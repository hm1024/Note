# Yarn 资源分配的流程

## AM 获取资源的流程

**ApplicationMaster**通过RPC调用ResourceManager端ApplicationMasterService#allocate方法申请资源

请求的参数

```
int responseID, 
float appProgress,
List<ResourceRequest> resourceAsk,
List<ContainerId> containersToBeReleased,
ResourceBlacklistRequest resourceBlacklistRequest
```
示例
```java
ask { 
    priority { priority: 20 } 
    resource_name: "*" 
    capability { memory: 1024 virtual_cores: 1 } 
    num_containers: 10 
} 
ask { 
    priority { priority: 20 } 
    resource_name: "/default-rack" 
    capability { memory: 1024 virtual_cores: 1 }
    num_containers: 10 
} 
ask { 
    priority { priority: 20 } 
    resource_name: "hdp01.bigdata.zll.360es.cn" 
    capability { memory: 1024 virtual_cores: 1 } 
    num_containers: 10 
} 
blacklist_request { } 
response_id: 0 
progress: 0.05 // 程序的运行进度
```

当ApplicationMasterService收到来自AppMaster的请求后，首先向AMLivelinessMonitor更新AppMaster的心跳时间(仅对已注册的AppMaster)， 然后判断AppMaster是否已注册，判断请求是否有效(AppMaster每次请求都会有一个响应id，随着请求递增)；向RMAppAttempt发送STATUS_UPDATE事件；更新AppMaster申请资源的黑名单列表，之后ResourceManager不会向`ApplicationMaster`分配黑名单列表机器上的任何资源；对申请资源做规整化处理；然后通过allocate方法调用调度器去获取资源，该方法更新新的资源请求后，返回已经分配的资源，

```java
//org.apache.hadoop.yarn.server.resourcemanager.ApplicationMasterService  
public AllocateResponse allocate(AllocateRequest request)
      throws YarnException, IOException {

    AMRMTokenIdentifier amrmTokenIdentifier = authorizeRequest();

    ApplicationAttemptId appAttemptId =
        amrmTokenIdentifier.getApplicationAttemptId();
    ApplicationId applicationId = appAttemptId.getApplicationId();
	// 更新AM的心跳事件
    this.amLivelinessMonitor.receivedPing(appAttemptId);

    /* check if its in cache */
    AllocateResponseLock lock = responseMap.get(appAttemptId);

    synchronized (lock) {
      /**ApplicationMaster每次与AMS交互，都会生成并记录一个AllocateResponse，AllocateResponse
      中记录的交互Id每次交互都会递增。从registerAppAtempt()中设置为-1，registerApplicationMaster()
      设置为0， 以后开始每次交互均递增*/
      AllocateResponse lastResponse = lock.getAllocateResponse();     
      // 判断AppMaster是否已注册 略

      // 判断请求是否是旧的请求

      // 验证程序进度的值是否合法，以小数表示（百分比）

      // 向appAttempt发送STATUS_UPDATE的事件 （把作业的进度信息保存到RM维护的appAttempt中）
      this.rmContext.getDispatcher().getEventHandler().handle(
          new RMAppAttemptStatusupdateEvent(appAttemptId, request
              .getProgress()));

      List<ResourceRequest> ask = request.getAskList();
      List<ContainerId> release = request.getReleaseList();

      ResourceBlacklistRequest blacklistRequest =
          request.getResourceBlacklistRequest();
      List<String> blacklistAdditions =
          (blacklistRequest != null) ?
              blacklistRequest.getBlacklistAdditions() : Collections.EMPTY_LIST;
      List<String> blacklistRemovals =
          (blacklistRequest != null) ?
              blacklistRequest.getBlacklistRemovals() : Collections.EMPTY_LIST;
      //ResourceManager维护的这个application的信息,运行时，这个app是一个RMAppImpl
      RMApp app =
          this.rmContext.getRMApps().get(applicationId);
      
      // 如果 resourceName = ANY，则为资源请求设置标签表达式
      ApplicationSubmissionContext asc = app.getApplicationSubmissionContext();
      for (ResourceRequest req : ask) {
        if (null == req.getNodeLabelExpression()
            && ResourceRequest.ANY.equals(req.getResourceName())) {
          req.setNodeLabelExpression(asc.getNodeLabelExpression());
        }
      }
              
      // 对申请资源做规整化处理
      RMServerUtils.normalizeAndValidateRequests(ask,
            rScheduler.getMaximumResourceCapability(), app.getQueue(),rScheduler, rmContext);

      //在work-preserving 关闭的情况下，不应该发生申请释放的container的applicationAttemptId
      //与当前AM的attemptId不一致的 情况，如果发生，则抛出异常
      if (!app.getApplicationSubmissionContext().getKeepContainersAcrossApplicationAttempts()) {
          RMServerUtils.validateContainerReleaseRequest(release, appAttemptId);
      }
	  // 在这里去更新资源请求，并获取已经分配的资源(FairScheduler.allocate)
      Allocation allocation =
          this.rScheduler.allocate(appAttemptId, ask, release, 
              blacklistAdditions, blacklistRemovals);


      RMAppAttempt appAttempt = app.getRMAppAttempt(appAttemptId);
      AllocateResponse allocateResponse =
          recordFactory.newRecordInstance(AllocateResponse.class);
      if (!allocation.getContainers().isEmpty()) {
        allocateResponse.setNMTokens(allocation.getNMTokens());
      }

      // update the response with the deltas of node status changes
      List<RMNode> updatedNodes = new ArrayList<RMNode>();
      if(app.pullRMNodeUpdates(updatedNodes) > 0) {
        List<NodeReport> updatedNodeReports = new ArrayList<NodeReport>();
        for(RMNode rmNode: updatedNodes) {
          SchedulerNodeReport schedulerNodeReport =  
              rScheduler.getNodeReport(rmNode.getNodeID());
          Resource used = BuilderUtils.newResource(0, 0);
          int numContainers = 0;
          if (schedulerNodeReport != null) {
            used = schedulerNodeReport.getUsedResource();
            numContainers = schedulerNodeReport.getNumContainers();
          }
          NodeId nodeId = rmNode.getNodeID();
          NodeReport report =
              BuilderUtils.newNodeReport(nodeId, rmNode.getState(),
                  rmNode.getHttpAddress(), rmNode.getRackName(), used,
                  rmNode.getTotalCapability(), numContainers,
                  rmNode.getHealthReport(), rmNode.getLastHealthReportTime(),
                  rmNode.getNodeLabels());

          updatedNodeReports.add(report);
        }
        allocateResponse.setUpdatedNodes(updatedNodeReports);
      }
	  //设置已经为这个application分配的container信息到response中
      allocateResponse.setAllocatedContainers(allocation.getContainers());
      //设置已经完成的container的状态信息到response中
      allocateResponse.setCompletedContainersStatuses(appAttempt
          .pullJustFinishedContainers());
      //responseID自增1，放到response中
      allocateResponse.setResponseId(lastResponse.getResponseId() + 1);
      //设置集群中可用的资源信息到response中
      allocateResponse.setAvailableResources(allocation.getResourceLimit());
      //设置集群中可用节点的数目信息到response中
      allocateResponse.setNumClusterNodes(this.rScheduler.getNumClusterNodes());

      // add preemption to the allocateResponse message (if any)
      //设置抢占信息到response中
      allocateResponse
          .setPreemptionMessage(generatePreemptionMessage(allocation));

      // update AMRMToken if the token is rolled-up
      MasterKeyData nextMasterKey =
          this.rmContext.getAMRMTokenSecretManager().getNextMasterKeyData();

     // token相关的处理

      lock.setAllocateResponse(allocateResponse);
      return allocateResponse;
    }    
  }
```

### 更新状态

RMAppAttempt 中注册了STATUS_UPDATE事件，收到该事件后会首先去更新该应用的进度

```java
//org.apache.hadoop.yarn.server.resourcemanager.rmapp.attempt.RMAppAttemptImpl
// Transitions from RUNNING State
.addTransition(RMAppAttemptState.RUNNING, RMAppAttemptState.RUNNING,
    RMAppAttemptEventType.STATUS_UPDATE, new StatusUpdateTransition())

   // StatusUpdateTransition#transition
   public void transition(RMAppAttemptImpl appAttempt,
        RMAppAttemptEvent event) {

      RMAppAttemptStatusupdateEvent statusUpdateEvent
        = (RMAppAttemptStatusupdateEvent) event;

      // Update progress
      appAttempt.progress = statusUpdateEvent.getProgress();

      // Ping to AMLivelinessMonitor
      appAttempt.rmContext.getAMLivelinessMonitor().receivedPing(
          statusUpdateEvent.getApplicationAttemptId());
    }
```

### 获取已分配的资源

这里以FairScheduler为例，在申请资源时首先会对资源请求做规整化处理；释放已完成的container，然后更新应用的资源请求，保存在调度器维护的数据结构中，等待调度器分配资源，

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler  
public Allocation allocate(ApplicationAttemptId appAttemptId,
      List<ResourceRequest> ask, List<ContainerId> release,
      List<String> blacklistAdditions, List<String> blacklistRemovals) {

    // Make sure this application exists
    FSAppAttempt application = getSchedulerApp(appAttemptId);

    // Sanity check
    SchedulerUtils.normalizeRequests(ask, DOMINANT_RESOURCE_CALCULATOR,
        clusterResource, minimumAllocation, getMaximumResourceCapability(),
        incrAllocation);

    // Set amResource for this app
    if (!application.getUnmanagedAM() && ask.size() == 1
        && application.getLiveContainers().isEmpty()) {
      application.setAMResource(ask.get(0).getCapability());
    }

    // 释放容器
    releaseContainers(release, application);

    synchronized (application) {
      if (!ask.isEmpty()) {
        // 更新应用的资源请求
        application.updateResourceRequests(ask);
      }
      
      Set<ContainerId> preemptionContainerIds = new HashSet<ContainerId>();
      for (RMContainer container : application.getPreemptionContainers()) {
        preemptionContainerIds.add(container.getContainerId());
      }
      // 更新该app设置的黑名单
      application.updateBlacklist(blacklistAdditions, blacklistRemovals);
      // 获取之前分配的container和nm token
      ContainersAndNMTokensAllocation allocation = application.pullNewlyAllocatedContainersAndNMTokens();
      Resource headroom = application.getHeadroom();
      application.setApplicationHeadroomForMetrics(headroom);
        
      return new Allocation(allocation.getContainerList(), headroom,
          preemptionContainerIds, null, null, allocation.getNMTokenList());
    }
  }
```

### 更新新请求的资源

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.SchedulerApplicationAttempt
public synchronized void updateResourceRequests(
      List<ResourceRequest> requests) {
    if (!isStopped) {
      appSchedulingInfo.updateResourceRequests(requests, false);
    }
  }
```



```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.AppSchedulingInfo  
synchronized public void updateResourceRequests(
      List<ResourceRequest> requests, boolean recoverPreemptedRequest) {
    QueueMetrics metrics = queue.getMetrics();
    
    // Update resource requests
    for (ResourceRequest request : requests) {
      Priority priority = request.getPriority();
      String resourceName = request.getResourceName();
      boolean updatePendingResources = false;
      ResourceRequest lastRequest = null;

      if (resourceName.equals(ResourceRequest.ANY)) {
        updatePendingResources = true;
        
        if (request.getNumContainers() > 0) {
          activeUsersManager.activateApplication(user, applicationId);
        }
      }

      Map<String, ResourceRequest> asks = this.requests.get(priority);

      if (asks == null) {
        asks = new ConcurrentHashMap<String, ResourceRequest>();
        this.requests.put(priority, asks);
        this.priorities.add(priority);
      }
      lastRequest = asks.get(resourceName);

      if (recoverPreemptedRequest && lastRequest != null) {
        // Increment the number of containers to 1, as it is recovering a single container.
        // 将容器数增加到1，因为它正在恢复单个容器。
        request.setNumContainers(lastRequest.getNumContainers() + 1);
      }
      // 更新资源请求  在后续调度器真正分配资源时，会根据此时存储的app资源请求进行分配。
      asks.put(resourceName, request);
      if (updatePendingResources) {
        
        // Similarly, deactivate application?
        if (request.getNumContainers() <= 0) {
          LOG.info("checking for deactivate of application :"
              + this.applicationId);
          checkForDeactivation();
        }
        
        int lastRequestContainers = lastRequest != null ? lastRequest
            .getNumContainers() : 0;
        Resource lastRequestCapability = lastRequest != null ? lastRequest
            .getCapability() : Resources.none();
        metrics.incrPendingResources(user, request.getNumContainers(),
            request.getCapability());
        metrics.decrPendingResources(user, lastRequestContainers,
            lastRequestCapability);
      }
    }
  }
```

## NM 心跳

NM 通过RPC定时向 RM 发送心跳nodeHeartbeat，心跳中主要包含节点上Container的状态、保持活跃的应用程序和节点的健康状况及Token等信息，在RM端由ResourceTrackerService 服务处理来自NodeManager的心跳，经过一系列的判断后会发送RMNodeEventType.STATUS_UPDATE事件

```java
  public NodeHeartbeatResponse nodeHeartbeat(NodeHeartbeatRequest request)
      throws YarnException, IOException {

    NodeStatus remoteNodeStatus = request.getNodeStatus();
    /**
     * Here is the node heartbeat sequence...
     * 1. 判断是否是合法的node，没有在exclude文件中。
     * 2. 判断该节点是否注册过。
     * 3. 判断该heartbeat是否重复。
     * 4. 发送一个RMNodeStatusEvent事件
     */
    NodeId nodeId = remoteNodeStatus.getNodeId();

    // Heartbeat response
    NodeHeartbeatResponse nodeHeartBeatResponse = YarnServerBuilderUtils
        .newNodeHeartbeatResponse(lastNodeHeartbeatResponse.
            getResponseId() + 1, NodeAction.NORMAL, null, null, null, null,
            nextHeartBeatInterval);
    // 把RMNode中该NodeManager 要clean的Container的列表、要结束的应用的列表、要从该NM中removed的container列表
    rmNode.updateNodeHeartbeatResponseForCleanup(nodeHeartBeatResponse);

    populateKeys(request, nodeHeartBeatResponse);

    ConcurrentMap<ApplicationId, ByteBuffer> systemCredentials =
        rmContext.getSystemCredentialsForApps();
    if (!systemCredentials.isEmpty()) {
      nodeHeartBeatResponse.setSystemCredentialsForApps(systemCredentials);
    }

    // 4. Send status to RMNode, saving the latest response.
    this.rmContext.getDispatcher().getEventHandler().handle(
        new RMNodeStatusEvent(nodeId, remoteNodeStatus.getNodeHealthStatus(),
            remoteNodeStatus.getContainersStatuses(), 
            remoteNodeStatus.getKeepAliveApplications(), nodeHeartBeatResponse));

    return nodeHeartBeatResponse;
  }
```

> NM心跳的返回的信息
>
> NodeHeartbeatResponse
>
> * 心跳id
> * 节点状态
> * 要清理的container
> * 要清理的应用
> * container及nm的token
> * 下次心跳的间隔



### RM对心跳的处理

在RMNodeImpl中注册了该事件(这里只看正常的流程)，当节点是健康节点时，会去更新该节点上新创建的以及已完成的container的列表，然后向调度器发送SchedulerEventType.NODE_UPDATE

```java
//org.apache.hadoop.yarn.server.resourcemanager.rmnode.RMNodeImpl
//Transitions from RUNNING state
.addTransition(NodeState.RUNNING,
    EnumSet.of(NodeState.RUNNING, NodeState.UNHEALTHY),
    RMNodeEventType.STATUS_UPDATE, new StatusUpdateWhenHealthyTransition())

public static class StatusUpdateWhenHealthyTransition implements
      MultipleArcTransition<RMNodeImpl, RMNodeEvent, NodeState> {
    @Override
    public NodeState transition(RMNodeImpl rmNode, RMNodeEvent event) {

      RMNodeStatusEvent statusEvent = (RMNodeStatusEvent) event;
      // 获取上一次心跳的响应
      rmNode.latestNodeHeartBeatResponse = statusEvent.getLatestResponse();
	  // 获取NM的健康状况
      NodeHealthStatus remoteNodeHealthStatus = statusEvent.getNodeHealthStatus();
      rmNode.setHealthReport(remoteNodeHealthStatus.getHealthReport());
      // 更新节点上次的更新时间
      rmNode.setLastHealthReportTime(remoteNodeHealthStatus.getLastHealthReportTime());
   	  // 节点不健康时的处理 略
        
      //处理该节点新启动以及已完成的container
      rmNode.handleContainerStatus(statusEvent.getContainers());
      //发送NodeUpdateSchedulerEvent事件
      if(rmNode.nextHeartBeat) {
        rmNode.nextHeartBeat = false;
        rmNode.context.getDispatcher().getEventHandler().handle(
            new NodeUpdateSchedulerEvent(rmNode));
      }

      return NodeState.RUNNING;
    }
  }
```

### 处理节点上的container

把在该节点上新启动的Container添加到已经启动的Container列表中,把该节点上已执行完成的container，从已启动的container列表中删除，并添加到已经完成的container列表中，然后将他们封装到UpdatedContainerInfo对象中，并添加到RMNode维护的节点更新队列中，而该队列中的信息就组成了在该节点上**最新的容器信息列表**，

```java
  private void handleContainerStatus(List<ContainerStatus> containerStatuses) {
    List<ContainerStatus> newlyLaunchedContainers =
        new ArrayList<ContainerStatus>();
    List<ContainerStatus> completedContainers =
        new ArrayList<ContainerStatus>();
    for (ContainerStatus remoteContainer : containerStatuses) {
      ContainerId containerId = remoteContainer.getContainerId();

      if (containersToClean.contains(containerId)) {
        LOG.info("Container " + containerId + " already scheduled for "
            + "cleanup, no further processing");
        continue;
      }
      if (finishedApplications.contains(containerId.getApplicationAttemptId()
          .getApplicationId())) {
        LOG.info("Container " + containerId
            + " belongs to an application that is already killed,"
            + " no further processing");
        continue;
      }

      // Process running containers
      if (remoteContainer.getState() == ContainerState.RUNNING) {
        if (!launchedContainers.contains(containerId)) {
          // Just launched container. RM knows about it the first time.
          launchedContainers.add(containerId);
          newlyLaunchedContainers.add(remoteContainer);
        }
      } else {
        // A finished container
        launchedContainers.remove(containerId);
        completedContainers.add(remoteContainer);
      }
    }
    if (newlyLaunchedContainers.size() != 0 || completedContainers.size() != 0) {
      nodeUpdateQueue.add(new UpdatedContainerInfo(newlyLaunchedContainers,
          completedContainers));
    }
  }
```



## 调度器进行调度

当FairScheduler收到该事件后，会调用nodeUpdate方法

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler  
public void handle(SchedulerEvent event) {
    case NODE_UPDATE:
    if (!(event instanceof NodeUpdateSchedulerEvent)) {
        throw new RuntimeException("Unexpected event type: " + event);
    }
    NodeUpdateSchedulerEvent nodeUpdatedEvent = (NodeUpdateSchedulerEvent)event;
    nodeUpdate(nodeUpdatedEvent.getRMNode());
    break;
  }
```

在nodeUpdate方法中会处理来自NM的心跳，从container的监控中清除新启动的Container（Container分配后会监控该container，如果在指定的事件内没有运行的话，会释放该Container），释放已完成的Container，然后判断是否开启持续调度

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler
private synchronized void nodeUpdate(RMNode nm) {
    long start = getClock().getTime();
    if (LOG.isDebugEnabled()) {
      LOG.debug("nodeUpdate: " + nm + " cluster capacity: " + clusterResource);
    }
    eventLog.log("HEARTBEAT", nm.getHostName());
    FSSchedulerNode node = getFSSchedulerNode(nm.getNodeID());
    // 获取当前节点上 最新的容器信息列表
    List<UpdatedContainerInfo> containerInfoList = nm.pullContainerUpdates();
    List<ContainerStatus> newlyLaunchedContainers = new ArrayList<ContainerStatus>();
    List<ContainerStatus> completedContainers = new ArrayList<ContainerStatus>();
    for(UpdatedContainerInfo containerInfo : containerInfoList) {
      newlyLaunchedContainers.addAll(containerInfo.getNewlyLaunchedContainers());
      completedContainers.addAll(containerInfo.getCompletedContainers());
    } 
    // 新启动的Container从containerAllocationExpirer中清除
    for (ContainerStatus launchedContainer : newlyLaunchedContainers) {
      containerLaunchedOnNode(launchedContainer.getContainerId(), node);
    }

    // 释放已完成的Container
    for (ContainerStatus completedContainer : completedContainers) {
      ContainerId containerId = completedContainer.getContainerId();
      LOG.debug("Container FINISHED: " + containerId);
      completedContainer(getRMContainer(containerId),
          completedContainer, RMContainerEventType.FINISHED);
    }
    //开启持续调度
    if (continuousSchedulingEnabled) {
      if (!completedContainers.isEmpty()) {
        attemptScheduling(node);
      }
    } else {
      attemptScheduling(node);
    }

    long duration = getClock().getTime() - start;
    fsOpDurations.addNodeUpdateDuration(duration);
  }
```

### 两种调度方式

ResourceManager进行资源分配的两种策略：

* 心跳调度

  NodeManager会定期的向ResourceManager发送心跳汇报自身的状态和资源情况(如：可用资源，正在使用的资源，已释放的资源)，在ResourceManager端心跳会触发一次资源调度，从维护的队列中取出合适的应用的资源请求(合适，指定的是这个资源请求既不违背队列的最大资源限制，也不违背这个NodeManager的剩余资源量限制)放到这个NodeManager上运行。这种调度方式的一个主要缺点是调度有延迟，当一个NodeManager即使有空闲的资源，调度只能在收到心跳后才能进行，不够及时。

* 持续调度

  不用等待NodeManager向ResourceManager发送心跳时才进行资源调度，而是由一个独立的线程进行实时的资源调度，与NodeManager的心跳触发的调度异步并行进行。当收到NodeManager心跳时，把调度的结果通过心跳响应告诉对应的NodeManager。

> 持续调度，在NodeManager心跳到来时，仅当NodeManager上有已经完成的Container的时才触发调度。二者最终的调度方法都是通过attemptScheduling(node)进行调度的

两种调度的不同之处主要由两点：

* 调度的时机不同：心跳调度仅仅发生在收到了某个NodeManager的心跳信息的情况下，持续调度则不依赖与NodeManager的心跳通信，是连续发生的，当心跳到来，会将调度结果直接返回给NodeManager；
* 调度范围不同：心跳调度机制下，当收到某个节点的心跳，就对这个节点且仅仅对这个节点进行一次调度，即谁的心跳到来就触发对谁的调度，而持续调度的每一轮，是会遍历当前集群的所有节点，每个节点依次进行一次调度，保证**一**轮下来每一个节点都被公平的调度一次；

当持续调度开启时，会启动一个持续调度的`ContinuousSchedulingThread`线程去分配资源，在`continuousSchedulingAttempt()`中会遍历所有节点，依次进行资源调度，这里的调度，就是试图找出一个container请求放到这个服务器上运行。为了让整个集群的资源分配在服务器节点之间能够更均匀，调度之前通过资源比较器对节点按照资源余量**从多到少**排序，从而让资源更充裕的节点先被调度，这样做更有利于让所有节点的资源使用量达到均衡，而不至于由于某些节点的序号排在前面而总是被先调度，造成**资源调度倾斜**。然后遍历排好序的节点，当节点的资源量满足Container的需求时，调用`attemptScheduling()`方法尝试调度分配Container。

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler
private class ContinuousSchedulingThread extends Thread {
    @Override
    public void run() {
      while (!Thread.currentThread().isInterrupted()) {
        try {
          continuousSchedulingAttempt();
          Thread.sleep(getContinuousSchedulingSleepMs());
        } catch (InterruptedException e) {
          LOG.warn("Continuous scheduling thread interrupted. Exiting.", e);
          return;
        }
      }
    }
  }
void continuousSchedulingAttempt() throws InterruptedException {
    long start = getClock().getTime();
    List<NodeId> nodeIdList = new ArrayList<NodeId>(nodes.keySet());
    //按节点上的可用空间对节点进行排序，为了首先在最空闲的节点上分配容器，使得Container均匀分布。 
    //加锁的是为了排序期间节点上的可用空间不会改变。
    synchronized (this) {
      Collections.sort(nodeIdList, nodeAvailableResourceComparator);
    }
    // iterate all nodes
    for (NodeId nodeId : nodeIdList) {
      FSSchedulerNode node = getFSSchedulerNode(nodeId);
      try {
        if (node != null && Resources.fitsIn(minimumAllocation,
            node.getAvailableResource())) {
          attemptScheduling(node);
        }
      } catch (Throwable ex) {
      }
    }

    long duration = getClock().getTime() - start;
    fsOpDurations.addContinuousSchedulingRunDuration(duration);
  }
```

### 最终的调度

两种调度方式的调度算法和原理是相同的，都是通过FairScheduler#attemptScheduling方法在节点上真正进行尝试分配的方法 attemptScheduling(node)，在该方法中

```java
// org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler  
synchronized void attemptScheduling(FSSchedulerNode node) {

    final NodeId nodeID = node.getNodeID();
    // Assign new containers...
    // 1. Check for reserved applications
    // 2. Schedule if there are no reservations
	// 检查该节点上是否有预留的应用程序
    FSAppAttempt reservedAppSchedulable = node.getReservedAppSchedulable();
    if (reservedAppSchedulable != null) {
      Priority reservedPriority = node.getReservedContainer().getReservedPriority();
      FSQueue queue = reservedAppSchedulable.getQueue();
      /*如果这个节点被这个应用预定，这里就去判断这个应用是不是有能够分配到这个node上的请求，
        如果有这样到请求，并且，没有超过队列到剩余资源，那么，就可以把这个预定的资源尝试进行分配（有可能分配失败）*/
      /*而如果发现这个应用没有任何一个请求适合在这个节点运行，或者，当前队列的剩余资源已经不够运行这个预留的、还没来得及执行的container，
        那么这个container就没有再预留的必要了*/
      if (!reservedAppSchedulable.hasContainerForNode(reservedPriority, node)
          || !fitsInMaxShare(queue,
          node.getReservedContainer().getReservedResource())) {
        //如果这个被预留的container已经不符合运行条件，就没有必要保持预留了，直接取消预留，让出资源
        reservedAppSchedulable.unreserve(reservedPriority, node);
        reservedAppSchedulable = null;
      } else {
        //对这个已经进行了reservation对节点进行节点分配，当然，有可能资源还是不足，因此还将处于预定状态
        node.getReservedAppSchedulable().assignReservedContainer(node);
      }
    }
    //该节点上没有reserve的app，尝试进行assignment
    if (reservedAppSchedulable == null) {
      int assignedContainers = 0;
      while (node.getReservedContainer() == null) {
        boolean assignedContainer = false;
        //尝试进行container的分配，并判断是否分配到资源,是否有预留(外层循环条件)
        if (!queueMgr.getRootQueue().assignContainer(node).equals(
            Resources.none())) {
          assignedContainers++;
          assignedContainer = true;
        }
        if (!assignedContainer) { break; }
        //判断是否一次心跳只能分配一个container
        if (!assignMultiple) { break; }
        if ((assignedContainers >= maxAssign) && (maxAssign > 0)) { break; }
      }
    }
    updateRootQueueMetrics();
  }
```

在上面的assignContainer(node)方法中，首先判断如果此队列的资源是使用否超出队列的限制，或该节点上有保留的Container时拒绝分配，然后根据调度策略对子队列进行排序，从排好序的子队列中逐个进行分配资源，如果分配到资源后立即返回分配的资源。

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FSParentQueue 
public Resource assignContainer(FSSchedulerNode node) {
    Resource assigned = Resources.none();

    // 如果此队列的资源是使用否超出队列的限制，或该节点上有保留的Container时拒绝分配
    if (!assignContainerPreCheck(node)) {
      return assigned;
    }
	// 根据放置策略对子队列进行排序
    Collections.sort(childQueues, policy.getComparator());
    for (FSQueue child : childQueues) {
      assigned = child.assignContainer(node);
      if (!Resources.equals(assigned, Resources.none())) {
        break;
      }
    }
    return assigned;
  }
```

接下来看看叶子队列(即FSLeafQueue)中是如何进行分配资源的，首先会判断，在子队列中会遍历自己目前所有的runnableApps，然后逐个尝试进行分配，只要有一个分配成功就退出

```java
// org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FSLeafQueue  
public Resource assignContainer(FSSchedulerNode node) {
    Resource assigned = Resources.none();
    if (LOG.isDebugEnabled()) {
      LOG.debug("Node " + node.getNodeName() + " offered to queue: " + getName());
    }
	// 如果此队列的已使用资源小于等于该队列的最大资源，或该节点上有保留的Container时拒绝分配
    if (!assignContainerPreCheck(node)) {
      return assigned;
    }
	
    Comparator<Schedulable> comparator = policy.getComparator();
    writeLock.lock();
    try {
      // 对正在应用进行排序
      Collections.sort(runnableApps, comparator);
    } finally {
      writeLock.unlock();
    }
    readLock.lock();
    try {
      //对排序完成的资源一次进行调度，即对他们进行资源分配尝试
      for (FSAppAttempt sched : runnableApps) {
        if (SchedulerAppUtils.isBlacklisted(sched, node, LOG)) {
          continue;
        }
		// 尝试在当前节点上为该任务分配资源
        assigned = sched.assignContainer(node);
        if (!assigned.equals(Resources.none())) {
          break;
        }
      }
    } finally {
      readLock.unlock();
    }
    return assigned;
  }
```

在FSAppAttempt的assignContainer方法中，为当前任务在传入的节点上分配资源，如果在传入的节点上有满足请求的资源，则分配本地资源，如果没有满足的资源，则判断是否接受同机架或任意节点的资源，如果接受就在获取相应的资源，如果不接受则返会null。

```java
// org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FSAppAttempt
private Resource assignContainer(FSSchedulerNode node, boolean reserved) {
    if (LOG.isDebugEnabled()) {
      LOG.debug("Node offered to app: " + getName() + " reserved: " + reserved);
    }

    Collection<Priority> prioritiesToTry = (reserved) ?
        Arrays.asList(node.getReservedContainer().getReservedPriority()) :
        getPriorities();

    synchronized (this) {
      for (Priority priority : prioritiesToTry) {
        // 
        if (getTotalRequiredResources(priority) <= 0 ||
            !hasContainerForNode(priority, node)) {
          continue;
        }

        addSchedulingOpportunity(priority);

        // Check the AM resource usage for the leaf queue
        if (getLiveContainers().size() == 0 && !getUnmanagedAM()) {
          if (!getQueue().canRunAppAM(getAMResource())) {
            return Resources.none();
          }
        }

        ResourceRequest rackLocalRequest = getResourceRequest(priority,
            node.getRackName());
        ResourceRequest localRequest = getResourceRequest(priority,
            node.getNodeName());

        if (localRequest != null && !localRequest.getRelaxLocality()) {
          LOG.warn("Relax locality off is not supported on local request: "
              + localRequest);
        }

        NodeType allowedLocality;
        // 这里判断是否开启持续调度，
        if (scheduler.isContinuousSchedulingEnabled()) {
          allowedLocality = getAllowedLocalityLevelByTime(priority,
              scheduler.getNodeLocalityDelayMs(),
              scheduler.getRackLocalityDelayMs(),
              scheduler.getClock().getTime());
        } else {
          allowedLocality = getAllowedLocalityLevel(priority,
              scheduler.getNumClusterNodes(),
              scheduler.getNodeLocalityThreshold(),
              scheduler.getRackLocalityThreshold());
        }

        if (rackLocalRequest != null && rackLocalRequest.getNumContainers() != 0
            && localRequest != null && localRequest.getNumContainers() != 0) {
          return assignContainer(node, localRequest,
              NodeType.NODE_LOCAL, reserved);
        }

        if (rackLocalRequest != null && !rackLocalRequest.getRelaxLocality()) {
          continue;
        }

        if (rackLocalRequest != null && rackLocalRequest.getNumContainers() != 0
            && (allowedLocality.equals(NodeType.RACK_LOCAL) ||
            allowedLocality.equals(NodeType.OFF_SWITCH))) {
          return assignContainer(node, rackLocalRequest,
              NodeType.RACK_LOCAL, reserved);
        }

        ResourceRequest offSwitchRequest =
            getResourceRequest(priority, ResourceRequest.ANY);
        if (offSwitchRequest != null && !offSwitchRequest.getRelaxLocality()) {
          continue;
        }

        if (offSwitchRequest != null &&
            offSwitchRequest.getNumContainers() != 0) {
          if (!hasNodeOrRackLocalRequests(priority) ||
              allowedLocality.equals(NodeType.OFF_SWITCH)) {
            return assignContainer(
                node, offSwitchRequest, NodeType.OFF_SWITCH, reserved);
          }
        }
      }
    }
    return Resources.none();
  }
```

不管是哪种类型(NODE_LOCAL、 RACK_LOCAL、 OFF_SWITCH)的资源，都会通过该方法进行调度

```java
// org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FSAppAttempt
private Resource assignContainer(
      FSSchedulerNode node, ResourceRequest request, NodeType type,
      boolean reserved) {

    // How much does this request need?
    Resource capability = request.getCapability();

    // How much does the node have?
    Resource available = node.getAvailableResource();

    // Can we allocate a container in this queue?
    FSQueue queue = (FSQueue) this.queue;
    Resource maxRes = queue.getMaxShare();
    Resource usedRes = getResourceUsage();
    Resource totalRes = Resource.newInstance(usedRes.getMemory() + capability.getMemory(),
        usedRes.getVirtualCores() + capability.getVirtualCores());
    if(!Resources.fitsIn(totalRes, maxRes)) {
      if (LOG.isDebugEnabled()) {
        LOG.debug("Queue " + queue.getName() + " has no enough resources to allocate");
      }
      return Resources.none();
    }

    Container container = null;
    if (reserved) {
      container = node.getReservedContainer().getContainer();
    } else {
      container = createContainer(node, capability, request.getPriority());
    }

    // Can we allocate a container on this node?
    if (Resources.fitsIn(capability, available)) {
      // Inform the application of the new container for this request
      // 在这里真正在当前节点上分配container请求的
      RMContainer allocatedContainer =
          allocate(type, node, request.getPriority(), request, container);
      if (allocatedContainer == null) {
        // Did the application need this resource?
        if (reserved) {
          unreserve(request.getPriority(), node);
        }
        return Resources.none();
      }

      // If we had previously made a reservation, delete it
      if (reserved) {
        unreserve(request.getPriority(), node);
      }

      // Inform the node
      node.allocateContainer(allocatedContainer);

      // If this container is used to run AM, update the leaf queue's AM usage
      if (getLiveContainers().size() == 1 && !getUnmanagedAM()) {
        getQueue().addAMResourceUsage(container.getResource());
        setAmRunning(true);
      }

      return container.getResource();
    } else {
      if (!FairScheduler.fitsInMaxShare(getQueue(), capability)) {
        return Resources.none();
      }
      Resource appMaxReserveResources = scheduler.getAllocationConfiguration().getAppMaxReserveResources();
      Resource currentReservation = getCurrentReservation();
      Resource reservationPlusCapability = Resources.add(currentReservation, capability);
      if(!Resources.fitsIn(reservationPlusCapability, appMaxReserveResources)) {
        return Resources.none();
      }
      // The desired container won't fit here, so reserve
      // 所需的容器在这里不合适，所以请预留
      reserve(request.getPriority(), node, container, reserved);

      return FairScheduler.CONTAINER_RESERVED;
    }
  }
```

判断该节点上是否有满足该优先级下所需的container需求

```java
  /**
   * Whether this app has containers requests that could be satisfied on the
   * given node, if the node had full space.
   */
  public boolean hasContainerForNode(Priority prio, FSSchedulerNode node) {
    ResourceRequest anyRequest = getResourceRequest(prio, ResourceRequest.ANY);
    ResourceRequest rackRequest = getResourceRequest(prio, node.getRackName());
    ResourceRequest nodeRequest = getResourceRequest(prio, node.getNodeName());

    return
        // There must be outstanding requests at the given priority:
        anyRequest != null && anyRequest.getNumContainers() > 0 &&
            // If locality relaxation is turned off at *-level, there must be a
            // non-zero request for the node's rack:
            (anyRequest.getRelaxLocality() ||
                (rackRequest != null && rackRequest.getNumContainers() > 0)) &&
            // If locality relaxation is turned off at rack-level, there must be a
            // non-zero request at the node:
            (rackRequest == null || rackRequest.getRelaxLocality() ||
                (nodeRequest != null && nodeRequest.getNumContainers() > 0)) &&
            // The requested container must be able to fit on the node:
            Resources.lessThanOrEqual(RESOURCE_CALCULATOR, null,
                anyRequest.getCapability(), node.getRMNode().getTotalCapability());
  }
```

真正在NM上分配container的方法，在该方法中会把申请的container保存到`List<RMContainer> newlyAllocatedContainers`中，然后等待AppMaster的心跳获取已分配的container

```java
  synchronized public RMContainer allocate(NodeType type, FSSchedulerNode node,
      Priority priority, ResourceRequest request,
      Container container) {
    // Update allowed locality level
    NodeType allowed = allowedLocalityLevel.get(priority);
    if (allowed != null) {
      if (allowed.equals(NodeType.OFF_SWITCH) &&
          (type.equals(NodeType.NODE_LOCAL) ||
              type.equals(NodeType.RACK_LOCAL))) {
        this.resetAllowedLocalityLevel(priority, type);
      }
      else if (allowed.equals(NodeType.RACK_LOCAL) &&
          type.equals(NodeType.NODE_LOCAL)) {
        this.resetAllowedLocalityLevel(priority, type);
      }
    }

    // Required sanity check - AM can call 'allocate' to update resource 
    // request without locking the scheduler, hence we need to check
    if (getTotalRequiredResources(priority) <= 0) {
      return null;
    }
    
    // Create RMContainer
    RMContainer rmContainer = new RMContainerImpl(container, 
        getApplicationAttemptId(), node.getNodeID(),
        appSchedulingInfo.getUser(), rmContext);

    // Add it to allContainers list.
    newlyAllocatedContainers.add(rmContainer);
    liveContainers.put(container.getId(), rmContainer);    

    // Update consumption and track allocations
    List<ResourceRequest> resourceRequestList = appSchedulingInfo.allocate(
        type, node, priority, request, container);
    Resources.addTo(currentConsumption, container.getResource());

    // Update resource requests related to "request" and store in RMContainer
    ((RMContainerImpl) rmContainer).setResourceRequests(resourceRequestList);

    // Inform the container
    rmContainer.handle(
        new RMContainerEvent(container.getId(), RMContainerEventType.START));

    if (LOG.isDebugEnabled()) {
      LOG.debug("allocate: applicationAttemptId=" 
          + container.getId().getApplicationAttemptId() 
          + " container=" + container.getId() + " host="
          + container.getNodeId().getHost() + " type=" + type);
    }
    RMAuditLogger.logSuccess(getUser(), 
        AuditConstants.ALLOC_CONTAINER, "SchedulerApp", 
        getApplicationId(), container.getId());
    
    return rmContainer;
  }
```

### 延迟调度

```java
  /**
   * Delay scheduling: We often want to prioritize scheduling of node-local
   * containers over rack-local or off-switch containers. To achieve this
   * we first only allow node-local assignments for a given priority level,
   * then relax the locality threshold once we've had a long enough period
   * without successfully scheduling. We measure both the number of "missed"
   * scheduling opportunities since the last container was scheduled
   * at the current allowed level and the time since the last container
   * was scheduled. Currently we use only the former.
   */
  private final Map<Priority, NodeType> allowedLocalityLevel =
      new HashMap<Priority, NodeType>();
```

延迟调度的核心逻辑如下，该方法的调用见FSAppAttempt#assignContainer(FSSchedulerNode node, boolean reserved)

```java
//org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FSAppAttempt 
/**
   * Return the level at which we are allowed to schedule containers, given the
   * current size of the cluster and thresholds indicating how many nodes to
   * fail at (as a fraction of cluster size) before relaxing scheduling
   * constraints.
   */
  public synchronized NodeType getAllowedLocalityLevel(Priority priority,
      int numNodes, double nodeLocalityThreshold, double rackLocalityThreshold) {
    // upper limit on threshold
    if (nodeLocalityThreshold > 1.0) { nodeLocalityThreshold = 1.0; }
    if (rackLocalityThreshold > 1.0) { rackLocalityThreshold = 1.0; }

    // If delay scheduling is not being used, can schedule anywhere
    if (nodeLocalityThreshold < 0.0 || rackLocalityThreshold < 0.0) {
      return NodeType.OFF_SWITCH;
    }

    // Default level is NODE_LOCAL
    if (!allowedLocalityLevel.containsKey(priority)) {
      allowedLocalityLevel.put(priority, NodeType.NODE_LOCAL);
      return NodeType.NODE_LOCAL;
    }

    NodeType allowed = allowedLocalityLevel.get(priority);

    // If level is already most liberal, we're done
    if (allowed.equals(NodeType.OFF_SWITCH)) return NodeType.OFF_SWITCH;

    double threshold = allowed.equals(NodeType.NODE_LOCAL) ? nodeLocalityThreshold :
      rackLocalityThreshold;

    // Relax locality constraints once we've surpassed threshold.
    if (getSchedulingOpportunities(priority) > (numNodes * threshold)) {
      if (allowed.equals(NodeType.NODE_LOCAL)) {
        allowedLocalityLevel.put(priority, NodeType.RACK_LOCAL);
        resetSchedulingOpportunities(priority);
      }
      else if (allowed.equals(NodeType.RACK_LOCAL)) {
        allowedLocalityLevel.put(priority, NodeType.OFF_SWITCH);
        resetSchedulingOpportunities(priority);
      }
    }
    return allowedLocalityLevel.get(priority);
  }
```


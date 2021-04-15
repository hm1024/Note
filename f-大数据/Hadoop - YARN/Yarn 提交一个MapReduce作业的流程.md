[toc]

# Yarn 提交一个MapReduce作业的流程

## 客户端提交

## RM 

### ClientRMService#submitApplication

```java
  @Override
  public SubmitApplicationResponse submitApplication(
      SubmitApplicationRequest request) throws YarnException {
    /* submissionContext 的内容见下文*/
    ApplicationSubmissionContext submissionContext = request
        .getApplicationSubmissionContext();
    ApplicationId applicationId = submissionContext.getApplicationId();

    // ApplicationSubmissionContext needs to be validated for safety - only
    // those fields that are independent of the RM's configuration will be
    // checked here, those that are dependent on RM configuration are validated
    // in RMAppManager.

    String user = null;
    String password = null;
    try {
      // Safety TODO 查看
      user = UserGroupInformation.getCurrentUser().getShortUserName();
      // user password
      password = submissionContext.getPassword();
    } catch (IOException ie) {
      LOG.warn("Unable to get the current user.", ie);
      RMAuditLogger.logFailure(user, AuditConstants.SUBMIT_APP_REQUEST,
          ie.getMessage(), "ClientRMService",
          "Exception in submitting application", applicationId, submissionContext.getApplicationType());
      throw RPCUtil.getRemoteException(ie);
    }

    // ldap authenticate

    // 限制用户最多能提交的App数量

    // Check whether app has already been put into rmContext,If it is, simply return the response
  
    try {
      /*call RMAppManager to submit application directly*/
      rmAppManager.submitApplication(submissionContext,
          System.currentTimeMillis(), user);

      LOG.info("Application with id " + applicationId.getId() + 
          " submitted by user " + user);
      RMAuditLogger.logSuccess(user, AuditConstants.SUBMIT_APP_REQUEST,
          "ClientRMService", applicationId, submissionContext.getApplicationType());
    } catch (YarnException e) {
      LOG.info("Exception in submitting application with id " +
          applicationId.getId(), e);
      RMAuditLogger.logFailure(user, AuditConstants.SUBMIT_APP_REQUEST,
          e.getMessage(), "ClientRMService",
          "Exception in submitting application", applicationId, submissionContext.getApplicationType());
      throw e;
    }

    SubmitApplicationResponse response = recordFactory
        .newRecordInstance(SubmitApplicationResponse.class);
    return response;
  }
```



client =》 RM  submissionContext

```json
application_id {
  id: 1
  cluster_timestamp: 1600141422113
}
application_name: "QuasiMonteCarlo"
queue: "default"
am_container_spec {
  localResources {
    key: "jobSubmitDir/job.splitmetainfo"
    value {
      resource {
        scheme: "hdfs"
        host: "HACluster"
        port: 8020
        file: "/home/yarn/staging/yarn/.staging/job_1600141422113_0001/job.splitmetainfo"
      }
      size: 452
      timestamp: 1600141526650
      type: FILE
      visibility: APPLICATION
    }
  }
  localResources {
    key: "job.jar"
    value {
      resource {
        scheme: "hdfs"
        host: "HACluster"
        port: 8020
        file: "/home/yarn/staging/yarn/.staging/job_1600141422113_0001/job.jar"
      }
      size: 272067
      timestamp: 1600141526586
      type: PATTERN
      visibility: APPLICATION
      pattern: "(?:classes/|lib/).*"
    }
  }
  localResources {
    key: "jobSubmitDir/job.split"
    value {
      resource {
        scheme: "hdfs"
        host: "HACluster"
        port: 8020
        file: "/home/yarn/staging/yarn/.staging/job_1600141422113_0001/job.split"
      }
      size: 712
      timestamp: 1600141526624
      type: FILE
      visibility: APPLICATION
    }
  }
  localResources {
    key: "job.xml"
    value {
      resource {
        scheme: "hdfs"
        host: "HACluster"
        port: 8020
        file: "/home/yarn/staging/yarn/.staging/job_1600141422113_0001/job.xml"
      }
      size: 107680
      timestamp: 1600141526736
      type: FILE
      visibility: APPLICATION
    }
  }
  tokens: "HDTS\000\000\001\025MapReduceShuffleToken\b\016\227\352\032\251\203\3450"
  environment {
    key: "HADOOP_CLASSPATH"
    value: "$PWD:job.jar/job.jar:job.jar/classes/:job.jar/lib/*:$PWD/*"
  }
  environment {
    key: "SHELL"
    value: "/bin/bash"
  }
  environment {
    key: "CLASSPATH"
    value: "$PWD:$HADOOP_SPINNER_CORE_DIR:$HADOOP_CONF_DIR:$HADOOP_COMMON_HOME/share/hadoop/common/*:$HADOOP_COMMON_HOME/share/hadoop/common/lib/*:$HADOOP_HDFS_HOME/share/hadoop/hdfs/*:$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*:$HADOOP_YARN_HOME/share/hadoop/yarn/*:$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*:job.jar/job.jar:job.jar/classes/:job.jar/lib/*:$PWD/*"
  }
  environment {
    key: "LD_LIBRARY_PATH"
    value: "$PWD"
  }
  command: "$JAVA_HOME/bin/java -Djava.io.tmpdir=$PWD/tmp -Dlog4j.configuration=container-log4j.properties -Dyarn.app.container.log.dir=<LOG_DIR> -Dyarn.app.container.log.filesize=1073741824 -Dyarn.app.container.log.backups=10 -Dhadoop.root.logger=INFO,CRLA -Dhadoop.root.logfile=syslog -Dyarn.app.container.log.stdout.filesize=1073741824 -Dyarn.app.container.log.stderr.filesize=1073741824 -Dhadoop.root.stdout.file=stdout -Dhadoop.root.stderr.file=stderr  -Xmx1230m org.apache.hadoop.mapreduce.v2.app.MRAppMaster 1><LOG_DIR>/stdout 2><LOG_DIR>/stderr "
  application_ACLs {
    accessType: APPACCESS_VIEW_APP
    acl: "*"
  }
  application_ACLs {
    accessType: APPACCESS_MODIFY_APP
    acl: " "
  }
}
cancel_tokens_when_complete: true
maxAppAttempts: 2
resource {
  memory: 1024
  virtual_cores: 1
}
applicationType: "MAPREDUCE"

```

返回 SubmitApplicationResponse

![image-20200915120354483](assets/image-20200915120354483.png)

### RMAPPManager#submitApplication

RMAPPManager 向 RMAppImpl 发送 RMAppEventType.START 事件

> RMAppManager 为该应用程序创建一个RMAppImpl对象以维护它的运行状态，并判断系统状态，如果是故障重启动状态，则向他发送一个RMAppEventType.RECOVER事件，否则发送一个RMAppEventType.START事件。

```java
protected void submitApplication(
      ApplicationSubmissionContext submissionContext, long submitTime,
      String user) throws YarnException {
    ApplicationId applicationId = submissionContext.getApplicationId();
	// 见下文
    RMAppImpl application =
        createAndPopulateNewRMApp(submissionContext, submitTime, user, false);
    ApplicationId appId = submissionContext.getApplicationId();
	// UGI => false
    // 调度程序此时尚未启动，因此应确保启动调度程序时首先处理排队的这些START事件。
    this.rmContext.getDispatcher().getEventHandler()
       .handle(new RMAppEvent(applicationId, RMAppEventType.START));
  }
```

###### RMAPPManager#createAndPopulateNewRMApp

```java
  private RMAppImpl createAndPopulateNewRMApp(
      ApplicationSubmissionContext submissionContext, long submitTime,
      String user, boolean isRecovery) throws YarnException {
    ApplicationId applicationId = submissionContext.getApplicationId();
    /*amReq= {Priority: 0, Capability: <memory:1024, vCores:1>, # Containers: 1, Location: *, Relax Locality: true} */
    ResourceRequest amReq =
        validateAndCreateResourceRequest(submissionContext, isRecovery);
    // Create RMApp
    RMAppImpl application =
        new RMAppImpl(applicationId, rmContext, this.conf,
            submissionContext.getApplicationName(), user,
            submissionContext.getQueue(),
            submissionContext, this.scheduler, this.masterService,
            submitTime, submissionContext.getApplicationType(),
            submissionContext.getApplicationTags(), amReq);
	// 将新创建的App添加到RM管理app(作业)的map集合中
    if (rmContext.getRMApps().putIfAbsent(applicationId, application) !=null) {
        // 如果程序进入这里，则表示 已经存在包含此applicationId的作业
        // 打印日志，抛异常
    }
    return application;
  }
```

### 

RMApp的状态机，RMAppImpl类中注册了该事件 `RMAppEventType.START`

```java
// Transitions from NEW state
.addTransition(RMAppState.NEW, RMAppState.NEW_SAVING,
	RMAppEventType.START, new RMAppNewlySavingTransition())
....
private static final class RMAppNewlySavingTransition extends RMAppTransition {
    @Override
    public void transition(RMAppImpl app, RMAppEvent event) {
      /*如果启用了恢复，则将应用程序信息存储在非阻塞调用中，因此请确保RM重新启动后,RM已存储重新启动AM所需的信息，而无需进一步的客户端通信*/
      LOG.info("Storing application with id " + app.applicationId);
      app.rmContext.getStateStore().storeNewApplication(app);
    }
}
....
//RMStateStore#storeNewApplication
public synchronized void storeNewApplication(RMApp app) {
    ApplicationSubmissionContext context = app.getApplicationSubmissionContext();
    assert context instanceof ApplicationSubmissionContextPBImpl;
    ApplicationStateData appState =
        ApplicationStateData.newInstance(app.getSubmitTime(), app.getStartTime(), context, app.getUser());
    dispatcher.getEventHandler().handle(new RMStateStoreAppEvent(appState));
}
```

### RMStateStore#storeNewApplication

发送事件RMStateStoreEventType.STORE_APP，状态机RMStateStore中注册了该事件

```java
.addTransition(RMStateStoreState.ACTIVE,
          EnumSet.of(RMStateStoreState.ACTIVE, RMStateStoreState.FENCED),
          RMStateStoreEventType.STORE_APP, new StoreAppTransition()) 

private static class StoreAppTransition
      implements MultipleArcTransition<RMStateStore, RMStateStoreEvent,
          RMStateStoreState> {
    @Override
    public RMStateStoreState transition(RMStateStore store,
        RMStateStoreEvent event) {
      boolean isFenced = false;
      // 要存储的状态信息
      /**
      submitTime,startTime,user,SubmissionContext,RMAppState,diagnostics,finishTime
      */
      ApplicationStateData appState =
          ((RMStateStoreAppEvent) event).getAppState();
      ApplicationId appId =
          appState.getApplicationSubmissionContext().getApplicationId();
      LOG.info("Storing info for app: " + appId);
      try {
        store.storeApplicationStateInternal(appId, appState);
        store.notifyApplication(new RMAppEvent(appId,
                   RMAppEventType.APP_NEW_SAVED));
      } catch (Exception e) {
        LOG.error("Error storing app: " + appId, e);
        isFenced = store.notifyStoreOperationFailedInternal(e);
      }
      return finalState(isFenced);
    };
  }
```

###### ZKRMStateStore#storeApplicationStateInternal

```java
  public synchronized void storeApplicationStateInternal(ApplicationId appId,
      ApplicationStateData appStateDataPB) throws Exception {
    // nodeCreatePath=/shjt/rmstore/ZKRMStateRoot/RMAppRoot/application_id
    String nodeCreatePath = getNodePath(rmAppRoot, appId.toString());
    if (LOG.isDebugEnabled()) {
      LOG.debug("Storing info for app: " + appId + " at: " + nodeCreatePath);
    }
    byte[] appStateData = appStateDataPB.getProto().toByteArray();
    createWithRetries(nodeCreatePath, appStateData, zkAcl,CreateMode.PERSISTENT);
  }
```

###### ZKRMStateStore#notifyApplication

发送`RMAppEventType.APP_NEW_SAVED`事件，

```java
private void notifyApplication(RMAppEvent event) {
    rmDispatcher.getEventHandler().handle(event);
}
```

在RMAppImpl中注册了该事件，

```java
// Transitions from NEW_SAVING state
.addTransition(RMAppState.NEW_SAVING, RMAppState.SUBMITTED,
   RMAppEventType.APP_NEW_SAVED, new AddApplicationToSchedulerTransition())
...
  private static final class AddApplicationToSchedulerTransition extends
      RMAppTransition {
    @Override
    public void transition(RMAppImpl app, RMAppEvent event) {
      app.handler.handle(new AppAddedSchedulerEvent(app.applicationId,
        app.submissionContext.getQueue(), app.user,
        app.submissionContext.getReservationID()));
    }
  }
```

当RMAPPImpl收到`RMAppEventType.APP_NEW_SAVED` 事件后，状态变为 SUBMITTED，并向RMAppAttempt发送RMAppAttemptEventType.START事件后，创建一个运行实例对象RMAppAttemptImplement。

### 创建实例对象RMAppAttemptImplement

RMAppAttempt 向 RMApp 发送 **RMAppEventType.APP_ACCEPTED** 事件，RMApp 的状态由 SUBMIITTED 变为 ACCEPTED

RMAppImpl 中注册了该事件

```java
// Transitions from SUBMITTED state
.addTransition(RMAppState.SUBMITTED, RMAppState.ACCEPTED,
      RMAppEventType.APP_ACCEPTED, new StartAppAttemptTransition())
    
  private static final class StartAppAttemptTransition extends RMAppTransition {
    @Override
    public void transition(RMAppImpl app, RMAppEvent event) {
      app.createAndStartNewAttempt(false);
    };
  }
```

RMAPPImpl# createNewAttempt   **创建RMAppAttempt**

```java
private void createAndStartNewAttempt(boolean transferStateFromPreviousAttempt) {
    createNewAttempt();
    // RMAppAttemptEventType.START
    handler.handle(new RMAppStartAttemptEvent(currentAttempt.getAppAttemptId(),transferStateFromPreviousAttempt));
}
private void createNewAttempt() {
    ApplicationAttemptId appAttemptId =
        ApplicationAttemptId.newInstance(applicationId, attempts.size() + 1);
    RMAppAttempt attempt =
        new RMAppAttemptImpl(appAttemptId, rmContext, scheduler, masterService,
          submissionContext, conf,
          /*如果（先前失败的尝试次数（不应包括抢占，硬件错误和NM重新同步）+1）等于最大尝试限制，则新创建的尝试可能是最后一次尝试。*/
          maxAppAttempts == (getNumFailedAppAttempts() + 1), amReq);
    attempts.put(appAttemptId, attempt);
    currentAttempt = attempt;
  }
```





。。。。。。。。。



FairScheduler#handle

```java
 case APP_ADDED:
      if (!(event instanceof AppAddedSchedulerEvent)) {
        throw new RuntimeException("Unexpected event type: " + event);
      }
      AppAddedSchedulerEvent appAddedEvent = (AppAddedSchedulerEvent) event;
      String queueName =
          resolveReservationQueueName(appAddedEvent.getQueue(),
              appAddedEvent.getApplicationId(),
              appAddedEvent.getReservationID());
      if (queueName != null) {
        addApplication(appAddedEvent.getApplicationId(),
            queueName, appAddedEvent.getUser(),
            appAddedEvent.getIsAppRecovering());
      }
      break;
```

FairScheduler#addApplication

```java
protected synchronized void addApplication(ApplicationId applicationId,
      String queueName, String user, boolean isAppRecovering) {
    RMApp rmApp = rmContext.getRMApps().get(applicationId);
    FSLeafQueue queue = assignToQueue(rmApp, queueName, user);
    
     // Enforce ACLs
    UserGroupInformation userUgi = UserGroupInformation.createRemoteUser(user);
    
     // RBAC check
    boolean rbacCheck = RBACAuthorizer.checkPermission(user, queue.getName(), QueueACL.SUBMIT_APPLICATIONS.name());
    
    /**
    SchedulerApplication
            private Queue queue;
            private final String user;
            private T currentAttempt;
    applications       
   		    ConcurrentMap<ApplicationId, SchedulerApplication<T>>        
    **/
    SchedulerApplication<FSAppAttempt> application = new SchedulerApplication<FSAppAttempt>(queue, user);
    applications.put(applicationId, application);
    queue.getMetrics().submitApp(user);
    
    LOG.info("Accepted application " + applicationId + " from user: " + user
            + ", in queue: " + queueName + "(assigned=" + queue.getName() + ")" + ", currently num of applications: "
            + applications.size());
    if (isAppRecovering) {
      if (LOG.isDebugEnabled()) {
        LOG.debug(applicationId + " is recovering. Skip notifying APP_ACCEPTED");
      }
    } else {
      rmContext.getDispatcher().getEventHandler()
        .handle(new RMAppEvent(applicationId, RMAppEventType.APP_ACCEPTED));
    }
}       
```



### RMAppAttemptImpl#AttemptStartedTransition


```java
// Transitions from NEW State
.addTransition(RMAppAttemptState.NEW, RMAppAttemptState.SUBMITTED,
       RMAppAttemptEventType.START, new AttemptStartedTransition())
    
    
  private static final class AttemptStartedTransition extends BaseTransition {
	@Override
    public void transition(RMAppAttemptImpl appAttempt,
        RMAppAttemptEvent event) {

      appAttempt.startTime = System.currentTimeMillis();

      // Register with the ApplicationMasterService
      appAttempt.masterService
          .registerAppAttempt(appAttempt.applicationAttemptId);

      
      // Add the applicationAttempt to the scheduler and inform the scheduler
      // whether to transfer the state from previous attempt. SchedulerEventType.APP_ATTEMPT_ADDED
      appAttempt.eventHandler.handle(new AppAttemptAddedSchedulerEvent(
        appAttempt.applicationAttemptId, transferStateFromPreviousAttempt));
    }
  }
```

发送事件 SchedulerEventType.APP_ATTEMPT_ADDED

FairScheduler#handle

```java
    case APP_ATTEMPT_ADDED:
      if (!(event instanceof AppAttemptAddedSchedulerEvent)) {
        throw new RuntimeException("Unexpected event type: " + event);
      }
      AppAttemptAddedSchedulerEvent appAttemptAddedEvent =
          (AppAttemptAddedSchedulerEvent) event;
      // FairScheduler 的私有方法
      addApplicationAttempt(appAttemptAddedEvent.getApplicationAttemptId(),
        appAttemptAddedEvent.getTransferStateFromPreviousAttempt(),
        appAttemptAddedEvent.getIsAttemptRecovering());
      break;
```

```java
//Add a new application attempt to the scheduler.  
protected synchronized void addApplicationAttempt(
      ApplicationAttemptId applicationAttemptId,
      boolean transferStateFromPreviousAttempt,
      boolean isAttemptRecovering) {
    SchedulerApplication<FSAppAttempt> application =
        applications.get(applicationAttemptId.getApplicationId());
    String user = application.getUser();
    FSLeafQueue queue = (FSLeafQueue) application.getQueue();

    FSAppAttempt attempt =
        new FSAppAttempt(this, applicationAttemptId, user,
            queue, new ActiveUsersManager(getRootQueueMetrics()),
            rmContext);
    if (transferStateFromPreviousAttempt) {
      attempt.transferStateFromPreviousAttempt(application
          .getCurrentAppAttempt());
    }
    application.setCurrentAppAttempt(attempt);

    boolean runnable = maxRunningEnforcer.canAppBeRunnable(queue, user);
    // 核心处理 List<FSAppAttempt> runnableApps = // apps that are runnable
    queue.addApp(attempt, runnable);
    if (runnable) {
      maxRunningEnforcer.trackRunnableApp(attempt);
    } else {
      maxRunningEnforcer.trackNonRunnableApp(attempt);
    }
    
    queue.getMetrics().submitAppAttempt(user);

    LOG.info("Added Application Attempt " + applicationAttemptId
        + " to scheduler from user: " + user);

    if (isAttemptRecovering) {
      if (LOG.isDebugEnabled()) {
        LOG.debug(applicationAttemptId
            + " is recovering. Skipping notifying ATTEMPT_ADDED");
      }
    } else {
      rmContext.getDispatcher().getEventHandler().handle(
        new RMAppAttemptEvent(applicationAttemptId,
            RMAppAttemptEventType.ATTEMPT_ADDED));
    }
  }

```

RMAppAttemptImpl 中注册了RMAppAttemptEventType.ATTEMPT_ADDED事件，执行该事件所触发的方法后，RMAppAttempt 的状态变为 SCHEDULED

```java
// Transitions from SUBMITTED state
.addTransition(RMAppAttemptState.SUBMITTED, EnumSet.of(RMAppAttemptState.LAUNCHED_UNMANAGED_SAVING,RMAppAttemptState.SCHEDULED),
      RMAppAttemptEventType.ATTEMPT_ADDED,new ScheduleTransition())
               
  public static final class ScheduleTransition
      implements
      MultipleArcTransition<RMAppAttemptImpl, RMAppAttemptEvent, RMAppAttemptState> {
    @Override
    public RMAppAttemptState transition(RMAppAttemptImpl appAttempt,
        RMAppAttemptEvent event) {
      ApplicationSubmissionContext subCtx = appAttempt.submissionContext;
      // getUnmanagedAM():true if the AM is not managed by the RM
      if (!subCtx.getUnmanagedAM()) { // 提交的新任务进这里
        // Currently, following fields are all hard code,
        // TODO: change these fields when we want to support
        // priority/resource-name/relax-locality specification for AM containers  allocation.
        appAttempt.amReq.setNumContainers(1);
        appAttempt.amReq.setPriority(AM_CONTAINER_PRIORITY);
        appAttempt.amReq.setResourceName(ResourceRequest.ANY);
        appAttempt.amReq.setRelaxLocality(true);
        
        // AM resource has been checked when submission
        Allocation amContainerAllocation =
            appAttempt.scheduler.allocate(appAttempt.applicationAttemptId,
                Collections.singletonList(appAttempt.amReq),
                EMPTY_CONTAINER_RELEASE_LIST, null, null);
        if (amContainerAllocation != null
            && amContainerAllocation.getContainers() != null) {
          assert (amContainerAllocation.getContainers().size() == 0);
        }
        return RMAppAttemptState.SCHEDULED;
      } else {
        // save state and then go to LAUNCHED state
        appAttempt.storeAttempt();
        return RMAppAttemptState.LAUNCHED_UNMANAGED_SAVING;
      }
    }
  }
```

## ApplicatonMaster 的启动

当作业的 appattempt 被分配到资源后，状态从`ALLOCATED_SAVING`变成`ALLOCATED`时，由`AttemptStoredTransition.transition`调用`appAttempt.launchAttempt()`进行启动，下面来看下具体代码:

```java
private void launchAttempt(){
    launchAMStartTime = System.currentTimeMillis();
    // Send event to launch the AM Container
    // 通过异步调度器得到该事件注册的handle (在ResourceManager中注册)
    // AMLauncherEvent 对应的handle是ApplicationMasterLauncher
    eventHandler.handle(new AMLauncherEvent(AMLauncherEventType.LAUNCH, this));
}
```

AMLauncherEvent对应的handle是ApplicationMasterLauncher，事件类型是LAUNCH，在`ApplicationMasterLauncher.handle`中会调用`launch(application)`，代码如下:

```java
private void launch(RMAppAttempt application) {
      // 创建一个线程
      Runnable launcher = createRunnableLauncher(application, AMLauncherEventType.LAUNCH);
      // 将线程放入阻塞队列中 BlockingQueue<Runnable> masterEvents = new LinkedBlockingQueue<Runnable>();
      masterEvents.add(launcher);
}
```

只从这个方法来分析，首先创建了一个launcher线程，然后将其放入一个队列中，等待另一个线程从队列中取出进行操作，这是典型的生产者消费者模型。那么我们就来看下`ApplicationMasterLauncher`(ApplicationMasterLauncher是一个事件也是一个服务)关于这块代码的具体实现：

```java
protected Runnable createRunnableLauncher(RMAppAttempt application, 
    AMLauncherEventType event) {
  Runnable launcher = new AMLauncher(context, application, event, getConfig());
  return launcher;
}
```

这里只是new了一个AMLauncher，AMLauncher实现了Runnable接口，是执行AM操作的线程，只执行`launch`和`cleanup`。

launcher线程创建之后add到阻塞队列masterEvents中，那么必然会有另一个线程来队列中take launcher，这个线程是`LauncherThread`类型的`launcherHandlingThread`，launcherHandlingThread将launcher取出丢给线程池去执行，代码如下:

```java
private class LauncherThread extends Thread {
    public LauncherThread() {
        super("ApplicationMaster Launcher");
    }
    @Override
    public void run() {
        while (!this.isInterrupted()) {
            Runnable toLaunch;
            try {
                // 从阻塞队列中取出
                toLaunch = masterEvents.take();
                // 交给线程执行
                // launcherPool = new ThreadPoolExecutor(threadCount, threadCount, 1,TimeUnit.HOURS, new LinkedBlockingQueue<Runnable>());
                launcherPool.execute(toLaunch);
            } catch (InterruptedException e) {
                LOG.warn(this.getClass().getName() + " interrupted. Returning.");
                return;
            }
        }
    }
}   
```

放入线程池之后，launcher线程就开始执行，调用的是`AMLauncher.run`

```java
  public void run() {
    switch (eventType) {
    case LAUNCH:
      try {
        LOG.info("Launching master" + application.getAppAttemptId());
        launch();
        handler.handle(new RMAppAttemptEvent(application.getAppAttemptId(),
            RMAppAttemptEventType.LAUNCHED));
      } catch(Exception ie) {
        String message = "Error launching " + application.getAppAttemptId()
            + ". Got exception: " + StringUtils.stringifyException(ie);
        LOG.info(message);
        handler.handle(new RMAppAttemptEvent(application
            .getAppAttemptId(), RMAppAttemptEventType.LAUNCH_FAILED, message));
      }
      break;
    case CLEANUP:
      try {
        LOG.info("Cleaning master " + application.getAppAttemptId());
        cleanup();
      } catch(IOException ie) {
        LOG.info("Error cleaning master ", ie);
      } catch (YarnException e) {
        StringBuilder sb = new StringBuilder("Container ");
        sb.append(masterContainer.getId().toString());
        sb.append(" is not handled by this NodeManager");
        if (!e.getMessage().contains(sb.toString())) {
          // Ignoring if container is already killed by Node Manager.
          LOG.info("Error cleaning master ", e);          
        }
      }
      break;
    default:
      LOG.warn("Received unknown event-type " + eventType + ". Ignoring.");
      break;
    }
  }
```

因为之前放入阻塞队列 masterEvents 的事件类型是 LAUNCH，则此处调用 `launch()` 方法：

```java
private void launch() throws IOException, YarnException {
    // 获取对应 NodeManager 的 rpc 客户端
    connect();
    ContainerId masterContainerID = masterContainer.getId();
    ApplicationSubmissionContext applicationContext =
        application.getSubmissionContext();
    LOG.info("Setting up container " + masterContainer
             + " for AM " + application.getAppAttemptId());  
    ContainerLaunchContext launchContext =
        createAMContainerLaunchContext(applicationContext, masterContainerID);
    // 构建request
    StartContainerRequest scRequest =
        StartContainerRequest.newInstance(launchContext,
                                          masterContainer.getContainerToken());
    List<StartContainerRequest> list = new ArrayList<StartContainerRequest>();
    list.add(scRequest);
    StartContainersRequest allRequests =
        StartContainersRequest.newInstance(list);
    // 远程调用 startContainers，在 NodeManager 上启动 AM Container
    StartContainersResponse response =
        containerMgrProxy.startContainers(allRequests);
    if (response.getFailedRequests() != null
        && response.getFailedRequests().containsKey(masterContainerID)) {
        Throwable t =
            response.getFailedRequests().get(masterContainerID).deSerialize();
        parseAndThrowException(t);
    } else {
        LOG.info("Done launching container " + masterContainer + " for AM "
                 + application.getAppAttemptId());
    }
}
```

AMLaunch.launch先在`connect()`中拿到对应node的rpc客户端`containerMgrProxy`，然后构造request，最后调用rpc函数`startContainers()`并返回response。看下*node端*的`startContainers`代码:

## NM

```java
public StartContainersResponse
    startContainers(StartContainersRequest requests) throws YarnException,
        IOException {
  ...
  UserGroupInformation remoteUgi = getRemoteUgi();
  NMTokenIdentifier nmTokenIdentifier = selectNMTokenIdentifier(remoteUgi);
  authorizeUser(remoteUgi,nmTokenIdentifier);
  List<ContainerId> succeededContainers = new ArrayList<ContainerId>();
  Map<ContainerId, SerializedException> failedContainers =
      new HashMap<ContainerId, SerializedException>();
  for (StartContainerRequest request : requests.getStartContainerRequests()) {
    ContainerId containerId = null;
    try {
      ContainerTokenIdentifier containerTokenIdentifier =
          BuilderUtils.newContainerTokenIdentifier(request.getContainerToken());
      verifyAndGetContainerTokenIdentifier(request.getContainerToken(),
        containerTokenIdentifier);
      containerId = containerTokenIdentifier.getContainerID();
      startContainerInternal(nmTokenIdentifier, containerTokenIdentifier,
        request);
      succeededContainers.add(containerId);
    } catch (YarnException e) {
      failedContainers.put(containerId, SerializedException.newInstance(e));
    } catch (InvalidToken ie) {
      failedContainers.put(containerId, SerializedException.newInstance(ie));
      throw ie;
    } catch (IOException e) {
      throw RPCUtil.getRemoteException(e);
    }
  }

  return StartContainersResponse.newInstance(getAuxServiceMetaData(),
    succeededContainers, failedContainers);
}
```

startContainers对request中的container请求进行遍历，调用`startContainerInternal`启动一个container，这个container是在nodemanager上准备运行task的。启动成功的放入succeededContainers列表中，失败的则放入failedContainers中，遍历结束构造一个response返回给rm。

看下启动container的startContainerInternal方法:

```java
 private void startContainerInternal(NMTokenIdentifier nmTokenIdentifier,
      ContainerTokenIdentifier containerTokenIdentifier,
      StartContainerRequest request) throws YarnException, IOException {
	...
    ContainerId containerId = containerTokenIdentifier.getContainerID();
    String containerIdStr = containerId.toString();
    String user = containerTokenIdentifier.getApplicationSubmitter();

    LOG.info("Start request for " + containerIdStr + " by user " + user);
	// 得到当前 container 的上下文信息
    ContainerLaunchContext launchContext = request.getContainerLaunchContext();
	...
    // 创建 container 对象，开始NodeManager上container的状态机转换
    // container 的初始化状态为 NEW
    Container container =
        new ContainerImpl(getConfig(), this.dispatcher,
            context.getNMStateStore(), launchContext,
          credentials, metrics, containerTokenIdentifier);
    ApplicationId applicationID =
        containerId.getApplicationAttemptId().getApplicationId();
    // 将container放入context的containers中
    if (context.getContainers().putIfAbsent(containerId, container) != null) {
      NMAuditLogger.logFailure(user, AuditConstants.START_CONTAINER,
        "ContainerManagerImpl", "Container already running on this node!",
        applicationID, containerId);
      throw RPCUtil.getRemoteException("Container " + containerIdStr
          + " already is running on this node!!");
    }

    this.readLock.lock();
    try {
      if (!serviceStopped) {
        // Create the application
        Application application =
            new ApplicationImpl(dispatcher, user, applicationID, credentials, context);
        // 如果是该application的第一个container，则进行一些辅助操作，如启动log aggregation服务
        if (null == context.getApplications().putIfAbsent(applicationID,
          application)) {
          LOG.info("Creating a new application reference for app " + applicationID);
          LogAggregationContext logAggregationContext =
              containerTokenIdentifier.getLogAggregationContext();
          Map<ApplicationAccessType, String> appAcls =
              container.getLaunchContext().getApplicationACLs();
          // logAggregationContext放入context中共用
          context.getNMStateStore().storeApplication(applicationID,
              buildAppProto(applicationID, user, credentials, appAcls,
                logAggregationContext));
          // 触发ApplicationEventType.INIT_APPLICATION事件类型
          dispatcher.getEventHandler().handle(
            new ApplicationInitEvent(applicationID, appAcls,
              logAggregationContext));
        }

        this.context.getNMStateStore().storeContainer(containerId, request);
        dispatcher.getEventHandler().handle(
          new ApplicationContainerInitEvent(container));
		// 触发ApplicationEventType.INIT_CONTAINER事件类型
        this.context.getContainerTokenSecretManager().startContainerSuccessful(
          containerTokenIdentifier);
          
        NMAuditLogger.logSuccess(user, AuditConstants.START_CONTAINER,
          "ContainerManageImpl", applicationID, containerId);
        ...metrics
      } else {
        throw new YarnException(
            "Container start failed as the NodeManager is " +
            "in the process of shutting down");
      }
    } finally {
      this.readLock.unlock();
    }
  }
```

startContainerInternal首先创建一个container对象，这也就开启了container的状态机之旅，新建的container状态是NEW。
随后判断该container是否是该application的第一个container，如果是则启动日志聚合功能，并触发ApplicationEventType.INIT_APPLICATION事件类型，使application的状态由*ApplicationState.NEW*变为*ApplicationState.INITING*，最后触发ApplicationEventType.INIT_CONTAINER事件类型，更新container的状态。如果不是则直接触发ApplicationEventType.INIT_CONTAINER事件类型。

if语句执行完之后，application的状态由NEW变为了INITING，此时将container信息存储在context中，并触发ApplicationEventType.INIT_CONTAINER事件类型，*由于此时application的状态是INITING，事件类型为INIT_CONTAINER，则处理次事件的handler是`InitContainerTransition`，随后application的状态依然是INITING*，看下InitContainerTransition.transition方法:

```java
public void transition(ApplicationImpl app, ApplicationEvent event) {
    ApplicationContainerInitEvent initEvent =
        (ApplicationContainerInitEvent) event;
    Container container = initEvent.getContainer();
    app.containers.put(container.getContainerId(), container);
    LOG.info("Adding " + container.getContainerId()
             + " to application " + app.toString());

    switch (app.getApplicationState()) {
        case RUNNING:
            app.dispatcher.getEventHandler().handle(new ContainerInitEvent(
                container.getContainerId()));
            break;
        case INITING:
        case NEW:
            // these get queued up and sent out in AppInitDoneTransition
            break;
        default:
            assert false : "Invalid state for InitContainerTransition: " +
                app.getApplicationState();
    }
}
```

看代码可见当application的状态是INITING和NEW时，触发INIT_CNONTAINER方法时不进行任何操作，则application的状态也不会发生变化。*只有当application的状态时RUNNING时*，才会触发由`ContainerEventType.INIT_CONTAINER`事件类型触发container的状态转移。

那么何时application的状态才会变为RUNNING呢？我们回到在新建application对象时触发的`ApplicationEventType.INIT_APPLICATION`事件类型上，此时application刚被new出来，则初始状态上NEW，则处理该事件类型的handler是`AppInitTransition`，下面看下`AppInitTransition.transition`

```java
public void transition(ApplicationImpl app, ApplicationEvent event) {
  ApplicationInitEvent initEvent = (ApplicationInitEvent)event;
  ...
  app.dispatcher.getEventHandler().handle(
      new LogHandlerAppStartedEvent(app.appId, app.user,
          app.credentials, ContainerLogsRetentionPolicy.ALL_CONTAINERS,
          app.applicationACLs, app.logAggregationContext)); 
}
```

该transtion中是一个异步调度器，处理的事件是`LogHandlerAppStartedEvent`，事件类型是`LogHandlerEventType.APPLICATION_STARTED`，此事件类型是在`LogAggregationService`中处理的，看下对应的handle方法:

```java
public void handle(LogHandlerEvent event) {
  switch (event.getType()) {
    case APPLICATION_STARTED:
      LogHandlerAppStartedEvent appStartEvent =
          (LogHandlerAppStartedEvent) event;
      initApp(appStartEvent.getApplicationId(), appStartEvent.getUser(),
          appStartEvent.getCredentials(),
          appStartEvent.getLogRetentionPolicy(),
          appStartEvent.getApplicationAcls(),
          appStartEvent.getLogAggregationContext());
      break;
    case CONTAINER_FINISHED:
      ...
      break;
    case APPLICATION_FINISHED:
      ...
      break;
    default:
      ; // Ignore
  }
}
```

LogAggregationService从名字来看，只要是用来进行日志聚合的，处理的事件类型有`APPLICATION_STARTED`、`CONTAINER_FINISHED`和`APPLICATION_FINISHED`。
APPLICATION_STARTED事件类型会调用`initApp`方法，

```java
private void initApp(final ApplicationId appId, String user,
    Credentials credentials, ContainerLogsRetentionPolicy logRetentionPolicy,
    Map<ApplicationAccessType, String> appAcls,
    LogAggregationContext logAggregationContext) {
  ApplicationEvent eventResponse;
  try {
    verifyAndCreateRemoteLogDir(getConfig());
    initAppAggregator(appId, user, credentials, logRetentionPolicy, appAcls,
        logAggregationContext);
    eventResponse = new ApplicationEvent(appId,
        ApplicationEventType.APPLICATION_LOG_HANDLING_INITED);
  } catch (YarnRuntimeException e) {
    LOG.warn("Application failed to init aggregation", e);
    eventResponse = new ApplicationEvent(appId,
        ApplicationEventType.APPLICATION_LOG_HANDLING_FAILED);
  }
  this.dispatcher.getEventHandler().handle(eventResponse);
}
```

logDir创建成功之后，会触发`ApplicationEventType.APPLICATION_LOG_HANDLING_INITED`事件类型，此时application的状态是INITING，则对应的handler是`AppLogInitDoneTransition`，看下transition方法:

```java
public void transition(ApplicationImpl app, ApplicationEvent event) {
  app.dispatcher.getEventHandler().handle(
      new ApplicationLocalizationEvent(
          LocalizationEventType.INIT_APPLICATION_RESOURCES, app));
}
```

当LogAggregationService为application创建了logDir并且启动日志聚合线程之后，才会通过AppLogInitDoneTransition处理APPLICATION_LOG_HANDLING_INITED事件。
在AppLogInitDoneTransition中触发`LocalizationEventType.INIT_APPLICATION_RESOURCES`在ResourceLocalizationService中被捕获，

```java
public void handle(LocalizationEvent event) {
  // TODO: create log dir as $logdir/$user/$appId
  switch (event.getType()) {
  case INIT_APPLICATION_RESOURCES:
    handleInitApplicationResources(
        ((ApplicationLocalizationEvent)event).getApplication());
    break;
  case INIT_CONTAINER_RESOURCES:
    handleInitContainerResources((ContainerLocalizationRequestEvent) event);
    break;
  case CACHE_CLEANUP:
    handleCacheCleanup(event);
    break;
  case CLEANUP_CONTAINER_RESOURCES:
    handleCleanupContainerResources((ContainerLocalizationCleanupEvent)event);
    break;
  case DESTROY_APPLICATION_RESOURCES:
    handleDestroyApplicationResources(
        ((ApplicationLocalizationEvent)event).getApplication());
    break;
  default:
    throw new YarnRuntimeException("Unknown localization event: " + event);
  }
}
```

handle中处理不同的事件类型，INIT_APPLICATION_RESOURCES由`handleInitApplicationResources`处理

```java
  private void handleInitApplicationResources(Application app) {
    // 0) Create application tracking structs
    String userName = app.getUser();
    privateRsrc.putIfAbsent(userName, new LocalResourcesTrackerImpl(userName,
        null, dispatcher, true, super.getConfig(), stateStore));
    String appIdStr = ConverterUtils.toString(app.getAppId());
    appRsrc.putIfAbsent(appIdStr, new LocalResourcesTrackerImpl(app.getUser(),
        app.getAppId(), dispatcher, false, super.getConfig(), stateStore));
    // 1) Signal container init
    //
    // This is handled by the ApplicationImpl state machine and allows
    // containers to proceed with launching.
    dispatcher.getEventHandler().handle(new ApplicationInitedEvent(
          app.getAppId()));
  }
```

这里会触发`ApplicationEventType.APPLICATION_INITED`，在application的状态机中处理，对应的handler是`AppInitDoneTransition`，处理之后application的状态由INITING转化为RUNNING。AppInitDoneTransition.transition方法如下:

```java
public void transition(ApplicationImpl app, ApplicationEvent event) {
  // Start all the containers waiting for ApplicationInit
  for (Container container : app.containers.values()) {
    app.dispatcher.getEventHandler().handle(new ContainerInitEvent(
          container.getContainerId()));
  }
}
```

这里会触发ContainerEventType.INIT_CONTAINER事件类型，由此事件类型开启container的状态转移。
在`startContainerInternal`中新建了一个container，初始化状态为NEW。此时当`ApplicationEventType.APPLICATION_INITED`触发之后，对应的handler会触发`ContainerEventType.INIT_CONTAINER`，container开始状态的转化，对应的handler是`RequestResourcesTransition`，看下transtion的代码:

```java
 public ContainerState transition(ContainerImpl container,
        ContainerEvent event) {
	  ...
      final ContainerLaunchContext ctxt = container.launchContext;
      container.metrics.initingContainer();

      container.dispatcher.getEventHandler().handle(new AuxServicesEvent
          (AuxServicesEventType.CONTAINER_INIT, container));

      // Inform the AuxServices about the opaque serviceData
      Map<String,ByteBuffer> csd = ctxt.getServiceData();
      if (csd != null) {
        // This can happen more than once per Application as each container may
        // have distinct service data
        for (Map.Entry<String,ByteBuffer> service : csd.entrySet()) {
          container.dispatcher.getEventHandler().handle(
              new AuxServicesEvent(AuxServicesEventType.APPLICATION_INIT,
                  container.user, container.containerId
                      .getApplicationAttemptId().getApplicationId(),
                  service.getKey().toString(), service.getValue()));
        }
      }
	  // 为public和private资源发送远程请求，这里的请求协议是yarn_protos.ContainerLaunchContextProto
      // Send requests for public, private resources
      Map<String,LocalResource> cntrRsrc = ctxt.getLocalResources();
      if (!cntrRsrc.isEmpty()) {
        try {
          for (Map.Entry<String,LocalResource> rsrc : cntrRsrc.entrySet()) {
            try {
              LocalResourceRequest req =
                  new LocalResourceRequest(rsrc.getValue());
              List<String> links = container.pendingResources.get(req);
              if (links == null) {
                links = new ArrayList<String>();
                container.pendingResources.put(req, links);
              }
              links.add(rsrc.getKey());
              storeSharedCacheUploadPolicy(container, req, rsrc.getValue()
                  .getShouldBeUploadedToSharedCache());
              switch (rsrc.getValue().getVisibility()) {
              case PUBLIC:
                container.publicRsrcs.add(req);
                break;
              case PRIVATE:
                container.privateRsrcs.add(req);
                break;
              case APPLICATION:
                container.appRsrcs.add(req);
                break;
              }
            } catch (URISyntaxException e) {
              LOG.info("Got exception parsing " + rsrc.getKey()
                  + " and value " + rsrc.getValue());
              throw e;
            }
          }
        } catch (URISyntaxException e) {
          // malformed resource; abort container launch
          LOG.warn("Failed to parse resource-request", e);
          container.cleanup();
          container.metrics.endInitingContainer();
          return ContainerState.LOCALIZATION_FAILED;
        }
        Map<LocalResourceVisibility, Collection<LocalResourceRequest>> req =
            new LinkedHashMap<LocalResourceVisibility,
                        Collection<LocalResourceRequest>>();
        if (!container.publicRsrcs.isEmpty()) {
          req.put(LocalResourceVisibility.PUBLIC, container.publicRsrcs);
        }
        if (!container.privateRsrcs.isEmpty()) {
          req.put(LocalResourceVisibility.PRIVATE, container.privateRsrcs);
        }
        if (!container.appRsrcs.isEmpty()) {
          req.put(LocalResourceVisibility.APPLICATION, container.appRsrcs);
        }
        
        container.dispatcher.getEventHandler().handle(
              new ContainerLocalizationRequestEvent(container, req));
        return ContainerState.LOCALIZING;
      } else {
        container.sendLaunchEvent();
        container.metrics.endInitingContainer();
        return ContainerState.LOCALIZED;
      }
    }
```

从该handler的名字上可以看出其主要作用是请求container的resources，方法中会判断该container是否需要请求resources，

- 如果需要则将资源进行本地化，触发`LocalizationEventType.INIT_CONTAINER_RESOURCES`，返回`ContainerState.LOCALIZING`，使*container由NEW转换为LOCALIZING状态*。而LocalizationEventType.INIT_CONTAINER_RESOURCES被`ResourceLocalizationService`进行捕获，开始Resource的本地化。
- 如果不需要则发送`LAUNCH_CONTAINER`事件，返回`ContainerState.LOCALIZED`，使*container从NEW直接转化为LOCALIZED*。

这里我们跟下不需要请求resources的情况，调用`sendLaunchEvent`发送`ContainersLauncherEventType.LAUNCH_CONTAINER`事件类型，由`ContainersLauncher`捕获。其handle方法如下:

```java
  public void handle(ContainersLauncherEvent event) {
    // TODO: ContainersLauncher launches containers one by one!!
    Container container = event.getContainer();
    ContainerId containerId = container.getContainerId();
    switch (event.getType()) {
      case LAUNCH_CONTAINER:
        Application app =
          context.getApplications().get(
              containerId.getApplicationAttemptId().getApplicationId());
		// 创建一个 ContainerLauncher线程，然后放入线程池containerLauncher 中执行
        ContainerLaunch launch =
            new ContainerLaunch(context, getConfig(), dispatcher, exec, app,
              event.getContainer(), dirsHandler, containerManager);
        containerLauncher.submit(launch);
        running.put(containerId, launch);
        break;
      case RECOVER_CONTAINER:
        ...
        break;
      case CLEANUP_CONTAINER:
   		...
        break;
    }
  }
```

ContainerLaunch线程提交到containerLauncher线程池之后开始执行此线程。ContainerLaunch实现了Callable接口，则线程的执行逻辑在call方法中，如下:

```java
public Integer call() {
  ...
  try {
    ...
    // LaunchContainer is a blocking call. We are here almost means the
    // container is launched, so send out the event.
    // 处理ContainerEventType.CONTAINER_LAUNCHED，使container由LOCALIZED变为RUNNING
    // 并开始监控这个container使用的内存(物理内存和虚拟内存)
    dispatcher.getEventHandler().handle(new ContainerEvent(
          containerID,
          ContainerEventType.CONTAINER_LAUNCHED));
    context.getNMStateStore().storeContainerLaunched(containerID);

    // Check if the container is signalled to be killed.
    if (!shouldLaunchContainer.compareAndSet(false, true)) {
      LOG.info("Container " + containerIdStr + " not launched as "
          + "cleanup already called");
      ret = ExitCode.TERMINATED.getExitCode();
    }
    else {
      exec.activateContainer(containerID, pidFilePath);
      // 执行启动container的脚本
      ret = exec.launchContainer(container, nmPrivateContainerScriptPath,
              nmPrivateTokensPath, user, appIdStr, containerWorkDir,
              localDirs, logDirs);
    }
  } catch (Throwable e) {
    LOG.warn("Failed to launch container.", e);
    dispatcher.getEventHandler().handle(new ContainerExitEvent(
        containerID, ContainerEventType.CONTAINER_EXITED_WITH_FAILURE, ret,
        e.getMessage()));
    return ret;
  } finally {
    completed.set(true);
    exec.deactivateContainer(containerID);
    try {
      context.getNMStateStore().storeContainerCompleted(containerID, ret);
    } catch (IOException e) {
      LOG.error("Unable to set exit code for container " + containerID);
    }
  }
  ...
  dispatcher.getEventHandler().handle(
      new ContainerEvent(containerID,
          ContainerEventType.CONTAINER_EXITED_WITH_SUCCESS));
  return 0;
}
```

ContainerLaunch将container运行环境准备好之后，触发`ContainerEventType.CONTAINER_LAUNCHED`事件类型，LaunchTransition捕获之后，**触发监控container的事件(监控container所需的物理内存和虚拟内存)**，使container由LOCALIZED变为RUNNING。

触发ContainerEventType.CONTAINER_LAUNCHED事件类型之后，继续执行，调用`exec.launchContainer`启动container，*该方法在container执行完毕之后才会返回*。如果正常结束ret为0，触发`ContainerEventType.CONTAINER_EXITED_WITH_SUCCESS`事件类型，*使container由RUNNING变为EXITED_WITH_SUCCESS*。



exec.launchContainer的实现是`DefaultContainerExecutor.launchContainer`，在launchContainer方法中会执行`bash default_container_executor.sh`命令，default_container_executor.sh脚本的内容是:

```shell
#!/bin/bash
/bin/bash "/xx/usercache/user/appcache/application_1499422474367_0001/container_1499422474367_0001_01_000001/default_container_executor_session.sh"
...
```

调用了default_container_executor_session.sh脚本

```shell
#!/bin/bash
...
exec /bin/bash "/xx/usercache/user/appcache/application_1499422474367_0001/container_1499422474367_0001_01_000001/launch_container.sh"
```

最后调用了launch_container.sh脚本，内容如下:

```shell
...
exec /bin/bash -c "$JAVA_HOME/bin/java -Dlog4j.configuration=container-log4j.properties -Dyarn.app.container.log.dir=/xx/application_1499422474367_0001/container_1499422474367_0001_01_000001 -Dyarn.app.container.log.filesize=0 -Dhadoop.root.logger=INFO,CLA  -Xmx1024m org.apache.hadoop.mapreduce.v2.app.MRAppMaster 1>/xx/application_1499422474367_0001/container_1499422474367_0001_01_000001/stdout 2>/xx/application_1499422474367_0001/container_1499422474367_0001_01_000001/stderr "
...
```

这个container是appmaster，所以这里调用的是`MRAppMaster`，并将标准输出写到stdout中，将标准错误输出写到stderr中。这也就是container的log目录里有三个文件的原因。

至此，AppMaster启动完毕，过程比较繁琐，下面附上一张图。随后介绍下MRAppMaster的运行过程。

![appMaster](images/Yarn 提交一个MapReduce作业的流程/appMaster.png)



参考引用链接：

* [YARN源码分析之ApplicationMaster启动流程 | big data decode club](http://bigdatadecode.club/YARNSrcApplicationMasterStart.html)


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

## AM 



## NM
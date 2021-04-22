# MRAppMaster 源码

MRAppMaster 主要组成

* ClientService
* ContainerAllocator
  * RMCommunicator
* ContainerLauncher
* TaskAttemptListener
  * TaskHeartbeatHandler
* Speculator

## MRAppMaster 的启动

MRAppMaster 的入口 main 函数

```java
  public static void main(String[] args) {
    try {
      ...
      ContainerId containerId = ConverterUtils.toContainerId(containerIdStr);
      ApplicationAttemptId applicationAttemptId =
          containerId.getApplicationAttemptId();
      long appSubmitTime = Long.parseLong(appSubmitTimeStr);
      // 创建 MR 对应的 MRAppMaster 对象
      MRAppMaster appMaster =
          new MRAppMaster(applicationAttemptId, containerId, nodeHostString,
              Integer.parseInt(nodePortString),
              Integer.parseInt(nodeHttpPortString), appSubmitTime);
      ShutdownHookManager.get().addShutdownHook(
        new MRAppMasterShutdownHook(appMaster), SHUTDOWN_HOOK_PRIORITY);
      JobConf conf = new JobConf(new YarnConfiguration());
      // 添加本地化作业的配置文件(job.xml)
      conf.addResource(new Path(MRJobConfig.JOB_CONF_FILE));
      
      MRWebAppUtil.initialize(conf);
      // 从系统环境变量中获取用户名
      String jobUserName = System
          .getenv(ApplicationConstants.Environment.USER.name());
      conf.set(MRJobConfig.USER_NAME, jobUserName);
      // 初始化并启动
      initAndStartAppMaster(appMaster, conf, jobUserName);
    } catch (Throwable t) {
      LOG.fatal("Error starting MRAppMaster", t);
      ExitUtil.terminate(1, t);
    }
  }
```

job.xml 为提交作业时由客户端生成的作业配置文件，该配置文件包含的内容：

* 客户端 core-site.xml/core-default.xml、yarn-site.xml/yarn-default.xml、hdfs-site.xml/hdfs-default.xml、maprd-site.xml/mapred-default.xml
* programatically （代码中通过conf.set() 添加的）
* from command line （提交作业时命令行）

在 launch_container.sh 中，会把 ApplicationConstants.Environment 所需的所有配置，export 到系统环境变量中，因此在程序中可直接从系统环境变量获取。

在 initAndStartAppMaster() 中初始化并启动AppMaster

```java
  protected static void initAndStartAppMaster(final MRAppMaster appMaster,
      final JobConf conf, String jobUserName) throws IOException,
      InterruptedException {
    UserGroupInformation.setConfiguration(conf);
	...   
    UserGroupInformation appMasterUgi = UserGroupInformation
        .createRemoteUser(jobUserName);
    appMasterUgi.addCredentials(credentials);
	...
    conf.getCredentials().addAll(credentials);
    appMasterUgi.doAs(new PrivilegedExceptionAction<Object>() {
      @Override
      public Object run() throws Exception {
        // 初始化 
        appMaster.init(conf);
        // 启动
        appMaster.start();
        if(appMaster.errorHappenedShutDown) {
          throw new IOException("Was asked to shut down.");
        }
        return null;
      }
    });
  }
```

应为 MRAppMaster 继承自 CompositeService，因此初始化最终会调用，MRAppMaster 的 serviceInit 方法。

```java
  protected void serviceInit(final Configuration conf) throws Exception {
    // 初始化作业运行的相关配置
    ...
    // 判断一些目录是否存在
    try {
       String user = UserGroupInformation.getCurrentUser().getShortUserName();
       // /tmp/hadoop-yarn/staging/user/.staging
       Path stagingDir = MRApps.getStagingAreaDir(conf, user);
       FileSystem fs = getFileSystem(conf);
       boolean stagingExists = fs.exists(stagingDir);
       // /tmp/hadoop-yarn/staging/user/.staging/jobId/COMMIT_STARTED
       Path startCommitFile = MRApps.getStartJobCommitFile(conf, user, jobId);
       boolean commitStarted = fs.exists(startCommitFile);
       // /tmp/hadoop-yarn/staging/user/.staging/jobId/COMMIT_SUCCESS
       Path endCommitSuccessFile = MRApps.getEndJobCommitSuccessFile(conf, user, jobId);
       boolean commitSuccess = fs.exists(endCommitSuccessFile);
       // /tmp/hadoop-yarn/staging/user/.staging/jobId/COMMIT_FAIL
       Path endCommitFailureFile = MRApps.getEndJobCommitFailureFile(conf, user, jobId);
       boolean commitFailure = fs.exists(endCommitFailureFile);
       ...
     } catch (IOException e) {
       throw new YarnRuntimeException("Error while initializing", e);
    }  
    ...
    if (errorHappenedShutDown) {
      // 当发生错误时的错误处理
    } else {
      // 创建 OutputCommitter
      committer = createOutputCommitter(conf);
	  // 创建 事件分发器
      dispatcher = createDispatcher();
      addIfService(dispatcher);

      //service to handle requests from JobClient
      // 创建 ClientService
      clientService = createClientService(context);
      // Init ClientService separately so that we stop it separately, since this
      // service needs to wait some time before it stops so clients can know the
      // final states
      clientService.init(conf);
      // 创建ContainerAllocator，负责向 ResourceManager 申请资源
      containerAllocator = createContainerAllocator(clientService, context);
      
      //service to handle the output committer
      committerEventHandler = createCommitterEventHandler(context, committer);
      addIfService(committerEventHandler);

      //service to handle requests to TaskUmbilicalProtocol
      // 监听 task 的状态变化
      taskAttemptListener = createTaskAttemptListener(context);
      addIfService(taskAttemptListener);

      //service to log job history events
      EventHandler<JobHistoryEvent> historyService = 
        createJobHistoryHandler(context);
      dispatcher.register(org.apache.hadoop.mapreduce.jobhistory.EventType.class,
          historyService);

      this.jobEventDispatcher = new JobEventDispatcher();

      //register the event dispatchers
      dispatcher.register(JobEventType.class, jobEventDispatcher);
      dispatcher.register(TaskEventType.class, new TaskEventDispatcher());
      dispatcher.register(TaskAttemptEventType.class, 
          new TaskAttemptEventDispatcher());
      dispatcher.register(CommitterEventType.class, committerEventHandler);
	  // 判断 map、reduce task 是否开启推测执行
      if (conf.getBoolean(MRJobConfig.MAP_SPECULATIVE, false)
          || conf.getBoolean(MRJobConfig.REDUCE_SPECULATIVE, false)) {
        //optional service to speculate on task attempts' progress
        speculator = createSpeculator(conf, context);
        addIfService(speculator);
      }
      // 创建推测执行时间分发器，并注册到全局事件分发器
      speculatorEventDispatcher = new SpeculatorEventDispatcher(conf);
      dispatcher.register(Speculator.EventType.class,
          speculatorEventDispatcher);

      // Now that there's a FINISHING state for application on RM to give AMs
      // plenty of time to clean up after unregister it's safe to clean staging
      // directory after unregistering with RM. So, we start the staging-dir
      // cleaner BEFORE the ContainerAllocator so that on shut-down,
      // ContainerAllocator unregisters first and then the staging-dir cleaner
      // deletes staging directory.
      addService(createStagingDirCleaningService());

      // service to allocate containers from RM (if non-uber) or to fake it (uber)
      addIfService(containerAllocator);
      dispatcher.register(ContainerAllocator.EventType.class, containerAllocator);

      // corresponding service to launch allocated containers via NodeManager
      containerLauncher = createContainerLauncher(context);
      addIfService(containerLauncher);
      dispatcher.register(ContainerLauncher.EventType.class, containerLauncher);

      // Add the JobHistoryEventHandler last so that it is properly stopped first.
      // This will guarantee that all history-events are flushed before AM goes
      // ahead with shutdown.
      // Note: Even though JobHistoryEventHandler is started last, if any
      // component creates a JobHistoryEvent in the meanwhile, it will be just be
      // queued inside the JobHistoryEventHandler 
      addIfService(historyService);
    }
    super.serviceInit(conf);
  } // end of init()
```

MRAppMaster 的初始化中，主要初始化一些运行所需的变量，及必须的一些服务。

MRAppMaster 的启动

```java
protected void serviceStart() throws Exception {
    // 判断 job 是否已 uber 方式启动
    if (job.isUber()) {
        MRApps.setupDistributedCacheLocal(getConfig());
        this.containerAllocator = new LocalContainerAllocator(
            this.clientService, this.context, nmHost, nmPort, nmHttpPort
            , containerID);
    } else {
        this.containerAllocator = new RMContainerAllocator(
            this.clientService, this.context);
    }
    ((Service)this.containerAllocator).init(getConfig());
    ((Service)this.containerAllocator).start();
    super.serviceStart();
}
```

MRAppMaster 启动方法中，先判断该job是否能以 Uber 方式启动，如果是则创建 LocalContainerAllocator，负责创建 RMContainerAllocator。然后初始化并启动 ContainerAllocator 。之后调用父类 CompositeService#serverStart  方法启动初始化阶段创建的子服务：

```java
protected void serviceStart() throws Exception {
  List<Service> services = getServices();
  if (LOG.isDebugEnabled()) {
    LOG.debug(getName() + ": starting services, size=" + services.size());
  }
  for (Service service : services) {
    // start the service. If this fails that service
    // will be stopped and an exception raised
    service.start();
  }
  super.serviceStart();
}
```





## RMCommunicator

```java
RMCommunicator#startAllocatorThread() //
    allocatorThread = new Thread(new AllocatorRunnable());
    allocatorThread.setName("RMCommunicator Allocator");
    allocatorThread.start();
        heartbeat();// yarn.app.mapreduce.am.scheduler.heartbeat.interval-ms=1000
            getResources();// return AllocatedContainers
                 makeRemoteRequest();// return response
                    scheduler.allocate(); // return allocateResponse 在这里通过与ResourceManager通过RPC交互
            ScheduledRequests#assign(allocatedContainers);//

            preemptReducesIfNeeded();// 抢占reduce资源，用来执行Map 
                                     // availableMemForMap must be sufficient to run at least 1 map

```



**reduce 抢占**

如果有 Map 没申请到资源，但有Reduce作业在跑时，map 将会去抢占reduce的资源

```java
RAMPDOWN_DIAGNOSTIC = "Reducer preempted "
      + "to make room for pending map attempts";
```





## TaskAttemptListener

该服务主要侦听 task 状态的更改。其包含一个子服务 TaskHeartbeatHandler, 主要追踪已启动的任务，确定任务是否仍在存活并且正在运行，如果长时间没有收到任务，则将其标记为已终止。


































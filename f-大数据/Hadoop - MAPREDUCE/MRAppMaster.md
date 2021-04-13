# MRAppMaster 源码

MRAppMaster 主要组成

* RMCommunicator
* 





## RMCommunicator

```java
RMCommunicator#startAllocatorThread() //
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


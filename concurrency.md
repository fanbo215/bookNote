如果对所有的资源的访问都是瞬间或可以接受的时间完成，那么，程序只需要按照顺序执行下来就行了，由于访问flash或网络请求等会需要较长的时间才能返回，如果将这些请求放在主线程上执行，会卡住UI界面，造成app无法及时响应用户的行为，用户体验不好，为了让程序能及时响应用户的行为，需要将这些耗时的操作放在别的线程，当它们执行完成后再通知主线程对结果进行后续的处理

* 并发和并行
	* 由于现在我们的设备可能不仅仅只有一个处理器，我们也可以将任务放在多个处理器上同时执行，这就是所谓的并行处理，并行是需要多个处理器，即硬件的支持
	* 由于现代的操作系统是多任务系统，可以同时允许多个线程同时执行，即所谓的并发执行，并发是需要操作系统支持，即系统软件的支持
	* 对于并发的线程，如果运行在多处理器上，可以实现并行执行

主线程（UI线程）

线程是一个非常底层的工具，他将线程的创建，管理，同步等交给了开发人员，提高了并发开发的难度，同时无法准确评估到底创建多少线程才合适

新的方式让开发人员只关注需要并发的任务本身，将剩下的事情交给系统来处理

异步和同步
同步是指一个函数或block（通常指任务）在其自身还没有执行完时是不会返回的，异步指它们还没有执行完就可以返回，没有执行完的任务还可以继续执行

异步与并发，异步机制利用多线程实现，并发也利用多线程，但是异步也可能是在同一个线程顺序执行
异步是对多线程的封装，并发是利用多线程

GCD
dispatch queue是基于c的用于执行自定义任务的机制，不管是并发队列还是串行队列，它们都是先进先出
有点：
1. 接口简单
2. 提供自动而全面的线程管理
3. 进行了细致的优化
4. 内存利用率高
5. 在加载时不会陷入内核
6. 异步的dispatch queue不会死锁队列
7. 串行队列比锁和其它同步机制效率高
串行队列按照先进先出的原则一次只执行一个任务，这些任务可以运行在不同的线程，主要用于同步访问特定资源
可以创建多个串行队列，每个队列可以并发执行，如4个串行队列，每个队列一个任务，那么这四个任务可以并发执行
并发队列可以执行一个或多个任务，任务仍然是按照先进先出的原则开始
main dispatch queue是一个全局的串行队列，用来在主线程上执行任务
系统可以根据当前系统环境动态决定分配多少线程，并且比手动创建线程执行任务要快
对于在block中要创建大量的oc对象，可以考虑在block开始处加入@autorelease blcok
可以为dispatch添加context数据

```
void myFinalizerFunction(void *context)
  {
	MyDataContext* theData = (MyDataContext*)context;
      // Clean up the contents of the structure
      myCleanUpDataContextFunction(theData);
      // Now release the structure itself.
      free(theData);
  }
  dispatch_queue_t createMyQueue()
  {
      MyDataContext*  data = (MyDataContext*) malloc(sizeof(MyDataContext));
      myInitializeDataContextFunction(data);
      // Create the queue and set the context data.
      dispatch_queue_t serialQueue =
  dispatch_queue_create("com.example.CriticalTaskQueue", NULL);
      if (serialQueue)
      {
          dispatch_set_context(serialQueue, data);
          dispatch_set_finalizer_f(serialQueue, &myFinalizerFunction);
      }
      return serialQueue;
  }. 

dispatch_sync和dispatch_sync_f函数会阻塞当前线程的执行，直到这个任务被执行完成
注意，如果在一个串行队列中调这连个函数肯定会导致死锁，对于并发队列也要避免这样调用

dispatch_queue_t myCustomQueue;
myCustomQueue = dispatch_queue_create("com.example.MyCustomQueue", NULL);
dispatch_async(myCustomQueue, ^{
    printf("Do some work here.\n");
});
printf("The first block may or may not have run.\n");
dispatch_sync(myCustomQueue, ^{
    printf("Do some more work here.\n");
});
printf("Both blocks have completed.\n"); //因为前面一个调用是同步调用，所以会先执行它里面的内容，又由于队列是先进先出，所以，队列之前的任务也会执行
void average_async(int *data, size_t len,
   dispatch_queue_t queue, void (^block)(int))
{
   // Retain the queue provided by the user to make
   // sure it does not disappear before the completion
   // block can be called.
   dispatch_retain(queue);
   // Do the work on the default concurrent queue and then
   // call the user-provided block with the results.
   dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0),
^{
      int avg = average(data, len);
      dispatch_async(queue, ^{ block(avg);});
      // Release the user-provided queue when done
      dispatch_release(queue);
   });
}

dispatch_apply可以允许将一个block提交到dispatch queue多次，如果是串行队列就是顺序执行，如果是并行队列就是并发执行，可以用于for循环，它极有可能会组塞当前线程，因此要特别注意不要放在主线程上，另外，也不要在串行队列的环境中用同一个队列来调用这个函数，极有可能造成死锁
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_apply(count, queue, ^(size_t i) {
   printf("%u\n",i);
});

如果你创建的不是cocoa应用程序并且没有设置runloop，必须调用dispatch_main函数来显式drain主dispatch queue，否则里面的任务是不能被执行的

挂起和回复dispatch queue，dispatch_suspend和dispatch_resume

使用dispatch semaphore来控制使用有限资源
对于有资源可用的情况，需要更少的时间来获取信号量
1. 当创建信号时，dispatch_semaphore_create，可以用来描述多少资源可用
2. 在每个任务中调用dispatch_semaphore_wait来等待信号
3. 当等待返回时，表示获取到资源，可以执行任务
4. 当使用完资源，释放它，并调用dispatch_semaphore_signal增加信号量值
当调用dispatch_semaphore_wait时，信号量会－1，如果值为负，内核会阻塞该线程，dispatch_semaphore_signal会将信号量＋1，如果有任务阻塞并等待资源，其中一个被阻塞的任务可以继续执行了

dispatch group是一种阻塞线程的方式，只有当它里面的任务结束执行后才会取消阻塞。对于这些任务不在同一个队列也可以追踪
-(void)fetchConfigurationWithCompletion:(void (^)(NSError* error))completion
{
    // Define errors to be processed when everything is complete.
    // One error per service; in this example we'll have two 
    __block NSError *configError = nil;
    __block NSError *preferenceError = nil;

    // Create the dispatch group
    dispatch_group_t serviceGroup = dispatch_group_create();

    // Start the first service
    dispatch_group_enter(serviceGroup);
    [self.configService startWithCompletion:^(ConfigResponse *results, NSError* error){
        // Do something with the results
        configError = error;
        dispatch_group_leave(serviceGroup);
    }];

    // Start the second service
    dispatch_group_enter(serviceGroup);
    [self.preferenceService startWithCompletion:^(PreferenceResponse *results, NSError* error){
        // Do something with the results
        preferenceError = error;
        dispatch_group_leave(serviceGroup);
    }];

    dispatch_group_notify(serviceGroup,dispatch_get_main_queue(),^{
        // Assess any errors
        NSError *overallError = nil;
        if (configError || preferenceError)
        {
            // Either make a new error or assign one of them to the overall error
            overallError = configError ?: preferenceError;
        }
        // Now call the final completion block
        completion(overallError);
    });
}



dispatch source也是基于c的异步处理系统事件的机制，它封装了系统的事件，当事件发生时，会提交一个用户的block给dispatch queue，用它可以取代异步回调函数，可以用于监控如下系统事件：
1. 定时器
2. 信号句柄
3. 描述符相关的事件
4. 进程相关的事件
5. mach port事件
6. 用户定义的事件
创建dispatch source的步骤
1. 创建dispatch source，dispatch_source_create;
2. 配置dispatch source:
	* 给dispatch source设置事件处理函数；
	* 对于时间源，用dispatch_source_set_timer
3. 可选设置取消dispatch source函数;
4. 调用dispatch_resume函数开始处理事件
当事件处理已经被排队并等待处理事件时对于新到来的事件，dispatch source会将这两个事件合并，具体的情况要视事件类型而定
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ,
                                 myDescriptor, 0, myQueue);
dispatch_source_set_event_handler(source, ^{
   // Get some data from the source variable, which is captured
   // from the parent context.
   size_t estimated = dispatch_source_get_data(source);
   // Continue reading the descriptor...
});
dispatch_resume(source);
当电脑睡眠时，所有的定时器dispatch source会被挂起。当电脑唤醒时，那些定时器会自动唤醒。

Operation Queue
是面向对象的dispatch queue，对于任务的执行顺序可以自定义，在定义任务时设置好依赖关系就能创造复杂的执行顺序
Operation对象可以生成kvo通知，用来监控任务的进度

Operation由NSOperation描述，它是一个抽象类，需要被继承后才能使用，该类提供了要实现一个操作所需的大量工作
NSInvocationOperation可以利用已有的类和接口来创建一个operation对象
NSBlockOperation可以用来执行一个或多个block，只有当多个block都执行完，这个操作才算执行完
好处：
支持操作对象间基于图的依赖，该依赖阻止一个操作执行，直到与它依赖的操作执行完
支持在主要任务完成后提供一个可选的完成block
支持通过kvo通知来监控操作执行状态的改变
支持通过优先级来影响相关执行的顺序
支持取消执行语意，允许你停止执行一个正在执行的操作
NSInvocationOperation使用

```
- (NSOperation *)taskWithData:(id)data {
	NSInvocationOperation *theOp = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(myTaskMethod:) object:data];
	return theOp;
}
- (void)myTaskMethod:(id)data {
	// perform the task;
}
```

NSBlockOperation使用

```
NSBlockOperation *theOp = [NSBlockOperation blockOperationWithBlock: ^{
	NSLog("Beginning operation. \n");
	// do some work
}];
```

如果是要添加更多block，采用addExecutionBlcok:方法
自定义操作需要实现方法
1. 自定义初始化操作，用于设置操作的状态
2. main，要执行的任务

```
@interface MyNonConcurrentOperation : NSOperation
@property id (strong) myData;
-(id)initWithData:(id)data;
@end
@implementation MyNonConcurrentOperation
- (id)initWithData:(id)data {
   if (self = [super init])
      myData = data;
   return self;
}
-(void)main {
   @try {
      // Do some work on myData and report the results.
   }
   @catch(...) {
      // Do not rethrow exceptions.
} }
@end
```

响应取消，需要周期性的调用isCancelled方法，如果为YES，则立即结束，这些地方包括
1. 在执行具体的任务前
2. 在每个循环前，或者每一个遍历前
3. 在比较适合中断操作时

```
- (void)main {
   @try {
      BOOL isDone = NO;
      while (![self isCancelled] && !isDone) {
          // Do some work and set isDone to YES when finished
} }
   @catch(...) {
      // Do not rethrow exceptions.
} }
```

对于并发的操作，需要覆盖的实现：
|:------方法---------|:-----------描述-----------------|
|start---------------| 必须覆写，手动执行需要调start方法, 不用调super start｜
|main----------------| 可选，用来实现任务|
|isExecuting, isFinished| 必须覆写，由用户来报告状态|
|isConcurrent--------| 必须覆写，标示操作是否并发|

```
@interface MyOperation : NSOperation {
      BOOL        executing;
      BOOL        finished;
￼￼￼￼}
- (void)completeOperation;
@end
@implementation MyOperation
- (id)init {
    self = [super init];
    if (self) {
        executing = NO;
        finished = NO;
    }
    return self;
}
- (BOOL)isConcurrent {
    return YES;
}
- (BOOL)isExecuting {
    return executing;
}
- (BOOL)isFinished {
    return finished;
}
@end

- (void)start {
   // Always check for cancellation before launching the task.
   if ([self isCancelled])
   {
      // Must move the operation to the finished state if it is canceled.
      [self willChangeValueForKey:@"isFinished"];
      finished = YES;
      [self didChangeValueForKey:@"isFinished"];
return; }
   // If the operation is not canceled, begin executing the task.
[self willChangeValueForKey:@"isExecuting"];
[NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
   executing = YES;
   [self didChangeValueForKey:@"isExecuting"];
}

- (void)main {
   @try {
       // Do the main work of the operation here.
       [self completeOperation];
   }
   @catch(...) {
      // Do not rethrow exceptions.
} }
- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
    executing = NO;
    finished = YES;
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}
```

对于操作间的依赖，可以通过调用addDependency:来建立。表示调用的操作依赖参数操作，即调用的操作不会执行，直到参数操作结束执行，这种依赖不局限在同一个queue中，但是要注意避免循环依赖，当所有的操作依赖都执行完成，该操作将变为ready状态，注意：依赖必须在运行操作或添加它们到operation queue前添加

对于一个添加到operation queue的操作，执行的顺序取决与操作的ready状态和自己的优先级，默认自己的优先级是normal，可以通过setQueuePriority:来设置，优先级🈯️限在同一个queue中，
可以设置操作所在线程的优先级，setThreadPriority:

通过setCompletionBlock:来设置完成block

避免在线程中存储或传递数据
操作需要自己引用
添加错误和异常处理

对于操作，建议处理比较耗时的操作，而不是极其短暂的操作

添加一个操作到operation queue

```
NSOperationQueue *aQueue = [[NSOperationQueue alloc] init]; 
[aQueue addOperation:anOp]; // Add a single operation
[aQueue addOperations:anArrayOfOps waitUntilFinished:NO]; // Add multiple operations [aQueue addOperationWithBlock:^{
   /* Do something. */
}];
```

永远不要将操作添加到队列后再修改操作

```
  - (BOOL)performOperation:(NSOperation*)anOp
  {
     BOOL        ranIt = NO;
     if ([anOp isReady] && ![anOp isCancelled])
     {
        if (![anOp isConcurrent])
           [anOp start];
        else
           [NSThread detachNewThreadSelector:@selector(start)
                     toTarget:anOp withObject:nil];
ranIt = YES; }
     else if ([anOp isCancelled])
     {
        // If it was canceled before it was started,
	//  move the operation to the finished state.
        [self willChangeValueForKey:@"isFinished"];
        [self willChangeValueForKey:@"isExecuting"];
        executing = NO;
        finished = YES;
        [self didChangeValueForKey:@"isExecuting"];
        [self didChangeValueForKey:@"isFinished"];
        // Set ranIt to YES to prevent the operation from
        // being passed to this method again in the future.
        ranIt = YES;
}
     return ranIt;
 }

NSOperation可以单独执行，也可以放在operation queue中执行，并且可以设置同步或异步，默认是同步，注意同步异步是针对调用该对象start对应的线程而言的，并且表明该调用是否要立即返回，与并发无关，如果要实现并发的操作，需要再创建一个线程，调用异步的系统函数或确保start函数能立即返回的接口，当然通常开发者不需要是实现并发的操作。

并发设计
定义所有的行为
根据前面的行为判断是没有先后顺序的行为采用并发，先后顺序会影响结果的采用序列，此时可以将行为细分为更小的执行单元
区分出不同任务放在不同的队列
特别注意的地方：
如果内存使用太多，可以考虑在任务中直接计算值
尽早识别出串行任务，让它能尽可能的并行
避免使用锁
多依赖系统并发库

对于涉及到大量数字并行计算的可以考虑采用OpenCL（open computing language）


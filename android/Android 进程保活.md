# Android 进程保活

## 背景

### 进程等级

**前台进程**

- 与用户正在交互的Activity
- 前台Activity以bind方式启动的Service
- Service调用了startForground，绑定了Notification
- 正在执行生命周期的Service，例如在执行onCreate、onStart、onDestory
- 正在执行onReceive方法的BroadcastReceiver

**可见进程**

- 托管不在前台、但仍对用户可见的 Activity（已调用其 onPause() 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。
- 托管绑定到可见（或前台）Activity 的 Service。

**服务进程**
 正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。

**后台进程**
 包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 onStop() 方法）。这些进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。

**空进程**
 不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。

## 总结

**提升进程优先级**的方式

- Activity提权，监听屏幕的息屏和解锁，使用一个1个像素的Activity
- Service提权，Service通过startForground方法来开启一个Notification

**进程拉活**

- 通过广播的方式
- 通过Service在onStartCommand的返回值，START_STICKY，由系统拉活，在短时间内如果多次被杀死可能就再也启动不了了
- 通过账户同步拉活
- 通过JobScheduler拉活
- 通过Service的bind启动的方式，双进程守护拉活
- 推送拉活
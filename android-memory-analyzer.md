### GC原理：

引用标记，



### 工具：

MAT，Android Studio Profiler

#### MAT：

dump hprof file之后需要 hprof-conv转换文件格式



### adb看内存，

- **VSS**- Virtual Set Size **虚拟**耗用内存（包含共享库占用的内存）
- **RSS**- Resident Set Size 实际使用物理内存（包含共享库占用的内存）
- **PSS**- Proportional Set Size 实际使用的物理内存（**比例分配共享库**占用的内存）
- **USS**- Unique Set Size 进程**独自占用**的物理内存（不包含共享库占用的内存）

一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS

| Item | 全称                    | 含义   | 等价                  |
| ---- | --------------------- | ---- | ------------------- |
| USS  | Unique Set Size       | 物理内存 | 进程独占的内存             |
| PSS  | Proportional Set Size | 物理内存 | PSS= USS+ 按比例包含共享库  |
| RSS  | Resident Set Size     | 物理内存 | RSS= USS+ 包含共享库     |
| VSS  | Virtual Set Size      | 虚拟内存 | VSS= RSS+ 未分配实际物理内存 |

top/ps：查看PSS、RSS

top：Usage: top [ -m max_procs ][ -n iterations ] [ -d delay ][ -s sort_column ] [ -t ][ -h ]

    -m num  Maximum number of processes to display.
    -n num  Updates to show before exiting.
    -d num  Seconds to wait between updates.
    -s col  Column to sort by (cpu,vss,rss,thr).
    -t      Show threads instead of processes.
    -h      Display this help screen.




### 进程生命周期

https://developer.android.google.cn/guide/components/processes-and-threads.html?hl=zh-cn#Processes

重要性层次结构一共有 5 级

- 前台进程
  - 正在交互的 Activity
- 可见进程
  - 托管不在前台、但仍对用户可见的 Activity
- 服务进程
  - 在后台播放音乐或从网络下载数据
- 后台进程
  - 对用户不可见的 Activity 
- 空进程
  - 不含任何活动应用组件的进程



### service绑定

- 扩展 Binder 类：本地应用使用，不需要跨进程工作
  - https://developer.android.google.cn/guide/components/bound-services.html?hl=zh-cn#Binder
- 使用 Messenger：可以远程，不需要AIDL，使用Messenger发送Message通信。
  - https://developer.android.google.cn/guide/components/bound-services.html?hl=zh-cn#Messenger
  - 双向通信，需要在客户端建立一个Messenger和Handler处理服务的消息




Unity UGUI曲面化，改变顶点。类似于canvas.drawBitmapMesh()。

对于图片，需要增加顶点

https://zhuanlan.zhihu.com/p/26799645

### 合理的职责划分、可测试性、易用性、模块化，易于封装与分发
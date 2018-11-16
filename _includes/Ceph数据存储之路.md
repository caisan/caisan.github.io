# Ceph数据存储之路

Ceph就是用来存储数据的。

从ceph存储数据的流程出发，看看ceph是如何实现数据存储的。

1.radosclient

radosclient中包含了几个模块：消息管理模块Messager，数据处理模块Objecter，Finisher线程模块。

2.IOCtx

为相关Pool创建IOCtx，ioctx中会指明radosclient和Objecter模块。同时记录Pool的信息，包括pool的参数。

以上两个步骤是每个一个Rados客户端在和Ceph集群通信的时候都会涉及到的步骤，这些客户端包括rbd，rgw，cephfs都会这样的做。




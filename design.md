# Design

## 持久化

- Dont fear the filesystem
  
  -   six  7200rpm sata RAID-5 
    
    - random writes only 100kb
    
    - liner write 600mb
  
  - -

## 性能优化

### 2.1 小额I/O优化

- 小的io服务器客户端都很多，

- sendfile0 拷贝技术
  
  - 原来的顺序
    
    - 操作系统从磁盘读 记录到 kernel space pagecache
    
    - copy data from pagecache to user-space buffer
    
    - write  data to kernel space socket buffer
    
    - copy dataw from socket to NICbuffer(查下这个是啥) 
  
  - 用了sendfile 直接从disk 到NIC buffer

### 2.2 batch压缩 end-to-end batch compression

- 有些场景cpu 用的少，网络是瓶颈

- 因为message 都相同type ， 字段为json，meta字段是相同的

- 一个batch可以一起被压缩，并在一个表格中被发送

- kakfa支持GZIP snappy  LZ4 和 Zstandards压缩协议

## 生产者

### 3.1 负载均衡 loading balancing

- producer sends message directly to broker without any intervening routing tier（中间路由层），kafka节点需要知道哪些server是alive哪些topic partition 的leaer，并适当发送

- clients 控制往哪个partition  。 能够随机指定，但是也可以语义函数指定。

### 3.2 异步发送 asynchronous send

- 生产者会尝试在内存中计算数据并在单个请求中带一个大的batch

- batch有一个固定的大小阈值，然后等待大小也有个阈值 10ms；这样减少大的请求，优化latency

## 消费者

- kafka消费者使用“fetch”请求表示消费者想消费。

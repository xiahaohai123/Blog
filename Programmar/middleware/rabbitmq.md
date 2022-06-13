
# 在Java中，为什么client有了connection又设计了channel?
简而言之: 分时复用。
connection作为连接对象，以一种资源的立场存在于程序中。
而channel可以基于connection对象上创建，以控制器的立场存在于程序中。
重点是，一个connection对象可以创建多个channel，在多线程场景下，每个channel可以独立做自己的事，不会互相干预，却都只需要使用同一个connection资源。
这种设计很像WIFI6。
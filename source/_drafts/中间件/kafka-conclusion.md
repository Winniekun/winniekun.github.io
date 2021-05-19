# Kafka的一系列总结

通过 [我用kafka两年踩过的一些非比寻常的坑](https://juejin.cn/post/6933190736888201223#comment) 文章类比联联换电业务，总结kafka出现的一些问题以及解决方式。



## 如何保证消息的顺序

> kafka的topic是无序的，但是一个topic可以有多个partition，每个partiion内部是有序的。
>
> 所以保证生产者写消息的时候，按照一定的规则写到同一个partition中，不同的消费者读不同的`partition`的消息，就能保证生产和消费者消息的顺序。
>
> 

## 防止消息的积压，也就是提升整体consumer的消费能力

### **线程封闭**

![image-20210402215138656](https://i.loli.net/2021/04/02/WuGbPIfQpwCcl6t.png)

### **方式二**

![image-20210402215323293](https://i.loli.net/2021/04/02/dSYjerLwPsDFani.png)

![image-20210402215631360](https://i.loli.net/2021/04/02/NVraMbfzXEHU83q.png)






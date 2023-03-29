## 队列(queue)

线性表结构

操作受限，只允许在一端插入，在另一端删除

先进先出(First In First Out)，简称FIFO结构

顺序队列(数组)，链式队列(链表)

顺序队列通过移动下标实现入队出队操作，队列满时，向前移动元素

### 循环队列

顺序存储结构，出队列操作伴随低效的数据元素移动。

换个思路，出队列时不移动元素而是移动队头

假溢出，入队端空间满了，但是出队段空间空闲

循环队列可以解决假溢出的问题，入队的一端空间用完了，就从头开始

队头指针和队尾指针可以一直增加，通过取模运算映射到队列结构上

利用阻塞队列，可以实现生产者-消费者模型

利用加锁或者原子操作，可以实现并发队列

利用循环队列做队列是需要预估业务量以确认队列长度

---

| 代码说明  | 代码位置                                      |
| --------- | --------------------------------------------- |
| 队列-链表 | CYangLi/shu4ju4jie2gou4/dui4lie4_lian4biao3.c |
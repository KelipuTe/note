2.Love is a dangerous disadvantage.

爱是种危险的劣势。

3.Jack Sparrow：You've seen it all, done it all. You survived. This is the trick, isn't it? To survive?

Cap'n Teague：It's not just about living forever, Jackie, the trick is living with yourself forever.

杰克船长：你什么都见过了，什么都做过了，你活下来了，一定有要诀吧？为了生存？

蒂格船长：目的不在永远活着，杰克，要诀是永远活出自己

4.Our destinies have been entwined, Elizabeth, but never joint.

我们的命运纠缠在一起，但无缘结连理。

5.Who am I for you?-Treasure.

-对于你来说，我是什么？-宝贝（最喜欢这句呜呜呜）
动图封面

6.If you choose to lock your heart away, you will lose her for certain.

If I might lend a machete to you intellectual thicket

change the facts.

Avoid the choice altogether.

如果你满腔投入，一定会全盘皆输

让我给你指点指点

换个角度

置身事外

7.They know they face extinction.

All that remains is where they make their final stand.

他们知道自己即将被消灭，所以他们要放手一搏。

8.Up is down.

黑白颠倒。

9.Genuine tragedies in the world are not conflicts between right and wrong. They are conflicts between two rights.

世界上真正的悲剧不是正确和错误间的冲突而是两种正确间的冲突。

网路传递具有不确定性。

良好的超时策略，可以尽可能让服务不堆积请求，尽快清空高延迟的请求。

项目要求实现简易MYSQL：
1 支持CS架构
2 支持数据存储，支持快速排序，支持查看，支持基础命令CRUD
3 支持多线程，多进程
4 支持启动方式为前台，和守护进程模式运行
5 支持事务功能
6 支持友好停止服务
7 支持主从复制

Safari 标签，TODO Reading
Bookmark，收藏
OneNote 分类存储链接

github.com/Terry-Mao/bfs 
Facebook haystack 


DDD，DTO=>DO=>PO，分别是在哪一层来处理模型转换的。 
DTO gRPC ，service -> biz
DO biz, DTO -> DO，Do business policy/rules，DO 持久化  -> repo
PO data, DO -> PO，orm /ent  



全局变量局部变量

存放变量的内存区域有大小和权限

全局变量 和 函数 在 readelf 命令的结果里面可以看得到，

readelf 命令的结果里面都是虚拟地址，要映射的

ldd 命令的结果中的 linux-vdso 是操作系统虚拟出来的 elf

局部变量一般是在栈上面的




虚拟地址 指针 虚拟地址编号 地址编号 进程虚拟地址 都是一个意思

虚拟地址有两种，编译时就确定的（符号 symbol 如 全局变量，函数等），运行时才确定的

虚拟地址的大小 和 cpu 架构有关 如果 cpu 是 64 位的 虚拟地址编号的大小就是 64 bit

注意有的命令，在打印 虚拟内存的时候会在前面补 0，显示出来的可能不够 64 bit

虚拟地址的递增方向是从低到高的




在 C 语言 下

局部变量 在函数执行后 不一定会释放 可能会真的释放 也有可能不释放

即使被释放了，说不定也能用


全局变量实际上被写到文件里去了，被写死了

局部变量可能在堆上在栈上，是可变的


gc、回收、释放空间，一般表示把内存上的数据擦除，不一定代表那个地址就不能用了


go 在编译的时候 可以确定变量应该放在 栈上还是堆上

go build -gcflags "-m -l"

小端字节序 ，想像一下 水流



核心本质技术知识培训机构不会告诉你的


　很多程序员放弃对于核心基础的学习，也不愿意去扩展自己的认知
甚至听从某大佬的意见去搞上层花哩胡哨的语言混饭。

从此你就失去了构建起软硬兼职的完整知识体系，失去了能够感知
计算机程序的运行机理和核心本质，高级语言是无法给予你计算机系统的
核心本质的，你也无法轻松的驾驭上层应用，你也不会更深入的用好上层高级语言
比如CPP，你只知道类成员怎么用，核心本质是什么你不懂
比如脚本语言更上层，你只知道怎么CRUD
比如你用的nginx redis mq ....你只知道会用，换了个东西你又重新去
卖命的学，核心技术你还是没有掌握，更谈不上你能更好的更深入的去理解和使用上层应用。



大概是18年的时候，我收集了几万本电子书，epub、mobi、azw3、pdf、txt各种格式，专业知识、通俗小说、心灵鸡汤、历史经典，大杂烩啥都有。用Calibre软件管理，编标签，搞分类，搞各种冷备份，折腾局域网书籍网站共享给手机，甚至动过买NAS的念头。为了看PDF，为此我还专门买了当时很热门的墨水屏阅读器，索尼13英寸的DPT-RP1，专门看A4篇幅的的PDF，并且将各种格式的电子书都转换为PDF格式然后用墨水屏阅读。过了一年多吧，也没看多少书，墨水屏也坏了（墨水屏也被我拆了，还拍了个视频发出来了）。平时打开Calibre书库就是来回的翻着目录，没有看的欲望。后来我就把库给删了，当时删库都删了好几个小时（二三十G的库，小文件太多了，机械盘处理小文件的通病，没有格式化分区是因为分区里还有其它大量小文件）。
现在看来，删库是完全正确的选择（作为个人的话是这样的，如果你是卖书的那就另说）。在我看来，任何没有阅读过的书籍都是负担而不是知识，过多的负担只会增加自己的管理成本，而不会让自己成长。

很认同，就像游戏机被破解了，里面都是游戏，结果一个也不想玩了。当下的好奇心和渴求感是无比珍贵的。

盖伦出轻语 沉默又破防


形式主义派生于官僚主义，官僚主义派生于私有制，为的就是权力寻租，然后获取超额暴利


重试间隔是为了绕开失败的原因
重试次数是为了绕开偶发性问题，如果已经得到了明确的失败，就不需要重试了
重试也要考虑会不会把下游打垮

限流，通过压力测试获得，通过统计近几天的峰值

降级，削峰，转异步，如果不能转异步，就返回一个特定的失败，让上游想办法

转异步可以存消息中间件比如kafka，如果存数据库而且数据库撑不住并发，则可以结合缓存从单体插入转批量插入

原设计 -> 加监控 -> 新设计1 -> 新问题1 -> 新监控2 -> 新设计2 如此循环

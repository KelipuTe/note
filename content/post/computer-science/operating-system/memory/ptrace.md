在用ptrace跟踪另外一个进程，寻找目标数据时，可能会在目标进程内存中多个位置找到目标数据，可能会有3个地方

变量本体，就是变量所在的位置；
堆栈框架，比如当这个变量定义在main函数里面的时候，main函数的堆栈框架里也可能会出现；
寄存器，在执行过程中，数据可能会暂时存储在cpu的寄存器中

当使用ptrace读取和修改目标进程的内存时，需要注意，读取和写入数据的单位是一个long
读取的时候想要读的是一个int，但是返回的时候会带着这个int后面的四个字节
修改的时候也需要注意，先把目标位置上的8个字节读出来，然后修改需要修改的部分，然后把八个字节完整的放回去，要不然会出错
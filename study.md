记录分析redis源代码的过程,选用目前最新的6.2版本

#编译redis
先编译redis, 整个编译过程还是很简单的, 直接根目录make就可以了

打断点的时候发现部分调试符号被optimized out了, 看readme
先make distclean
根据src/Makefile, 可以看到noopt target对应gcc -O0, 直接make noopt
发现jemalloc编译出错, 太多了, 搞不过来
直接make MALLOC=libc noopt, 先不使用jemalloc, 成功后会产生所有调试信息

产出的二进制直接就在src/下

make完后可以make test一下, 但需要安装tcl, 没用过, 先跳过

#clion中使用code inspection
为了在clion里面使用code inspection, 先照着src/Makefile和src/Makefile.dep写一个简单的Cmakelists.txt(这货跟其他的jetbrains ide比起来真是拉胯)
一点小技巧:
先将Makefile.dep里ctrl+f查找"\"换行符, ctrl+shift+j选择全部, 将所有换行去除
查看src/Makefile里, REDIS_SERVER_OBJ的定义, 知道了编译redis-server需要的obj
在bash里, 令 objs=(adlist.o quicklist.o ...)
for((i=0;i<${#objs[@]};i++));do cat Makefile.dep |grep ${objs[i]};done>deps.txt
在Makefile.dep里面grep出各个.o的依赖
在dep.txt里ctrl+f,勾选正则表达式,输入.+\.o: ,ctrl+shift+j select all occurrence, delete, 
就可以去掉开头的*.o: , 得到依赖的文件列表了
这时候为了好看还要每个文件一行
ctrl+r, 正则替换 ,将" *"替换为"\n"
然后"^(.*)$"替换为"src/$1"就可以完成操作
加入CMakelists.txt即可

#看源码
看一下源码文件结构, src里是实现, deps是依赖, test是测试
src里面文件组织是真的乱, 都不分个文件夹, 所有模块的都混在一起了
make之后的产出物也直接是在原地, 应该放到debug/bin下

先看redis教程, 学习redis基本用法,https://www.redis.com.cn/tutorial.html

服务端的主程序在src/servcer.c中

#redis核心数据结构
#dict
其实就是个hash实现的map，hash函数用的murmurhash2（据说）。
dict用来存储的其实是一个大小2^n的数组,hash出来的值就是数组下标，hash值有冲突的，作为链表插入到最前面。
涉及到扩缩容，就有负载因子（dict.used/dict.size，已有节点数/dict可承载大小），server.h:#define HASHTABLE_MAX_LOAD_FACTOR 1.618   /* Maximum hash table load factor. */,最大负载因子。
#skiplist
https://blog.csdn.net/helloworld_ptt/article/details/105801262

#re
自己搞的一个事件库，统一了select、epoll、kqueue的抽象。在主线程内进行IO多路复用，对应的处理函数处理发生的事件。
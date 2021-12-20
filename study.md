记录分析redis源代码的过程,选用目前最新的6.2版本

先编译redis, 整个编译过程还是很简单的, 直接根目录make就可以了

打断点的时候发现部分调试符号被optimized out了, 看readme
先make distclean
根据src/Makefile, 可以看到noopt target对应gcc -O0, 直接make noopt
发现jemalloc编译出错, 太多了, 搞不过来
直接make MALLOC=libc noopt, 先不使用jemalloc, 成功后会产生所有调试信息

make完后可以make test一下, 但需要安装tcl, 没用过, 先跳过

为了在clion里面使用code inspection, 先照着src/Makefile和src/Makefile.dep写一个简单的Cmakelists.txt(这货跟其他的jetbrains ide比起来真是拉胯)

看一下源码文件结构, src里是实现, deps是依赖, test是测试
src里面文件组织是真的乱, 都不分个文件夹, 所有模块的都混在一起了

先看redis教程, 学习redis基本用法,https://www.redis.com.cn/tutorial.html
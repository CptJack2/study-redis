记录分析redis源代码的过程,选用目前最新的6.2版本

先编译redis, 整个编译过程还是很简单的, 直接根目录make就可以了

打断点的时候发现部分调试符号被optimized out了, 看readme
先make distclean
根据src/Makefile, 可以看到noopt target对应gcc -O0, 直接make noopt
发现jemalloc编译出错, 太多了, 搞不过来
直接make MALLOC=libc noopt, 先不使用jemalloc, 成功后会产生所有调试信息

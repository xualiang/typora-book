```sh
jstat -gc 15404 100 10        // 15404为进程号， 100毫秒输出一次， 输出10次
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
53248.0 78336.0  0.0    0.0   510976.0 413677.5  199168.0   54216.5   36480.0 35465.0 4864.0 4645.5     12    0.288   4      0.277    0.565
C即Capacity 总容量，U即Used 已使用的容量
S0C : survivor0区的总容量
S1C : survivor1区的总容量
S0U : survivor0区已使用的容量
S1C : survivor1区已使用的容量
EC : Eden区的总容量
EU : Eden区已使用的容量
OC : Old区的总容量
OU : Old区已使用的容量
PC : 当前perm的容量 (KB)
PU : perm的使用 (KB)
YGC : 新生代垃圾回收次数
YGCT : 新生代垃圾回收时间
FGC : 老年代垃圾回收次数
FGCT : 老年代垃圾回收时间
GCT : 垃圾回收总消耗时间


```


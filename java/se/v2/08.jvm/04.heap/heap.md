## 堆体系结构概述

1. diagram

   ![avatar](/static/image/java/heap.png)

2. struct

   - [10]新生区: new/young[区域小, 存活率低]
     > [8]伊甸园区
     > [1]幸存者 0 区 [from 区][1]幸存者 1 区 [to 区]
   - [20]养老区: old/tenure[区域大, 存活率大]
   - [logic]永久区: implement of MethodArea

3. new Object() new 出来的对象放在 `伊甸园区`; 如果不停地 new, 新生区满了, 会触发 YGC

4. processor

   - new 出来的对象放在 `伊甸园区`
   - `伊甸园区` 满了出发 YGC
   - `伊甸园区` 中多次存活的数据到 `from` 区 $\color{red}{from 和 to 区进行交换}$
   - 15 次 YGC 后还存活的数据放到 `养老区`
   - `养老区` 满了出发 Full GC
   - 多次 FGC 后还是满的, 抛出 OOM Error

5. explain 4.3 交换

   - from 区 和 to 区并不是固定的, 每次 YGC 后会交换, 谁空谁是 to 区
   - YGC 时伊甸园区并须全清空, 幸存数据复制存入 From 区, 下次 YGC 将 from 区和伊甸园区都算作伊甸园区进行收割, 剩下的 to 区则成为存放存货数据的 from 区;

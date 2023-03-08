# CPU Microarchitectrue

## intel
intel芯片手册中列举了从1978年的8088到2013年的4代core产品的介绍, 部分产品的微架构, 后续产品使用的技术介绍
让我们可以一窥intel cpu产品的技术发展历程.

### 4004/8008/8080
1972年intel推出第一款商用cpu, 4004
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/cpu-microarch-img/4004.jpg)

### 8086/8088 
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/cpu-microarch-img/8088.jpg)
8086拥有16位寄存器, 20位数据总线, 20位地址总线, 支持1M内存, 8088与之类似, 不同的地方在于8086的指令是6byte, 8088是4byte, 
8088使用8位数据总线, 因此8088一次只能处理一个字节.

### P6 netburst microarchitecture
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/cpu-microarch-img/netburst.jpg)
netburst架构引入前后端流水线, 乱序发射, 对缓存提出了更高的要求

### intel sandy bridge 
L1 DCache 32k 8way, L2 Cache 256k 8way

The L1D cache is still 32 KiB and 8-way set associative. It is physically tagged, virtuallly index, and uses 64 B lines along with a write-back policy. The load-to-use latency is 4 cycles for integers and a few more cycles for SIMD/FP as they need to cross domains. Westmere added support for 1 GiB "Huge Pages"

支持1G大页内存

### intel skylake platform
coffeelake与skylake架构相同
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/cpu-microarch-img/skylake.webp)

## arm
A53/A73:
L1 2way
L2 2way

A55/A75: 
L1 ICache 16k, 32k, 64k, 4way 
L1 DCache 16k, 32k, 64k, 4way 

## rocket-chip
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/cpu-microarch-img/rocket-chip.png)

##ref
[Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
[nternal Architecture of 8088 Microprocessor](https://www.eeeguide.com/internal-architecture-of-8088-microprocessor/#:~:text=The%20architecture%20of%208088%20is%20same%20as%208086,blocks%20are%20the%20same%20as%20the%208086%20processor.)
[Sandy Bridge](https://en.wikichip.org/wiki/intel/microarchitectures/sandy_bridge_(client)#System_Architecture)
[Coffee Lake - Microarchitectures - Intel](https://en.wikichip.org/wiki/intel/microarchitectures/coffee_lake)
[arm a55/a75](https://www.anandtech.com/show/11441/dynamiq-and-arms-new-cpus-cortex-a75-a55/4)

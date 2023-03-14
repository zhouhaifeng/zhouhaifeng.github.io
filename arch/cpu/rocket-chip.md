# rocket-chip

## rocket

### summary
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/rocket-chip-img/arch.jpg)
![img](https://github.com/zhouhaifeng/zhouhaifeng.github.io/arch/cpu/rocket-chip-img/pipeline.jpg)

Rocket is a 5-stage in-order scalar core generator that implements the RV32G and RV64G
ISAs4 . It has an MMU that supports page-based virtual memory, a non-blocking data cache, and
a front-end with branch prediction. Branch prediction is configurable and provided by a branch
target buffer (BTB), branch history table (BHT), and a return address stack (RAS). For floating-
point, Rocket makes use of Berkeley’s Chisel implementations of floating-point units5 . Rocket
also supports the RISC-V machine, supervisor, and user privilege levels. A number of parameters
are exposed, including the optional support of some ISA extensions (M, A, F, D), the number of
floating-point pipeline stages, and the cache and TLB sizes.
Rocket can also be thought of as a library of processor components. Several modules originally
designed for Rocket are re-used by other designs, including the functional units, caches, TLBs,
the page table walker, and the privileged architecture implementation (i.e., the control and status
register file).
### pipeline
### L1 Cache
### tilelink(bus)
### system bus
### L2 Cache
### memory bus
## ref
[the rocket chip generator](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2016/EECS-2016-17.pdf)
[rocket chip memo](https://zhuanlan.zhihu.com/p/100151292)
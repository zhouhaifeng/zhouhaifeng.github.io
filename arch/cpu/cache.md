#Cache

## intel and amd memeory microarchitecture
registor, instruction, state, protocol, algorithm
mm->TLB->L1->L2->L3->memory

## intel MESI
IA-32 processors (beginning with the Pentium processor) and Intel 64 processors use the MESI (modified, exclu-
sive, shared, invalid) cache protocol to maintain consistency with internal caches and caches in other processors
(see Section 11.4, “Cache Control Protocol”).
When the processor recognizes that an operand being read from memory is cacheable, the processor reads an
entire cache line into the appropriate cache (L1, L2, L3, or all). This operation is called a cache line fill. If the
memory location containing that operand is still cached the next time the processor attempts to access the
operand, the processor can read the operand from the cache instead of going back to memory. This operation is
called a cache hit.
When the processor attempts to write an operand to a cacheable area of memory, it first checks if a cache line for
that memory location exists in the cache. If a valid cache line does exist, the processor (depending on the write
policy currently in force) can write the operand into the cache instead of writing it out to system memory. This
operation is called a write hit. If a write misses the cache (that is, a valid cache line is not present for area of
memory being written to), the processor performs a cache line fill, write allocation. Then it writes the operand into
the cache line and (depending on the write policy currently in force) can also write it out to memory. If the operand
is to be written out to memory, it is written first into the store buffer, and then written from the store buffer to
memory when the system bus is available. (Note that for the Pentium processor, write misses do not result in a
cache line fill; they always result in a write to memory. For this processor, only read misses result in cache line fills.)
Vol. 3A 11-5MEMORY CACHE CONTROL
When operating in an MP system, IA-32 processors (beginning with the Intel486 processor) and Intel 64 processors
have the ability to snoop other processor’s accesses to system memory and to their internal caches. They use this
snooping ability to keep their internal caches consistent both with system memory and with the caches in other
processors on the bus. For example, in the Pentium and P6 family processors, if through snooping one processor
detects that another processor intends to write to a memory location that it currently has cached in shared state,
the snooping processor will invalidate its cache line forcing it to perform a cache line fill the next time it accesses
the same memory location.
Beginning with the P6 family processors, if a processor detects (through snooping) that another processor is trying
to access a memory location that it has modified in its cache, but has not yet written back to system memory, the
snooping processor will signal the other processor (by means of the HITM# signal) that the cache line is held in
modified state and will preform an implicit write-back of the modified data. The implicit write-back is transferred
directly to the initial requesting processor and snooped by the memory controller to assure that system memory
has been updated. Here, the processor with the valid data may pass the data to the other processors without actu-
ally writing it to system memory; however, it is the responsibility of the memory controller to snoop this operation
and update memory.

## AMD MOESI
Implementations that support caching support a cache-coherency protocol for maintaining coherency 
between main memory and the caches. The cache-coherency protocol is also used to maintain 
coherency between all processors in a multiprocessor system. The cache-coherency protocol 
supported by the AMD64 architecture is the MOESI (modified, owned, exclusive, shared, invalid) 
protocol. The states of the MOESI protocol are:
• Invalid—A cache line in the invalid state does not hold a valid copy of the data. Valid copies of the
data can be either in main memory or another processor cache.
• Exclusive—A cache line in the exclusive state holds the most recent, correct copy of the data. The
copy in main memory is also the most recent, correct copy of the data. No other processor holds a
copy of the data. 
• Shared—A cache line in the shared state holds the most recent, correct copy of the data. Other
processors in the system may hold copies of the data in the shared state, as well. If no other
processor holds it in the owned state, then the copy in main memory is also the most recent. 
• Modified—A cache line in the modified state holds the most recent, correct copy of the data. The
copy in main memory is stale (incorrect), and no other processor holds a copy.
• Owned—A cache line in the owned state holds the most recent, correct copy of the data. The
owned state is similar to the shared state in that other processors can hold a copy of the most recent,
correct data. Unlike the shared state, however, the copy in main memory can be stale (incorrect).
Only one processor can hold the data in the owned state—all other processors must hold the data in
the shared state.

## 优化:
fetch
huge page
多核,多线程,无锁

CMPEXHG指令用于实现无锁环形队列

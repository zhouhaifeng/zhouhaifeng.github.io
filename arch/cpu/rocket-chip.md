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
### ICache & DCache
/** [[ICache]] is a set associated cache I$(Instruction Cache) of Rocket.
 * {{{
  * Keywords: Set-associated
  *           3 stage pipeline
  *           Virtually-Indexed Physically-Tagged (VIPT)
  *           Parallel access to tag and data SRAM
  *           Random replacement algorithm
  * Optional Features:
  *           Prefetch
  *           ECC
  *           Instruction Tightly Integrated Memory(ITIM)}}}
  *{{{
  * PipeLine:
  *   Stage 0 : access data and tag SRAM in parallel
  *   Stage 1 : receive paddr from CPU
  *             compare tag and paddr when the entry is valid
  *             if hit : pick up the target instruction
  *             if miss : start refilling in stage 2
  *   Stage 2 : respond to CPU or start a refill}}}
  *{{{
  * Note: Page size = 4KB thus paddr[11:0] = vaddr[11:0]
  *       considering sets = 64, cachelineBytes =64
  *       use vaddr[11:6] to access tag_array
  *       use vaddr[11:2] to access data_array}}}
  *{{{
  * ITIM:
  * │          tag         │    set    │offset│
  *                    ├way┘                    → indicate way location
  *                    │    line       │ }}}
  *   if `way` == b11 (last way), deallocate
  *   if write to ITIM all I$ will be invalidate
  *
  * The optional dynamic configurable ITIM sharing SRAM with I$ is set by  [[icacheParams.itimAddr]].
  * if PutFullData/PutPartialData to the ITIM address, it will dynamically allocate base address to the address of this accessing from SRAM.
  * if access to last way of ITIM, it set will change back to I$.
  *
  * If ITIM is configured:
  *   set: if address to access is not to be configured to ITIM yet,
  *        a memory accessing to ITIM address range will modify `scratchpadMax`,
  *        from ITIM base to `scratchpadMax` will be used as ITIM.
  *   unset: @todo
  *
  * There will always be one way(the last way) used for I$, which cannot be allocated to ITIM.
  *
  * @param icacheParams parameter to this I$.
  * @param staticIdForMetadataUseOnly metadata used for hart id.
  */

  DCache:
  hit, fill, refill, tag, tilelink

### MSHR
NBMSHR

class ProbeUnit(implicit edge: TLEdgeOut, p: Parameters) extends L1HellaCacheModule()(p) {
  val io = IO(new Bundle {
    val req = Flipped(Decoupled(new TLBundleB(edge.bundle)))
    val rep = Decoupled(new TLBundleC(edge.bundle))
    val meta_read = Decoupled(new L1MetaReadReq)
    val meta_write = Decoupled(new L1MetaWriteReq)
    val wb_req = Decoupled(new WritebackReq(edge.bundle))
    val way_en = Input(Bits(nWays.W))
    val mshr_rdy = Input(Bool())
    val block_state = Input(new ClientMetadata())
  })

  val (s_invalid :: s_meta_read :: s_meta_resp :: s_mshr_req ::
       s_mshr_resp :: s_release :: s_writeback_req :: s_writeback_resp :: 
       s_meta_write :: Nil) = Enum(9)
  val state = RegInit(s_invalid)

  val req = Reg(new TLBundleB(edge.bundle))
  val req_idx = req.address(idxMSB, idxLSB)
  val req_tag = req.address >> untagBits

  val way_en = Reg(Bits())
  val tag_matches = way_en.orR
  val old_coh = Reg(new ClientMetadata)
  val miss_coh = ClientMetadata.onReset
  val reply_coh = Mux(tag_matches, old_coh, miss_coh)
  val (is_dirty, report_param, new_coh) = reply_coh.onProbe(req.param)

  io.req.ready := state === s_invalid
  io.rep.valid := state === s_release
  io.rep.bits := edge.ProbeAck(req, report_param)

  assert(!io.rep.valid || !edge.hasData(io.rep.bits),
    "ProbeUnit should not send ProbeAcks with data, WritebackUnit should handle it")

  io.meta_read.valid := state === s_meta_read
  io.meta_read.bits.idx := req_idx
  io.meta_read.bits.tag := req_tag
  io.meta_read.bits.way_en := ~(0.U(nWays.W))

  io.meta_write.valid := state === s_meta_write
  io.meta_write.bits.way_en := way_en
  io.meta_write.bits.idx := req_idx
  io.meta_write.bits.tag := req_tag
  io.meta_write.bits.data.tag := req_tag
  io.meta_write.bits.data.coh := new_coh

  io.wb_req.valid := state === s_writeback_req
  io.wb_req.bits.source := req.source
  io.wb_req.bits.idx := req_idx
  io.wb_req.bits.tag := req_tag
  io.wb_req.bits.param := report_param
  io.wb_req.bits.way_en := way_en
  io.wb_req.bits.voluntary := false.B

  // state === s_invalid
  when (io.req.fire) {
    state := s_meta_read
    req := io.req.bits
  }

  // state === s_meta_read
  when (io.meta_read.fire) {
    state := s_meta_resp
  }

  // we need to wait one cycle for the metadata to be read from the array
  when (state === s_meta_resp) {
    state := s_mshr_req
  }

  when (state === s_mshr_req) {
    old_coh := io.block_state
    way_en := io.way_en
    // if the read didn't go through, we need to retry
    state := Mux(io.mshr_rdy, s_mshr_resp, s_meta_read)
  }

  when (state === s_mshr_resp) {
    state := Mux(tag_matches && is_dirty, s_writeback_req, s_release)
  }

  when (state === s_release && io.rep.ready) {
    state := Mux(tag_matches, s_meta_write, s_invalid)
  }

  // state === s_writeback_req
  when (io.wb_req.fire) {
    state := s_writeback_resp
  }

  // wait for the writeback request to finish before updating the metadata
  when (state === s_writeback_resp && io.wb_req.ready) {
    state := s_meta_write
  }

  when (io.meta_write.fire) {
    state := s_invalid
  }
}

### TLB

/** =Overview=
  * [[TLB]] is a TLB template which contains PMA logic and PMP checker.
  *
  * TLB caches PTE and accelerates the address translation process.
  * When tlb miss happens, ask PTW(L2TLB) for Page Table Walk.
  * Perform PMP and PMA check during the translation and throw exception if there were any.
  *
  *  ==Cache Structure==
  *  - Sectored Entry (PTE)
  *   - set-associative or direct-mapped
  *    - nsets = [[TLBConfig.nSets]]
  *    - nways = [[TLBConfig.nWays]] / [[TLBConfig.nSectors]]
  *    - PTEEntry( sectors = [[TLBConfig.nSectors]] )
  *   - LRU(if set-associative)
  *
  *  - Superpage Entry(superpage PTE)
  *   - fully associative
  *    - nsets = [[TLBConfig.nSuperpageEntries]]
  *    - PTEEntry(sectors = 1)
  *   - PseudoLRU
  *
  *  - Special Entry(PTE across PMP)
  *   - nsets = 1
  *   - PTEEntry(sectors = 1)
  *
  * ==Address structure==
  * {{{
  * |vaddr                                                 |
  * |ppn/vpn                                   | pgIndex   |
  * |                                          |           |
  * |           |nSets             |nSector    |           |}}}
  *
  * ==State Machine==
  * {{{
  * s_ready: ready to accept request from CPU.
  * s_request: when L1TLB(this) miss, send request to PTW(L2TLB), .
  * s_wait: wait for PTW to refill L1TLB.
  * s_wait_invalidate: L1TLB is waiting for respond from PTW, but L1TLB will invalidate respond from PTW.}}}
  *
  * ==PMP==
  * pmp check
  *  - special_entry: always check
  *  - other entry: check on refill
  *
  * ==Note==
  * PMA consume diplomacy parameter generate physical memory address checking logic
  *
  * Boom use Rocket ITLB, and its own DTLB.
  *
  * Accelerators:{{{
  *   sha3: DTLB
  *   gemmini: DTLB
  *   hwacha: DTLB*2+ITLB}}}
  * @param instruction true for ITLB, false for DTLB
  * @param lgMaxSize @todo seems granularity
  * @param cfg [[TLBConfig]]
  * @param edge collect SoC metadata.
  */

### tilelink(bus)
### system bus
### memory bus
## ref
[the rocket chip generator](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2016/EECS-2016-17.pdf)
[rocket chip memo](https://zhuanlan.zhihu.com/p/100151292)
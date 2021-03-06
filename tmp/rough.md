# Rough Notes

__NOTE:__ These are rough notes intentionally not linked from elsewhere, and may
be full of errors, swearing and bullshit.

## TODO

* Go into detail about `handle_mm_fault()` and how various page faults are
  handled, incl. returning the zero page.

* Discuss split page table locks in page-tables.md.

## Ideas

* `pXX_large()` seems to duplicate `pXX_huge()`, though the latter seems to be
  in hugetlb context, and there are some variations, for example:

```
pud_large -> pse/present pud_huge -> pse
pmd_large -> pse         pmd_huge -> pse/not present
```

* Cover `switch_mm()` etc.

* Fixup link to vmalloc in [functions][funcs] page from being a link to
  http://www.makelinux.net/books/lkd2/ch11lev1sec5 to a local description once
  it's written.

* Cover [mem_section][mem_section]/memory sections. Seems like the _physical_
  memory map is divided into these 'sections' which determine whether a given
  block of memory (seem to be in 128MiB blocks on x86-64) is actually backed by
  RAM or not :]. Also, go into detail about the sparse memory model for x86-64.

* [Page Attribute Table (PAT)][pat]? [Memory type range register (MTRR)][mtrr]?
  Some fine-grained caching stuff going on there, cover. Also PCD (Page-level
  Cache Disable) and PWT (Page-level Write-Through) flags. PCID seems related?

* Confirm that, by default, accessed and dirty flags are cleared in memory
  pages.

* Expand page flags list to be more exhaustive.

* Examine the wonders of `include/linux/pfn_t.h` - Seems to be a specific PFN
  type with flag bitfields added in.

* Check out the devmap stuff more thoroughly - descriptions like 'Determines if
  the PTE page pointed at by the specified PMD entry is part of a device
  mapping.' might not be valid. Surely a part of the
  [device mapper][device-mapper] functionality?

* Maybe add `_PAGE_HIDDEN` stuff - used by kmemcheck.

* The comment at line 93 in `arch/x86/include/asm/pgtable.h` says:

```c
/*
 * The following only work if pte_present() is true.
 * Undefined behaviour if not..
 */
```

* The 'following' suggests all functions below it, so the `pte_present()` caveat
  should probably be added to these flag functions (and possibly just all of
  them.)

* Look into why `pmd_mkclean()` and `pte_mkclean()` do not also clear the
  soft-dirty flag.

* Be consistent with discussion of 'user-defined' flags in `mk/clr` functions
  too.

* `preempt_disable/enable()` appears to simply call `barrier()` on x86 rather
  than actually doing anything to prevent preemption, why?!

* Weird that [flush_tlb_kernel_range()][flush_tlb_kernel_range] flushes
  individual pages given kernel mappings are marked `_PAGE_GLOBAL` (and afaict
  an `invlpg` instruction does not override this), seems mostly used by vmalloc
  stuff, maybe those pages aren't marked global?

* Talk about `rmap`s.

* `__handle_mm_fault()` seems to be a really core function.

* `struct per_cpu_pages` - is this used by modern kernels to obviate the need
  for specific 'quick lists' for allocating page table pages? Covered in what's
  new in linux 2.6 section of UtLVMM.

* What are `%esp` fixup stacks referenced in the [x86-64 memory map][x86-64-mm]?
  Presumably a means of handling 32-bit processes (given `esp` not `rsp`.)

* Cover ASLR (maybe mention DEP in passing, have already covered the NX bit.)

* `MODULES_END = 0xffffffffff000000` but the [x86-64 memory map][x86-64-mm] says
  they end at `0xffffffffff600000`. Why the difference?

* Cover anonymous memory.

* Cover spurious faults, e.g. [spurious_fault()][spurious_fault].

* Look at vsyscall emulation.

* Look at vmacache - https://lwn.net/Articles/589475/

* Trace through a fork/exec.

* Look into prefaulting - https://lwn.net/Articles/127301/ - is this still how
  this is done?

* Dive into `tlb_flush_pending` more deeply.

* Look into the use of the [empty_zero_page][empty_zero_page].

* Look in more depth into top-down vs. legacy bottom-up memory allocation e.g.
  as determined by [arch_pick_mmap_layout()][arch_pick_mmap_layout].

* Why is [TASK_UNMAPPED_BASE][TASK_UNMAPPED_BASE] set to (page-aligned)
  `TASK_SIZE/3`?

* Investigate behaviour at `sysctl -w vm.overcommit_memory=2` - this killed
  chrome for me, so presumably in the transition from one mode to another tasks
  are performed to put the system into the alternative mode. Perhaps when moving
  to no overcommit, all allocations are faulted in? I didn't see an oom killer
  invocation, but did see `traps: chrome[20500] trap invalid opcode
  ip:563d68f60422 sp:7ffc425e68b0 error:0 in
  chrome[563d66ac1000+5d99000]`. Confirmed that demand paging still happens
  under this mode.

* [for_each_process_thread()][for_each_process_thread]
  vs. [for_each_process()][for_each_process] - does the former double-count, or
  does the latter simply skip thread processes? Check.

* What happens when a new kernel mapping is added in page tables? Are all shared
  mappings updated? Or kept to a single PUD maybe? Probably just pre-mapped. -
  `KERNEL_IMAGE_SIZE_DEFAULT` seems to indicate that the 512MiB kernel is
  pre-mapped.

* Add discussion of [tlb_gather_mmu()][tlb_gather_mmu] and
  [tlb_finish_mmu()][tlb_finish_mmu].

* Investigate teardown of pages, e.g. via
  [unmap_page_range()][unmap_page_range].

## Interesting functions

* [free_pgd_range()][free_pgd_range].

* [vm_normal_page()][vm_normal_page] - has some discussion about 'special' PTEs,
  worth investigating.

[TASK_UNMAPPED_BASE]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/include/asm/processor.h#L785
[arch_pick_mmap_layout]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/mmap.c#L100
[device-mapper]:https://en.wikipedia.org/wiki/Device_mapper
[empty_zero_page]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/kernel/head_64.S#L527
[flush_tlb_kernel_range]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/tlb.c#L296
[for_each_process]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L2696
[for_each_process_thread]:https://github.com/torvalds/linux/blob/v4.6/include/linux/sched.h#L2718
[free_pgd_range]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L473
[mem_section]:https://github.com/torvalds/linux/blob/v4.6/include/linux/mmzone.h#L1040
[mtrr]:https://en.wikipedia.org/wiki/Memory_type_range_register
[pat]:https://en.wikipedia.org/wiki/Page_attribute_table
[spurious_fault]:https://github.com/torvalds/linux/blob/v4.6/arch/x86/mm/fault.c#L1045
[tlb_finish_mmu]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L273
[tlb_gather_mmu]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L219
[unmap_page_range]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L1268
[vm_normal_page]:https://github.com/torvalds/linux/blob/v4.6/mm/memory.c#L742
[x86-64-mm]:https://github.com/torvalds/linux/blob/v4.6/Documentation/x86/x86_64/mm.txt

[funcs]:../funcs/funcs.md

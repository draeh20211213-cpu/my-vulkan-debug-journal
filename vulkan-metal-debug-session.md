# Vulkan → Metal Debug Session  
**Date:** 2025-10-05  
**Target:** `vkGetQueryPoolResults` via MoltenVK on macOS  
**Tool:** LLDB + disassembly  

---

## 1. Break at `execute_add`
```asm
0x147df9eee:  callq  *0x1c8(%rax)        ; vkGetQueryPoolResults
2. Step into loader stub (libvulkan.1.4.321.dylib)

asm

Copy
0x155e7e5c0 <+0>:  testq  %rdi, %rdi
0x155e7e5c3 <+3>:  je     0x155e7e5e5
3. Loader forwards to ICD (libMoltenVK.dylib)

asm

Copy
0x1569bb240 <+0>:  pushq  %rbp
0x1569bb241 <+1>:  movq   %rsp, %rbp
4. Inside MVKTraceVulkanCallStartImpl

asm

Copy
0x1569b8b40 <+0>:  pushq  %rbp
0x1569b8b4d <+13>: subq   $0x128, %rsp
5. Core logic: MVKQueryPool::getResults

asm

Copy
0x1569cf5f0 <+0>:  pushq  %rbp
0x1569cf601 <+17>: movq   0x20(%rdi), %rax
0x1569cf605 <+21>: movl   0xc(%rax), %r12d
6. Return to caller (execute_add)

asm

Copy
0x1569bb306 <+198>: retq
7. Back-trace (final)


Copy
frame #0: vkGetQueryPoolResults + 198
frame #1: execute_add + 1175
frame #2: vulkan_add_timed + 555
...
Summary

Successfully stepped from Python → pybind11 → Vulkan → MoltenVK → Metal-ready driver
No direct Metal API call observed in this pass; query-result copying is handled inside MVKQueryPool::getResults
Full instruction-level log captured above

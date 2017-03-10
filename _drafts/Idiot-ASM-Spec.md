---
layout: post
---

```
r0~11

r12:    rflags
r13:    rsp
r14:    rbp
r15:    rip

opcode reg/mem, reg/mem/imm

mem:
    [imm]
    [reg]
    [base + index * scale + disp]
        base/index: reg
        scale:      1/2/4/8
        disp:       imm

mov r0, 1
add r0, 2

push
pop
```

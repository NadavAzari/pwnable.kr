
# tiny_hard

Tiny hard binary looks **exactly** like tiny_easy, So whats makes it so hard?

![nx_enabled](images/nx_enabled.png)

In tiny_easy, we sprayed shellcode into the stack and tried to ump somewhere there, this time its not possible, but in every other hard challenge U cant just jump to a shellcode in the stack, why is this one different?

tiny_hard binary is super small, which means you dont got almost any gadget, u dont have `libc` , u dont have external functions, u stuck with nothing!.

![no_got.png](images/no_got.png)

I ran vmmap, to double check libc isnt there, and something else came to my mind!

![vdso.png](images/vdso.png)

## VDSO

`vdso` is code that the linux kernel inject to any process running on a linux machine, sounds like a start point!
i took memory dump from start address to end address of the vdso in the binary, and ran `ROPGadget` on it
![syscall_gadget.png](images/syscall_gadget.png)

Perfect! this is the most effective gadget we can find since we can control any single byte of the stack frame of the gadget we are gonna jump to...
how can we control eax? its not that big of a deal..
![eax_control_instruction.png](images/eax_control_instruction.png)
first instruction is 
```assembly
pop eax
```

so basicly `$eax = argc`
so we have arbitrary syscall!

but we cant control any register.. and we cant do `SROP` because we cant control the stack frame either..

during the call, i noticed every register is `0`
So i started to looking for syscalls that can help me even if they get 0 and came up with a super cool idea:

## PTRACE

ptrace is a super powerfull syscall which being use for debugging purposes, every debugger uses this, all the powerfull features `gdb` offers, u can do using ptrace.
The only problem with using `ptrace_attach` is it downgrade permissions, but `ptrace(0,0,0,0)` calls **ptrace_traceme**
which basicly says, everyone can attach to me now, without downgrading the permissions!

From now on we just needs to write an exploit, which runs tiny_hard in the vulnerable way, attach to it, and we control everything!
registers, memory, everything!.

I wrote /bin/sh into the memory, prepared the registers to call `execve` and changed `eip` to point to our syscall gadget!

good luck!!
![win.png](images/win.png)
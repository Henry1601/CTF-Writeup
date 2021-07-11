# gogo
AUTHOR: THELSHELL
## Problem
> Hmmm this is a weird file... [enter_password](https://github.com/Henry1601/PicoCTF-Writeup/blob/main/Reverse%20Engineering/gogo/enter_password). There is a instance of the service running at `mercury.picoctf.net:4052`.
## Solution
```bash
	$ file enter_password 
	enter_password: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), statically linked, Go BuildID=3-hVI6nMz0HbfIUMSEzq/TkiA8oRk8FHsCuRXIle2/C1my_KvOIt2KUk44LyQs/-XrwOx7UDhcGGdtF5xpG, with debug_info, not stripped
```
The given file is executable and not stripped, that means it is able to debug it. Let's run the program to see what it does.
```bash
	$ chmod +x enter_password
	$ ./enter_password                        
	Enter Password:
```
After a few times trying, I figure out that if the input string is not less than 32-character length, it will print out "Try again!" (our first clue).

Now I use radare2 to debug the program. Some resources I think might be helpful for those are new to reverse engineering and **Radare2**: [A journey into Radare 2 - Part 1](https://www.megabeets.net/a-journey-into-radare-2-part-1/), [Radare2 Tutorial](https://www.youtube.com/watch?v=oW8Ey5STrPI&list=PLg_QXA4bGHpvsW-qeoi3_yhiZg8zBzNwQ)
```bash
	$ r2 -Ad enter_password
	...
	[0x080913d0]> afl | grep "main"
	0x0806d640   33 881          dbg.runtime.main
	0x0808def0    3 54           dbg.runtime.main.func1
	0x0808df30    5 44           dbg.runtime.main.func2
	0x080d4800    7 640          dbg.main.main
	0x080d4a80   15 247          dbg.main.checkPassword
	0x080d4b80    3 281          dbg.main.get_flag
	0x080d4ca0    7 63           dbg.main.check
	0x080d4ce0   11 368          dbg.main.ambush
	0x080d4e50    7 87           dbg.main.init
	[0x080913d0]> s dbg.main.checkPassword 
	[0x080d4a80]> V
```
> `r2 -Ad` parameter is to open program in debug mode and analyze all the debug-info.
> 
> `afl` command is to list all functions of the program.
> 
> `s` command is to move to the specified function.
>
> `V` command is to see in visual mode.

The disassembler show a lot but we just find what we need. Look at those first lines:
```bash
 	 0x080d4a80      658b0d000000.  mov ecx, dword gs:[0]       ; enter_password.go:30    ; void main.checkPassword(string input,bool ~r1);                                            
 	 0x080d4a87      8b89fcffffff   mov ecx, dword [ecx - 4]
	 0x080d4a8d      3b6108         cmp esp, dword [ecx + 8]
 ┌─< 0x080d4a90      0f86d7000000   jbe 0x80d4b6d
 │   0x080d4a96      83ec44         sub esp, 0x44
 │   0x080d4a99      8b4c244c       mov ecx, dword [arg_4ch]
 │   0x080d4a9d      83f920         cmp ecx, 0x20               ; enter_password.go:31    ; 32
┌──< 0x080d4aa0      0f8cab000000   jl 0x80d4b51
```
From the line at [0x080d4a9d] and with the clue we find out at first, I'm sure that the program only accept equal or more than 32-character length input since the `jl` command at line [0x080d4aa0] will lead to exit function
```bash
	0x080d4b51      c70424000000.  mov dword [esp], 0          ; enter_password.go:32
	0x080d4b58      e883f3fdff     call dbg.os.Exit            ;[1]
```
Now look at processing part of the function
```bash
      ┌──< 0x080d4b0c      eb01           jmp 0x80d4b0f               ; enter_password.go:70
      │╎   ; CODE XREFS from dbg.main.checkPassword @ 0x80d4b35, 0x80d4b38
    ┌┌───> 0x080d4b0e      40             inc eax
    ╎╎│╎   ; CODE XREF from dbg.main.checkPassword @ 0x80d4b0c
    ╎╎└──> 0x080d4b0f      83f820         cmp eax, 0x20               ; 32
    ╎╎┌──< 0x080d4b12      7d26           jge 0x80d4b3a
    ╎╎│╎   0x080d4b14      39d0           cmp eax, edx                ; enter_password.go:71
   ┌─────< 0x080d4b16      734e           jae 0x80d4b66
   │╎╎│╎   0x080d4b18      0fb62c01       movzx ebp, byte [ecx + eax]
   │╎╎│╎   0x080d4b1c      83f820         cmp eax, 0x20               ; enter_password.go:70    ; 32
  ┌──────< 0x080d4b1f      7345           jae 0x80d4b66               ; enter_password.go:71
  ││╎╎│╎   0x080d4b21      0fb6740404     movzx esi, byte [esp + eax + 4]
  ││╎╎│╎   0x080d4b26      31f5           xor ebp, esi
  ││╎╎│╎   0x080d4b28      0fb6740424     movzx esi, byte [esp + eax + 0x24]
  ││╎╎│╎   0x080d4b2d      95             xchg eax, ebp
  ││╎╎│╎   0x080d4b2e      87de           xchg esi, ebx
  ││╎╎│╎   0x080d4b30      38d8           cmp al, bl
  ││╎╎│╎   0x080d4b32      87de           xchg esi, ebx
  ││╎╎│╎   0x080d4b34      95             xchg eax, ebp
  ││└────< 0x080d4b35      75d7           jne 0x80d4b0e
  ││ ╎│╎   0x080d4b37      43             inc ebx                     ; enter_password.go:72
  ││ └───< 0x080d4b38      ebd4           jmp 0x80d4b0e
  ││  │╎   ; CODE XREF from dbg.main.checkPassword @ 0x80d4b12
  ││  └──> 0x080d4b3a      83fb20         cmp ebx, 0x20               ; enter_password.go:75    ; 32
  ││  ┌──< 0x080d4b3d      7509           jne 0x80d4b48
  ││  │╎   0x080d4b3f      c644245001     mov byte [arg_50h], 1       ; enter_password.go:76
  ││  │╎   0x080d4b44      83c444         add esp, 0x44
  ││  │╎   0x080d4b47      c3             ret
  ││  │╎   ; CODE XREF from dbg.main.checkPassword @ 0x80d4b3d
  ││  └──> 0x080d4b48      c644245000     mov byte [arg_50h], 0       ; enter_password.go:78
  ││   ╎   0x080d4b4d      83c444         add esp, 0x44
  ││   ╎   0x080d4b50      c3             ret
```
From this code, we see that the program take each character of the input string to XOR with each character of some key string, which is located at [esp+4], then compare the result with another string located at [esp+0x24].

Now, put a breakpoint at the end of this code block to see what inside the stack (esp) but of course before the function return.
```bash
	:> db 0x080d4b3d
	:> dc
	(4480) Created thread 4536
	(4480) Created thread 4537
	(4480) Created thread 4538
	[+] SIGNAL 19 errno=0 addr=0x00000000 code=0 si_pid=0 ret=0
	[+] signal 19 aka SIGSTOP received 0
	:> dc
	Enter Password: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
	hit breakpoint at: 0x80d4b3d
```
> `db` command is to put breakpoint at specified address.
>
>`dc` command is to continue running the program.
>
> The stack is usually small in visual mode, use can expand it with command `e stack.size = 128` which would set the stack size to 128 bytes.

And here is the hexdump from [esp]:
```bash
	- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
	0x19c3df24  0000 0000 3836 3138 3336 6631 3365 3364  ....861836f13e3d
	0x19c3df34  3632 3764 6661 3337 3562 6462 3833 3839  627dfa375bdb8389
	0x19c3df44  3231 3465 4a53 475d 4145 0354 5d02 5a0a  214eJSG]AE.T].Z.
	0x19c3df54  5357 450d 0500 5d55 5410 010e 4155 574b  SWE...]UT...AUWK
	0x19c3df64  4550 4601 c248 0d08 2000 c919 2000 0000  EPF..H.. ... ...
```
Here are what we're looking for, 32 bytes from [esp+4] will be the key and 32 bytes from [esp+0x24] will be the final string which must be match after the XOR process. So what we gonna do now is just simply XOR the final string with the key to find to password.
```bash
	$ python
	>>> key = 0x3836313833366631336533643632376466613337356264623833383932313465
	>>> result = 0x4a53475d414503545d025a0a5357450d05005d555410010e4155574b45504601
	>>> password = key ^ result
	>>> bytes.fromhex(hex(password)[2:])
	b'reverseengineericanbarelyforward'
```
Let's try the password
```bash
	$ ./enter_password 
	Enter Password: reverseengineericanbarelyforward
	=========================================
	This challenge is interrupted by psociety
	What is the unhashed key?
```

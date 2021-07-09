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
	[0x080913c0]> afl | grep main
	0x0806d640   33 881          dbg.runtime.main
	0x0808def0    3 54           dbg.runtime.main.func1
	0x0808df30    5 44           dbg.runtime.main.func2
	0x080d4800    7 640          dbg.main.main
	0x080d4a80   15 247          dbg.main.checkPassword
	0x080d4b80    3 281          dbg.main.get_flag
	0x080d4ca0    7 63           dbg.main.check
	0x080d4ce0   11 368          dbg.main.ambush
	0x080d4e50    7 87           dbg.main.init
	[0x080913c0]> s dbg.main.checkPassword
```
> `r2 -Ad` parameter is to open program in debug mode and analyze the debug information.
> `afl` command is to list all functions of the program.
> `s` command is to move to the specified function.

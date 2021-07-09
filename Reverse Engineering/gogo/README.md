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
	[0x080d4a80]> pdd
```
> `r2 -Ad` parameter is to open program in debug mode and analyze the debug information.
> 
> `afl` command is to list all functions of the program.
> 
> `s` command is to move to the specified function.
>
> `pdd` command is to decompile the function "checkPassword"
```bash
	#include <stdint.h>
 
	int32_t dbg_main_checkPassword (int32_t arg_48h, int32_t arg_4ch, int32_t arg_50h) {
		uint8 key;
		string input;
		bool ~r1;
		int32_t var_4h;
		int32_t var_8h;
		int32_t var_ch;
		int32_t var_10h;
		int32_t var_14h;
		int32_t var_18h;
		int32_t var_1ch;
		int32_t var_20h;
		int32_t var_24h;
		/* void main.checkPassword(string input,bool ~r1); */
	label_2:
		ecx = *(gs:0);
		ecx = *((ecx - 4));
		if (esp <= *((ecx + 8))) {
			goto label_3;
		}
		ecx = arg_4ch;
		if (ecx < 0x20) {
			goto label_4;
		}
	label_1:
    edi = &var_4h;
    eax = 0;
    fcn_08090b18 ();
    edi = &var_24h;
    esi = obj_main_statictmp_4;
    eax = fcn_08090fe0 (0x38313638, 0x31663633, 0x64336533, 0x64373236, 0x37336166, 0x62646235, 0x39383338, 0x65343132);
    ecx = arg_48h;
    edx = arg_4ch;
    eax = 0;
    ebx = 0;
    while (al != bl) {
label_0:
        eax++;
        if (eax >= 0x20) {
            goto label_5;
        }
        if (eax >= edx) {
            goto label_6;
        }
        ebp = *((ecx + eax));
        if (eax >= 0x20) {
            goto label_6;
        }
        esi = *((esp + eax + 4));
        ebp ^= esi;
        esi = *((esp + eax + 0x24));
        tmp_0 = eax;
        eax = ebp;
        ebp = tmp_0;
        tmp_1 = esi;
        esi = ebx;
        ebx = tmp_1;
        tmp_2 = esi;
        esi = ebx;
        ebx = tmp_2;
        tmp_3 = eax;
        eax = ebp;
        ebp = tmp_3;
    }
    ebx++;
    goto label_0;
label_5:
    if (ebx == 0x20) {
        arg_50h = 1;
        return eax;
    }
    arg_50h = 0;
    return eax;
label_4:
    os_Exit (0);
    ecx = arg_4ch;
    goto label_1;
label_6:
    runtime_panicindex ();
    __asm ("ud2");
label_3:
    runtime_morestack_noctxt ();
    goto label_2;
}
```

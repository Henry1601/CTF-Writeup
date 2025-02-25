# asm4
AUTHOR: SANJAY C
## Problem
> What will asm4("picoCTF_f97bb") return? Submit the flag as a hexadecimal value (starting with '0x'). [Source](https://github.com/Henry1601/PicoCTF-Writeup/blob/main/Reverse%20Engineering/asm4/test4.S)
## Solution
As I mentioned [here](https://github.com/Henry1601/PicoCTF-Writeup/tree/main/Reverse%20Engineering/asm1), I will only explain handwork solution in this post.

Also, this explaination requires knowledge of Flag Register in x86 architecture. If you haven't heard or still confuse about it, this will be helpful: [Assembly Language Tutorial 4: Flags Register CF, OF, ZF ,AF, SF, PF](https://www.youtube.com/watch?v=oQKa5q-jVzY)

Here is the code:
```bash
asm4:
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	push   ebx
	<+4>:	sub    esp,0x10
	<+7>:	mov    DWORD PTR [ebp-0x10],0x27a
	<+14>:	mov    DWORD PTR [ebp-0xc],0x0
	<+21>:	jmp    0x518 <asm4+27>
<+23>:		add    DWORD PTR [ebp-0xc],0x1
<+27>:		mov    edx,DWORD PTR [ebp-0xc]
	<+30>:	mov    eax,DWORD PTR [ebp+0x8]
	<+33>:	add    eax,edx
	<+35>:	movzx  eax,BYTE PTR [eax]
	<+38>:	test   al,al
	<+40>:	jne    0x514 <asm4+23>
	<+42>:	mov    DWORD PTR [ebp-0x8],0x1
	<+49>:	jmp    0x587 <asm4+138>
<+51>:		mov    edx,DWORD PTR [ebp-0x8]
	<+54>:	mov    eax,DWORD PTR [ebp+0x8]
	<+57>:	add    eax,edx
	<+59>:	movzx  eax,BYTE PTR [eax]
	<+62>:	movsx  edx,al
	<+65>:	mov    eax,DWORD PTR [ebp-0x8]
	<+68>:	lea    ecx,[eax-0x1]
	<+71>:	mov    eax,DWORD PTR [ebp+0x8]
	<+74>:	add    eax,ecx
	<+76>:	movzx  eax,BYTE PTR [eax]
	<+79>:	movsx  eax,al
	<+82>:	sub    edx,eax
	<+84>:	mov    eax,edx
	<+86>:	mov    edx,eax
	<+88>:	mov    eax,DWORD PTR [ebp-0x10]
	<+91>:	lea    ebx,[edx+eax*1]
	<+94>:	mov    eax,DWORD PTR [ebp-0x8]
	<+97>:	lea    edx,[eax+0x1]
	<+100>:	mov    eax,DWORD PTR [ebp+0x8]
	<+103>:	add    eax,edx
	<+105>:	movzx  eax,BYTE PTR [eax]
	<+108>:	movsx  edx,al
	<+111>:	mov    ecx,DWORD PTR [ebp-0x8]
	<+114>:	mov    eax,DWORD PTR [ebp+0x8]
	<+117>:	add    eax,ecx
	<+119>:	movzx  eax,BYTE PTR [eax]
	<+122>:	movsx  eax,al
	<+125>:	sub    edx,eax
	<+127>:	mov    eax,edx
	<+129>:	add    eax,ebx
	<+131>:	mov    DWORD PTR [ebp-0x10],eax
	<+134>:	add    DWORD PTR [ebp-0x8],0x1
<+138>: 	mov    eax,DWORD PTR [ebp-0xc]
	<+141>:	sub    eax,0x1
	<+144>:	cmp    DWORD PTR [ebp-0x8],eax
	<+147>:	jl     0x530 <asm4+51>
	<+149>:	mov    eax,DWORD PTR [ebp-0x10]
	<+152>:	add    esp,0x10
	<+155>:	pop    ebx
	<+156>:	pop    ebp
	<+157>:	ret
```
As usual, we are setting up the stack at these first lines.
```bash
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	push   ebx
	<+4>:	sub    esp,0x10
	<+7>:	mov    DWORD PTR [ebp-0x10],0x27a
	<+14>:	mov    DWORD PTR [ebp-0xc],0x0
	<+21>:	jmp    0x518 <asm4+27>
```
The stack now would look like this:
```bash
	     Stack
	|-------------|		(low memory)
	|----0x27a----|		<--- ebp - 0x10 (esp)
	|-----0x0-----|		<--- ebp - 0xc
	|-------------|		<--- ebp - 0x8
	|-----ebx-----|		<--- ebp - 0x4
	|-----ebp-----|
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|----string---|		<--- ebp + 0x8 (input string)
	|-------------|		(high memory)
```
Now, take the jump to line 27.
```bash
	<+23>:	add    DWORD PTR [ebp-0xc],0x1
	<+27>:	mov    edx,DWORD PTR [ebp-0xc]
	<+30>:	mov    eax,DWORD PTR [ebp+0x8]
	<+33>:	add    eax,edx
	<+35>:	movzx  eax,BYTE PTR [eax]
	<+38>:	test   al,al
	<+40>:	jne    0x514 <asm4+23>
```
Look at this and we know that this is a loop through the input string in order to count the length of the string. Let's me explain this.

At line 27, we move value in [ebp-0xc] into **EDX** and address of the string into **EAX**. Note that **EAX** now is a pointer, which points to the string location, treat the string as an Array. After adding 0 to **EAX** (since **EDX** = 0), take the first byte of the string (index [0]), which is character "p", zero extend and align it back into **EAX**.
>`movzx` "**move with zero extension**" is special version of the `mov` instruction that perform zero extension from the source to the destination. This is the only instruction that allows the source and destination to be different sizes.
>
>For example:
>```bash
>		al = 1101 0011 (8 bits)
>		movzx	ax,al
>		ax = 0000 0000 1101 0011 (16 bits)
>```

The purpose of `test	al,al` is to check if that was end of the string. `test` instruction will set ZF (zero flag) if and only if `al` equals 0. `jne` instruction will take the jump if ZF is set, or else, we back to line 23 to increase the pointer **EAX** to the next character in the string.
>*Note that at line 35, we only take a BYTE from **EAX**.*

This code block re-written in C would look like this:
```bash
	int strLen (char *letter) {
 		int i = 0;
 		while (*(letter + i) != '\0') {
 			i++;
 		}
 		return i;
	}
	//	strLen("picoCTF_f97bb") = 13
```
The value 13 is now stored at [ebp-0xc] since each round we add 1 to it.
```bash
	<+42>:	mov    DWORD PTR [ebp-0x8],0x1
	<+49>:	jmp    0x587 <asm4+138>
```
At line 42 and 49, we stored 1 into [ebp-0x8] (*I guess this will be another count variable for the next loop*) and take unconditionally jump to line 138. Before that, let's take a look at our stack at this time:
```bash
	     Stack
	|-------------|		(low memory)
	|----0x27a----|		<--- ebp - 0x10
	|-----0xd-----|		<--- ebp - 0xc (13 = 0xd)
	|-----0x1-----|		<--- ebp - 0x8
	|-----ebx-----|		<--- ebp - 0x4
	|-----ebp-----|
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|----string---|		<--- ebp + 0x8 (input string)
	|-------------|		(high memory)
```
This is the main loop of the function. I wrote each line from 51 to 134 one by one into C pseudo code.
>Note that the square brackets ([ ]) in assembly mean *pointer*.
```bash
<+51>:		edx = *[ebp-0x8]
	<+54>:	eax = *[ebp+0x8]
	<+57>:	eax = eax + edx
	<+59>:	movzx  eax,BYTE PTR [eax]
	<+62>:	movsx  edx,al
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
	<+65>:	eax = *[ebp-0x8]
	<+68>:	ecx = *[eax - 0x1]
	<+71>:	eax = *[ebp+0x8]
	<+74>:	eax = eax + ecx
	<+76>:	movzx  eax,BYTE PTR [eax]
	<+79>:	movsx  eax,al
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
	<+82>:	edx = edx - eax
	<+84>:	eax = edx
	<+86>:	edx = eax
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
	<+88>:	eax = *[ebp-0x10]
	<+91>:	ebx = *edx + (*eax)*1
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
	<+94>:	eax = *[ebp-0x8]
	<+97>:	lea    edx,[eax+0x1]
	<+100>:	mov    eax,DWORD PTR [ebp+0x8]
	<+103>:	add    eax,edx
	<+105>:	movzx  eax,BYTE PTR [eax]
	<+108>:	movsx  edx,al
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
	<+111>:	mov    ecx,DWORD PTR [ebp-0x8]
	<+114>:	mov    eax,DWORD PTR [ebp+0x8]
	<+117>:	add    eax,ecx
	<+119>:	movzx  eax,BYTE PTR [eax]
	<+122>:	movsx  eax,al
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
	<+125>:	sub    edx,eax
	<+127>:	mov    eax,edx
	<+129>:	add    eax,ebx
	<+131>:	mov    DWORD PTR [ebp-0x10],eax
	<+134>:	add    DWORD PTR [ebp-0x8],0x1
<+138>: 	mov    eax,DWORD PTR [ebp-0xc]
	<+141>:	sub    eax,0x1
	<+144>:	cmp    DWORD PTR [ebp-0x8],eax
	<+147>:	jl     0x530 <asm4+51>
```
At line 138, we move the value in [ebp-0xc], which is the length of the input string (13) into **EAX**. After subtract 1 from **EAX**, jump to line 51 if it is greater than [ebp-0x8].

By the way, take a quick look at the whole block, we see that there's only 1 caculation on [ebp-0x8], which is adding 1 to it at line 134. So we can imagine the loop would be something like:
```bash
	int len = 13;		//[ebp-0xc]
	int i = 0;		//[ebp-0x8]
	do {
	
		...		//some code here
	
		len = len - 1;
		i++;
	} while (len > i);
```

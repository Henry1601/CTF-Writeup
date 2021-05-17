# asm4
AUTHOR: SANJAY C
## Problem
> What will asm4("picoCTF_f97bb") return? Submit the flag as a hexadecimal value (starting with '0x'). [Source](https://github.com/Henry1601/PicoCTF-Writeup/blob/main/Reverse%20Engineering/asm4/test4.S)
## Solution
As I mentioned [here](https://github.com/Henry1601/PicoCTF-Writeup/tree/main/Reverse%20Engineering/asm1), I will only explain handwork solution in this post.

Here is the code:
```
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
```
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	push   ebx
	<+4>:	sub    esp,0x10
	<+7>:	mov    DWORD PTR [ebp-0x10],0x27a
	<+14>:	mov    DWORD PTR [ebp-0xc],0x0
	<+21>:	jmp    0x518 <asm4+27>
```
The stack now would look like this:
```
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
```
	<+23>:	add    DWORD PTR [ebp-0xc],0x1
	<+27>:	mov    edx,DWORD PTR [ebp-0xc]
	<+30>:	mov    eax,DWORD PTR [ebp+0x8]
	<+33>:	add    eax,edx
	<+35>:	movzx  eax,BYTE PTR [eax]
	<+38>:	test   al,al
	<+40>:	jne    0x514 <asm4+23>
```
Look at this and we know that this is a loop through the input string in order to count the length of the string. Let's me explain this.

At line 27, we move value in [ebp-0xc] into **EDX** and address of the string into **EAX**. Note that **EAX** now is a pointer, which point to the string location. After adding 0 to **EAX** (since **EDX** = 0), take the first byte of the string, which is letter "p", zero extend it and align it back into **EAX**.
>`movzx` "**move with zero extension**" is special version of the `mov` instruction that perform zero extension from the source to the destination. This is the only instruction that allows the source and destination to be different sizes.
>For example:
>```
>	al = 1101 0011 (8 bits)
>	movzx	ax,al
>	ax = 0000 0000 1101 0011 (16 bits)
>```

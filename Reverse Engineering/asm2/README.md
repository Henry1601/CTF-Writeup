# asm2
AUTHOR: SANJAY C
## Problem
> What does asm2(0xb,0x2e) return? Submit the flag as a hexadecimal value (starting with '0x'). [Source](https://github.com/Henry1601/PicoCTF-Writeup/blob/9caa979d09341e7326d72a19b2862da2f389e28a/Reverse%20Engineering/asm2/test2.S)
## Solution
As I mentioned [here](https://github.com/Henry1601/PicoCTF-Writeup/blob/ac4c178de5946ae21a2cbeabbe06448f9dd20d17/Reverse%20Engineering/asm1/README.md), I will only explain handwork solution in this post.

Here is the code:
```
asm2:
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	sub    esp,0x10
	<+6>:	mov    eax,DWORD PTR [ebp+0xc]
	<+9>:	mov    DWORD PTR [ebp-0x4],eax
	<+12>:	mov    eax,DWORD PTR [ebp+0x8]
	<+15>:	mov    DWORD PTR [ebp-0x8],eax
	<+18>:	jmp    0x509 <asm2+28>
	<+20>:	add    DWORD PTR [ebp-0x4],0x1
	<+24>:	sub    DWORD PTR [ebp-0x8],0xffffff80
	<+28>:	cmp    DWORD PTR [ebp-0x8],0x63f3
	<+35>:	jle    0x501 <asm2+20>
	<+37>:	mov    eax,DWORD PTR [ebp-0x4]
	<+40>:	leave  
	<+41>:	ret
```
As usual, first 3 lines are the set-up step of the function.
```
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	sub    esp,0x10
```
We push **EBP** into the stack, then put **ESP** 4 double words backward.
> Note that all values in assembly are hexadecimal, which means 0x10 = 16 bytes = 4 double words.
```
	     Stack
	|-------------|		(high memory)
	|-----0x2e----|		<--- ebp + 0xc (second input value)
	|-----0xb-----|		<--- ebp + 0x8 (first input value)
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|-----ebp-----|		<--- ebp
	|-------------|		(low memory)
```

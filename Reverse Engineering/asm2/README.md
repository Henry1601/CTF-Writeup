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
As usual, first 3 lines are the set-up steps of the function.
```
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	sub    esp,0x10
```
We push **EBP** into the stack, move stack pointer (**ESP**) into it, then subtract 0x10 from **ESP**, which means we align the stack to a 16-byte boundary. Remember that stack grows towards lower memory addresses, that's why we subtract **ESP**.
> *Note that all values in assembly are hexadecimal, 0x10 = 16 bytes*
```
	     Stack
	|-------------|		(low memory)
	|-----esp-----|		<--- ebp - 0x10
	|-------------|		<--- ebp - 0xc
	|-------------|		<--- ebp - 0x8
	|-------------|		<--- ebp - 0x4
	|-----ebp-----|
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|-----0xb-----|		<--- ebp + 0x8 (first input value)
	|-----0x2e----|		<--- ebp + 0xc (second input value)
	|-------------|		(high memory)
```
```
	<+6>:	mov    eax,DWORD PTR [ebp+0xc]
	<+9>:	mov    DWORD PTR [ebp-0x4],eax
	<+12>:	mov    eax,DWORD PTR [ebp+0x8]
	<+15>:	mov    DWORD PTR [ebp-0x8],eax
	<+18>:	jmp    0x509 <asm2+28>
```
Next, we move 0x2e (second input value) to [ebp-0x4] at line 6 and 9, 0xb (first input value) to [ebp-0x8] at line 12 and 15. Then, take the jump to line 28.
```
	     Stack
	|-------------|		(low memory)
	|-----esp-----|		<--- ebp - 0x10
	|-------------|		<--- ebp - 0xc
	|-----0xb-----|		<--- ebp - 0x8
	|-----0x2e----|		<--- ebp - 0x4
	|-----ebp-----|
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|-----0xb-----|		<--- ebp + 0x8 (first input value)
	|-----0x2e----|		<--- ebp + 0xc (second input value)
	|-------------|		(high memory)
```
```
	<+20>:	add    DWORD PTR [ebp-0x4],0x1
	<+24>:	sub    DWORD PTR [ebp-0x8],0xffffff80
	<+28>:	cmp    DWORD PTR [ebp-0x8],0x63f3
	<+35>:	jle    0x501 <asm2+20>
```
This is the loop of the function. As you can see, we compare [ebp-0x8] with 0x63f3 and jump to line 20 "**if less or equal**" (`jle`). At line 20 and 24, we add 0x1 to [ebp-0x4], subtract 0xffffff80 from [ebp-0x8] then continue looping until [ebp-0x8] greater than 0x63f3. Note that we are working on x86 architecture, so 0xffffff80 = -0x80.

**[ebp-0x8] - (-0x80) = [ebp-0x8] + 0x80**

Now, we can re-write line 24:
```
	<+24>:  add    DWORD PTR [ebp-0x8],0x80
```
In the last 3 lines, we return the value in **EAX**, which is [ebp-0x4].
```
	<+37>:	mov    eax,DWORD PTR [ebp-0x4]
	<+40>:	leave  
	<+41>:	ret
```
We can imagine the code written in C like this:
```
	asm2(int a, int b) {
		while(a <= 0x63f3) {
			b += 1;
			a += 0x80;
		}
		return b;
	}
```
Finally (easy now), calculate the loop and we get the flag.

*Have fun hacking!*
## Flag
`0xf6`

# asm1
AUTHOR: SANJAY C
## Problem
> What does asm1(0x8be) return? Submit the flag as a hexadecimal value (starting with '0x'). [Source](https://github.com/Henry1601/PicoCTF-Writeup/blob/6eecbc6c032866c91e5e09daebb27ac1482603a7/Reverse%20Engineering/asm1/test1.S)
## Solution
To me, there are two way to solve assembly problems like this one. First and recommend way is to read it line by line so we could understand it better. Second is to make it executable and let computer do it, we would get the result faster and more convenient. Here I will explain both but the second way will only be explained in this post.

Note that this will require basic knowledge of assembly code that I can't cover it all so you can look up on the Internet.

Let's begin! Here is the code
```
asm1:
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
	<+3>:	cmp    DWORD PTR [ebp+0x8],0x71c
	<+10>:	jg     0x512 <asm1+37>
	<+12>:	cmp    DWORD PTR [ebp+0x8],0x6cf
	<+19>:	jne    0x50a <asm1+29>
	<+21>:	mov    eax,DWORD PTR [ebp+0x8]
	<+24>:	add    eax,0x3
	<+27>:	jmp    0x529 <asm1+60>
	<+29>:	mov    eax,DWORD PTR [ebp+0x8]
	<+32>:	sub    eax,0x3
	<+35>:	jmp    0x529 <asm1+60>
	<+37>:	cmp    DWORD PTR [ebp+0x8],0x8be
	<+44>:	jne    0x523 <asm1+54>
	<+46>:	mov    eax,DWORD PTR [ebp+0x8]
	<+49>:	sub    eax,0x3
	<+52>:	jmp    0x529 <asm1+60>
	<+54>:	mov    eax,DWORD PTR [ebp+0x8]
	<+57>:	add    eax,0x3
	<+60>:	pop    ebp
	<+61>:	ret>
```
I'll break it into pieces and explain each one.
```
	<+0>:	push   ebp
	<+1>:	mov    ebp,esp
```
We know that we are putting 0x8be (input value of function) to stack, which gets pushed into **EBP** and then moved into **ESP** on line 0 and 1.
```
	     Stack
	|-------------|		(high memory)
	|----0x8be----|		<--- ebp + 0x8 (input value)
	|-----ret-----|		<--- ebp + 0x4 (return addr)
	|-----ebp-----|		<--- ebp
	|-------------|		(low memory)
```
```
	<+3>:	cmp    DWORD PTR [ebp+0x8],0x71c
	<+10>:	jg     0x512 <asm1+37>
```
In the next 2 line we are seeing that we are comparing 0x8be (value at [ebp+0x8]) with 0x71c and since 0x8be is greater than 0x71c, we take the jump to asm1+37. The `jg` means "**jump if greater**". Now we are going to line 37.
```
	<+37>:	cmp    DWORD PTR [ebp+0x8],0x8be
	<+44>:	jne    0x523 <asm1+54>
	<+46>:	mov    eax,DWORD PTR [ebp+0x8]
	<+49>:	sub    eax,0x3
	<+52>:	jmp    0x529 <asm1+60>
```
Here we are comparing input value with 0x8be. The `jne` means "**jump if not equal**" and obviously 0x8be is equal 0x8be so we do not take the jump. In line 46 and 49, we subtract 0x3 from 0x8be then move to EAX, which would be 0x8bb now. We then unconditionally jump `jmp` to line 60.
```
	<+60>:	pop    ebp
	<+61>:	ret>
```

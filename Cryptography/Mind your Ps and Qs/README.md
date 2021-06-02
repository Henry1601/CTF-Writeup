# Mind your Ps and Qs
AUTHOR: SARA
## Problem
> In RSA, a small `e` value can be problematic, but what about `N`? Can you decrypt this? [values](https://mercury.picoctf.net/static/bf5e2c8811afb4669f4a6850e097e8aa/values)
## Solution
Download the attached file and look into the file content, it would be something like this:
```
	Decrypt my super sick RSA:
	c: 421345306292040663864066688931456845278496274597031632020995583473619804626233684
	n: 631371953793368771804570727896887140714495090919073481680274581226742748040342637
	e: 65537
```
Ok, this's important! You need to do some research about RSA cipher to have better unstanding how this algorithm actually works inside.

> This will be heplful: [RSA Algorithm](https://www.di-mgt.com.au/rsa_alg.html)

You may confuse a bit at first since Cryptography required some knowledge of math such as Prime, Multiplicative inverse,... It may take time but keep learning and trust me. It's worth to do so. By the way, it's not too hard to understand.

First thing to do with RSA is to look at it public key (n,e). In RSA, `n` is a product of 2 prime integers and we need to factorize it before doing anything. This is also the hardest part of breaking an RSA, which make RSA secure. I used this tool to factorize: [Prime Factors Decomposition](https://www.dcode.fr/prime-factors-decomposition).
```
	n = 1461849912200000206276283741896701133693 * 431899300006243611356963607089521499045809
```
Next, we caculate of Euler Phi Function of n `φ(n)` which is the number of non-negative integers less than n that are relatively prime to n. If n is a product of 2 prime integers `p` and `q`, then `φ(n) = (p-1)*(q-1)`.
```
	φ(n) = 631371953793368771804570727896887140714061729769155038068711341335911329840163136
```
Now, we will find `d` - Multiplicative Inverse of `e` mod `φ(n)`. This pratical explain would be helpful: [How To Find The Inverse of a Number ( mod n ) - Inverses of Modular Arithmetic - Example](https://www.youtube.com/watch?v=shaQZg8bqUM).

And since this number is out of normal caculator's range, I used this tool: [Modular Inverse Calculator (A^-1 Modulo N)](https://www.dcode.fr/modular-inverse).
```
	d*e = 1 (mod n)
	d = 86820026055294556838164569629472617179839240561509150603097892917271661878321409
```
Last step, message `m` will be `c^d mod n`. I used this tool: [Modular Exponentiation Calculator - Power Mod](https://www.dcode.fr/modular-exponentiation).
```
	m = c^d (mod n) = 13016382529449106065927291425342535437996222135352905256639647889241102700065917
```
Convert the final `m` to ASCII and that's our flag!

Let me say that again: It's important to understanding the inside process of the RSA cipher. This will be your base knowledge to get into Cryptography. Don't be dependent on tools, tools help you caculate cipher faster but not help you to understand the cipher.

Here is the tool for the full cipher process: [RSA Cipher Calculator](https://www.dcode.fr/rsa-cipher). By the way, this page has many cool caculation tools to work with large numbers since normal caculators can not do so.

*Have fun hacking!*
## Flag
`picoCTF{sma11_N_n0_g0od_55304594}`

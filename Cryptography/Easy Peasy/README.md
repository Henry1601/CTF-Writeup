# Easy Peasy
AUTHOR: MADSTACKS
## Problem
> A one-time pad is unbreakable, but can you manage to recover the flag? (Wrap with picoCTF{}) `nc mercury.picoctf.net 11188` [otp.py](https://github.com/Henry1601/PicoCTF-Writeup/blob/main/Cryptography/Easy%20Peasy/otp.py)
## Solution
First, I use Netcat to find out what happen at the given host
```bash
	$ nc mercury.picoctf.net 11188
	******************Welcome to our OTP implementation!******************
	This is the encrypted flag!
	551e6c4c5e55644b56566d1b5100153d4004026a4b52066b4a5556383d4b0007
	
	What data would you like to encrypt? test123
	Here ya go!
	22390b11096e4b

	What data would you like to encrypt? a
	Here ya go!
	00

	What data would you like to encrypt?
```
As we can see, the program prints out the 32-byte encrypted flag then let us try to encrypt any character on input. Seem like it wants us to perform a "Known ciphertext Attack". If you haven't known about basic cipher attacks, this will be helpful: [The difference between these 4 breaking Cipher techniques?](https://crypto.stackexchange.com/questions/13274/the-difference-between-these-4-breaking-cipher-techniques)

Luckily, we've got the source code of the cipher algorithm attached. Let's take a look at it.
```bash
	#!/usr/bin/python3 -u
	import os.path

	KEY_FILE = "key"
	KEY_LEN = 50000
	FLAG_FILE = "flag"

	def startup(key_location) :
		flag = open(FLAG_FILE).read()
		kf = open(KEY_FILE, "rb").read()

		start = key_location
		stop = key_location + len(flag)

		key = kf[start:stop]
		key_location = stop

		result = list(map(lambda p, k : "{:02x}".format(ord(p) ^ k), flag, key))
		print("This is the encrypted flag!\n{}\n".format("".join(result)))

		return key_location

	def encrypt(key_location) :
		ui = input("What data would you like to encrypt? ").rstrip()
		if len(ui) == 0 or len(ui) > KEY_LEN :
			return -1

		start = key_location
		stop = key_location + len(ui)

		kf = open(KEY_FILE, "rb").read()

		if stop >= KEY_LEN :
			stop = stop % KEY_LEN
			key = kf[start:] + kf[:stop]
		else :
			key = kf[start:stop]
		key_location = stop

		result = list(map(lambda p, k : "{:02x}".format(ord(p) ^ k), ui, key))

		print("Here ya go!\n{}\n".format("".join(result)))

		return key_location


	print("******************Welcome to our OTP implementation!******************")
	c = startup(0)
	while c >= 0 :
		c = encrypt(c)
```
If you haven't known Python yet, look up on Google, do some research on it, Python is very easy to learn.

So far, we know that the encrypted flag is `551e6c4c5e55644b56566d1b5100153d4004026a4b52066b4a5556383d4b0007`, the length of the flag is 32 bytes since from the source code, each character we input will be encrypt with key and format into `{:02x}`, which are 2 hex characters. By the side, the length of the key is 50000 bytes and each time we input a string, the start point of key will move forward regarding of the length of the inputed string. If the start point of the the key exceeds 50000, it will return to the beginning.

The algorithm of the cipher is just basically **XOR** the each inputed character with corresponding key character in the key string. So that means if we somehow get the first 32 bytes of the key string, the problem will be solved since `(key^flag)^key = flag`.

We know that each time we execute the program, we get the encrypted flag, which means the first 32 bytes of the key string are already be used and the start point of the key now at the 33th position in the key string. So now if we input (50000-32) characters, the start point will return to the beginning. Then, continue to input 32 `\x00` characters (NULL characters) and we get the first 32-byte key string since `'\x00'^A = A` for any A.

This is my command using Python:
```bash
	$ python3 -c "print('\x00'*(50000-32)+'\n'+'\x00'*32)" | nc mercury.picoctf.net 11188
	.
	.
	.
	What data would you like to encrypt? Here ya go!
	62275c7838335c7866305c786462775c7862355c7865365c7861615a5c786536
	
	What data would you like to encrypt?	
```
Now all that is left is to **XOR** the encrypted flag with the key:
```bash
	$ python3
	>>> enc_flag = 0x551e6c4c5e55644b56566d1b5100153d4004026a4b52066b4a5556383d4b0007
	>>> key = 0x62275c7838335c7866305c786462775c7862355c7865365c7861615a5c786536
	>>> flag = hex(enc_flag^key)
	>>> bytes.fromhex(flag[2:]).decode('ascii')
	'7904ff830f1c5bba8f763707247ba3e1'
```

*Have fun hacking!*
## Flag
`picoCTF{7904ff830f1c5bba8f763707247ba3e1}`

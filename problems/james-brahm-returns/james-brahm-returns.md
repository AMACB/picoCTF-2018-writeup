# James Brahm Returns

> Dr. Xernon has finally approved an update to James Brahm's spy terminal. (Someone finally told them that ECB isn't secure.) Fortunately, CBC mode is safe! Right? Connect with `nc 2018shellX.picoctf.com XXXXX`. Source.

> Hint:  
> Who killed SSL3?

Before even starting the problem, connecting with ``nc 2018shellX.picoctf.com XXXXX`` and running a few trials is a good idea. Interestingly we get two inputs, and the ciphertext is pretty monstrously long for even very small inputs. After a few trials, we can see that changing, say, the last character or so causes the system to totally reject it. Not hugely surpising.

Now, it's useful to understand what is going on regarding the encryption of our messages simply because the ciphertext is a bit overwhelmingly long. Looking at the source code, the following few blocks are the ones that we care about.
```python
def encrypt(key, plain, IV):
	cipher = AES.new( key.decode('hex'), AES.MODE_CBC, IV.decode('hex') )
	return IV + cipher.encrypt(plain).encode('hex')
```
So indeed, this is CBC encryption. After having solved Magic Padding Oracle previously, padding oracle attacks as a general attack against CBC comes to mind. (I explain what this is later.) As a quick check, the initialization vectors are indeed randomized:
```python
		IV = ''.join(random.choice(digits + 'abcdef') for _ in range(32))
```
Looking at what gets encrypted more thoroughly, we see that:
```python
	message = """Agent,
Greetings. My situation report is as follows:
{0}
My agent identifying code is: {1}.
Down with the Soviets,
006
""".format( sitrep, agent_code )
```
and later on
```python
	message = message+PS

	h.update(message)
	message = pad(message+ h.digest())
```
It looks like our first input (the situation report) is placed before the flag, then our second input (the PS) is placed after the flag. (This is important.) After, the message is hashed, and the hash is appended to the message. Finally, the message is padded. This message is later encrypted as in the ``encrypt`` function.

Alright, if we want to do a padding oracle attack, we need to know precisely when we have good padding. Looking at that check, we see that:
```python
		message = decrypt(key, c, iv)
		message = check_padding(message)
		if message:
				if verify_mac(message):
					print("Successful decryption.")
				else:
					print("Ooops! Did not decrypt successfully. Please send again.")
		else:
			print("Ooops! Did not decrypt successfully. Please send again.")
```
Dangit. We get the same error message if the hash doesn't match (the ``verify_mac`` function) and if the padding doesn't match. Because of this, a simple padding oracle attack isn't going to work because we need to be darn sure that the hash already matches so that if we get an error we'll *know* it's because of a padding issue.

So in comes the hint: Who killed SSL3?

In short, the given program uses pretty much the same protocol that SSL3 uses: Create a message, append a hash, and then add some padding. However, the SSL3 protocol has an issue with checking the padding when it uses CBC, as described by a Google team in <a href="https://www.openssl.org/~bodo/ssl-poodle.pdf">this article</a>. (This is the "research" part of this challenge.)

There are two parts to this vulneratibility (named the "POODLE Attack"): First, a browser is coerced somehow in using the outdated SSL3 protocol, and then second, the actual decryption attack takes place with a clever padding oracle. Because the program in the challenge is already using SSL3, we only care about the second part of the attack.

The specifics of the attack relies on two facts about SSL3:
1. The program uses pkcs7 padding, as below. (We need to know how it's padded if we can do a padding oracle attack.)
1. When decrypting, the program only checks the *last* byte of the padding and strips off the corresponding number of bytes.

For an example of the second fact, if the final byte of the padding is ``\x01``, then the program will just strip off the final byte and continue with decryption. And if the final byte is ``\x10``, then the program will strip off a full block of 16 bytes and then continue with decryption. importantly, SSL3 does not care what's in the other bytes, and they can pretty much be whatever. Indeed, in the ``unpad`` function of our source code, we see that:
```python
def check_padding(message):
	check_char = ord(message[-2:].decode('hex'))
	if (check_char < 17) and (check_char > 0): #bud
		return message[:-check_char*2]
	else:
		return False
```
So only the last character of the padding is checked and then the corresponding number of bytes are stipped. What's the issue? Well, suppose we know that there is a full block of padding in a message (a full block of padding is added whenever the message takes up an integer number of blocks because without doing so ``deadbee\x01`` could mean ``deadbee\x01`` or ``deadbee``). Then *we can just replace the last block of padding with some of the ciphertext from inisde of the message*; it will probably not have valid padding once decrypted, but maybe it will. Importantly, notice that the hash and rest of the message remain unchanged, so if the padding just so happens to be correct, then obviously the rest of the message will work as well because it didn't change.

And conversely, if the padding is incorrect, then either we have a bogus padding byte at the end and we get an error (``return False``), or the length of the message after stripping won't be a multiple of 16 bytes, so the AES decryption will throw an error. Thus, in theory, we can conduct a padding oracle attack because in our very careful manner we can determine exactly when we have correct padding and exactly when we do not; that hash/mac verification has been completely bypassed. Do notice that this will be one byte at a time, however, because as noted, only one byte of the padding is checked. (Details are coming.)

Okay, specifics. (I'll go into what a padding oracle attack is here.) I'm going to introduce some notation here that will be useful throughout but mostly later. From <a href="https://blog.skullsecurity.org/2013/padding-oracle-attacks-in-depth">here</a>.
<pre>
E - Encryption function. Here, we're using AES.
D - Decryption function. Here, we're using AES.
P<sub>n</sub>[k] - the k<sup>th</sup> byte of the n<sup>th</sup> block of the plaintext. (I include the initialization vector.)
C<sub>n</sub>[k] - the k<sup>th</sup> byte of the n<sup>th</sup> block of the ciphertext.
"[k]" in the last two definitions can be omitted to mean the entire block.
</pre>
In order to ensure that we get that desired full block of padding, we can just try inputs of different lengths until the size of the ciphertext abruptly increases by 16 bytes/32 ciphertext characters. I'll let this nice visual explain why this works.
<pre>
INPUt LENGTTH     PADDED MESSAGE 
0             --> P<sub>0</sub> = deadbeefdead\x04\x04\x04\x04
1 (A)         --> P<sub>0</sub> = deadbeefdeadA\x03\x03\x03
2 (AA)        --> P<sub>0</sub> = deadbeefdeadAA\x02\x02
3 (AAA)       --> P<sub>0</sub> = deadbeefdeadAAA\x01
4 (AAAA)      --> P<sub>0</sub> = deadbeefdeadAAAA\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10\x10
</pre>
Viola. After some brute force with our program, we see that having 14 ``a``s gives a ciphertext much longer than with 13 ``a``s.
```
Select an option:
Encrypt message (E)
Send & verify (S)
e
Please enter your situation report: aaaaaaaaaaaaa  # 13 "a"s
Anything else?
encrypted: bf089c779e1ce77fc982b572d86710e4c1d5e697d4fee8ae1a01d146e66552b6270908554c9eae4fc3fc91264b7a8cde8e291ca5d8dab0e0a821bd0de668c38f6865021522a5add33d7f30c648c3d373cb675936624776b48ec749a52b64a86c6877912966d92156ef2fecad547c5a3468d4889b51730630aae32a561d0d41c983ed21c58d8e697c90dde51ff7f0350a7f3be7d0df23539c3da0adc1eaac3c061b98292da0db1c6d598980fd3bf66b2ab2a858322e8384c86aa104d609193f39
Select an option:
Encrypt message (E)
Send & verify (S)
e
Please enter your situation report: aaaaaaaaaaaaaa # 14 "a"s
Anything else?
encrypted: 00a2d629e6272a2c358f3dd4c8c9efd536abd08307266ada26108599c782ab70a6dbd30ead570dfa6e2c6e17ce28af39ef547b27dd524b71d4304fd3c600df11b386bb692563a3c263ea8c5e75a9af4d0188f6d32855df22bd02e186515bb9f19c36e98f9e1c231cc325bf31fb17ee2fb05b107863a877202e94e878ba1da6b76eedf6e8e3b38fa4c920a17d05b8986f8167397d2e52227f8adcdddf5d73ee87907b83332b7a9a4ab3dbaa7a2adf048cd48b1ba9a931c50385dadc3c4303a80a84b83cba7acaab64bafe8c686b9645f5
Select an option:
Encrypt message (E)
Send & verify (S)
```
So 14 ``a``s give a full block of padding. Notice that having 14, 30, 46, etc. ``a``s will all give us this full block of padding as well because they simply shift things over by an integer number of blocks.

Now, we could go off and try to decode the entire message, but it's faster to decode exactly what we want. Thus, we need to know the structure of the message so that we can find the blocks with the flag. Luckily, this is in the source code. Looking at the given structure of the message, we can guess lengths of the flags that will give lengths of ciphertext as described above. As an example run with 14 ``a``s, we split the message by blocks.
<pre>
THEORETICAL PLAINTEXT MESSAGE WITH IV
P<sub>0</sub> = [Inialization vector not yet stripped off]
P<sub>1</sub> = Agent,\nGreetings
P<sub>2</sub> = . My situation r
P<sub>3</sub> = eport is as foll
P<sub>4</sub> = ows:\naaaaaaaaaaa
P<sub>5</sub> = aaa\nMy agent ide
P<sub>6</sub> = ntifying code is
P<sub>7</sub> = : picoCTF{deadbe
P<sub>8</sub> = efdeadbeefdead}.
P<sub>9</sub> = \nDown with the S
P<sub>10</sub> = oviets,\n006\n2��
P<sub>11</sub> = i��`\���[��n�l�
P<sub>12</sub> = 
</pre>
The last group of unprintables in the message come from the hash and padding. Importantly, that entire last block looks like a similar unprintable, and indeed looking at ``ord(c)`` for each character tells us those are ``\x10``, as desired. So, we guess that the flag looks something like ``picoCTF{deadbeefdeadbeefdead}``. (Indeed, the above message has 208 bytes, which exactly matches the 416 charcters of the ciphertext described before.)

Importantly, we can put characters before and after the flag, so we can move where the flag lives in the plaintext (bytewise) while still maintaining a full block of padding. For instance, the following moves the ``p`` of the flag to the end of the ninth block.
<pre>
THEORETIECAL PLAINTEXT MESSAGE WITH IV
P<sub>0</sub> = [Inialization vector not yet stripped off]
P<sub>1</sub> = Agent,\nGreetings
P<sub>2</sub> = . My situation r
P<sub>3</sub> = eport is as foll
P<sub>4</sub> = ows:\naaaaaaaaaaa
P<sub>5</sub> = aaaaaaaaaaaaaaaa
P<sub>6</sub> = aaaaaaaaaaaaaaaa
P<sub>7</sub> = aaaaaaaaaaaaaaaa
P<sub>8</sub> = \nMy agent identi
P<sub>9</sub> = fying code is: p
P<sub>10</sub> = icoCTF{deadbeefd
P<sub>11</sub> = eadbeefdead}.\nDo
P<sub>12</sub> = wn with the Sovi
P<sub>13</sub> = ets,\n006\n<b>aaa</b>�!*�             # 3 "a"s added here.
P<sub>14</sub> = a�O�+<�Vq3EZ�
P<sub>15</sub> = 
</pre>
Now we are ready for the attack. We will attempt to decrypt that ``p``, which is a good test case because we know that the flag had better begin with a ``p``. The key step (as mentioned briefly before) is just to use the desired ciphertext block to replace the padding block. Here, after accounting for the initialization vector, we move the ninth block to the end. (This is a "man-in-the-middle" technique.)
<pre>
CIPHERTEXT BLOCKS AFTER ALTERING		CORRESPONDING PLAINTEXT AFTER ALTERING
C<sub>0</sub> = 6b8c055b7670548ed22d8793f56e2e98		P<sub>0</sub> = [Initialization vector]
C<sub>1</sub> = 15dac8676327c16072c36eb8e6e21f80		P<sub>1</sub> = Agent,\nGreetings
C<sub>2</sub> = cec31b8087a899e358be1fac0deec10d		P<sub>2</sub> = . My situation r
C<sub>3</sub> = 695135b390849b2dac0e399cd8f4e268		P<sub>3</sub> = eport is as foll
C<sub>4</sub> = cf9b38013fe3ee33dd6ff3212b6908b7		P<sub>4</sub> = ows:\naaaaaaaaaaa
C<sub>5</sub> = 65e862ad7c8d5cfd3229d630acee2414		P<sub>5</sub> = aaaaaaaaaaaaaaaa
C<sub>6</sub> = e19ca4b95d358e63cb73f656b68c7d0e		P<sub>6</sub> = aaaaaaaaaaaaaaaa
C<sub>7</sub> = 63c5c8d5f974cffe32914006c6ad84a1		P<sub>7</sub> = aaaaaaaaaaaaaaaa
C<sub>8</sub> = e7caa1638a15ecea1f4650ebde144ae5		P<sub>8</sub> = \nMy agent identi
C<sub>9</sub> = 575f2014846dcf94cb4059e69aa2e575		P<sub>9</sub> = fying code is: p
C<sub>10</sub> = 448855460a8ceaf3152d12a917f64d82		P<sub>10</sub> = icoCTF{deadbeefd
C<sub>11</sub> = 2ccca2a1a4f69c3d510354c4a249b438		P<sub>11</sub> = eadbeefdead}.\nDo
C<sub>12</sub> = 2231a7a00ce7192dc241b7291d8b47bd		P<sub>12</sub> = wn with the Sovi
C<sub>13</sub> = 2bf8149fd067b22c1362ab76b2bc46f1		P<sub>13</sub> = ets,\n006\naaa�!*�
C<sub>14</sub> = c726fe387b7d8c184ceae6455c422dda		P<sub>14</sub> = a�O�+<�Vq3EZ�
<b>C<sub>9</sub> = 575f2014846dcf94cb4059e69aa2e575</b>		P<sub>15</sub> = used to be "fying code is: p"
</pre>
After the message is decrypted, the first thing that happens is that the padding is checked. Because CBC is used, we know the following.
<pre>P<sub>15</sub>[15] = C<sub>14</sub>[15] ⊕ D(C<sub>15</sub>)[15]
P<sub>15</sub>[15] = C<sub>14</sub>[15] ⊕ D(C<sub>9</sub>)[15]            # C<sub>15</sub> = C<sub>9</sub>  
P<sub>15</sub>[15] = C<sub>14</sub>[15] ⊕ D(E(C<sub>8</sub> ⊕ P<sub>9</sub>))[15]  
P<sub>15</sub>[15] = C<sub>14</sub>[15] ⊕ C<sub>8</sub>[15] ⊕ P<sub>9</sub>[15]
P<sub>9</sub>[15] = C<sub>14</sub>[15] ⊕ C<sub>8</sub>[15] ⊕ P<sub>15</sub>[15]</pre>
Looking at this equation, we know that when the padding is checked, the only way we will fully go through with a working message is if 
<pre>P<sub>15</sub>[15] = \x10</pre>
This was discussed before: The only way that the message has a valid padding byte and ends up with an integer number of blocks is if the padding byte is ``\x10`` because the message started with a full block of padding. Notice that if padding is good and the above holds, then we can actually solve for the desired plaintext byte!
<pre>P<sub>9</sub>[15] = C<sub>14</sub>[15] ⊕ C<sub>8</sub>[15] ⊕ 16</pre>
Obviously we know all bytes of the ciphertext because they are given to us.

This is all fine and good: If we happen to get good padding, we can find the plaintext. So ... how can we gaurentee that we can get good padding? Well, we can't really, but we can just iterate the above algorithm. In particular, if we don't get good padding when we try the above the first time, then we can throw our hands up in there and just try, try again. In particular, because we've moved a block where it's not supposed to be in terms of the encryption, <code>D(C<sub>15</sub>)[15]</code> is pretty much random gibberish, and because <code>C<sub>14</sub>[15]</code> is an encrypted hash, it's pretty much random gibberish too. Because we want this random gibberish to just happen to xor out to ``\x10``, this will occur about 1 in 256 times. With Python scripting, a reasonably fast connection, and teammates, an average 256 trials per byte of the flag is not unreasonable. (This is the "padding oracle attack.") So, we can just brute force the final byte.

There is one last thing to talk about before bringing in the code. That is how exactly we'll solve for a byte other than the last one. Careful readers will already have noticed that I placed 3 ``a``s in the "PS" section of the input in order to place the ``p`` at the end of the ninth block while still giving us valid padding. By just removing ``a``s from the "situation report" section of the input and placing them at the end allows us to shift the flag's position in the message while keeping a full block of padding. For example, if we wanted to place the ``c`` in ``picoCTF`` at the end of the ninth block , we could move our ``a``s around in the following way.
<pre>
THEORETIECAL DECRYPTED MESSAGE
P<sub>0</sub> = [Inialization vector not yet stripped off]
P<sub>1</sub> = Agent,\nGreetings
P<sub>2</sub> = . My situation r
P<sub>3</sub> = eport is as foll
P<sub>4</sub> = ows:\naaaaaaaaaaa
P<sub>5</sub> = aaaaaaaaaaaaaaaa
P<sub>6</sub> = aaaaaaaaaaaaaaaa
P<sub>7</sub> = aaaaaaaaaaaaaa\nM
P<sub>8</sub> = y agent identify
P<sub>9</sub> = ing code is: pic
P<sub>10</sub> = oCTF{deadbeefdea
P<sub>11</sub> = dbeefdead}.\nDown
P<sub>12</sub> = with the Soviet
P<sub>13</sub> = s,\n006\n<b>aaaaa</b>�!*�             # There are 5 "a"s here, but the total number of "a"s has not changed.
P<sub>14</sub> = a�O�+<�Vq3EZ�
P<sub>15</sub> = 
</pre>
This way, we can use the same algorithm we used to brute force the ``p`` to brute force any character in the flag. The following code implements what we just described taking the desired index of the flag as an argument from ``sys.argv``.
```python
import subprocess
import sys
import time

""" This function will place the index_of_flag at the end of the 9th block. """
""" It will then take the 9th block, replace it with the block at the end, and ask the oracle. """
def test(index_of_flag):
	while True:
		try:
			p = subprocess.Popen(["nc", "2018shellX.picoctf.com", "XXXXX"], stdout=-1, stdin=-1, stderr=-1, bufsize=1)
			p.stdout.readline() # Welcome, Agent 006!\n
			p.stdout.readline() # Select an option:\n
			p.stdout.readline() # Encrypt message (E)\n
			p.stdout.readline() # Send & verify (S)\n
			
			p.stdin.write(b"e\n")
			p.stdin.flush()
			
			# 14 is typical offset stuff
			numAs = 14 + 16 * 3 - 3 - index_of_flag
			# This specific length will push the desired character to the end of the 9th block
			p.stdin.write(bytes("a" * numAs + "\n", 'utf-8'))
			p.stdin.flush()
			
			# Anything to add? Why yes.
			p.stdin.write(bytes("a" * (3 + index_of_flag) + "\n", 'utf-8'))
			p.stdin.flush()
			
			line = p.stdout.readline()
			""" Format of line: Please enter... || Anything else? || encrypted: [WHAT WE WANT] || \n """
			message = line.split(b" ")[-1][:-1]
			print("MESSAGE:", message)
			
			# Remove the last block, and replace it with the 9th block.
			altered = message[:-32] + message[9*32 : 10*32]
			print("ALTERED:", altered)
			
			p.stdout.readline() # Select an option:\n
			p.stdout.readline() # Encrypt a message (E)\n
			p.stdout.readline() # Send & verify (S)
			
			p.stdin.write(b"s\n")
			p.stdin.flush()
			
			p.stdin.write(altered + b"\n")
			p.stdin.flush()
			
			#print(p.stdout.readline())
			return b"Successful" in p.stdout.readline(), altered
		except IOError as e: # Time-outs mostly
			print("Got an error:", e)
			print("Trying again in 10 seconds ...")
			time.sleep(10)
	
index = 0 if len(sys.argv) == 1 else int(sys.argv[1])
tests = 0
done = False
final = ""
while not done:
	print("TEST NUMBER", tests)
	done, final = test(index)
	print(done, "\n")
	tests += 1

print("--------------------------------------------")
blocks = [final[32*i : 32*(i+1)] for i in range(len(final) // 32)]
print("At index", index, "of the flag, the character is:")
print('"', chr(16 ^ int(blocks[-2][-2:], 16) ^ int(blocks[8][-2:], 16)), '"') # Using quotes helps us notice newline characters and spaces.
```
Using the above program to slowly construct the flag (byte by byte) is doable, and at the end, we get what we want.

> ``picoCTF{g0_@g3nt006!_#######}``

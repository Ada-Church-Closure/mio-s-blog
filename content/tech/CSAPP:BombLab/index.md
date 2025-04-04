+++
date = '2025-03-09T18:11:55+08:00'
draft = false
title = 'CSAPP:BombLab'
+++

> 本期封面是动漫《轻音少女》第一季时唯一律澪之间发生小矛盾的故事，此时，她们三个正在远远的看着mio......

# CSAPP:BombLab

> [!NOTE]
>
> 本文主要参考博客：arthals.ink，如果你要学习方法，你只要看TA写的就可以了，我只看了前两层，    只是做个记录，我认为对于我来说很好的解决问题方式就是写注释(所以我这里有逐行的注释)。

gdb**指令**：

```assembly
p $rax  			# 打印寄存器 rax 的值
p $rsp  			# 打印栈指针的值
p/x $rsp  			# 打印栈指针的值，以十六进制显示
p/d $rsp  			# 打印栈指针的值，以十进制显示

x/2x $rsp  			# 以十六进制格式查看栈指针 %rsp 指向的内存位置 M[%rsp] 开始的两个单位。
x/2d $rsp 			# 以十进制格式查看栈指针 %rsp 指向的内存位置 M[%rsp] 开始的两个单位。
x/2c $rsp 			# 以字符格式查看栈指针 %rsp 指向的内存位置 M[%rsp] 开始的两个单位。
x/s $rsp 			# 把栈指针指向的内存位置 M[%rsp] 当作 C 风格字符串来查看。

x/b $rsp 			# 检查栈指针指向的内存位置 M[%rsp] 开始的 1 字节。
x/h $rsp 			# 检查栈指针指向的内存位置 M[%rsp] 开始的 2 字节（半字）。
x/w $rsp 			# 检查栈指针指向的内存位置 M[%rsp] 开始的 4 字节（字）。
x/g $rsp 			# 检查栈指针指向的内存位置 M[%rsp] 开始的 8 字节（双字）。

info registers  	# 打印所有寄存器的值
info breakpoints  	# 打印所有断点的信息

delete breakpoints 1  # 删除第一个断点，可以简写为 d 1
```

## phase1:

比较简单，就是对比一下字符串，熟悉一下。

```assembly
00000000000015ab <phase_1>:
    15ab:	f3 0f 1e fa          	endbr64 
    15af:	48 83 ec 08          	sub    $0x8,%rsp
    15b3:	48 8d 35 f2 1a 00 00 	lea    0x1af2(%rip),%rsi        # 这很简单，你只要查看rsi里面存放了什么东西就可以
    15ba:	e8 f3 05 00 00       	call   1bb2 <strings_not_equal>
    15bf:	85 c0                	test   %eax,%eax
    15c1:	75 05                	jne    15c8 <phase_1+0x1d>
    15c3:	48 83 c4 08          	add    $0x8,%rsp
    15c7:	c3                   	ret    
    15c8:	e8 f9 06 00 00       	call   1cc6 <explode_bomb>
    15cd:	eb f4                	jmp    15c3 <phase_1+0x18>
```

## phase2:

先看phase_2的代码，这是典型的循环

```assembly
00000000000015cf <phase_2>:
    15cf:	f3 0f 1e fa          	endbr64                         # 用来防止ROP攻击 
    15d3:	55                   	push   %rbp                     # 两个局部变量
    15d4:	53                   	push   %rbx
    15d5:	48 83 ec 28          	sub    $0x28,%rsp               # 分配了40个字节
    15d9:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax            # 金丝雀值，用来防止恶意修改
    15e0:	00 00 
    15e2:	48 89 44 24 18       	mov    %rax,0x18(%rsp)          # 把金丝雀值保存起来，这样被修改的时候就会提醒
    15e7:	31 c0                	xor    %eax,%eax                # ???
    15e9:	48 89 e6             	mov    %rsp,%rsi                # 栈指针的值赋给了rsi
    15ec:	e8 2d 07 00 00       	call   1d1e <read_six_numbers>  # 调用一个读取6个数字的函数
    15f1:	83 3c 24 01          	cmpl   $0x1,(%rsp)              # 第一个数字为1
    15f5:	75 0a                	jne    1601 <phase_2+0x32>
    15f7:	48 89 e3             	mov    %rsp,%rbx                # rbx为当前栈顶的地址
    15fa:	48 8d 6c 24 14       	lea    0x14(%rsp),%rbp          # rbp存放rsp + 20bytes的地址 0 4 8 12 16 20刚好六个数字用栈传递
    15ff:	eb 10                	jmp    1611 <phase_2+0x42>      # 无条件跳转1611
    1601:	e8 c0 06 00 00       	call   1cc6 <explode_bomb>
    1606:	eb ef                	jmp    15f7 <phase_2+0x28>
    1608:	48 83 c3 04          	add    $0x4,%rbx                # 相等的情况下考察第二个参数的情况
    160c:	48 39 eb             	cmp    %rbp,%rbx                # 循环终止条件
    160f:	74 10                	je     1621 <phase_2+0x52>
    1611:	8b 03                	mov    (%rbx),%eax              # 取第一个参数到eax
    1613:	01 c0                	add    %eax,%eax                # eax = eax * 2
    1615:	39 43 04             	cmp    %eax,0x4(%rbx)           # 和第二个参数作比较
    1618:	74 ee                	je     1608 <phase_2+0x39>      # 相等继续，不相等爆炸
    161a:	e8 a7 06 00 00       	call   1cc6 <explode_bomb>
    161f:	eb e7                	jmp    1608 <phase_2+0x39>
    1621:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
    1626:	64 48 2b 04 25 28 00 	sub    %fs:0x28,%rax
    162d:	00 00 
    162f:	75 07                	jne    1638 <phase_2+0x69>
    1631:	48 83 c4 28          	add    $0x28,%rsp
    1635:	5b                   	pop    %rbx
    1636:	5d                   	pop    %rbp
    1637:	c3                   	ret    
    1638:	e8 13 fc ff ff       	call   1250 <__stack_chk_fail@plt>
```

再看它调用的读取六个数字的function

```asm
0000000000001d1e <read_six_numbers>:
    1d1e:	f3 0f 1e fa          	endbr64 
    1d22:	48 83 ec 08          	sub    $0x8,%rsp                # 再分配8个字节
    1d26:	48 89 f2             	mov    %rsi,%rdx                # 记住rsi是上一层栈顶的位置放到rdx这里
    1d29:	48 8d 4e 04          	lea    0x4(%rsi),%rcx           # 把这些参数用寄存器向下一个sscanf传递
    1d2d:	48 8d 46 14          	lea    0x14(%rsi),%rax
    1d31:	50                   	push   %rax
    1d32:	48 8d 46 10          	lea    0x10(%rsi),%rax
    1d36:	50                   	push   %rax
    1d37:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
    1d3b:	4c 8d 46 08          	lea    0x8(%rsi),%r8
    1d3f:	48 8d 35 b6 15 00 00 	lea    0x15b6(%rip),%rsi        # 32fc <array.0+0x1fc> 这里应该是我们输入的数字 int sscanf(const char *str, const char *format, ...);
    1d46:	b8 00 00 00 00       	mov    $0x0,%eax
    # 分析一下sscanf的参数 rdi:就是我们输入的string,rsi是格式,就是"%d %d %d %d %d %d",rdx是第一个数,rcx是第二个数,r8是第三个数,r9是第四个数,现在寄存器不够用，用栈传递参数，并且是从右向左的这就很好理解了
    1d4b:	e8 b0 f5 ff ff       	call   1300 <__isoc99_sscanf@plt> #这里要调用sscanf函数
    1d50:	48 83 c4 10          	add    $0x10,%rsp
    1d54:	83 f8 05             	cmp    $0x5,%eax
    1d57:	7e 05                	jle    1d5e <read_six_numbers+0x40>
    1d59:	48 83 c4 08          	add    $0x8,%rsp
    1d5d:	c3                   	ret    
    1d5e:	e8 63 ff ff ff       	call   1cc6 <explode_bomb>

```

## phase3:

本层就是关于一些条件的判断(大概就是switch语句)（理解提升了，之前自己肯定没办法做出来的）：

```asm
000000000000163d <phase_3>:                                         # 提醒是关于switch语句,不是哥们是否有些太长了
    163d:	f3 0f 1e fa          	endbr64                         # 我们按照线性的方法先走一遍程序
    1641:	48 83 ec 28          	sub    $0x28,%rsp               # 分配了40个字节
    1645:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax            # 金丝雀值
    164c:	00 00 
    164e:	48 89 44 24 18       	mov    %rax,0x18(%rsp)          # 金丝雀值放在24个字节开始的位置
    1653:	31 c0                	xor    %eax,%eax                # 检查
    1655:	48 8d 4c 24 0f       	lea    0xf(%rsp),%rcx           # rcx = rsp + 15 占一个字节 第四个参数 a2
    165a:	48 8d 54 24 10       	lea    0x10(%rsp),%rdx          # rdx = rsp + 16 占四个字节 第三个参数 a1
    165f:	4c 8d 44 24 14       	lea    0x14(%rsp),%r8           # r8 = rsp + 20  占四个字节 第五个参数 a3
    1664:	48 8d 35 5e 1a 00 00 	lea    0x1a5e(%rip),%rsi        # 30c9 <_IO_stdin_used+0xc9> 这里的rsi是"%d %c %d"
    166b:	e8 90 fc ff ff       	call   1300 <__isoc99_sscanf@plt>
    1670:	83 f8 02             	cmp    $0x2,%eax                # sscanf的返回值是读取的参数的个数,若参数小于2错
    1673:	7e 20                	jle    1695 <phase_3+0x58>
    1675:	83 7c 24 10 07       	cmpl   $0x7,0x10(%rsp)          # a1大于7就爆炸
    167a:	0f 87 0a 01 00 00    	ja     178a <phase_3+0x14d>     
    1680:	8b 44 24 10          	mov    0x10(%rsp),%eax          # rax = a1(我们先假设a1 = 6)
    1684:	48 8d 15 55 1a 00 00 	lea    0x1a55(%rip),%rdx        # 30e0 <_IO_stdin_used+0xe0> rdx中加载了一个-68?
    168b:	48 63 04 82          	movslq (%rdx,%rax,4),%rax       # rax = 4 * rax + rdx 
    168f:	48 01 d0             	add    %rdx,%rax                # rax = rax + rdx(可能是跳表位置的计算？)
    1692:	3e ff e0             	notrack jmp *%rax               # 其作用是 跳转到 RAX 寄存器存储的地址，并且不记录 return address 到 影子调用栈（Shadow Stack
    1695:	e8 2c 06 00 00       	call   1cc6 <explode_bomb>
    169a:	eb d9                	jmp    1675 <phase_3+0x38>
    169c:	b8 77 00 00 00       	mov    $0x77,%eax
    16a1:	81 7c 24 14 a8 01 00 	cmpl   $0x1a8,0x14(%rsp)
    16a8:	00 
    16a9:	0f 84 e5 00 00 00    	je     1794 <phase_3+0x157>
    16af:	e8 12 06 00 00       	call   1cc6 <explode_bomb>
    16b4:	b8 77 00 00 00       	mov    $0x77,%eax
    16b9:	e9 d6 00 00 00       	jmp    1794 <phase_3+0x157>
    16be:	b8 70 00 00 00       	mov    $0x70,%eax
    16c3:	81 7c 24 14 bc 00 00 	cmpl   $0xbc,0x14(%rsp)
    16ca:	00 
    16cb:	0f 84 c3 00 00 00    	je     1794 <phase_3+0x157>
    16d1:	e8 f0 05 00 00       	call   1cc6 <explode_bomb>
    16d6:	b8 70 00 00 00       	mov    $0x70,%eax
    16db:	e9 b4 00 00 00       	jmp    1794 <phase_3+0x157>
    16e0:	b8 78 00 00 00       	mov    $0x78,%eax
    16e5:	81 7c 24 14 40 03 00 	cmpl   $0x340,0x14(%rsp)
    16ec:	00 
    16ed:	0f 84 a1 00 00 00    	je     1794 <phase_3+0x157>
    16f3:	e8 ce 05 00 00       	call   1cc6 <explode_bomb>
    16f8:	b8 78 00 00 00       	mov    $0x78,%eax
    16fd:	e9 92 00 00 00       	jmp    1794 <phase_3+0x157>
    1702:	b8 6e 00 00 00       	mov    $0x6e,%eax
    1707:	83 7c 24 14 39       	cmpl   $0x39,0x14(%rsp)
    170c:	0f 84 82 00 00 00    	je     1794 <phase_3+0x157>
    1712:	e8 af 05 00 00       	call   1cc6 <explode_bomb>
    1717:	b8 6e 00 00 00       	mov    $0x6e,%eax
    171c:	eb 76                	jmp    1794 <phase_3+0x157>
    171e:	b8 74 00 00 00       	mov    $0x74,%eax
    1723:	81 7c 24 14 c4 03 00 	cmpl   $0x3c4,0x14(%rsp)
    172a:	00 
    172b:	74 67                	je     1794 <phase_3+0x157>
    172d:	e8 94 05 00 00       	call   1cc6 <explode_bomb>
    1732:	b8 74 00 00 00       	mov    $0x74,%eax
    1737:	eb 5b                	jmp    1794 <phase_3+0x157>
    1739:	b8 67 00 00 00       	mov    $0x67,%eax
    173e:	81 7c 24 14 95 03 00 	cmpl   $0x395,0x14(%rsp)
    1745:	00 
    1746:	74 4c                	je     1794 <phase_3+0x157>
    1748:	e8 79 05 00 00       	call   1cc6 <explode_bomb>
    174d:	b8 67 00 00 00       	mov    $0x67,%eax
    1752:	eb 40                	jmp    1794 <phase_3+0x157>
    1754:	b8 71 00 00 00       	mov    $0x71,%eax           # a1小于7会跳转到这里 eax = 71,这里已经重新赋值了
    1759:	81 7c 24 14 f2 01 00 	cmpl   $0x1f2,0x14(%rsp)    # 看第三个参数的值,不等于498就爆炸
    1760:	00 
    1761:	74 31                	je     1794 <phase_3+0x157>
    1763:	e8 5e 05 00 00       	call   1cc6 <explode_bomb>
    1768:	b8 71 00 00 00       	mov    $0x71,%eax
    176d:	eb 25                	jmp    1794 <phase_3+0x157>
    176f:	b8 6e 00 00 00       	mov    $0x6e,%eax
    1774:	81 7c 24 14 83 03 00 	cmpl   $0x383,0x14(%rsp)
    177b:	00 
    177c:	74 16                	je     1794 <phase_3+0x157>
    177e:	e8 43 05 00 00       	call   1cc6 <explode_bomb>
    1783:	b8 6e 00 00 00       	mov    $0x6e,%eax
    1788:	eb 0a                	jmp    1794 <phase_3+0x157>
    178a:	e8 37 05 00 00       	call   1cc6 <explode_bomb>
    178f:	b8 6e 00 00 00       	mov    $0x6e,%eax
    1794:	38 44 24 0f          	cmp    %al,0xf(%rsp)        # 等于498的情况下来到这里,看输入的字符和al的值是否相等，查ASCII这里应该是G
    1798:	75 15                	jne    17af <phase_3+0x172> # 不相等爆炸
    179a:	48 8b 44 24 18       	mov    0x18(%rsp),%rax      # 第二个是q，OK结束，完全不知道具体的switch但是能做
    179f:	64 48 2b 04 25 28 00 	sub    %fs:0x28,%rax
    17a6:	00 00 
    17a8:	75 0c                	jne    17b6 <phase_3+0x179>
    17aa:	48 83 c4 28          	add    $0x28,%rsp
    17ae:	c3                   	ret    
    17af:	e8 12 05 00 00       	call   1cc6 <explode_bomb>
    17b4:	eb e4                	jmp    179a <phase_3+0x15d>
    17b6:	e8 95 fa ff ff       	call   1250 <__stack_chk_fail@plt>
```

## phase4:

关于递归函数？

这是phase4中调用的方法：

```assembly
00000000000017bb <func4>:                                       # 开始的参数 edi:a1(x1) esi:0(x2) edx:14(x3) 设返回值为x
    17bb:	f3 0f 1e fa          	endbr64                     # 这应该是一个递归函数
    17bf:	53                   	push   %rbx                 # 临时变量 (temp)
    17c0:	89 d0                	mov    %edx,%eax            # x = x3
    17c2:	29 f0                	sub    %esi,%eax            # x -= x2
    17c4:	89 c3                	mov    %eax,%ebx            # temp = x
    17c6:	c1 eb 1f             	shr    $0x1f,%ebx           # 这里相当于是取了temp的符号
    17c9:	01 c3                	add    %eax,%ebx            # temp += x
    17cb:	d1 fb                	sar    %ebx                 # temp /= 2(这里就是默认省略了1,愚蠢)
    17cd:	01 f3                	add    %esi,%ebx            # temp += x2
    17cf:	39 fb                	cmp    %edi,%ebx            # 比较和x1相不相等
    17d1:	7f 06                	jg     17d9 <func4+0x1e>    # 如果大于>
    17d3:	7c 10                	jl     17e5 <func4+0x2a>    # 如果小于<
    17d5:	89 d8                	mov    %ebx,%eax            # x = temp
    17d7:	5b                   	pop    %rbx
    17d8:	c3                   	ret    
    17d9:	8d 53 ff             	lea    -0x1(%rbx),%edx      # 大于x1的情况 x3 = temp - 1
    17dc:	e8 da ff ff ff       	call   17bb <func4>         # 递归调用
    17e1:	01 c3                	add    %eax,%ebx            # temp += x
    17e3:	eb f0                	jmp    17d5 <func4+0x1a>    # 返回
    17e5:	8d 73 01             	lea    0x1(%rbx),%esi       # 小于的情况 x1 = temp + 1
    17e8:	e8 ce ff ff ff       	call   17bb <func4>         # 递归调用
    17ed:	01 c3                	add    %eax,%ebx            # temp += x
    17ef:	eb e4                	jmp    17d5 <func4+0x1a>    # 返回
```

```assembly
00000000000017f1 <phase_4>:
    17f1:	f3 0f 1e fa          	endbr64 
    17f5:	48 83 ec 18          	sub    $0x18,%rsp               # 分配24个字节
    17f9:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax            # 金丝雀值
    1800:	00 00 
    1802:	48 89 44 24 08       	mov    %rax,0x8(%rsp)           # 把金丝雀值放到了stack上面
    1807:	31 c0                	xor    %eax,%eax
    1809:	48 8d 4c 24 04       	lea    0x4(%rsp),%rcx           # 栈上参数传递 第四个参数 4字节 a2
    180e:	48 89 e2             	mov    %rsp,%rdx                # 第三个参数 4字节 a1
    1811:	48 8d 35 f0 1a 00 00 	lea    0x1af0(%rip),%rsi        # 参数格式："%d %d"
    1818:	e8 e3 fa ff ff       	call   1300 <__isoc99_sscanf@plt>
    181d:	83 f8 02             	cmp    $0x2,%eax                # 是否读取正确
    1820:	75 06                	jne    1828 <phase_4+0x37>      # 不正确爆炸
    1822:	83 3c 24 0e          	cmpl   $0xe,(%rsp)              # a1 <= 14不然爆炸
    1826:	76 05                	jbe    182d <phase_4+0x3c>
    1828:	e8 99 04 00 00       	call   1cc6 <explode_bomb>      # a1 <= 14跳到这里 这里大概是为调用f4做准备
    182d:	ba 0e 00 00 00       	mov    $0xe,%edx                # 第三个参数是14
    1832:	be 00 00 00 00       	mov    $0x0,%esi                # 第二个参数是0
    1837:	8b 3c 24             	mov    (%rsp),%edi              # 第一个参数是a1
    183a:	e8 7c ff ff ff       	call   17bb <func4>             # 调用了f4,那就是说我们根据f4的逻辑来设置a1的输入
    183f:	83 f8 12             	cmp    $0x12,%eax               # 将返回值和18作比较
    1842:	75 07                	jne    184b <phase_4+0x5a>      # 不相同就爆炸
    1844:	83 7c 24 04 12       	cmpl   $0x12,0x4(%rsp)          # 把a2和18作比较
    1849:	74 05                	je     1850 <phase_4+0x5f>      # 不相同爆炸，相同结束
    184b:	e8 76 04 00 00       	call   1cc6 <explode_bomb>
    1850:	48 8b 44 24 08       	mov    0x8(%rsp),%rax
    1855:	64 48 2b 04 25 28 00 	sub    %fs:0x28,%rax
    185c:	00 00 
    185e:	75 05                	jne    1865 <phase_4+0x74>
    1860:	48 83 c4 18          	add    $0x18,%rsp
    1864:	c3                   	ret    
    1865:	e8 e6 f9 ff ff       	call   1250 <__stack_chk_fail@plt>

```

我对于fun4做了逆向：

```C
#include<stdio.h>
//手动逆向代码fun4
int fun4(int num1, int num2, int num3){
    int x = num3 - num2;
    int temp = x;
    if(temp < 0){
        ++temp;
    }
    temp /= 2;
    temp += num2;
    if(temp > num1){
        //要注意调用完成之后获取的rax的使用（因为这里只调用但没有获取值浪费了很长时间）
       return fun4(num1, num2, temp - 1) + temp;
    }else if(temp < num1){
       return fun4(num1, temp + 1, num3) + temp;
    }else{
        return temp;
    } 
}
//就是给一个输入，使得返回值为0x12
int main(){
    int num1;
    //scanf("%d", &num1);
    //当输入11时，答案为18,也就是answer
    int value = fun4(11, 0, 0xe);
    printf("%d\n", value);
}

```

## phase5:

hint：我的输入和array之间的转换关系,也不是很难

phase5:

```assembly
000000000000186a <phase_5>:
    186a:	f3 0f 1e fa          	endbr64 
    186e:	53                   	push   %rbx                     # 一个局部变量
    186f:	48 83 ec 10          	sub    $0x10,%rsp               # 开了16字节空间
    1873:	48 89 fb             	mov    %rdi,%rbx                # 局部变量存放rdi,rdi就是字符串的首地址
    1876:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax            # 金丝雀值
    187d:	00 00 
    187f:	48 89 44 24 08       	mov    %rax,0x8(%rsp)           # 放在栈上，也就是说有8字节的可用空间
    1884:	31 c0                	xor    %eax,%eax                # 校验
    1886:	e8 06 03 00 00       	call   1b91 <string_length>     # 调用string_length,这里应该是rdi作为参数读入了一个string
    188b:	83 f8 06             	cmp    $0x6,%eax                # 返回值和6比较，不相等爆炸，输入的字符串的长度要是6才可以
    188e:	75 55                	jne    18e5 <phase_5+0x7b>
    1890:	b8 00 00 00 00       	mov    $0x0,%eax                # eax = 0？下面大概是为strings_not_equal做准备，不相等爆炸
    1895:	48 8d 0d 64 18 00 00 	lea    0x1864(%rip),%rcx        # maduiersnfotvbylWow! You've defused the secret stage!
    189c:	0f b6 14 03          	movzbl (%rbx,%rax,1),%edx       # 就是我输入数字,从上面这个stirng中找值，构造一个rdi                                                             
    18a0:	83 e2 0f             	and    $0xf,%edx
    18a3:	0f b6 14 11          	movzbl (%rcx,%rdx,1),%edx
    18a7:	88 54 04 01          	mov    %dl,0x1(%rsp,%rax,1)
    18ab:	48 83 c0 01          	add    $0x1,%rax
    18af:	48 83 f8 06          	cmp    $0x6,%rax                # rax就是一个index作为循环控制量
    18b3:	75 e7                	jne    189c <phase_5+0x32>
    18b5:	c6 44 24 07 00       	movb   $0x0,0x7(%rsp)           # 最后为我们构造的字符串添加了一个结束符号
    18ba:	48 8d 7c 24 01       	lea    0x1(%rsp),%rdi
    18bf:	48 8d 35 0c 18 00 00 	lea    0x180c(%rip),%rsi        # *rsi = "bruins" 通过上面的操作，*rdi要等于"bruins"怎么操作？
    18c6:	e8 e7 02 00 00       	call   1bb2 <strings_not_equal> # 意思就是两个字符串不相同就爆炸
    18cb:	85 c0                	test   %eax,%eax
    18cd:	75 1d                	jne    18ec <phase_5+0x82>
    18cf:	48 8b 44 24 08       	mov    0x8(%rsp),%rax
    18d4:	64 48 2b 04 25 28 00 	sub    %fs:0x28,%rax
    18db:	00 00 
    18dd:	75 14                	jne    18f3 <phase_5+0x89>
    18df:	48 83 c4 10          	add    $0x10,%rsp
    18e3:	5b                   	pop    %rbx
    18e4:	c3                   	ret    
    18e5:	e8 dc 03 00 00       	call   1cc6 <explode_bomb>
    18ea:	eb a4                	jmp    1890 <phase_5+0x26>
    18ec:	e8 d5 03 00 00       	call   1cc6 <explode_bomb>
    18f1:	eb dc                	jmp    18cf <phase_5+0x65>
    18f3:	e8 58 f9 ff ff       	call   1250 <__stack_chk_fail@plt>
```

我们不必细究它调用的两个方法的具体实现了，就和函数名字一样。我输入的string是"M63487",因为实际上会和0xf作与运算，所以每个字符都是可选的。

## phase6:

> [!CAUTION]
>
> 应该是最难的一层了，hint：链表，那就要用到结构体了吧。（做完：其实还好，只要你理解它在干什么。）

```assembly
00000000000018f8 <phase_6>:
    18f8:	f3 0f 1e fa          	endbr64                        # 关于链表操作,最逆天的一层，孩子们
    18fc:	41 57                	push   %r15                    # 6个局部变量,都是拿来干嘛的？？？
    18fe:	41 56                	push   %r14
    1900:	41 55                	push   %r13
    1902:	41 54                	push   %r12
    1904:	55                   	push   %rbp
    1905:	53                   	push   %rbx
    1906:	48 83 ec 78          	sub    $0x78,%rsp              # 分配120个bytes
    190a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax           # 金丝雀值
    1911:	00 00 
    1913:	48 89 44 24 68       	mov    %rax,0x68(%rsp)         # 放到栈上，有104个bytes是可用的
    1918:	31 c0                	xor    %eax,%eax               # 检测金丝雀值
    191a:	4c 8d 74 24 10       	lea    0x10(%rsp),%r14         # 此时r14存放的是rsp + 16的地址 
    191f:	4c 89 74 24 08       	mov    %r14,0x8(%rsp)          # 把rsp + 16的地址放在rsp + 8的位置
    1924:	4c 89 f6             	mov    %r14,%rsi               # 把rsp + 16的地址作为第二个参数
    1927:	e8 f2 03 00 00       	call   1d1e <read_six_numbers> # 读取了六个数字 rsp + 16 20 24 28 32 36放在这六个位置
    192c:	4d 89 f4             	mov    %r14,%r12               # r12中放 rsp + 16的地址
    192f:	41 bf 01 00 00 00    	mov    $0x1,%r15d              # r15 = 1
    1935:	4d 89 f5             	mov    %r14,%r13               # r13中放 rsp + 16的地址
    1938:	e9 c6 00 00 00       	jmp    1a03 <phase_6+0x10b>    # 跳转
    193d:	e8 84 03 00 00       	call   1cc6 <explode_bomb>
    1942:	e9 ce 00 00 00       	jmp    1a15 <phase_6+0x11d>
    1947:	48 83 c3 01          	add    $0x1,%rbx               # rbx刚刚为1,这里就是作为一个循环控制变量 ++index（第二层循环）
    194b:	83 fb 05             	cmp    $0x5,%ebx               # 和5比较
    194e:	0f 8f a7 00 00 00    	jg     19fb <phase_6+0x103>    # 如果大于5跳转
    1954:	41 8b 44 9d 00       	mov    0x0(%r13,%rbx,4),%eax   # r15小于5的情况：eax中存放 *(rsp + 4 * index)
    1959:	39 45 00             	cmp    %eax,0x0(%rbp)          # 和首元素做比较
    195c:	75 e9                	jne    1947 <phase_6+0x4f>     # 不相等跳转，相等直接爆炸
    195e:	e8 63 03 00 00       	call   1cc6 <explode_bomb>
    1963:	eb e2                	jmp    1947 <phase_6+0x4f>
    1965:	48 8b 54 24 08       	mov    0x8(%rsp),%rdx          # 至此输入检查已经结束,rdx = rsp + 16（不要想错了，这里存放的值是rsp + 16）
    196a:	48 83 c2 18          	add    $0x18,%rdx              # rdx = rsp + 36
    196e:	b9 07 00 00 00       	mov    $0x7,%ecx               # rcx = 7
    1973:	89 c8                	mov    %ecx,%eax               # eax = 7
    1975:	41 2b 04 24          	sub    (%r12),%eax             # eax为7减去数组中的元素
    1979:	41 89 04 24          	mov    %eax,(%r12)             # 再把这个减了之后的值加载回去
    197d:	49 83 c4 04          	add    $0x4,%r12               # 下一个数字
    1981:	4c 39 e2             	cmp    %r12,%rdx               # 检查终止条件
    1984:	75 ed                	jne    1973 <phase_6+0x7b>
    1986:	be 00 00 00 00       	mov    $0x0,%esi               # 现在输入的每个数字都成了它对于7的补 rsi = 0，假设输入2 6 1 5 4 3 此时的值就是 5 1 6 2 3 4 rsi = 0
    198b:	8b 4c b4 10          	mov    0x10(%rsp,%rsi,4),%ecx  # rcx = *(rsp + 16 + 4 * rsi) 为数组的第一个值
    198f:	b8 01 00 00 00       	mov    $0x1,%eax               # eax = 1
    1994:	48 8d 15 75 38 00 00 	lea    0x3875(%rip),%rdx       # gdb查看内存这里就是把一个链表的node1的地址加载给了rdx,尝试用gdb去查看链表的具体结构，大概就是结构体{value + key + nextAddress}
    199b:	83 f9 01             	cmp    $0x1,%ecx               # rcx处的值和1比较
    199e:	7e 0b                	jle    19ab <phase_6+0xb3>     # 小于等于1就跳转
    19a0:	48 8b 52 08          	mov    0x8(%rdx),%rdx          # rdx此时应该为节点指向的节点的地址
    19a4:	83 c0 01             	add    $0x1,%eax               # ++eax
    19a7:	39 c8                	cmp    %ecx,%eax               # rcx和 eax比较
    19a9:	75 f5                	jne    19a0 <phase_6+0xa8>     # 不相等跳转，直到数组的第一个值和链表第一个节点的值相等就跳转
    19ab:	48 89 54 f4 30       	mov    %rdx,0x30(%rsp,%rsi,8)  # *(rsp + 48 + 8 * rsi) = rdx 把这个地址存放在stack上面
    19b0:	48 83 c6 01          	add    $0x1,%rsi               # ++rsi
    19b4:	48 83 fe 06          	cmp    $0x6,%rsi               # 循环终止条件
    19b8:	75 d1                	jne    198b <phase_6+0x93>     # 不相等继续
    19ba:	48 8b 5c 24 30       	mov    0x30(%rsp),%rbx         # 现在我们已经把按照输入数字顺序节点指向的地址放在了栈上（人话？）rbx为第一个地址
    19bf:	48 8b 44 24 38       	mov    0x38(%rsp),%rax         # rax是第二个地址
    19c4:	48 89 43 08          	mov    %rax,0x8(%rbx)          # 以下就是把链表按照我们输入的顺序连接在一起，看不明白就画图
    19c8:	48 8b 54 24 40       	mov    0x40(%rsp),%rdx
    19cd:	48 89 50 08          	mov    %rdx,0x8(%rax)
    19d1:	48 8b 44 24 48       	mov    0x48(%rsp),%rax
    19d6:	48 89 42 08          	mov    %rax,0x8(%rdx)
    19da:	48 8b 54 24 50       	mov    0x50(%rsp),%rdx
    19df:	48 89 50 08          	mov    %rdx,0x8(%rax)
    19e3:	48 8b 44 24 58       	mov    0x58(%rsp),%rax
    19e8:	48 89 42 08          	mov    %rax,0x8(%rdx)
    19ec:	48 c7 40 08 00 00 00 	movq   $0x0,0x8(%rax)          # 0就是null节点
    19f3:	00 
    19f4:	bd 05 00 00 00       	mov    $0x5,%ebp                # rbp = 5
    19f9:	eb 35                	jmp    1a30 <phase_6+0x138>     # 连接完了之后跳转
    19fb:	49 83 c7 01          	add    $0x1,%r15                # 这是应该是第一层循环
    19ff:	49 83 c6 04          	add    $0x4,%r14                # 下一个
    1a03:	4c 89 f5             	mov    %r14,%rbp                # 在成功读取六个数字之后跳转到这里，rbp存放rsp + 16地址（第一次）
    1a06:	41 8b 06             	mov    (%r14),%eax              # 读取的第一个数字
    1a09:	83 e8 01             	sub    $0x1,%eax                # 读取的数字-1
    1a0c:	83 f8 05             	cmp    $0x5,%eax                # 和5作比较
    1a0f:	0f 87 28 ff ff ff    	ja     193d <phase_6+0x45>      # 大于5爆炸（这意味着不能输入大于6的数字）
    1a15:	41 83 ff 05          	cmp    $0x5,%r15d               # r15刚刚赋值为1,现在和5作比较
    1a19:	0f 8f 46 ff ff ff    	jg     1965 <phase_6+0x6d>      # 大于5跳转（到这里为止，经过了一个类似于冒泡排序的比较，这意味着我们输入的数字不能有重复的也不能大于6）
    1a1f:	4c 89 fb             	mov    %r15,%rbx                # rbx = 1   
    1a22:	e9 2d ff ff ff       	jmp    1954 <phase_6+0x5c>      
    1a27:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
    1a2b:	83 ed 01             	sub    $0x1,%ebp
    1a2e:	74 11                	je     1a41 <phase_6+0x149>		
    1a30:	48 8b 43 08          	mov    0x8(%rbx),%rax           # 连接之后在这里 
    1a34:	8b 00                	mov    (%rax),%eax
    1a36:	39 03                	cmp    %eax,(%rbx)
    1a38:	7d ed                	jge    1a27 <phase_6+0x12f>		# 也就是说链表必须是递增还是递减的一个顺序？
    1a3a:	e8 87 02 00 00       	call   1cc6 <explode_bomb>
    1a3f:	eb e6                	jmp    1a27 <phase_6+0x12f>
    1a41:	48 8b 44 24 68       	mov    0x68(%rsp),%rax
    1a46:	64 48 2b 04 25 28 00 	sub    %fs:0x28,%rax
    1a4d:	00 00 
    1a4f:	75 0f                	jne    1a60 <phase_6+0x168>
    1a51:	48 83 c4 78          	add    $0x78,%rsp
    1a55:	5b                   	pop    %rbx
    1a56:	5d                   	pop    %rbp
    1a57:	41 5c                	pop    %r12
    1a59:	41 5d                	pop    %r13
    1a5b:	41 5e                	pop    %r14
    1a5d:	41 5f                	pop    %r15
    1a5f:	c3                   	ret    
    1a60:	e8 eb f7 ff ff       	call   1250 <__stack_chk_fail@plt>
```

不容易，终于写完了。

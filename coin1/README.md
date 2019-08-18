# coin1
這題搞了3天 binary search 就是寫不好 氣死
```=
---------------------------------------------------
-              Shall we play a game?              -
---------------------------------------------------

You have given some gold coins in your hand
however, there is one counterfeit coin among them
counterfeit coin looks exactly same as real coin
however, its weight is different from real one
real coin weighs 10, counterfeit coin weighes 9
help me to find the counterfeit coin with a scale
if you find 100 counterfeit coins, you will get reward :)
FYI, you have 60 seconds.

- How to play -
1. you get a number of coins (N) and number of chances (C)
2. then you specify a set of index numbers of coins to be weighed
3. you get the weight information
4. 2~3 repeats C time, then you give the answer

- Example -
[Server] N=4 C=2 	# find counterfeit among 4 coins with 2 trial
[Client] 0 1 		# weigh first and second coin
[Server] 20			# scale result : 20
[Client] 3			# weigh fourth coin
[Server] 10			# scale result : 10
[Client] 2 			# counterfeit coin is third!
[Server] Correct!

- Ready? starting in 3 sec... -

N=974 C=10
time expired! bye!
```
script
```=
from pwn import *
import time
context.log_level = 'DEBUG'
def chk(s,n,c):
    left=s
    right=n

    for i in range(c):
        mid=(left+right)/2
        result=auto(left,mid)
        if(result==0):

            left=mid

        if(result==1):

            right=mid

    # r.interactive()
    r.sendline(str(left))
    r.recv()
def auto(s,n):
    send=" "
    for i in range(s,n):
        send+=str(i)
        send+=' '
    r.sendline(send)
    re=r.recv()
    # if(re=='format error\n'):
        # r.interactive()
    # if(c==0):
        # r.sendline(int(n))
    if((int(re))%10==0):
        # print '1'
        # log.info('1')
        # auto(n/2,n,c-1)
        return 0;
    else:
        # auto(0,n/2,c-1)
        return 1;
    # log.info(re)
r=remote('pwnable.kr',9007)
r.read()
time.sleep(4)
for i in range(100):
    str0=r.recv()
    # log.info(str)
    str0=str0.strip()
    log.info(str0)
    str1=str0.split(' ')[0]
    # log.info(str1)
    str2=str0.split(' ')[1]
    # log.info(str2)
    n=str1.split('=')[1]
    c=str2.split('=')[1]
    # log.info(n)
    # log.info(c)
    chk(0,int(n),int(c))

```

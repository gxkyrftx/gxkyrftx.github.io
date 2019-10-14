---
layout:     post
title:      rootersctf-writeup
subtitle:   ctf题解
date:       2019-10-13
author:     gxkyrftx
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - ctf
---
# crypto

## 1 rsa

### 1.1 题目

```python
#!/usr/bin/env python3

from Crypto.Util.number import *
import gmpy2, binascii
from secret import flag

p = getPrime(128)
q = getPrime(128)

n = p*q
e = 65537

encryptedFlag = bytes_to_long(flag)
encryptedFlag = pow(encryptedFlag, e, n)
encryptedFlag = binascii.hexlify(long_to_bytes(encryptedFlag))

file = open("flag.enc", 'w')

file.write("ciphertext: {}\nn: {}\ne: {}".format(encryptedFlag, str(n), str(e)))

file.close()

```

输出文件flag.enc

```python
ciphertext: b'a0d8e768fa9d7238a6974b4a54073165fede084494d52b2ed600e6d0e77c1113'
n: 76277366118709104496037524736448350894293924248428417186334555697457434498837
e: 65537
```

标准的rsa，n的位数很小，直接分解n就行

### 1.2 求解

```python
#!coding:utf-8
#RSA
import libnum

p =  244117596642286059282649228191796982671
q = 312461564294690980358123469116445272347
n = p * q
e = 65537
c = 0xa0d8e768fa9d7238a6974b4a54073165fede084494d52b2ed600e6d0e77c1113
phi = (p - 1) * (q - 1)
d = libnum.modular.invmod(e, phi)
m = libnum.n2s(pow(c, d, n)) 
print m
```

## 2 Really Silly Algorithm LIBrary(unsolved)

### 2.1 题目

```python
#!/usr/bin/env python3

from Crypto.Util.number import *
import gmpy2, binascii
from sillyLibrary import rsalib_p, rsalib_q
from secret import flag

n = rsalib_p*rsalib_q

e = 0x10001

ciphertext = binascii.hexlify(long_to_bytes(pow(bytes_to_long(flag), e, n)))

file = open('flag.enc', 'w')

file.write("ciphertext: {}\nn: {}\ne: ".format(ciphertext, str(n), str(e)))
```



```c
ciphertext: 014107b18849e23cc0494a1f32d2176a0c0a497e2ad35d054941716d6a60c5be5656b369e0e4bba72fbcaba4586bc0cd352d4da34023b5a8
n: 11127212863544389237262565582342312793444654886136310133705565382574067475157051952050355000097126157942244686899397157941082263117851
e: 0x10001
```

### 2.2 求解



## 3 Digene

One of the most guessing challenge. We spend a lot of time trying to googling the correct cipher. Trying to googling about digene, turn out it is a digestive medicine for stomach. So we try to search Italian cure digestive stomach and found out about Amari. Trying to google Amari Italian, I found an article in wikipedia about Amaro. Reading through the wiki, I found the variant of Amaro liqueur called fernet, which is seems familiar to us. We realized that it has the same name with fernet encryption. Using python, We solve the challenge.

### 3.1 题目

```python
gAAAAABdnF6L7Gb1WxFUr4AVSs3Lg0lHetYDwxJvo84WN9DZd2_UASlX26XhPuFgIFs5v54yzxAbmQaZet9tOP-__y46eLqW5OeyLNlKlRpX_UfMm-aDLAM5p-DrBEBK_IzH_2kXJxc3

eH9Yta38cisI5gPZiNA3C6yKRRqqEy-K9Q9ZwfF5r1k=
```

首先根据题目描述

```
Challenge name : Digene 
Challenge Description : I wish there was something Italian to cure me! 
```

查看Digene ，然后通过google发现相关的一种描述digestive medicine,然后换关键字继续搜索，这次关键字为Italian ，digestive stomach。然后找到了Amari，根据谷歌Amari的结果，发现了关键词Amaro，然后在维基百科上看到了，有一种名为Amaro liqueur的东西，叫做fernet。好了这是一种常见的加密算法，AES-CBC 128。然后可以求解了

### 3.2 求解

```python
gxk@gxk-virtual-machine:~$ python
Python 2.7.12 (default, Nov 12 2018, 14:36:49) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from cryptography.fernet import Fernet
>>> c=b"gAAAAABdnF6L7Gb1WxFUr4AVSs3Lg0lHetYDwxJvo84WN9DZd2_UASlX26XhPuFgIFs5v54yzxAbmQaZet9tOP-__y46eLqW5OeyLNlKlRpX_UfMm-aDLAM5p-DrBEBK_IzH_2kXJxc3"
>>> k=b"eH9Yta38cisI5gPZiNA3C6yKRRqqEy-K9Q9ZwfF5r1k="
>>> cipher=Fernet(k)
>>> cipher.decrypt(c)
'rooters{d!g3st!f_th4t_d!g3sts_y0ur_m3ss4g3s}ctf'
>>> 
```

# reverse

## 1 easy_re

### 1.1 题目

#### 1.1.1 main()

```c
signed __int64 __fastcall main(signed int a1, char **a2, char **a3)
{
...
  v3 = time(0LL);
  srand(v3);
  if ( a1 > 1 )
  {
    v5 = strlen(a2[1]);
    if ( sub_55DB96D93273((__int64)a2[1], v5) )
    {
      puts("Correct Password!!!!!!!");
      result = 0LL;
    }
...
```

#### 1.1.2 sub_55DB96D93273()

```c
bool __fastcall sub_55DB96D93273(__int64 a1, unsigned __int64 a2)
{
...
  v3 = 0;
  for ( i = 0; a2 > i; ++i )
    v3 |= *((char *)off_55DB96D963E0 + i) ^ a2 ^ *(char *)(i + a1);
  return v3 == 0;
}
```

#### 1.1.3 off_55DB96D963E0

这是一段长为26的字符，内容如下

```
[0x54,0x49,0x49,0x52,0x43,0x54,0x55,0x5D,0x17,0x52,0x79,0x51,0x12,0x55,0x79,0x43,0x47,0x5C,0x5F,0x79,0x4F,0x79,0x52,0x16,0x4A,0x42,0x79,0x5F,0x47,0x79,0x79,0x1C,0x76,0x79,0x5B,0x45,0x52,0x40,0x00]

```

逻辑很清楚，最后v3为0，就可以满足条件，使用z3求解

### 1.2 求解

```python
from z3 import *
s = Solver()
n = 26
X = [BitVec('x_%s' % i, 8) for i in range(n)]

sn=[0x54,0x49,0x49,0x52,0x43,0x54,0x55,0x5D,0x17,0x52,0x79,0x51,0x12,0x55,0x79,0x43,0x47,0x5C,0x5F,0x79,0x4F,0x79,0x52,0x16,0x4A,0x42,0x79,0x5F,0x47,0x79,0x79,0x1C,0x76,0x79,0x5B,0x45,0x52,0x40,0x00]

flag="rooters{"+"x"*n+"}ctf"
result=0

for i in range(8):
    result|=sn[i]^ (n+12) ^ ord(flag[i])
for i in range(8,n+8):
	result|=sn[i]^  (n+12) ^ X[i-8]
for i in range(n+8,n+12):
	result|=sn[i]^ (n+12) ^ ord(flag[i])
s.add(result==0)

if s.check() == sat:
    flag = ""
    m = s.model()
    for i in range(n):
        flag += chr(m[X[i]].as_long())
    print(flag)
else:
    print("unsat")

```

## 
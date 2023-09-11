1.RSA
-

1.已知n，c，e

```
import gmpy2
import libnum
from Crypto.Util.number import *
from binascii import a2b_hex, b2a_hex

flag = "*****************"

p = 319576316814478949870590164193048041239
q = 275127860351348928173285174381581152299
c = 1323932973882505127567997561415668792873249397183695004797826407988557497789
e = 65537
n = p * q
phi = (p - 1) * (q - 1)
d = gmpy2.invert(e, phi)
m = pow(c, d, n)
print(libnum.n2s(int(m)))
```

分解n：
<http://factordb.com/>

<br>
有时上面这种代码运行后会出现\x形式的字符串，如果出现这样的问题则试试下面的：

```
from Crypto.Util.number import bytes_to_long, long_to_bytes, getPrime
p = 
q = 
c = 
e = 65537
d = pow(e, -1, (p-1)*(q-1))
m = long_to_bytes(pow(ct, d, p*q))
print(m)
```

 2 . pycryptodome 库破解哈希（PCTF2023 Breakfast club）
-
题目给出了一个txt：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/bc0cf6b5-6665-4cbf-8731-ce9f0d29280c)
<br>

在这里使用pycryptodome对每一行进行破解：
```
from Crypto.Hash import SHA, SHA1, MD2, MD4, MD5, SHA224, SHA256, SHA384, SHA512, SHA3_224, SHA3_256, SHA3_384, SHA3_512, TupleHash128, TupleHash256, BLAKE2s, BLAKE2b
algos = [SHA, SHA1, MD2, MD4, MD5, SHA224, SHA256, SHA384, SHA512, SHA3_224, SHA3_256, SHA3_384, SHA3_512, TupleHash128, TupleHash256, BLAKE2s, BLAKE2b]
import string
with open("D:/bluewhale-oj/BreakfastPasswords.txt", "r") as f:
    hashes = f.readlines()

flag = ""
for i in range(len(hashes)):
    algo, hash = hashes[i].split(" ")
    algo = algos[i]
    for char in string.printable:
        test_hash = algo.new()
        test_hash.update(char.encode())
        if test_hash.hexdigest() in hash:
            flag += char

print(flag)
```

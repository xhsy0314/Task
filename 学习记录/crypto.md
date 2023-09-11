RSA
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

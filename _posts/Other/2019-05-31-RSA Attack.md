---
layout: post
tag: Other
author: aiQG_
---


1. 爆破出n的因数, p/q相差很大或很小 可以分解n

2. 低模指数e=3, 已知m高位和n,c 可以求出完整明文
```python
# sage
n = 
e = 3
c = 
m = #完整的m
k = #未知的位数
PR.<x> = PolynomialRing(Zmod(n))
f = (x + m)^e-c
x0 = f.small_roots(X=2^kbits, beta=1)[0]
#明文为m+x0
```

2. 已知p（或q）高位, 已知n,c,e 可求出p,q
```python
#sage
n = 
e = 
p = #完整的p
pk = #原p的位数
k = #p未知的位数
pbar = p_fake & (2^pbits-2^kbits)
#
PR.<x> = PolynomialRing(Zmod(n))
f = x + pbar
x0 = f.small_roots(X=2^kbits, beta=0.4)[0]  
print x0 + pbar #计算出来的p
```

3. 低模指数e=3, 已知n, c和私钥d的低位(在模指数低的情况下，d高位的一半可以认为已知)
```python
#sage
def partial_p(p0, kbits, n):
    PR.<x> = PolynomialRing(Zmod(n))
    nbits = n.nbits()

    f = 2^kbits*x + p0
    f = f.monic()
    roots = f.small_roots(X=2^(nbits//2-kbits), beta=0.3)  # find root < 2^(nbits//2-kbits) with factor >= n^0.3
    if roots:
        x0 = roots[0]
        p = gcd(2^kbits*x0 + p0, n)
        return ZZ(p)

def find_p(d0, kbits, e, n):
    X = var('X')

    for k in xrange(1, e+1):
        results = solve_mod([e*d0*X - k*X*(n-X+1) + k*n == X], 2^kbits)
        for x in results:
            p0 = ZZ(x[0])
            p = partial_p(p0, kbits, n)
            if p:
                return p


if __name__ == '__main__':
    n =
    e = 3
    d = 

    beta = 0.5
    epsilon = beta^2/7
    nbits = n.nbits() #获取n的位数
    kbits = floor(nbits*(beta^2+epsilon)) #已知部分d的位数 一般是正常d的一半
    d0 = d & (2^kbits-1) #已知的部分d 
    print "lower %d bits (of %d bits) is given" % (kbits, nbits)

    p = find_p(d0, kbits, e, n)
    print "found p: %d" % p
    q = n//p
    print d
    print inverse_mod(e, (p-1)*(q-1))

```

4. e太小m也不大的情况下，c=pow(m,e,n)=pow(m,e)，gmpy2的iroot开根爆破

5. 在e太大(d小)的情况下($ d < \cfrac{1}{3}N^{\frac{1}{4} }\ mod\ n \equiv m^{(re_1+se_2)}\ mod\ n \equiv m\ mod\ n$) 
```python 
import [RSAwienerHacker](https://github.com/pablocelayes/rsa-wiener-attack#rsa-wiener-attack)
RSAwienerHacker.hack_RSA(e,n) #得到d
```

6. 相同的N不同的e(私钥d)，加密同一明文
$re_1 + se_2 = 1\ mod\ n$求出$r$和$s$
$c_1^rc_2^s \equiv m^{re_1}m^{se_2}\  mod\  n \equiv m^{(re_1+se_2)}\ mod\ n \equiv m\ mod\ n$ 求出$m$
```python
# -*- coding:utf-8 
#py3
from libnum import n2s, s2n
from gmpy2 import invert
n= raw_input( "Input the n: ")
e1 = raw_input( " Input the e1:")
e1 = raw_input ("Input the e2:")
def egcd(a, b) :
    if a == O:
        return (b , 0, 1)
    else:
        g,y,x=egcd(b%a,a)
        return (g, x - (b/1a)*y,y)
fo1 = open(' flag.enc1','rb' )
fo2 = open(' flag.enc2', 'rb')
datafo1 = fo1.read( )
datafo2 = fo2.read( )
c1 = s2n(datafo1)
c2 = s2n(datafo2)
fo1.close()
fo2.close()
c2 = invert(c2, n)
S = egcd(e1, e2)
s1 = s[1]
s2 = s[2]
s2 = - s2
m = pow(c1, s1, n)*pow(c2, s2, n) % n
print n2s(m)
```

8. 低加密指数广播攻击：相同的e，对相同的明文m加密了N(至少为e)次
```python 
import gmpy
def my_parse_number(number):
    string = "%x" % number
    erg = []
    while string != '':
        erg = erg + [chr(int(string[:2], 16))]
        string = string[2:]
    return ''.join(erg)
def e_gcd(a, b):
    x,y = 0, 1
    lastx, lasty = 1, 0
    while b:
        a, (q, b) = b, divmod(a,b)
        x, lastx = lastx-q*x, x
        y, lasty = lasty-q*y, y
    return (lastx, lasty, a)
def chinese_remainder_theorem(items):
  N = 1
  for a, n in items:
    N *= n
  result = 0
  for a, n in items:
    m = N/n
    r, s, d = e_gcd(n, m)
    if d != 1:
      raise "Input not pairwise co-prime"
    result += a*s*m
  return result % N, N
e=
n=[,,,...]#
c=[,,,...]#
data=[]
for i in range(len(c)):
    data += [(c[i],n[i])]
x, n = chinese_remainder_theorem(data)
realnum = gmpy.mpz(x).root(e)[0].digits()
print my_parse_number(int(realnum))
```

7. 同一公钥加密的**具有线性关系的**两条明文（Related Message Attack）
```python
#import FranklinReiter
#?#import hastads
#sage
# c2=c1+r
franklinReiter(n,e,r,c1,c2)
#?#hastads(,,,,,)
```

8. $d<N^{0.292}$ 
```python
#import [boneh_durfee](https://github.com/mimoo/RSA-and-LLL-attacks/blob/master/boneh_durfee.sage)
example()
```

9. 两个n不互相素，可以gmpy2.gcd(n1,n2)得到p,q





$\infty .​$ 
[CTF-Crypto](https://github.com/ValarDragon/CTF-Crypto)
[CTF-RSA-tool](https://github.com/3summer/CTF-RSA-tool)
[总结](http://lanvnal.com/2018/07/28/RSA%E9%A2%98%E5%9E%8B%E6%80%BB%E7%BB%93/index.html)
[RsaCtfTool(成套)](https://github.com/Ganapati/RsaCtfTool)
[RSA-and-LLL-attacks](https://github.com/mimoo/RSA-and-LLL-attacks)
[this man TQL.github](https://github.com/mimoo/RSA-and-LLL-attacks)

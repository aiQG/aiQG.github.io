---
layout: post
tag: Note
author: aiQG_
---


openssl实现了大部分的加密
```shell
  -out #输出文件
  -in #输入文件

  - RSA
  	openssl genrsa -out 1.key 1024 #生成私钥对：RSA PRIVATE KEY
  	openssl rsa -in k.txt -pubout -out p.txt #取出公钥：PUBLIC KEY（文件中记录算法类型）
  openssl rsa -in k.txt -RSAPublicKey_out  -out p.txt #取出RSAPublicKey格式公钥：RSA PUBLIC KEY
```

快速使用: 

```shell
echo -n "Y0u_h4v3_4_Sm4rt_Br41n" | openssl sha1
# -n 表示不接收换行符
```

openssl生成的公私钥和一些信息是用ASN.1(RSAPublicKey)/PKCS#8(PUBLIC KEY)等编码格式来保存的，所以解base64并不能直接得到公私钥（得到DER编码格式）
可以使用在线平台解码 http://lapo.it/asn1js/
也可以用openssl的asn1parse命令查看
> asn1parse是一个诊断工具，可以解析ASN.1结构的密钥，证书等

---

生成RSA:

私钥：

```shell
openssl genrsa -out k.txt 50 #生成50位的私钥
openssl asn1parse -i -in k.txt #解码ASN.1
```


返回：
>0:d=0  hl=2 l=  55 cons: SEQUENCE 
>2:d=1  hl=2 l=   1 prim:  INTEGER           :00
>5:d=1  hl=2 l=   7 prim:  INTEGER           :03A20D44C84679
>14:d=1  hl=2 l=   3 prim:  INTEGER           :010001
>19:d=1  hl=2 l=   7 prim:  INTEGER           :021B398DFA6501
>28:d=1  hl=2 l=   4 prim:  INTEGER           :01F85261
>34:d=1  hl=2 l=   4 prim:  INTEGER           :01D81B19
>40:d=1  hl=2 l=   4 prim:  INTEGER           :015DE4E1
>46:d=1  hl=2 l=   3 prim:  INTEGER           :5435F1
>51:d=1  hl=2 l=   4 prim:  INTEGER           :E068D6


对应结构：
>RSAPrivateKey ::= SEQUENCE {
>versionVersion,
>modulusINTEGER, -- n
>publicExponentINTEGER, -- e
>privateExponentINTEGER, -- d
>prime1INTEGER, -- p
>prime2INTEGER, -- q
>exponent1INTEGER, -- d mod (p-1)
>exponent2INTEGER, -- d mod (q-1)
>coefficientINTEGER, -- (inverse of q) mod p
>otherPrimeInfosOtherPrimeInfos OPTIONAL
>}

---

记录算法类型的公钥：

```shell
openssl rsa -in k.txt -pubout -out p.txt #从私钥生成公钥，这里生成的公钥是
openssl asn1parse -i -in p.txt #分析ASN.1格式
```
返回：

>0:d=0  hl=2 l=  34 cons: SEQUENCE 
>2:d=1  hl=2 l=  13 cons:  SEQUENCE 
>4:d=2  hl=2 l=   9 prim:   OBJECT            :rsaEncryption 
>15:d=2  hl=2 l=   0 prim:   NULL 
>17:d=1  hl=2 l=  17 prim:  BIT STRING 

对应结构：
>SubjectPublicKeyInfo ::=SEQUENCE {
>  algorithmAlgorithmIdentifier{{SupportedAlgorithms}},
>  subjectPublicKeyBIT STRING
>}

这里偏移17的地方：```BIT STRING ```对应了结构中的```subjectPublicKeyBIT STRING```
这里还保存了加密算法类型

继续分析偏移17的地方：
```shell
openssl asn1parse -i -in p.txt -strparse 17
```
返回：
>0:d=0  hl=2 l=  14 cons: SEQUENCE 
>2:d=1  hl=2 l=   7 prim:  INTEGER           :03A20D44C84679
>11:d=1  hl=2 l=   3 prim:  INTEGER           :010001

对应结构：
>RSAPublicKey ::= SEQUENCE  {
>modulus INTEGER, -- n
>publicExponentINTEGER -- e
>}

---

不记录算法类型的公钥：

```shell
openssl rsa -in k.txt -RSAPublicKey_out  -out p.txt #生成公钥
openssl asn1parse -i -in p.txt #分析ASN.1格式
```
返回：
>0:d=0  hl=2 l=  14 cons: SEQUENCE 
2:d=1  hl=2 l=   7 prim:  INTEGER           :03A20D44C84679
11:d=1  hl=2 l=   3 prim:  INTEGER           :010001

对应格式：
> RSAPublicKey ::= SEQUENCE  {
>   modulus INTEGER, -- n
>   publicExponentINTEGER -- e
>  }

---

加密：
> openssl rsautl -encrypt  -inkey p.txt -pubin -in flag.txt -out enflag.txt

用公钥p.txt加密
/*
注意，如果生成的公钥开头为```-----BEGIN RSA PUBLIC KEY-----```开头（RSA公钥）(pkcs1公钥)
openssl似乎并不支持，会返回```unable to load Public Key```的错误
openssl支持```-----BEGIN PUBLIC KEY-----```的公钥（SSL公钥）(pkcs8公钥)
```shell
openssl rsa -pubin -in p.txt -RSAPublicKey_out -out p2.txt #pkcs8公钥转pkcs1公钥
openssl rsa -RSAPublicKey_in -in p2.txt -pubout -out p.txt #pkcs1公钥转换为pkcs8公钥
```
*/
//如果明文信息太长会出现```data too large for key size```的错误，这时应该生成更长的key。或者使用大文件加密方法。


解密：
> openssl rsautl -decrypt -inkey k.txt -in enflag.txt -out deflag.txt

如果私钥错误会返回```RSA operation error```

---
签名：
```shell
openssl rsautl -sign -inkey k.txt -in flag.txt -out sigflag.txt #用私钥签名
openssl rsautl -verify -inkey p.txt -pubin -in sigflag.txt -out checkflag.txt #用公钥验证签名
```
注意如果签名的是明文本身，验证出来的也是明文本身，所以一般对hash签名
一般采取先签名后加密的方法

---

生成加密的私钥
```shell
openssl genrsa -des -out k.txt 1000 #用DES加密（默认CBC模式）
openssl rsa -in k.txt -text #查看私钥信息
openssl rsa -in k.txt -out k.txt #删除密码
```

---

---

---

其他加密算法（DES为例）：

```shell
#DES
openssl des -in flag.txt  #可以发现密文前面都默认带有"Salted__"
#-base64 加密后或者解密前进行base64
#-K 指定密码 并且需要-vi指定初始化向量
#-k 指定密码 初始化向量可以自动生成
#-d 解密 如果密文格式不正确会报"bad magic number" //注意openssl并不只保存密文，还有初始化向量等数据 //一般解密用openssl enc -d -算法
#-salt 使用随机盐
#-S 指定盐
#-p 输出各参数
```

key的填充和盐有关
猜测密文格式应该是"Salted__"+随机或指定的盐+初始化向量+密文






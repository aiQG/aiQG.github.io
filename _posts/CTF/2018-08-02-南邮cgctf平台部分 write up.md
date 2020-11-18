---
layout: post
tag: CTF
author: aiQG_
---

[My_CSDN_Blog](blog.csdn.net/qq_42863361)
[CG-CTF](https://cgctf.x1ct34m.com/)
2018.7
>Welcome to http://aiqg.vip/
---------------------------
## web
 - md5 collision
看源代码
```php
<?php
$md51 = md5('QNKCDZO');
$a = @$_GET['a'];
$md52 = @md5($a);
if(isset($a)){
if ($a != 'QNKCDZO' && $md51 == $md52) {
    echo "nctf{*****************}";
} else {
    echo "false!!!";
}}
else{echo "please input a";}
?>
```
要满足md51==md52 并且 a!='QNKCDZO'
结合php的md5想到0e绕过

令a=s878926199a
（过md5后是0e开头）
构造
>http://chinalover.sinaapp.com/web19/?a=s878926199a

即可
nctf{md5_collision_is_easy}

---
- 签到2
从响应头可以看到字符串长度被限制在10个(网页源代码里也可以看到)
直接将源代码的10修改成11，输入zhimakaimen
nctf{follow_me_to_exploit}

---
·这题不是web
打开看到一奇怪的图片
![这里写图片描述](https://i.imgur.com/anihqQY.gif)
下载下来用文本打开
最后一行
nctf{photo_can_also_hid3_msg} 

---
- 层层递进
打开似乎看到一大堆广告
查看Network发现一个奇怪的404.html
![这里写图片描述](https://i.imgur.com/hNvovtZ.png)
进去看源码
![这里写图片描述](https://i.imgur.com/BfaxFyc.png)
nctf{this_is_a_fl4g}

---
- AAencode
js的aa加密 打开时一堆表情+乱码￼
![这里写图片描述](https://i.imgur.com/C0B1Z6z.png)
(若都是表情则省去下面一步 具体情况因人而异)
下载此页￼￼
![这里写图片描述](https://i.imgur.com/VceRIBM.png)
![这里写图片描述](https://i.imgur.com/V828vlE.png)
随便找一个网站解去
![这里写图片描述](https://i.imgur.com/LOSwRbW.png)
nctf{javascript_aaencode}

---
- 单身二十年
点击链接很明显发生了一个跳转
直接下载链接地址
![这里写图片描述](https://i.imgur.com/VGsDDxU.png)
![这里写图片描述](https://i.imgur.com/YFjCJ9B.png)
搞定
nctf{yougotit_script_now}

---
- php decode
找个在线运行php的网站
复制代码
将eval改成echo即可
nctf{gzip_base64_hhhhhh}

---
- 文件包含
提示了LFI 
网页输出show.php 也就是输出file变量的文件
再想办法爆出文件源代码
Tip :https://www.cnblogs.com/wh4am1/p/6542398.html
用base64编码index.php后输出
令file=php://filter/read=convert.base64-encode/resource=index.php
再拿去解码￼
![这里写图片描述](https://i.imgur.com/3oWLbaB.png)
nctf{edulcni_elif_lacol_si_siht}

---
- 单身一百年也没用
点开链接 发现响应头里藏着flag
![这里写图片描述](https://i.imgur.com/w9h7lOf.png)
nctf{this_is_302_redirect}

---
- cookie
点开链接发现请求头
Cookie:Login=0
根据tip将0改成1￼
![这里写图片描述](https://i.imgur.com/R5ytPx8.png)

nctf{cookie_is_different_from_session}


---
- MYSQL
 科普了robots协议
进入robots.txt看看(别少了s别少了s别少了s)￼
![这里写图片描述](https://i.imgur.com/PcyHbWe.png)
发现源代码
发现1024为判断关键
分析
GET到的id不能为1024
保留整数后id为1024￼
![这里写图片描述](https://i.imgur.com/9qjxufJ.png)
nctf{query_in_mysql}

---
- sql injection 3
![这里写图片描述](https://i.imgur.com/nPcIpKB.png)
看到id=1
就想试试234...
![id=2](https://i.imgur.com/soDKbpw.png)
![id=3](https://i.imgur.com/jwZ5Ree.png)
得到提示 gbk
输入id='
('=%27; #=%23; 空格=%20)￼
![id='](https://i.imgur.com/Vwseku9.png)
被转义掉了
构造宽字节,用来闭合单引号(低位为%27即可 )
令id=%ee%27
出现warning 说明思路正确
最后加上#（%23）把后面的语句注释掉 运行正常￼
![这里写图片描述](https://i.imgur.com/p35FHoF.png)
接下来就是union select了
先找到返回的是哪个字段 (1,2,3,666什么的多试几次)￼
![这里写图片描述](https://i.imgur.com/DmzQ1Nx.png)
将该字段替换成要查找的内容..

[Union select 手工注入](https://www.cnblogs.com/ermei/p/7929501.html)
[关于information_schema](https://www.cnblogs.com/lqw4/p/4831463.html)

`information_schema`保存了数据库的信息 
`group_concat() `用于打印

查看当前库
>id=%ee%27union select 1,database()%23

查看库中所有表
>id=%ee%27union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()%23￼

![这里写图片描述](https://i.imgur.com/BvjXKJ0.png)
可以看到一共五个库,之前id=3时返回的"the fourth table"明显提示了flag在ctf4中
再看看ctf4中有哪些列
>id=%ee%27union select 1,group_concat(column_name) from information_schema.columns where table_name=0x63746634%23

(其中0x63746634是ctf4的16进制gbk编码 [转码网站](http://www.mytju.com/classcode/tools/encode_gb2312.asp))
![这里写图片描述](https://i.imgur.com/HZQnbIJ.png)
flag！
 
在ctf4中搜索flag
令
>id=%ee%27union select 1,flag from ctf4%23

![这里写图片描述](https://i.imgur.com/I4e9unp.png)
总的来说就是 爆库 爆表 爆字段(列)
nctf{gbk_3sqli}
 
说说踩的坑: 之前用
>id=%ee%27 union select 1,table_name from information_schema.tables where table_schema=database()%23

只能爆出第一个ctf 然后在里面转得各种头晕 还找到了admin和一串密文(解码后是njupt)

---
- /x00
分析源码

	```php
	if (isset ($_GET['nctf'])) {
		if (@ereg ("^[1-9]+$", $_GET['nctf']) === FALSE)
			echo '必须输入数字才行';
        else if (strpos ($_GET['nctf'], '#biubiubiu') !== FALSE)   
            die('Flag: '.$flag);
        else
            echo '骚年，继续努力吧啊~';
    }
    ```
    
必须输入数字  必须包含#biubiubiu
题目提示了00截断
令nctf=666%00%23biubiubiu (%23为#)￼
![这里写图片描述](https://i.imgur.com/RtoWePN.png)
nctf{use_00_to_jieduan}

---
- bypass again
一看到源码有md5 a!b之类的 就想到0e
```php
if (isset($_GET['a']) and isset($_GET['b'])) {
	if ($_GET['a'] != $_GET['b'])
		if (md5($_GET['a']) == md5($_GET['b']))
			die('Flag: '.$flag);
		else
			print 'Wrong.';
}
```
网上找两段md5后为0e开头的字符串 赋值即可
nctf{php_is_so_cool}
也可令ab为数组(md5不处理数组)

---
- 变量覆盖
有源码
关键部分
```php
<?php if ($_SERVER["REQUEST_METHOD"] == "POST") { ?>
	<?php
    extract($_POST);
    if ($pass == $thepassword_123) { ?>
	    <div class="alert alert-success">
	        <code><?php echo $theflag; ?></code>
        </div>
    <?php } ?>
<?php } ?>
```
POST方式提交 
pass==thepassword_123输出flag
![这里写图片描述](https://i.imgur.com/3FUi9Rq.png)
nctf{bian_liang_fu_gai!}

---
- sql注入1
源码
```php
<?php
if($_POST[user] && $_POST[pass]) {
    mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = trim($_POST[user]);
  $pass = md5(trim($_POST[pass]));
  $sql="select user from ctf where (user='".$user."') and (pw='".$pass."')";
    echo '</br>'.$sql;
  $query = mysql_fetch_array(mysql_query($sql));
  if($query[user]=="admin") {
      echo "<p>Logged in! flag:******************** </p>";
  }
  if($query[user] != "admin") {
    echo("<p>You are not admin!</p>");
  }
}
echo $query[user];
?>
```
只需要user==admin 
令user=admin’)#￼
![这里写图片描述](https://i.imgur.com/1x5HGk4.png)
nctf{ni_ye_hui_sql?}

---
- pass check
```php
<?php
$pass=@$_POST['pass'];
$pass1=***********;//被隐藏起来的密码
if(isset($pass))
{
if(@!strcmp($pass,$pass1)){
echo "flag:nctf{*}";
}else{
echo "the pass is wrong!";
}
}else{
echo "please input pass!";
}
?>
```
要满足!strcmp($pass,$pass1)
让strcmp 返回0 即可(比较非字符串时会返回0 )￼
![这里写图片描述](https://i.imgur.com/ywUg1Vm.png)
nctf{strcmp_is_n0t_3afe}

---
- 起名字真难
```php
<?php
 function noother_says_correct($number)
{
	$one = ord('1');
	$nine = ord('9');
	for ($i = 0; $i < strlen($number); $i++)
	{   
		$digit = ord($number{$i});
		if ( ($digit >= $one) && ($digit <= $nine) )
		{
			return false;
		}
	}
	return $number == '54975581388';
}
$flag='*******';
if(noother_says_correct($_GET['key']))
    echo $flag;
else 
    echo 'access denied';
?>
```
number 任意一位都不能是1-9的数字
number =='54975581388'
那么只能是0和字母
而且是个整数
那么就是十六进制数了￼
![这里写图片描述](https://i.imgur.com/7lpvDfo.png)
nctf{follow_your_dream}

---
- 综合题
jother编码
放到控制台运行返回
1bc29b36f623ba82aaf6724fd3b16718.php
访问此文件的响应头有tip:history of bash
百度一下 是bash的history文件
进入发现flag 下载即可￼
![这里写图片描述](https://i.imgur.com/GmMda6D.png)
nctf{bash_history_means_what}

---
- sql注入2
```php
<?php
if($_POST[user] && $_POST[pass]) {
   mysql_connect(SAE_MYSQL_HOST_M . ':' . SAE_MYSQL_PORT,SAE_MYSQL_USER,SAE_MYSQL_PASS);
  mysql_select_db(SAE_MYSQL_DB);
  $user = $_POST[user];
  $pass = md5($_POST[pass]);
  $query = @mysql_fetch_array(mysql_query("select pw from ctf where user='$user'"));
  if (($query[pw]) && (!strcasecmp($pass, $query[pw]))) {
      echo "<p>Logged in! Key: ntcf{**************} </p>";
  }
  else {
    echo("<p>Log in failure!</p>");
  }
}
?>
```
先封闭单引号
然后搜索md5后的密码
ntcf{union_select_is_wtf}

---

#逆向

- ReadAsm2
开始有三个mov到了内存 
显然是三个参数
分别是
input 28 1
其中1是计数变量
读两遍汇编基本就知道函数在干什么了
![这里写图片描述](https://i.imgur.com/Mq4MPVh.png)
flag{read_asm_is_the_basic}

---
- py交易
.pyc文件
搜索py在线反编译工具
```python2
import base64

def encode(message):
    s = ''
    for i in message:
        x = ord(i) ^ 32
        x = x + 16
        s += chr(x)
    
    return base64.b64encode(s)

correct = 'XlNkVmtUI1MgXWBZXCFeKY+AaXNt'
flag = ''
print 'Input flag:'
flag = raw_input()
if encode(flag) == correct:
    print 'correct'
else:
    print 'wrong'
```
flag经过encode后是correct
反过来做一遍就好了
![这里写图片描述](https://i.imgur.com/Gu6VCML.png)

nctf{d3c0mpil1n9_PyC}

---
- WxyVM
丢到ida里F5()
![这里写图片描述](https://i.imgur.com/j34kFXb.png)
加密函数 sub_4005b6
![这里写图片描述](https://i.imgur.com/L4lTxb9.png)

输入被一个数组加密
数组的第n个元素选择运算方式（加减乘异或）
数组的第n+1个元素指定输入的哪一位被运算
数组的第n+2个元素是运算数
数组总长15000 每三个一组

加密完成后对比结果

ok分析完毕
找到0x1060到0x1060+24*4到data（24个4字节的数据）
手动录入（FF与00的填充不管，小端序）

然后导出15000字节的数组 (用winhex选择block)(从0x10c0到0x10c0+15000) (少导出十个字节让我头疼了一上午)
逆向
```c++
#include<iostream>
#include<fstream>
using namespace std;

int main()
{
	ifstream f15000("arr");
    char s24[]={0xC4,0x34,0x22,0xb1,
        0xd3,0x11,0x97,0x07,
        0xdb,0x37,0xc4,0x06,
        0x1d,0xfc,0x5b,0xed,
        0x98,0xdf,0x94,0xd8,
        0xb3,0x84,0xcc,0x08
    };
	char s15[15000];
	char ch;
    int j=0;
	while(f15000.get(ch))
	{
		s15[j]=ch;
        j++;
	}
	
	for(int i=14997;i>=0;i=i-3)
	{
		char v0=s15[i];
		char v3=s15[i+2];
		char r=s15[i+1];
		switch(v0)
		{
			case 1:
			s24[r]-=v3;
			break;
			
			case 2:
			s24[r]+=v3;
			break;

			case 3:
			s24[r]^=v3;
			break;

			case 4:
			s24[r]/=v3;
			break;

			case 5:
			s24[r]^=s24[s15[i+2]];
			break;
				
			default:
			continue;
		}
	}
	cout<<s24<<endl;
	return 0;
}
```
nctf{Embr4ce_Vm_j0in_R3}

---

- Single
//开始没注意到single啥意思..走了很多弯路......
丢到IDA里
![在这里插入图片描述](https://i.imgur.com/zRCBF4R.png)

发现三个子函数
```C++
void sub_40070E(char* input)
{
    int result;
    int i;
    if (strlen(input)>0x51) //input 超过0x51(81)个字符GG
    {
        exit(1);
    }
    
    for(i=0;;i++)
    {
        result = strlen(input);
        if(i>=result)   break;
        
        if(input[i]<=47||input[i]>57)//input[i]不符合GG
        {
            exit(1);
        }
    }
    
    return result;
}
```
```C++
void sub_40078B(char* input,char* with_input)  
{
    int result;
    int i;
    
    for(i=0;;i++)
    {
        result=strlen(input);
        if(i>=result)   break;
        
        if(input[i]!=48) 
        {
            if( !input[i] || with_input[i] )
            {
                exit(1);
            }
            with_input[i]=input[i]-48;  //with_input[i]为 0 的地方会被修改
        }
    }
}

```
第三个有三个检查函数
```C++
void sub_400AD4_1(char* with_input)
{
    for(int i=0;i<=8;i++)
    {
        char s[24]=0;//初始化s
        for(int j=0;j<=8;j++)
        {
            s[ with_input[9*i+j] ]+=1;  //s[ with_input[0~80] ]+=1
        }
        for(int k=1;k<=9;k++)  //s[1~9]要为1
        {
            if (s[k]!=1)
            {
                exit(1);
            }
        }
    }
}

void sub_400AD4_2(char* with_input)
{
    
    
    for(int i=0;i<=8;i++)
    {
        char s[24]=0;//初始化s
        for(int j=0;j<=8;j++)
        {
            s[ with_input[9*j+i] ]+=1; 
        }
        for(int k=1;k<=9;k++)  //s[1~9]要为1
        {
            if (s[k]!=1)
            {
                exit(1);
            }
        }
    }
}

void sub_400AD4_3(char* with_input)
{
    int v6=3;
    int v7=3;
    
    for(int i=0;i<=8;i++)
    {
        char s[24]=0;  //初始化s
        for(int j=v6-3;j<v6;j++)
        {
            for(int k=v7-3;k<v7;k++)
            {
                s[ with_input[9*j+k] ]+=1;
            }
        }
        for(int l=1;1<=9;l++)//s[1~9]要为1
        {
            if(s[l]!=1)
            {
                exit(1);
            }
        }
        if(v7==9)
        {
            v7=3;
            v6+=3;
        }
        else
        {
            v7+=3;
        }
    }
}
void sub_400AD4(char* with_input)
{
sub_400AD4_1(with_input);
sub_400AD4_2(with_input);
sub_400AD4_3(with_input);
}
```
看到第三个函数...甚至让我想无脑爆破...//然后发现爆破成本太高....
后来学习到single还有数独的意思....//当时我的内心是崩溃的
一个9*9的数独
sub_40070E//检查输入：不超过81个字符，只能输入1～9
sub_40078B//将输入填入数独表，如果填到原本有数字的地方就GG
sub_400AD4//检查有没有数字重复
sub_400AD4_1//检查每一行
sub_400AD4_2//检查每一列
sub_400AD4_3//检查每一宫

数独表就是那个unk_602080
![在这里插入图片描述](https://i.imgur.com/9CEwzjZ.png)
百度在线解数独...
![在这里插入图片描述](https://i.imgur.com/EAMKeVn.png)
灰色的数字是题目中的，在flag中灰色为0
横着一行行拼接成flag

---

- 480小时精通C++
//这题学到了很多东西...
丢到ida里一脸懵B:![在这里插入图片描述](https://i.imgur.com/j6iEq4m.png)
![在这里插入图片描述](https://i.imgur.com/WjTKbeA.png)
 
 //那么多的加密函数主函数里怎么没有调用？(v6 v7 v8 v9 v10的字符串不是flag)
 然后把v6~v10的字符串提出来用各种姿势跑了 StringEncryptFunction()    //然鹅并没有什么X用
 后来发现main里有一大段被nop掉了 (感谢五号机?)
 才知道这是叫Self-Modifying Code(SMC) (感谢人人人)
 >[浅析SMC技术](https://blog.csdn.net/whatday/article/details/7265931)

![在这里插入图片描述](https://i.imgur.com/EJwsVtD.png)

这样就需要把程序跑起来 让他自己还原自己了
然鹅在win上跑不了elf 所以在linux上用ida远程调试
[IDA远程调试(elf文件)设置](https://blog.csdn.net/qq_42863361/article/details/82927093)
![在这里插入图片描述](https://i.imgur.com/vyVdZVT.png)
ok 加密后的代码出来了 
![在这里插入图片描述](https://i.imgur.com/YIVIJR7.png)
让它跑完 StringEncryptFunction() flag就出来了
flag{N0body_c4n_Mast3r_c_p1us_pLus!}

---
- 你大概需要一个优秀的mac

三个加密函数 很容易整理出流程
![在这里插入图片描述](https://i.imgur.com/tgtHlDm.png)
最后和check里的unk_100000ED0比较
![在这里插入图片描述](https://i.imgur.com/JoJfYM3.png)
flag{I5_th1s_7he_PR1c3_I'M_PAyiNG_f0r_my_pA57_m1stAk35?}

---
- HomuraVM
//作为萌新的我表示...学到了好多...?

 丢到ida里 程序思路还算清楚.
 获取input(flag)---->加密---->字符串对比
//难的就是不知道怎么加密的
两堆字符被传入了sub_8DC//猜测是加密函数
//有个sub_8AA函数点进去发现是getpid getsid什么的 基本上可以确定是反调试
![在这里插入图片描述](https://i.imgur.com/uLXMeKN.png)
传入进来后按照每位减67的方式 跳转到dword_1048+deword_1048[v1]位置去执行代码(学习到了JUMPOUT居然还有这种用处...)
![在这里插入图片描述](https://i.imgur.com/8sxW0Eh.png)
dword_1048长度是59 可以发现里面存着一堆十六进制数: 0xFFFFF***(能猜测到是相关地址)
so 代码的位置就是dword_1048首地址 + 0xFFFFF*** = 0x00000*** (1048的‘1’被‘FFFFF’进位丢失了) 所以代码地址在0x048+0x***处 
然后到各个地址处看汇编(又复习了各种test cmp jz jnz) 可以总结出![在这里插入图片描述](https://i.imgur.com/t2t1dkF.png)其中bdb相当于什么也没做

再回到main里的两大堆字符 可以发现有大量重复的字符
(与代码联系起来 猜测和循环有关)
于是可以发现有大量重复的hv{aG}[ur]ov......MCh{mG}结构(中间夹着不同数量的a和r)
跑个脚本 给hv{aG}[ur]ov  MCh{mG}  a  v  h[ur]ov(仅出现在开头) 这五种结构各位各减67 看看对应了哪些代码
得到![在这里插入图片描述](https://i.imgur.com/9hqD2j0.png)这里把hv{aG}[ur]ov作为每个结构的开始 MCh{mG}作为结束ind是下标(对应的汇编是将input的偏移地址调整4位//感谢前辈的指点)
经过整理后发现开始的h[ur]ov不起作用 得到
```C++
void head()
{
	Num_1=ch2//ch2是input的下一个字符
}
//其中有多少个a就减Num_1多少
//其中有多少个r就加Num_1多少
void tail()
{
	ch2=ch1+Num_1-2*(ch1&Num_1);//ch1是input的当前字符
	ind++;
}
//最后得到的input每一个字符应该与main中v5向下33个变量相同
```
*思考过通过从对比字符串逆运算得到input
发现加密运算中有直接赋值的语句 所以看来是不可逆的了*
那就爆破
```python
code_v5=[27,114,17,118,8,74,126,5,55,124,31,88,104,7,112,7,49,108,4,47,4,105,54,77,127,8,80,12,109,28,127,80,29,96]
ch1=ord('}')
i=1#这里不清楚为什么要从code_v5[1]开始 导致flag从第三位开始
def anloop(a,r,ch):
    Num_1=ch-a+r
    ch=ch1+Num_1-2*(ch1 & Num_1)
    return ch
    for ch in range(0,128):#flag的内容一定都在ascii码里 所以上限是128
    ch2=anloop(3,0,ch); #第一个结构 a有3个 r有0个 ...
    if code_v5[i]==ch2:
        i+=1
        ch1=ch2#准备下一位
        print(chr(ch),end='')
        break
	for ch in range(0,128):
    ch2=anloop(3,3,ch);
    if code_v5[i]==ch2:
        i+=1
        ch1=ch2
        print(chr(ch),end='')
        break
        ...
```
由于加密函数是一位一位修改的 每次修改都基于上一位字符
从main得知 input字符串的第一位与flag的最后一位相同
所以能猜到input第一位是'}'
之后以
```python
for ch in range(0,128):
    ch2=anloop(a,r,ch);
    if code_v5[i]==ch2:
        i+=1
        ch1=ch2
        print(chr(ch),end='')
        break
```
为单位 把各个结构的a和r数量填上去....//学习使用了py...虽然还不是很懂...![在这里插入图片描述](https://i.imgur.com/UzqW3Tf.png)
这里从code_v5[1]开始 导致flag从第三位开始(不清楚为什么要从1开始)
flag{D3v1L_H0mur4_f**k_y0uR_bra1N}
//踩的坑：之前把code_v5填错了 导致浪费半天时间 

---

- 丘比龙De女神
下载文件
![在这里插入图片描述](https://i.imgur.com/l6kJPvL.gif)

winhex打开能看到 nvshen.jpg  (由于有文件名所以猜测是压缩文件)
搜索jpg 能找到几个jpg的位置 其中发现附近有一个 love(应该就是提示了)
![在这里插入图片描述](https://i.imgur.com/EBPebW1.png)
love 附近的数据形式与zip(0x504B030414...)相似 
>更多文件头 : https://www.cnblogs.com/WangAoBo/p/6366211.html

love正好覆盖了0x504B0304 
更改后用.zip打开 有密码 猜测当然是love了～![在这里插入图片描述](https://i.imgur.com/MGom9LQ.png)

计算图片md5 得到flag![在这里插入图片描述](https://i.imgur.com/xQP0OuR.png)
flag{a6caad3aaafa11b6d5ed583bef4d8a54}

---

- WxyVM 2

丢到ida发现
input必须是25个字节
一大堆的+-^(但是真正影响到input的只有那么几个byte形的东西)
复制下来py处理一下吧![在这里插入图片描述](https://i.imgur.com/WyXyIzT.png)
再把它们倒过来 +改成- ，-改成+ ，^不变
然后找到各个相对input的偏移![在这里插入图片描述](https://i.imgur.com/Rnw989o.png)
复制到程序里编译运行即可
nctf{th3_vM_w1th0ut_dAta}

---

- simple machine
这道题可以说是标准的逆向题了
首先丢到ida里 能看到第一个是给变量分配内存的函数 第二个是反调试的(小小分析一下就知道了) 主要是后面两个加密的函数
![在这里插入图片描述](https://i.imgur.com/bvagYMa.png) 
进去看了看 发现逻辑有点复杂 无意义的语句很多, 决定从output开始 向上分析函数作用 (不静态分析单个函数的作用)

encode_2(将被异或的input按一定规则放进output):
先找到函数对output更改的语句，将这句中的变量用上面的语句替换(直到代入的都是常量) 
能够写出此函数对应的解码函数
```cpp
//decode_2:
unsigned char output[]={0x00,0x03,0x09,0x3A,0x05,0x0E,0x02,0x16,0x0F,0x1F,0x12,0x56,0x3B,0x0B,0x51,0x50,0x39,0x00,0x09,0x1F,0x50,0x04,0x14,0x57,0x3B,0x12,0x07,0x3C,0x1C,0x3A,0x15,0x05,0x0B,0x08,0x06,0x01,0x04,0x12,0x16,0x39,0x05,0x0B,0x50,0x57,0x09,0x12,0x0A,0x27,0x13,0x17,0x0E,0x02,0x55,0x18};//这就是最后拿来对比的字符串
    for(int i=0;i<=2;i++)
    {
        for(int j=0;j<18;j++)
        {
            input[j*3+i]=output[j+18*i];
        }
    }
```

encode_1(对input的每一位都按顺序异或str[]):
与上面同样的操作//这个函数规则更简单一点
```cpp
//decode_1:
char str[]="feeddeadbeefcafe";
    for(int i=0;i<54;i++)
    {
        input[i] ^= str[i%16];
    }
```

最后打印出input即可
flag{wh4t_a_fuck1ng_4ss3mbly_styl3_C_progr4mm1ng_c0de}
//把str抄反了 足足晕了半天(看到字符串就忘了小端序)

---

- Baby RSA
学习RAS
> 维基百科: https://zh.wikipedia.org/zh-hans/RSA加密演算法 

题目中有公钥(e,n)
以及加密后的信息m (听说是出题者把密文打成了m...)

首先将n分解为p,q，通过同余方程 de=1 (mod(p−1)(q−1)) 解出d
然后代入  m^d^≡ M(mod n)  求得明文M

上工具 
先将p, q 从n分解出来
并计算出d
最后将密文复制进去解密
![在这里插入图片描述](https://i.imgur.com/tpTpJoX.png)
flag{Acdxvf5vD_15_W7f}

---

# misc
- Remove Boyfriend
打开流量包可以看见有FTP流量
追踪流可以看到各种操作![在这里插入图片描述](https://i.imgur.com/7IisDFW.png)
两个文件 一个py 一个图片
py用来解密flag
flag{who_am_1}

---

- Coding Gay
可以从这张图片提取出4张图片
![](https://i.imgur.com/UaLoPtp.jpg)

文件结构大概是这样：
A图片在最前面显示，C图片中插着两张B图片（C图片看似png但是改成jpg格式才能打开）
没思路。。。

---
layout: post
title: 常见加密算法介绍
category: java
tags: [java]
keywords: DES、AES、RSA、对称加密、非对称加密、公钥、私钥
---

## 几个基本概念

+ 明文：原始信息

+ 密钥：加密与解密算法的参数，直接影响对明文进行变换的结果。

+ 密文：对明文进行变换的结果。

+ 加密算法：以密钥为参数，对明文进行多种置换和转换的规则和步骤，变换结果为密文。

+ 解密算法：加密算法的逆运算，以密文为输入，密钥为参数，变换结果为明文。

## 凯撒密码介绍

凯撒密码作为一种最古老的加密技术，在古罗马的时候都已经开始流行，他的基本思想是，通过把字母移动一定的位数来实现加密和解密。明文中的所有字母都在字母表上向后（或向前）按照一个固定数目进行偏移后被替换成密文。

例如，当偏移量是`3`的时候，所有的字母A将被替换成`D`，`B`变成E，由此可见，位数就是凯撒密码加密和解密的密钥。

字符串`ABC`的每个字符都右移3位则变成`DEF`，解密的时候`DEF`的每个字符左移`3`位即能还原。

**代码实现如下：**
```java
	public class CaesarDemo {
	public static void main(String args[]) {
		//明文：原始数据
		String clearText = "hellocaesar";
		//加密规则：将字母按照字母表的顺序向右移动3位
		int key = 3;
		
		//密文：通过加密算法将明文混淆之后的信息
		String cipher = encrypt(clearText, key);
		System.out.println("密文:" + cipher);

		String decrypt = decrypt(cipher, key);
		System.out.println("明文:" + decrypt);
	}

	/**
	 * 加密
	 *
	 * @param clearText 原始数据
	 * @param key       密钥
	 * @return 密文
	 */
	private static String encrypt(String clearText, int key) {
		char[] charArray = clearText.toCharArray();

		for (int i = 0; i < charArray.length; i++) {
			charArray[i] += key;
		}
		return new String(charArray);
	}

	/**
	 * 解密
	 *
	 * @param cipher 明文
	 * @param key    密钥
	 * @return 明文
	 */
	private static String decrypt(String cipher, int key) {
		char[] charArray = cipher.toCharArray();

		for (int i = 0; i < charArray.length; i++) {
			charArray[i] -= key;
		}
		return new String(charArray);
	}
}
```
**输出结果**
```
密文:khoorfdhvdu
明文:hellocaesar
```

### 破解凯赛密码-频率分析法

如果我们知道一个密码是用凯撒密码加密的，可以试用暴力破解法来解密，凯撒移位码只有`25`种，最多就是将`25`种可能性挨个检测一下就可以了。

那在不知道的前提下，又怎样判断是凯撒密码呢？这得用频率分析法。

在任何一种书面语言中，不同的字母或字母组合出现的频率各不相同。而且，对于以这种语言书写的任意一段文本，都具有大致相同的特征字母分布。

比如，在英语中，字母`E`出现的频率很高，而`X`则出现的较少。英语文本中典型的字母分布情况如下图所示。

![](https://www.major818.com/assets/images/2019/java/letter.png)

**破解流程：**

`1.` 统计密文里出现次数最多的字符，例如出现次数最多的字符是`h`

`2.` 计算字符`h`到`e`的偏移量，值为3，则表示原文偏移了3个位置。

`3.` 将密文所有的字符恢复偏移3个位置。

**注意点：**

`1.` 统计密文里出现次数最多的字符时，需要统计几个备选，因为最多的可能是空格或者其他字符。

`2.` 短文可能严重偏离标准频率，假如文章少于100个字母，那么对它的解密就会比较困难，而且不是所有文章都适用标准频率。


## 对称加密

**1.概述** 

加密和解密都试用同一把密钥，这种加密方法称为对称加密，也称为单密钥加密。
 
![](https://www.major818.com/assets/images/2019/java/symmetric-encryption.png)

基于“对称密钥”的加密算法主要有`DES`算法、`3DES`算法、`AES`算法、`Blowfish`算法，`Rc5`算法等。

**2.对称密码常用的数学运算**

**移位和循环移位**

移位就是将一段数码按照规定的位数整体性地左移或右移，循环右移就是当右移时，把数据的最后的位移到数码的最前头，循环左移正相反

例如，对十进制数码12345678循环右移1位的结果为81234567。

**置换**

就是将数码中的某一位的值根据置换表的规定，用另一位代替。它不想移位操做那样整齐有序，看上去杂乱无章。这正是加密所需，被经常应用。

**扩展**

就是将一段数码扩展成比原来位数更长的数码，扩展方法有多种，例如，可以用置换的方法，以扩展置换表来规定扩展后的数码每一位的替代值。

**压缩**

就是将一段数码压缩成比原来位数更短的数码。压缩方法有多种，例如，也可以用置换的方法，以表来规定压缩后的数码每一位的替代值。

**异或**

这是一种二进制布尔代数运算。异或的数学符号为⊕，它的运算法则如下，

		1 ⊕ 1=0
		0 ⊕ 0=0
		1 ⊕ 0=1
		0 ⊕ 1=1

也可以简单的理解为，参与异或运算的两数位相等，则结果为0，不等则为1。

**迭代**

迭代 就是多次重复相同的运算，这在密码运算中经常使用，以使得形成的密文更加难以破解。

**总结：**这些运算主要是对比特位进行操做，其共同目的就是把被加密的明文数码尽可能深地打乱，从而加大破译的难度。

## 对称加密算法

### 1.DES算法

`DES`是`Data Encryption Standard`(数据加密标准)的缩写。它是由`IBM`公司研制的一种对称加密算法，美国国家标准局于1977年公布把它作为非机要部门使用的数据加密标准，三十年来，它一直活跃在国际保密通信的舞台上，扮演了十分重要的角色。

`DES`是一个分组加密算法，典型的`DES`以`64`位为分组对数据加密，加密和解密用的是同一个算法。它的密钥长度是`56`位（因为每个第`8`位都用作奇偶校验），密钥可以是任意的`56`位的数，其保密性依赖于密钥。

**DES加密的算法框架大致如下：**

首先要生成一套加密密钥，从用户处取得一个`64`位长度的密码口令，然后通过等分、移位、选取和迭代形成一套`16`个加密密钥，分别供每一轮运算中使用。

`DES`对`64`位（bit）的明文分组`M`进行操做，`M`经过一个初始置换`IP`，置换成`m0`。将`m0`明文分成左半部分和右半部分，各`32`位长。

然后进行`16`轮完全相同的运算（迭代），在每一轮运算过程中数据与相应的密钥结合。

经过`16`轮迭代后，左、右半部分合在一起经过一个末置换（数据整理）这样就完成 了加密过程。

**加密流程如图所示：**

![](https://www.major818.com/assets/images/2019/java/des.png)

**代码实现如下：**

```java
public class DESDemo {
	public static void main(String args[]) throws Exception {
		String clearText = "hellocaesar";
		// 提供原始秘密 ：长度64位，8个字节
		String originKey = "12345678";

		String cipherText = desEncryption(clearText, originKey);
		System.out.println("密文：" + cipherText);

		String clearText2 = desDecrypt(cipherText, originKey);
		System.out.println("明文:" + clearText2);

	}

	/**
	 * 加密
	 *
	 * @param clearText 明文
	 * @param originKey 原始密钥
	 * @return
	 * @throws NoSuchPaddingException
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeyException
	 */
	private static String desEncryption(String clearText, String originKey)
			throws NoSuchPaddingException,
			NoSuchAlgorithmException,
			InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
		//1.获取加密算法工具类对象
		Cipher cipher = Cipher.getInstance("DES");
		//2.对工具类对象进行初始化
		//mode: 加密/解密模式
		//key: 对原始密钥处理之后的密钥
		SecretKeySpec key = getKey(originKey);
		cipher.init(Cipher.ENCRYPT_MODE, key);

		//3.用加密功能工具类对象对明文进行加密
		byte[] doFinal = cipher.doFinal(clearText.getBytes());

		return new String(doFinal);
		//return Base64.encode(doFinal);
	}

	/**
	 * 构造密钥
	 *
	 * @param originKey 原始密钥
	 * @return 构造密钥
	 */
	private static SecretKeySpec getKey(String originKey) {
		byte[] buffer = new byte[8];
		byte[] originByte = originKey.getBytes();

		for (int i = 0; i < 8 && i < originByte.length; i++) {
			buffer[i] = originByte[i];
		}
		//根据给定的字节数组构造一个密钥
		return new SecretKeySpec(buffer, "DES");
	}

	/**
	 * 解密
	 *
	 * @param cipherText 密文
	 * @param originKey 密钥
	 * @return
	 * @throws NoSuchPaddingException
	 * @throws NoSuchAlgorithmException
	 * @throws InvalidKeyException
	 * @throws BadPaddingException
	 * @throws IllegalBlockSizeException
	 */
	private static String desDecrypt(String cipherText, String originKey)
			throws NoSuchPaddingException, NoSuchAlgorithmException,
			InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
		Cipher cipher = Cipher.getInstance("DES");
		SecretKeySpec key = getKey(originKey);
		cipher.init(Cipher.DECRYPT_MODE, key);
		byte[] doFinal = cipher.doFinal(cipherText.getBytes());
		//byte[] doFinal = cipher.doFinal(Base64.decode(cipherText));

		return new String(doFinal);
	}
}
```
**输出结果如图所示：**

![](https://www.major818.com/assets/images/2019/java/desDemoInput.png)

为什么会造成加密后的密文乱码并抛异常呢？

是因为在加密的过程中都是对比特位进行操做的，需要将明文转换成字节数组传给`doFinal(clearText.getBytes())`加密方法，该方法最终再返回一个字节数组。其实问题就出在加密过程中通过`DES`的一系列

算法过后会造成加密前的字节数组和加密后的字节数组长度不再相等，`String`类再通过平台码去解码时由于找不到与之对应的字符就会出现乱码的情况。

有什么方法可以防止乱码呢?

目前最好的解决办法就是通过Base64去编码，将加密和解密方法中注释的那一行代码放开后再来看看输出结果。

![](https://www.major818.com/assets/images/2019/java/desDemoSuccess.png)

### 2.Base64编码

加密后的结果是字节数组，这些被加密后的字节在码表（例如GBK、UTF-8码表）上找不到对应字符，会出现乱码，当乱码字符串再次转换为字节数组时，长度会变化，导致解码失败，所以转换后的数据是不安全的。

使用`Base64`对字节数组进行编码，任何字节都能够映射成对应的`Base64`字符，之后能恢复到字节数组，利于加密后数据的保存和传输，所以转换是安全的。

**Base64码表如下：**


![](https://www.major818.com/assets/images/2019/java/base64.png)

**Base64编解码原理**

*`a 、`当字符串字符个数为`3`的整数倍时：*

比如字符串`ABC`,其在计算机内存中的十六进制表示为41、42、43，十进制表示为65，66，67二进制表示为：

`01000001`	`01000010`	`01000011`   将这三个二进制数依次取`6bit`

`010000/01`	`0100/0010`	`01/000011` 就转换成了

`010000`  `010100`  `001001`  `000011`，将这四个二进制数转换成十进制数为：16，20，9，3。

找到对应的字符为`Q`、`U`、`J`、`D`。

也就是说字符串`ABC`经过`Base64`编码后得出`QUJD`。这是最简单的情况，即`ASCLL`码字符数刚好可以被`3`整除。


*`b、`当字符串字符个数除以3余数为2时：*

比如字符串`ce`，其在内存中的十六进制表示为63，65;十进制表示99，101；二进制表示为：

`01100011`	`01100101`  依此取6bit

`011000/11`	`0110/0101`  这时，第`3`个字符不足`6`位，在后面补零，也就是`0101`变成了`010100`。转换结果为

`011000`  `110110`	`010100` 这`3`个二进制数转换成十进制为24，54，20。对照码表得出结果`Y2U`。编码后每补两个零就用一个`=`填充，最后编码得出`Y2U=`。

*`c、`当余数为1时*

比如字符串`{`，其在内存中的十六进制表示为`$78`，十进制为123，二进制位表示为：

`01111011` 依此取6bit

`011110/11 ` 补零后为

`011110/110000`  转换结果为`011110`和`110000`。这两个二进制数转换成十进制数为 30，48。对照码表得出结果为`ew`，补上`=`，最后编码得出`ew==`。解码也很简单，是编码的逆过程，即将每个字符对

照码表换算成6bit的二进制数，然后重组起来，按8位进行截取，得出源码。

## 3DES和AES算法

### 1. 3DES算法

DES的安全性首先取决于密钥的长度。密钥越长，破译者利用穷举法搜索密钥的难度就越大。DES采用64bit分组长度和56bit密钥长度：

密钥大小（bit）| 密钥个数 | 每微妙执行一次解密所需要的时间 | 每微妙执行一百万次解密所需要的时间
------------- | ---------| ----------------------- | -------------------------------
  56          | 2^56=7.2*10^16 |  2^55µm =1142年 | 1001小时

随着软硬件技术的发展，多核CPU、分布式计算、量子计算等理论的实现，DES在穷举方式的暴力攻击下还是相当脆弱的，因此很多人想办法用某种算法替代它，面对这种需求广泛被采用的有两种方案：

`1.`设计一套全新的算法，例如AES。

`2.`为了保护已有软件的投资，仍使用DES，但使用多个密钥进行多次加密，这就是多重DES加密。

例如后来演变出的`3-DES`算法使用了3个独立密钥（密钥长度为168bit）进行三重加密，这就比`DES`大大提高 了安全性，如果`56`位`DES`用穷举搜索来破译需要$$2^56$$次运算，而`3-DES`则需要2^112次。

### 2.AES算法

3DES缺陷是算法运行行对较慢。因为原来DES是为70年代的硬件设计的，算法代码并不高效，而3DES是DES算法的3轮迭代，因此更慢。而且DES和3DES的分组大小都是64bit，3DES密钥长度却是168bit，处于加密效率和安全的考虑，需要更大的分组长度。于是AES应运而生。

Advanced Encryption Standard高效加密标准，该标准是美国国家标准技术研究所于2001年颁布的，AES旨在取代DES成为广泛使用的标准，2006年AES已成为最流行的对称加密算法。

AES使用的分组大小为128bit，密钥长度可以为128，192，256bit。最简单最常用的也就是128bit（16字节）的密钥。

**AES算法过程：**

AES加密过程涉及到4种操做：字节替代（SubBytes）、行移位（ShiftRows）、列混淆（MixColumns）和轮密钥加（AddRoundKey）。

解密过程分别为对应的逆过程。由于每一步操做都是可逆的，按照相反的顺序进行解密即可恢复明文。加解密中每轮的密钥分别由初始密钥扩展得到。算法中16（byte）字节的明文、密文和轮密钥都以一个4*4的矩阵表示。

*三种对称加密算法对比：*

分组（块）加密标准 | 分组大小 | 密钥长度
----------------- | -------|-------
DES   | 64bit = 8byte|56 bit = 7 byte
3DES  | 64 bit = 8byte | 168 bit = 21 byte
AES   | 128 bit = 16byte | 128\|192\|256 bit = 16\|24\|32byte

**在实际开发中，对称加密算法这部分的运算大致如下图所示：** 

![](https://www.major818.com/assets/images/2019/java/sendingCiphertext.png)

## 非对称加密-RSA

### 1.常用的加密方式

**对称加密：**加密方和解密方使用同一个密钥
	
优点：加密解密过程简单、高效。
			
缺点：有一方泄漏，则整个加密就失去了意义。

**非对称加密：**加密方和解密方使用不同的密钥

优点：解密的密钥无法由加密的密钥，即使加密方暴露出了密钥也没事，这种加密方和解密方使用不同的密钥，大大提高了安全性。

缺点：效率比较低，过程比较繁琐。

### 2. 辅助概念

	1.质数的概念
	
	2.互为质数的概念

### 3. RSA加密密钥的获取

step1：随机选取两个数`p`、`q`，满足互质

Step2：`n = p * q`, //公开模数Public Modules ，其二进制位数即为密钥长度。

Step3：`𝑔= ɐ (p) ɐ(q) = (p-1)*(q-1)` //欧拉函数

Step4：在`1`和`𝑔`之间任意一个整数`e`，满足 `1 < e < 𝑔` //Public Exponent ,公开旨数

Step5:	由`e * d mode g = 1` 关系式推导出来 `d`, //Private Exponent ,私有指数

### 4.RSA加密密钥的获取

**RSA算法中的：**
	
	公开密钥 = (e,n)
	
	私有密钥 = (d,n)

### 5.RSA加密解密算法

**加密算法：**设M为需要加密的明文数据

	则加密算法为：`Encrypt_Message   =  M^e mod  n`


**解密算法：**设D为需要解密的密文数据

	则解密算法为：`Decrypt_Message = D^d mod n`

### 6.RSA算法的缺点

1.效率非常地下

2.密文数据较之原数据，其长度增加，即数据冗余太严重。






 











 






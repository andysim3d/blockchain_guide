## 数字签名
类似在纸质合同上签名确认合同内容，数字签名用于证实某数字内容的完整性（integrity）和来源（或不可抵赖，non-repudiation）。

一个典型的场景是，A 要发给 B 一个文件（一份信息），B 如何获知所得到的文件即为 A 发出的原始版本？A 先对文件进行摘要，然后用自己的私钥进行加密，将文件和加密串都发给 B。B 收到后文件和加密串，用 A 的公钥来解密加密串，得到原始的数字摘要，跟对文件进行摘要后的结果进行比对。如果一致，说明该文件确实是 A 发过来的，并且文件内容没有被修改过。

### HMAC
全称是 Hash-based Message Authentication Code，即“基于 Hash 的消息认证码”。基本过程为对某个消息，利用提前共享的对称密钥和 Hash 算法进行加密处理，得到 HMAC 值。该 HMAC 值提供方可以证明自己拥有共享的对称密钥，并且消息自身可以利用 HMAC 确保未经篡改。

HMAC(K, H, Message)

其中，K 为提前共享的对称密钥，H 为提前商定的 Hash 算法（一般为公认的经典算法），Message 为要处理的消息内容。如果不知道 K 和 H，则无法根据 Message 得到准确的 HMAC 值。

HMAC 一般用于证明身份的场景，如 A、B 提前共享密钥，A 发送随机串给 B，B 对称加密处理后把 HMAC 值发给 A，A 收到了自己再重新算一遍，只要相同说明对方确实是 B。

HMAC 主要问题是需要共享密钥。当密钥可能被多方拥有的场景下，无法证明消息确实来自某人（Non-repudiation）。反之，如果采用非对称加密方式，则可以证明。

### 盲签名

1983 年由 David Chaum [提出](http://www.hit.bme.hu/~buttyan/courses/BMEVIHIM219/2009/Chaum.BlindSigForPayment.1982.PDF)。签名者在无法看到原始内容的前提下对信息进行签名。

盲签名主要是为了实现防止追踪（unlinkability），签名者无法将签名内容和结果进行对应。典型的实现包括 [RSA 盲签名](https://en.wikipedia.org/wiki/RSA_(algorithm) )。

### 多重签名
n 个持有人中，收集到至少 m 个（$$n\ge{}m\ge{}1$$）的签名，即认为合法，这种签名被称为多重签名。

其中，n 是提供的公钥个数，m 是需要匹配公钥的最少的签名个数。

### 群签名
1991 年由 Chaum 和 van Heyst 提出。群签名属于群体密码学的一个课题。

群签名有如下几个特点：只有群中成员能够代表群体签名（群特性）；接收者可以用公钥验证群签名（验证简单性）；接收者不能知道由群体中哪个成员所签（无条件匿名保护）；发生争议时，群体中的成员或可信赖机构可以识别签名者（可追查性）。

Desmedt 和 Frankel 在 1991 年提出了基于门限的群签名实现方案。在签名时，一个具有 n 个成员的群体共用同一个公钥，签名时必须有 t 个成员参与才能产生一个合法的签名，t 称为门限或阈值。这样一个签名称为(n, t)不可抵赖群签名。

### 环签名

环签名由 Rivest,shamir 和 Tauman 三位密码学家在 2001 年首次提出。环签名属于一种简化的群签名。

签名者首先选定一个临时的签名者集合,集合中包括签名者自身。然后签名者利用自己的私钥和签名集合中其他人的公钥就可以独立的产生签名,而无需他人的帮助。签名者集合中的其他成员可能并不知道自己被包含在其中。

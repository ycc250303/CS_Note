## RSA握手

### 第一次握手

* **客户端向服务器发送**加密通信请求 `ClientHello`，消息包含：

  * 客户端支持的TLS版本
  * 客户端产生的随机数 `Client Random`
  * 客户端支持的密码套件列表

### 第二次握手

* **服务器向客户端发送**响应 `ServerHello`，消息包含：

  * 确认TLS协议版本
  * 服务器产生的随机数 `Server Random`
  * 确认的密码套件列表
  * 服务器的数字证书 `Server Certificate`
  * `Server Hello Done`表示结束

### 客户端验证证书

* 数字证书包含：
  * 公钥
  * 持有者信息
  * 证书有效期
  * 证书认证机构CA
  * CA的数字签名及算法
* 证书签发
  * 将相关信息打包，进行Hash计算得到Hash值
  * 使用私钥将Hash值加密，生成 `Certificate Signature`
  * 将 `Certificate Signature`添加到文件证书上
* 校验证书
  * 使用同样的算法得到Hash值H1
  * 使用CA的公钥解密Certificate Signature，得到Hash值H2
  * 比较H1和H2，相同则为可信赖的证书

### 第三次握手

* 客户端确认证书真实，回应服务器

  * 随机数 `pre-master key`
  * 利用三个随机数生成会话密钥
  * 加密通信算法通知**Change Cipher Spec**，表示随后的信息都用会话密钥加密通信
  * **Encrypted Handshake Message（Finishd）**：验证加密通信是否可用以及之前的握手信息是否被篡改

### 第四次握手

* 服务器最后回应
  * 根据之前三个随机数计算出通信的会话密钥
  * 向客户端发送加密通信算法改变通知
  * 服务端握手结束通知

## RSA算法缺陷

不支持前向保密：私钥泄露后，所有密文都会被破解

## ECDHE握手

具有前向安全性

### 第一次握手

* 客户端发送 `Client Hello`，包含：

  * TLS版本号
  * 支持的密码套件列表
  * 随机数 `Client Random`

### 第二次握手

* 服务端返回 `Server Hello`：
  * 确认TLS版本号
  * 随机数 `Server Random`
  * 选择密码套件
  * 发送证书和 `Server Key Exchange`
    * 选择椭圆曲线
    * 生成随机数作为椭圆曲线私钥
    * 计算出服务端的公钥（RSA加密）
* 共享的数据：**Client Random、Server Random 、使用的椭圆曲线、**椭圆曲线基点 G、服务端椭圆曲线的公钥****

### 第三次握手

* 客户端校验证书是否合法
* 生成客户端椭圆曲线公钥
* 用 `Client Key Exchange`发送给服务端
* 发送 `Change Cipher Spec`，表示后续改用加密算法通信
* 发送 `Encrypted Handshake Message`：摘要，加密，验证密钥是否可以使用

### 第四次握手

* 类似第三次的操作
* 最后一次握手时就可以提前发送HTTP数据

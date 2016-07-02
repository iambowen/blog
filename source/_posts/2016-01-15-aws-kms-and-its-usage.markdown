---
layout: post
title: "AWS KMS and its usage"
date: 2016-01-15 09:33:30 +0800
comments: true
categories: ["AWS", "KMS"]
---

##什么是KMS
---
[KMS](https://aws.amazon.com/kms/)是AWS提供的中心化的key托管服务，它使用硬件安全模块 (HSM)保护密钥安全。它可以被集成到其它的AWS服务中，如S3, EBS, RDS等，同时所有关于key的使用都会在CloudTrail中记录，以方便审计。

## KMS的优点
----
基本上来自于[文档](https://aws.amazon.com/kms/)，其好处有如下几点：

1. 中心化的key托管服务。举个例子，对于不同的环境(staging/production)，我们需要维护不同private key 去做部署，调试等等，还得考虑定期rotate。出于安全考虑，这些private key不推荐和部署的repo放在一起。一般情况下你得把它们放在一个统一的地方去保存，如[Rattic](http://rattic.org/)或者[Vault](https://www.vaultproject.io/)去管理。这样的话，你的承担这些工具的维护任务。KMS可以让你免除维护的压力。
2. 和 AWS 服务的集成。S3，EBS，RDS的数据加密，都可以使用KMS。同时，它也支持命令行或者API去管理key，进行key的rotate，加密解密等。
3. 可伸缩性、耐用性和高可用性。KMS会自动帮你保存key多份拷贝，耐用性99.999999999%，同时KMS会在多个AZ部署，保证高可用性。
4. 安全。KMS在服务端通过硬件加密，保证了你在上面存储的key的安全性。其实现的细节在[这里](https://d0.awsstatic.com/whitepapers/KMS-Cryptographic-Details.pdf)
5. 审计。对于key的请求，都会被记录在CloudTrail中，方便审计。

可以看到的好处有很多，比如直接把加密过后的private key或者密码扔到repo中，再也不用担心被别人拿去干坏事。

## 使用 KMS 服务
----
要使用KMS服务，首先得创建一个新的master key。key是按照region划分， 自己创建key的价格是1刀一个月，每个月的前20000次请求是免费的。

### 创建新key
---
在AWS Console -> IAM界面的`Encryption Keys`中找到创建Key和Key管理的选项，如key的`Enable`、`Disable`或者删除等。当然，我们可以通过AWS CLI来创建key，这样可以将整个过程用代码管理起来:

1. 假设AWS account为`123456789`,指定key policy并保存到文件(e.g `policy.json`)中

```json
{
  "Id": "KeyPolicy-1",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow access for Admin",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789:root"
      },
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    }
  ]
}
```

2. 创建key，并绑定对应的policy

``` bash
 ~> aws kms create-key --key-usage "encryption key" --description "master key" --policy "$(cat policy.json)"
```

返回的内容可能如下
```json
{
  "KeyMetadata": {
    "KeyId": "aabbccdd-4444-5555-6666-778899001122",
      "Description": "master key",
      "Enabled": true,
      "KeyUsage": "encryption key",
      "CreationDate": 2433401783.841,
      "Arn": "arn:aws:kms:ap-southeast-2:123456789:key/aabbccdd-4444-5555-6666-778899001122",
      "AWSAccountId": "123456789"
  }
}
```
3. 授权IAM user/role去使用或者管理key,这是除了policy之外的另一种访问管理控制的机制。

```bash
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": [
    "arn:aws:iam::111122223333:user/KMSUser",
    "arn:aws:iam::111122223333:role/KMSRole",
  ]},
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

4. 创建alias,可以作为keyid的替身使用

```bash
aws kms create-alias --alias-name "alias/test-encryption-key" --target-key-id aabbccdd-4444-5555-6666-778899001122
```

5. 使用key去加密文件。加密后的输出为base64编码后的密文，可以进一步解码为二进制文件。

```bash
aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedFile
```

6. 解密文件，原理如加密的过程。

```bash
aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedFile --output text --query Plaintext | base64 --decode > ExamplePlaintextFile
```

### 局限性
这种使用KMS的方式只能加密最多4KB的数据。想要加密更大的数据可以使用KMS去生成一个[Data Key](http://docs.aws.amazon.com/kms/latest/developerguide/workflow.html)，然后利用Data Key去加密数据。

### 使用场景举例
在REA项目中，在AWS上部署的大多数APP都是（尽量）遵循12factors原则的。应用运行时依赖的配置是通过`user-data`传入环境变量设置。在一个instance上启动服务的过程大致如下：
1. 在`launchConfiguration`中为instance添加`instanceProfile`，对应的role有使用KMS的权限；
2. 在`user-data`中设置cypher text并且解密到环境变量中:

```bash
cipher="CiBwo3lXT5T+pTZu7P9Cqkh0Iolpaz9FMzha5jJb6kTdiBKNAQEBAgB4cKN5V0+U/qU2buz/QqpIdCKJaWs/RTM4WuYyW+pE3YgAAABkMGIGCSqGSIb3DQEHBqBVMFMCAQAwTgYJKoZIhvcNAQcBMB4GCWCGSAFlAwQBLjARBAwIxkIN0TeX1HiWyj0CARCAIVaSfD/spTBFAfBVIp/Wy6TadlwUKKz/oTMWUUob9fcxdg=="
cipher_blob=$(mktemp /tmp/blob.123)
echo -n "${cipher}" | base64 -D > cipher_blob
PASSWORD=$(aws kms decrypt --ciphertext-blob fileb://$cipher_blob \
             --query "Plaintext"                         \
             --output text                               \
             --region ap-southeast-2  | \
             base64 -D
)

docker run -d -e PASSWORD=$PASSWORD .......
```

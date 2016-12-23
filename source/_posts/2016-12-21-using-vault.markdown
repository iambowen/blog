以前曾经介绍过关于[KMS的用法](http://www.jianshu.com/p/041387fe6c4e)，其中，提到了它的优点和用处，我们使用的场景有如下几点:

1. 我们产品的环境的所有的配置都保存在git上(Config As Code?)，所以相关的密码、private key等需要加密
2. 对AWS上应用/服务涉及的敏感数据进行加密
3. AWS上传输的数据进行加密，比如SQS

如果脱离AWS，选择好像还真是不太多，Harshicorp的[Vault](https://www.vaultproject.io)是我仅知道的一个，[RatticDB]算半个吧。

##什么是Vault
--- 

Vault提供了对token，密码，证书，API key等的安全存储(key/value)和控制，。它能处理key的续租、撤销、审计等功能。通过API访问可以获取到加密保存的密码、ssh key、X.509的certs等。它的特性包括:

1. 加密存储. 没有做到KMS对存储的HMS(硬件加密)，但是它传输后端提供与KMS类似的功能，允许存储加密密钥并执行加密操作。 它可以存储已存在的credentials，也可以为你的基础设施动态生成新的credential来限制第三方的访问，这些credentials到期会被撤销，也可以续租。同时还有访问控制策略来进行访问的权限管理。
2. rotate key。如果把Vault当做加密服务来使用的话，可以设置rotate的时间来生成一个新的key。
3. 审计的日志。所有对API的调用都会记录在一个审计日志上，

因为使用Vault的目的是为了

1. 持续集成服务器上运行测试或者部署需要的密码、API key、以及private key等需要加密
2. 服务部署是将加密后的应用需要的配置解密
同时我不希望在服务器上安装vault的命令行工具，所以在下面的使用中我都用Restful API的方式。

## 启动Vault服务
--- 

Vault存储[backend(https://www.vaultproject.io/docs/secrets/index.html)]可以是

1. 内存(开发模式)
2. 磁盘/数据库
3. [Consoul](https://www.consul.io)
4. AWS
5. ...

我在用Docker启动Vault服务的时候使用的Production模式，为了简单使用了磁盘作为存储，但是为了persist，所以将valut的文件存在了docker volume上面。
`docker-compose`文件如下:

```bash
version: "2"
services:
  vault:
    image: vault:0.6.4
    volumes:
        - vault-volume:/vault/file
    environment: 
        VAULT_LOCAL_CONFIG:  '{"backend": {"file": {"path": "/vault/file"}}, "default_lease_ttl": "168h", "max_lease_ttl": "720h", "listener": {"tcp": {"address": "0.0.0.0:8200", "tls_disable": "1"}}, "disable_mlock": true}'
    command: "server"
    cap_add: 
      - IPC_LOCK  #--cap-add: Add Linux capabilities,  in order for Vault to lock memory
    ports:
        - 8200:8200
volumes:
   vault-volume:
      external: true    
```
创建volume，启动vault服务器，配置通过环境变量`VAULT_LOCAL_CONFIG`传入。

```
docker volume create --name vault-volume
docker-compose up -d
```
从本地的`8200`端口应该就可以访问到了:

```
 telnet 0 8200
Trying 0.0.0.0...
Connected to 0.
Escape character is '^]'.
^]
```
### 初始化
----

下面的API请求可以对Vault进行初始化，两个参数的意思将master key分成几份以及还原，这里就用1吧。

```bash
curl -X PUT -d "{\"secret_shares\":1, \"secret_threshold\":1}"  http://127.0.0.1:8200/v1/sys/init | jq
```
返回结果:

```bash
{
  "keys": [
    "36d8ae19eb3e9d48965011e49af99865ca2bc6c78f4e900b7e14482d048d5ea2"
  ],
  "keys_base64": [
    "NtiuGes+nUiWUBHkmvmYZcorxsePTpALfhRILQSNXqI="
  ],
  "root_token": "e4347e60-0e72-fa8f-05e8-94c7388bb12c"
}
```
第一个是master key的public key，第二个是unseal key,最后一个是root token。unseal vault之后才能验证进行具体的操作。

```bash
curl -X PUT -d '{"key": "NtiuGes+nUiWUBHkmvmYZcorxsePTpALfhRILQSNXqI="}'  http://127.0.0.1:8200/v1/sys/unseal | jq
```
结果:

```json
{
  "sealed": false,
  "t": 1,
  "n": 1,
  "progress": 0,
  "version": "0.6.4",
  "cluster_name": "vault-cluster-603ef85a",
  "cluster_id": "ac454859-dc57-ee1d-38c1-71a0226a8cf3"
}
```
### 创建新token
为了安全起见，我们可以用root token创建出有限权限的新token，来继续后面的操作。假设这个token可以读写的路径为`secret/*`。在这个之前我们先创建访问这个路径的policy:

```bash
 curl -v  -X POST  -H "X-Vault-Token:e4347e60-0e72-fa8f-05e8-94c7388bb12c" -d '{"rules":"path \"secret/*\" {\n  policy = \"write\"\n}"}'   http://127.0.0.1:8200/v1/sys/policy/admin-policy

 curl -X GET  -H "X-Vault-Token:e4347e60-0e72-fa8f-05e8-94c7388bb12c"  http://127.0.0.1:8200/v1/sys/policy/admin-policy   | jq

{
  "rules": "path \"secret/*\" {\n  policy = \"write\"\n}",
  "name": "admin-policy",
  "request_id": "c7e5a70d-bca8-54b9-eb1d-793f3b027e1f",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "name": "admin-policy",
    "rules": "path \"secret/*\" {\n  policy = \"write\"\n}"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
} 
```
对应的创建`user-policy`:

```bash
curl -X POST  -H "X-Vault-Token:e4347e60-0e72-fa8f-05e8-94c7388bb12c" -d '{"rules":"path \"secret/*\" {\n  policy = \"read\"\n}"}'   http://127.0.0.1:8200/v1/sys/policy/user-policy

curl -X GET  -H "X-Vault-Token:e4347e60-0e72-fa8f-05e8-94c7388bb12c"  http://127.0.0.1:8200/v1/sys/policy/user-policy | jq

{
  "name": "user-policy",
  "rules": "path \"secret/*\" {\n  policy = \"read\"\n}",
  "request_id": "104fd2f3-f0ae-86f6-c1f7-ed3df01be962",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "name": "user-policy",
    "rules": "path \"secret/*\" {\n  policy = \"read\"\n}"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```
然后我们分别创建两个token，admin token和 user token:

``` bash
curl -X POST -d '{"policies": ["admin-policy"]}' -H "X-Vault-Token:e4347e60-0e72-fa8f-05e8-94c7388bb12c"  http://127.0.0.1:8200/v1/auth/token/create | jq

{
  "request_id": "a58efd8f-4585-d736-4d69-e8ee1d29f19a",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "a1ab3379-665f-15bf-0035-32e059e5d055",
    "accessor": "a1e6d92c-9788-a980-04a6-bfa45b0b7c26",
    "policies": [
      "admin-policy",
      "default"
    ],
    "metadata": null,
    "lease_duration": 604800,
    "renewable": true
  }
}
```

``` bash
 curl -X POST -d '{"policies": ["user-policy"]}' -H "X-Vault-Token:e4347e60-0e72-fa8f-05e8-94c7388bb12c"  http://127.0.0.1:8200/v1/auth/token/create | jq

{
  "request_id": "c32dc4a6-e432-48db-c59d-5a32c5d780cb",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "5d9e8e7f-e133-4cfa-6f67-880f2799235b",
    "accessor": "ed1c10ec-ca1b-7d1f-3a0a-ca4763ea3cec",
    "policies": [
      "default",
      "user-policy"
    ],
    "metadata": null,
    "lease_duration": 604800,
    "renewable": true
  }
}
```
这样就有了对于`secrets/*`路径下进行读写的两个 token，可以尝试去admin的token去添加新的键值对:

```bash
export ADMIN_TOKEN="a1ab3379-665f-15bf-0035-32e059e5d055"
curl -X POST -H "X-Vault-Token:$ADMIN_TOKEN" -d '{"token":"c192d0211cb81fbfeee53fb16e2a7465"}' http://127.0.0.1:8200/v1/secret/api/search

export USER_TOKEN="5d9e8e7f-e133-4cfa-6f67-880f2799235b"
 curl -X GET -H "X-Vault-Token:$USER_TOKEN" http://127.0.0.1:8200/v1/secret/api/search | jq

{
  "request_id": "dd2fcbae-b74f-6fc8-b895-a2c78667b1f0",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 604800,
  "data": {
    "token": "c192d0211cb81fbfeee53fb16e2a7465"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}

 curl -X POST -H "X-Vault-Token:$USER_TOKEN" -d '{"token":"286755fad04869ca523320acce0dc6a4"}' http://127.0.0.1:8200/v1/secret/api/search | jq

{
  "errors": [
    "permission denied"
  ]
}
```
这样就有了基本的权限管理。假设vault服务器和应用服务器或者CI的服务器部署在同一私有网络中，应用服务器和CI slave是不可以ssh的，那么通过应用服务器或者CI在启动的时候通过HTTP请求，利用读权限的`user-token`就可以拿到API-TOKEN，同时没有暴露给外部。

### 使用AppRoles的验证方式
----
AWS的EC2 instance上可以绑定`instanceProfile`, instanceProfile对应的是IAM的role，这个role可以设置对AWS资源的访问权限，比如对S3某个bucket的写权限，或者对Dynamodb的写权限等。Vault提供的AppRoles的功能比`instanceProfile`要差很多，不过确实可以将机器和一定权限的Role绑定起来，控制访问的范围。

允许使用approle的验证方式同时创建一个给CI slave使用的role:

```bash
export VAULT_TOKEN="e4347e60-0e72-fa8f-05e8-94c7388bb12c"
curl -X POST -H "X-Vault-Token:$VAULT_TOKEN" -d '{"type":"approle"}' http://127.0.0.1:8200/v1/sys/auth/approle   

curl -X POST -H "X-Vault-Token:$VAULT_TOKEN" -d '{"policies":"user-policy"}' http://127.0.0.1:8200/v1/auth/approle/role/deploy-role    #这里请求的参数可以指定请求来源的CIDR block '{"policies":"user-policy", "bound_cidr_list":"172.20.32.0/28"}'，只有在这个ip range里面的服务器才能使用这个role从vault拿到指定数据 

```

下面的API请求可以拿到`role-id`，以及根据`role-id`生成`secret-id`，利用它们可以登录获得从vault读取数据的权限:

```bash
curl -X GET -H "X-Vault-Token:$VAULT_TOKEN"          http://127.0.0.1:8200/v1/auth/approle/role/deploy-role/role-id | jq .

{
  "request_id": "553d9fa9-50a2-274e-77f9-675c200d8cd1",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "role_id": "9d830303-2e06-b432-9023-677eb886041c"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}

  curl -X POST -H "X-Vault-Token:$VAULT_TOKEN" http://127.0.0.1:8200/v1/auth/approle/role/deploy-role/secret-id | jq .

{
  "request_id": "ab5edb82-d1a9-b751-47c6-e2e0fe7ca333",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": {
    "secret_id": "6788593e-2c9b-0bae-0b0d-4b2c8d84e81d",
    "secret_id_accessor": "f88c33fd-431d-35aa-6295-6d7ecd6b34d1"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```
使用`secret_id` 和 `role_id`联合登录，拿到新的token:
```
 curl -X POST  -d '{"role_id":"9d830303-2e06-b432-9023-677eb886041c","secret_id":"6788593e-2c9b-0bae-0b0d-4b2c8d84e81d"}' http://127.0.0.1:8200/v1/auth/approle/login | jq '{client_token: .auth.client_token}'

{
  "client_token": "e0fe3cfb-1323-4ec6-9490-d6ac06dc3c69"
}
```
然后利用`client_token`去读取前面写入到vault中的search api token:

```bash
 curl -X GET -H "X-Vault-Token:e0fe3cfb-1323-4ec6-9490-d6ac06dc3c69"  http://127.0.0.1:8200/v1/secret/api/search | jq '{"api-token": .data.token}'
{
  "api-token": "c192d0211cb81fbfeee53fb16e2a7465"
}
```

## 秘钥管理
----
AWS的EC2 instance的服务器，在启动时，可以绑定一对ssh keypair，以方便用户使用缺省的`ec2-user`ssh到服务器上。从最佳实践的角度来说，应该把服务器当做immutable的设施，不允许ssh。但是现实比较嘲讽，这样的需求仍然存在，那么我们可以换种方式，尽量做好ssh key的管理。
比如，对于每个新启动的服务器，动态的生成一对ssh keypair，只应用在这台服务器上，服务器销毁后，吊销key pair。
Vault提供了ssh keypair的管理功能，利用这个功能我们可以对key的生命周期的管理。它支持两种方式:
1. One-Time-Password (OTP) Type
2. Dynamic Key Type
个人倾向于使用动态key，但是官方的文档推荐OTP类型，原因是无法对动态的key的使用进行audit，还有一个就是生成动态的key会消耗资源导致vault服务的停顿(好失望:()。Anyway，不管它。

要让vault发放keypair, 需要先注册一个private key，这个key必须有服务器的管理权限.之后，你需要创建一个role，包括一些限定条件，如admin用户的名字，缺省用户，目标服务器的IP地址应该匹配的CIDR地址，具体过程如下:
```
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=e4347e60-0e72-fa8f-05e8-94c7388bb12c

vault write ssh/keys/deploy-role key=@shared_deploy_key.pem  

vault write ssh/roles/deploy-role \
    key_type=dynamic \
    key=shared_deploy_key \
    admin_user=root \
    default_user=ec2-user \
    cidr_list=172.23.0.0/16   #目标服务器的IP地址需要在CIDR的范围内

vault write ssh/creds/deploy-role ip=172.23.0.6
```
整个的过程大概是这样，vault利用注册的private key登陆到目标服务器上，然后将新生成的key pair中的public key写入到目标服务器的`authorized_keys`文件中。在你想登陆到服务器上的时候，用对应的client token验证，获取private key，ssh登陆。key有过期时间，过期之后就被revoke了。

### PKI 管理
---
我觉得全站https困难之一就在于PKI的管理，今年出现过几次certficate过期导致的产品环境的问题，还好及时的切换到了AWS Certificate Manager，免费而且到期自动续租，基本上不用担心管理的问题了。而我们原有的内部PKI管理要稍微麻烦些，需要用自己业务线的LOB Intermediate CA去签发新的certs，运行一些自动化的脚本，还得从RatticDB里面找到对应的private key。
但是如果基础设施不是基于AWS，好像就没有很合适的工具了，只能自己维护PKI，Vault也提供了PKI的管理，可以在这方面提供帮助。考虑环境的一致性，开发、测试以及staging也需要certificate，我觉得这个功能是有意义的。 

```bash
vault mount -path=example -description="example Root CA" -max-lease-ttl=87600h pki

vault write example/root/generate/internal common_name=example.com ttl=87600h key_bits=4096
Key          	Value
---          	-----
certificate  	-----BEGIN CERTIFICATE-----
MIIDFDCCAfygAwIBAgIUCugj4117yjDXigWcflp7nA6lAp0wDQYJKoZIhvcNAQEL
BQAwFjEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMjIzMDU0MjQ3WhcNMjYx
MjIxMDU0MzE3WjAWMRQwEgYDVQQDEwtleGFtcGxlLmNvbTCCASIwDQYJKoZIhvcN
....
-----END CERTIFICATE-----
serial_number	0a:e8:23:e3:5d:7b:ca:30:d7:8a:05:9c:7e:5a:7b:9c:0e:a5:02:9d
```
Vault会安全的保存Root CA 的private key。 可以通过http请求拿到ca的pem文件。

```bash
curl -s http://127.0.0.1:8200/v1/example/ca/pem | openssl x509 -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            0a:e8:23:e3:5d:7b:ca:30:d7:8a:05:9c:7e:5a:7b:9c:0e:a5:02:9d
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=example.com
        Validity
            Not Before: Dec 23 05:42:47 2016 GMT
            Not After : Dec 21 05:43:17 2026 GMT
        Subject: CN=example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
....
```
需要给Vault配置访问CA和CRL(certificate revocation list)的地址。

```bash
vault write example/config/urls issuing_certificates="http://127.0.0.1:8200/v1/example"
Success! Data written to: example/config/urls
```
创建intermediate CA.

```bash
vault mount -path=example_lob  -description="Example LOB Intermediate CA" -max-lease-ttl=26280h pki
Successfully mounted 'pki' at 'example_lob'!

vault write example_lob/intermediate/generate/internal  common_name="Example LOB Intermediate CA" ttl=26280h key_bits=4096 exclude_cn_from_sans=true

Key	Value
---	-----
csr	-----BEGIN CERTIFICATE REQUEST-----
...
-----END CERTIFICATE REQUEST-----

```
把这个csr内容存在`example_lob.csr`文件中，请求root ca签发这个intermediate ca。
```
vault write example/root/sign-intermediate csr=@example_lob.csr common_name="Example LOB Intermediate CA" ttl=8760h

Key          	Value
---          	-----
certificate  	-----BEGIN CERTIFICATE-----
MIIElTCCA32gAwIBAgIUH1UgMxdv8fGTMAfTFc86JHVnfB4wDQYJKoZIhvcNAQEL
....
-----END CERTIFICATE-----
expiration   	1514009491
issuing_ca   	-----BEGIN CERTIFICATE-----
MIIDFDCCAfygAwIBAgIUCugj4117yjDXigWcflp7nA6lAp0wDQYJKoZIhvcNAQEL
....
-----END CERTIFICATE-----
serial_number	1f:55:20:33:17:6f:f1:f1:93:30:07:d3:15:cf:3a:24:75:67:7c:1e
```
得到Root CA的签发过的intermediate CA certs后，保存为文件，导入。最后就是要设置CA/CRL，和上面相同。

```bash
vault write example_lob/intermediate/set-signed certificate=@example_lob.crt
Success! Data written to: example_lob/intermediate/set-signed

curl -s http://localhost:8200/v1/example_lob/ca/pem | openssl x509 -text | head -15
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            23:0e:28:69:08:d1:5e:c9:10:8d:61:fe:46:4b:c6:09:ef:44:73:db
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=example.com
        Validity
            Not Before: Dec 23 06:23:55 2016 GMT
            Not After : Dec 23 06:24:25 2017 GMT
        Subject: CN=Example LOB Intermediate CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (4096 bit)
                Modulus (4096 bit):

vault write example_lob/config/urls issuing_certificates="http://127.0.0.1:8200/v1/example_lob/ca" 
Success! Data written to: example_lob/config/urls
```
在开始给server签发certs前，需要创建role，之后就可以签发了。

```bash
vault write example_lob/roles/web_server key_bits=2048 max_ttl=8760h allow_any_name=true
Success! Data written to: example_lob/roles/web_server

vault write example_lob/issue/web_server common_name="auth.lob.example.com" ip_sans="172.23.0.2" ttl=720h format=pem


Key             	Value
---             	-----
lease_id        	example_lob/issue/web_server/a5c7ba76-34b5-b2a8-5262-3cd3fbc1630e
lease_duration  	719h59m59s
lease_renewable 	false
ca_chain        	[-----BEGIN CERTIFICATE-----
....
-----END CERTIFICATE-----]
certificate     	-----BEGIN CERTIFICATE-----
....
-----END CERTIFICATE-----
issuing_ca      	-----BEGIN CERTIFICATE-----
....
-----END CERTIFICATE-----
private_key     	-----BEGIN RSA PRIVATE KEY-----
....
-----END RSA PRIVATE KEY-----
private_key_type	rsa
serial_number   	5a:11:20:aa:35:ad:3a:dd:22:a8:8c:4b:26:04:a8:e5:ce:fb:96:74
```
拿到private key和crt就可以放在服务器上测试了，感觉用这个[grunt-connect-proxy](https://www.npmjs.com/package/grunt-connect-proxy)测试起来可能会快点。

### 其他
---
Vault还有一个我觉得很好的特性是可以将LDAP作为auth的backend，感觉维护的压力又小了很多:) 剩下的特性大家可以自己尝试，我们也正在考虑把替换RatticDB保存一些private key之类的credential，下次的security meetup上面我可以展示下spike的成果……。






# OpenSSL 使用技巧

## 查看证书信息

```bash
openssl x509 -in ca.crt -noout -text
```

## 查看在线域名的证书信息

```bash
openssl s_client -connect ms.icometrix.com:443 -cipher 'DEFAULT:!ECDH'
```




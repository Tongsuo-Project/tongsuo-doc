<a name="oCL6W"></a>
## 版本
<a name="S2pR1"></a>
### Tengine

- pr：[https://github.com/alibaba/tengine/pull/1595](https://github.com/alibaba/tengine/pull/1595)
- 发布版本：[https://github.com/alibaba/tengine/releases/tag/2.3.4](https://github.com/alibaba/tengine/releases/tag/2.3.4)
- 编译参数参考：
```bash
./configure --add-module=modules/ngx_openssl_ntls \
            --with-openssl=../Tongsuo \
            --with-openssl-opt='--strict-warnings --api=1.1.1 enable-ntls' \
            --with-http_ssl_module \
            --with-stream \
            --with-stream_ssl_module \
            --with-stream_sni \
```
<a name="xXDCS"></a>
### Tongsuo

- 版本：8.3.1 发布版本或者 master分支
- 注意：如果不是使用 master 分支，请将 Tengine 编译参数中的 `--api=1.1.1`去掉
<a name="jGg6U"></a>
## 配置
```bash
  	server {
				listen 443 ssl;

        #开启国密功能
        enable_ntls     on;

        #国际 RSA 证书
        ssl_certificate                 certs/test_rsa.crt;
        ssl_certificate_key             certs/test_rsa.key;

        #国际 ECC 证书（可选）
        ssl_certificate                 certs/test_ecc.crt;
        ssl_certificate_key             certs/test_ecc.key;

        #国密签名证书
        ssl_sign_certificate            certs/SS.cert.pem;
        ssl_sign_certificate_key        certs/SS.key.pem;

        #国密加密证书
        ssl_enc_certificate             certs/SE.cert.pem;
        ssl_enc_certificate_key         certs/SE.key.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

        default_type            text/plain;

        add_header  "Content-Type" "text/html;charset=utf-8";

        location / {
            return 200 "tengine ntls test OK, ssl_protocol is $ssl_protocol (NTLSv1.1 表示国密，其他表示国际)";
        }
		}
```
<a name="fWz8D"></a>
## 测试
<a name="NQJBN"></a>
### Tongsuo 编译
```bash
./config --strict-warnings --api=1.1.1 --prefix=/opt/tongsuo -enable-ntls
make -j
sudo make install
```
<a name="v8EGc"></a>
### 测试文件
```bash
GET / HTTP/1.1
Host: test.com

```
```bash
-----BEGIN CERTIFICATE-----
MIICJzCCAcygAwIBAgIUfHbYwHKmQURFdSxsYr8pc2EJUncwCgYIKoEcz1UBg3Uw
gYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl
MCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM
U09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTE5MDUyMzAy
NDU0OFoXDTIzMDcwMTAyNDU0OFowgYYxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC
SjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu
b2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRowGAYDVQQDDBFzZXJ2
ZXIgc2lnbiAoU00yKTBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABKLD833Sm2zL
epLM5z0EnkYPNJJpTPIBkiDKMkEtqAP5B2D3PFHVqGsfqd5U+nNU1g0/1NPAVY4/
Bio1XRUSQhejGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgbAMAoGCCqBHM9VAYN1
A0kAMEYCIQDjYHcHhHgjYbQKC8QVYaiix2rgZ/u6i2CcOpF5tpSpLwIhAIgHsYRz
744eAjIlV2oL5t1yDFeWwgVmwn+Z4bGxx4mz
-----END CERTIFICATE-----
```
```bash
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBG0wawIBAQQgCjJidE9Lz0pqqq4f
/pY+XpVEEQwisZWUa57v1bry2h2hRANCAASiw/N90ptsy3qSzOc9BJ5GDzSSaUzy
AZIgyjJBLagD+Qdg9zxR1ahrH6neVPpzVNYNP9TTwFWOPwYqNV0VEkIX
-----END PRIVATE KEY-----
```
```bash
-----BEGIN CERTIFICATE-----
MIICJDCCAcugAwIBAgIUfHbYwHKmQURFdSxsYr8pc2EJUngwCgYIKoEcz1UBg3Uw
gYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl
MCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM
U09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTE5MDUyMzAy
NDU0OFoXDTIzMDcwMTAyNDU0OFowgYUxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC
SjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu
b2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRkwFwYDVQQDDBBzZXJ2
ZXIgZW5jIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEV1zefsVNw8/F
+Wnbb4SlYrI4Gdfqq9CfdBIACMLpOfzbJT/y0mwSYe7JLovuvXiluMURd8Z4YfxO
vQoXaIcscKMaMBgwCQYDVR0TBAIwADALBgNVHQ8EBAMCAzgwCgYIKoEcz1UBg3UD
RwAwRAIgCJMHFkOqjFWmLB4kzeuRYnffCv0g3vSkKsTlVsAWPFcCIC4A3QVFtQxv
HeHmS/swFJYT+LXSEcIqPksbv1vYItL1
-----END CERTIFICATE-----
```
```bash
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBG0wawIBAQQgzSMRhRfhOEZUy15G
ph0g0afn4u914iiNBxCSUONZCJ2hRANCAARXXN5+xU3Dz8X5adtvhKVisjgZ1+qr
0J90EgAIwuk5/NslP/LSbBJh7skui+69eKW4xRF3xnhh/E69Chdohyxw
-----END PRIVATE KEY-----
```
```bash
-----BEGIN CERTIFICATE-----
MIICJjCCAcygAwIBAgIUfHbYwHKmQURFdSxsYr8pc2EJUnkwCgYIKoEcz1UBg3Uw
gYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl
MCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM
U09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTE5MDUyMzAy
NDU0OFoXDTIzMDcwMTAyNDU0OFowgYYxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC
SjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu
b2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRowGAYDVQQDDBFjbGll
bnQgc2lnbiAoU00yKTBZMBMGByqGSM49AgEGCCqBHM9VAYItA0IABPKZvVLKuwxN
ibqsJ9kig19actvV0kHZkXulR8smwaRCs7ilYt/lyVRXfQpEYDMY/YJsAXSKxPg/
3rqXVede+YijGjAYMAkGA1UdEwQCMAAwCwYDVR0PBAQDAgbAMAoGCCqBHM9VAYN1
A0gAMEUCIQDCPLbdTI/Q4WUp8xBlf4BpZhda0N9C44Ya8aolhoiGvgIgcCOHiWpD
4oIag3HgIzecMQMeX6MRzsB3hQcEHUhD0Ng=
-----END CERTIFICATE-----
```
```bash
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBG0wawIBAQQgDp4wtjWXPzQ+kMcU
P9kXs9NOYJoC2JZ7KYApCWie99KhRANCAATymb1SyrsMTYm6rCfZIoNfWnLb1dJB
2ZF7pUfLJsGkQrO4pWLf5clUV30KRGAzGP2CbAF0isT4P966l1XnXvmI
-----END PRIVATE KEY-----
```
```bash
-----BEGIN CERTIFICATE-----
MIICJTCCAcqgAwIBAgIUfHbYwHKmQURFdSxsYr8pc2EJUnowCgYIKoEcz1UBg3Uw
gYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl
MCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM
U09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTE5MDUyMzAy
NDU0OFoXDTIzMDcwMTAyNDU0OFowgYQxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC
SjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu
b2xvZ3kgTFRELjEVMBMGA1UECwwMQlNSQyBvZiBUQVNTMRgwFgYDVQQDDA9jbGll
bnQgZW5jKFNNMikwWTATBgcqhkjOPQIBBggqgRzPVQGCLQNCAAROyCUcsultcbNL
mLZ9z2jAvVD+1/FUBYqpBvHN2Qt4xObNnFGxZOL2WpsUwp6yVIqAu4a9bgktOdpF
18/n/AtUoxowGDAJBgNVHRMEAjAAMAsGA1UdDwQEAwIDODAKBggqgRzPVQGDdQNJ
ADBGAiEAqyfhBcSFBbTjhLQikuUkrq4gMhC1PQU/gTnmmXk4isUCIQCqg1fn3CyM
ofg7oEnXHWM4Ui2Fe2bitJ0q3Eu920ZhiA==
-----END CERTIFICATE-----
```
```bash
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqBHM9VAYItBG0wawIBAQQg3D0NXP1aHf0wAXbM
h2L2n3azQ7bmYYL36hkyTrUDu8yhRANCAAROyCUcsultcbNLmLZ9z2jAvVD+1/FU
BYqpBvHN2Qt4xObNnFGxZOL2WpsUwp6yVIqAu4a9bgktOdpF18/n/AtU
-----END PRIVATE KEY-----
```
```bash
-----BEGIN CERTIFICATE-----
MIIDATCCAekCFHtrL6LgAAo9od9xDpCoG1EqJQgqMA0GCSqGSIb3DQEBCwUAMD0x
CzAJBgNVBAYTAkNOMQswCQYDVQQIDAJDRDEhMB8GA1UECgwYSW50ZXJuZXQgV2lk
Z2l0cyBQdHkgTHRkMB4XDTE5MTExMjAyMjQ0NVoXDTIwMTExMTAyMjQ0NVowPTEL
MAkGA1UEBhMCQ04xCzAJBgNVBAgMAkNEMSEwHwYDVQQKDBhJbnRlcm5ldCBXaWRn
aXRzIFB0eSBMdGQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCsQ+jc
s33WqhynMDJl6tMbwhvobEN3zl65TcWII+chetPwN9xnfNfGDfpu8F/8w47ONRRw
LDCNmM/8tSDJiqtaLaxZF6wHdiWBQR55jUHSwVoy6eVAmOHgArKCloj3xK7HkQz6
68SalZ1LSePJeIlZycimgFrnSzsP6jbdtA9cMHuMWGBCVb23C8A5+L2C/zO1bo22
djrtxsm/TiBM39WplXc6OLfaPPI5THDA/EnSyvv7Wur9WSmT/ZZPl0aEPtYwoYLL
SDYgtmjx+7tjqLe+jHZdWUzknaEPFlQU3wlVdTMjbCu7pi0tBp8fk+G4EQX1Vr4W
bkKHAs/UeF95s15HAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAB/MLRHcW2En94Qx
npTZjJ7c05SOgaYXE/E2OEuQeqbekfqdkpxV6YV9CURhYGdbhPh6CO6Z8wy1d1jN
76dNlcKXxxZ9qM2IRllsfJkw/Ao/5eSYX/gxBJx51Yxxj/itwqtrROk5nlyh5hNa
2y+2ANyqJmBDtKU93qHCAeyIDEo5rPqfvazflwfGDu+9zmUUeyMoNQxXMINgbtJb
NQUN74uZXxBoFQiYmfQEEZpr6NNOFkFHXwsE6fsu97SIyz8PLB7o9C3JK3Yhbnnz
forZHIy4STtfWoXAo43uZa0ufRnuVJvXQgjUkBgSqdS8xVEyAbuOJB/w5TBeo6Be
Rd1/wJ0=
-----END CERTIFICATE-----
```
```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEArEPo3LN91qocpzAyZerTG8Ib6GxDd85euU3FiCPnIXrT8Dfc
Z3zXxg36bvBf/MOOzjUUcCwwjZjP/LUgyYqrWi2sWResB3YlgUEeeY1B0sFaMunl
QJjh4AKygpaI98Sux5EM+uvEmpWdS0njyXiJWcnIpoBa50s7D+o23bQPXDB7jFhg
QlW9twvAOfi9gv8ztW6NtnY67cbJv04gTN/VqZV3Oji32jzyOUxwwPxJ0sr7+1rq
/Vkpk/2WT5dGhD7WMKGCy0g2ILZo8fu7Y6i3vox2XVlM5J2hDxZUFN8JVXUzI2wr
u6YtLQafH5PhuBEF9Va+Fm5ChwLP1HhfebNeRwIDAQABAoIBACVsF0EiqPqiN9lG
ChzD15qXH3LtWfbD2SYONBQwIHzQbwwnRnUg1MsMrFO/WkrRvMslEsyPfPi1srEm
M/o0aqcLdv5fuxpf2yPqHpGvUxZStKKM1yWiUKdWTqs5woV4r7Ng2l4EK9CdIe+C
HL7etY/Q2wr4pUbLvAfoDIU7IX8YGcMWVwSU02d+0XNIhv4I86x/2hCDk+qoIfLW
S62FE4ID8UAL+GLn6d/IHiPBT+vw5s2m9FfrL8HJ2MvmRVnGELGyGqS+8f+jOMhi
2M82I8z3WmQr/VUFCYTUF31kaLVMNq6yZrU6uGdV+yBxisZPutOuOXbjjBmqwLj9
HcdZwxkCgYEA36BzwA9Unnw5Y08D2i15r0f8EaFN3AAFfm9mfrLLJgB/WJSxAg2J
kXfnDojlARlY0YW4d8MKz+A9+B/uvCrvxDqm4lodryiBIxaAHHfZF/qqBUslupf3
zcpRabD9cnPNcwaUyRQDS6XRuVf2xbJs7qyaaosvnGdMcfE7LJIqsnMCgYEAxTQD
qN4eDd89J6xdMVOwPwLz0G0qKUStBLM2ExE6u/CpdC8VTiO5kUZHMcecFPg4z1/K
qAqJxDlSIxvOR1mEZPLtAj7A/92OSnMbpqbs7vP2vHUS6ImiPL/bumhFPXKapHWB
dp7oWXHNTIRxA+4BJEuTZfRqz6zCrUw8H8gqK90CgYEAmGSqnMaVvs8e+JsfH+5/
j0B5+bW37mWhWNEnws2q/QG3xrDFk4WQKy7PqasGjGIukdITrKGg25qQAGgac+a6
sDncAkKxGe17W2L4+O1/ZwTuGl9knaz0NSxboK/5d6aM6ocgm4rk2AdvTWQxifYW
n+vF6zdgwa/ve3KOBciyChsCgYEAvpVTSBtJ/mwWJUZuVmKj/XG0AmXODk4hzF4K
T4kiM0oV6oQqWfcquxypZ5Ga5aUy+i+AosB0fmBLYkTYKZp42jrwFXBig6Uyg/8U
5Q2EBDdg6KdYm8WQNpfRGij1abpde71YXjSbJv5Vw7Jnqr2U+ufTTwBVTdmP133K
yYhgQT0CgYEAwnvnoIIpesOnBOh03WSCGd7n7jGa5m0kWjq04odYt10LlCiIcGVn
Nf0bizYHL6xHMFJZbNRWDAolEv8+3Vx6pJI/+FA0Xbdo2dbRWE+XPFG1bBQ5vwqT
o38OxpwGRkwQUMSx9rX1AUt6iuSZeEeimFCxhEp7JjUQ5wzTftiUu4Y=
-----END RSA PRIVATE KEY-----
```
```bash
-----BEGIN CERTIFICATE-----
MIIBjDCCATMCCQCT/HGOf/tvyTAJBgcqhkjOPQQBME8xCzAJBgNVBAYTAmNuMQsw
CQYDVQQIDAJjZDEVMBMGA1UEBwwMRGVmYXVsdCBDaXR5MRwwGgYDVQQKDBNEZWZh
dWx0IENvbXBhbnkgTHRkMB4XDTE5MDgyMzA3MzEyNVoXDTIwMDgyMjA3MzEyNVow
TzELMAkGA1UEBhMCY24xCzAJBgNVBAgMAmNkMRUwEwYDVQQHDAxEZWZhdWx0IENp
dHkxHDAaBgNVBAoME0RlZmF1bHQgQ29tcGFueSBMdGQwWTATBgcqhkjOPQIBBggq
hkjOPQMBBwNCAASHWxVhsXJfEjojWhSGl+a+g5udAc3Uq/46nVNxGilEb1LshvS5
kezj9FPimeFXeJXoXrae3TyEtqP+ukJIL822MAkGByqGSM49BAEDSAAwRQIgTdsj
H7u2hvEVFTYh3kdaUjMmcNpHiCwcxegPHAVS1JkCIQDhrtwjCHsamSk0alrqyb7b
zAoZyg7ZBDa4hvT9ELoThw==
-----END CERTIFICATE-----
```
```bash
-----BEGIN EC PARAMETERS-----
BggqhkjOPQMBBw==
-----END EC PARAMETERS-----
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIENj33rWi4FrRg+sc0etNx4df8qq85vbBUiVk5+BmtpKoAoGCCqGSM49
AwEHoUQDQgAEh1sVYbFyXxI6I1oUhpfmvoObnQHN1Kv+Op1TcRopRG9S7Ib0uZHs
4/RT4pnhV3iV6F62nt08hLaj/rpCSC/Ntg==
-----END EC PRIVATE KEY-----
```
```bash
-----BEGIN CERTIFICATE-----
MIICZTCCAgugAwIBAgIUGE5cdCFARYgAXNod7aHnThArA4YwCgYIKoEcz1UBg3Uw
gYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl
MCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM
U09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTE5MDUyMzAy
NDU0OFoXDTIzMDcwMTAyNDU0OFowgYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC
SjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu
b2xvZ3kgTFRELjEVMBMGA1UECwwMU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0
IENBIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEyJlpzUrZ7HgViOep
/yGUfPfnbb3zcJAShtW48ZnjJ5/x+uC5o6bT53QM/MV6N+ZWQkiO1f2ejZ0ApvG0
fHvWI6NdMFswHQYDVR0OBBYEFMXy6LXt68nqzLu9nn5r2AyUvmUxMB8GA1UdIwQY
MBaAFMXy6LXt68nqzLu9nn5r2AyUvmUxMAwGA1UdEwQFMAMBAf8wCwYDVR0PBAQD
AgEGMAoGCCqBHM9VAYN1A0gAMEUCIQCPjXZprl4C0AWvyVAoFx34aVFvvYFhEioJ
Dqz727tUTgIgdnRzfwdTJHkfIin8Ksr1+xMblSVC8+X8e+yONW9YoFg=
-----END CERTIFICATE-----
```
**（注意：以上证书和私钥仅用于测试，不能配置到生产环境中）**
<a name="OWqGO"></a>
### 单证书测试
```bash
(cat req.txt; sleep 2) | /opt/tongsuo/bin/openssl s_client -connect localhost:443 -cipher ECC-SM2-WITH-SM4-SM3 -enable_ntls -ntls
```
<a name="hIDqs"></a>
### 双证书测试
```bash
(cat req.txt; sleep 2) | /opt/tongsuo/bin/openssl s_client -connect localhost:443 -cipher ECDHE-SM2-WITH-SM4-SM3 -enable_ntls -ntls -sign_cert Tongsuo/certs/CS.cert.pem -sign_key CS.key.pem -enc_cert CE.cert.pem -enc_key CE.key.pem -verifyCAfile CA.cert.pem
```
<a name="I13pa"></a>
## 浏览器测试

1. 下载和安装360安全浏览器: [http://jinjiu.oss.aliyuncs.com/360se10.1.1670.0.exe](http://jinjiu.oss.aliyuncs.com/360se10.1.1670.0.exe)  （注意：360安全浏览器其他版本没有测试通过，可能是用法不对，也可能是客户端 bug）
2. 启用国密

![image.png](https://cdn.nlark.com/yuque/0/2022/png/26770235/1666752825317-502d7e35-d30d-4416-ba4d-2117afd8bb29.png#averageHue=%23fcfcfb&clientId=ud5d17871-e8b4-4&crop=0&crop=0&crop=1&crop=1&height=848&id=HF2CZ&name=image.png&originHeight=1696&originWidth=1598&originalType=binary&ratio=1&rotation=0&showTitle=false&size=269996&status=done&style=none&taskId=u2c571e1d-e530-4211-b3c6-a4c3cdff6f8&title=&width=799)

3. 配置信任的根证书

创建文件 `C:\Users\Administrator\AppData\Roaming\360se6\User Data\Default\gmssl\ctl\ctl.dat`<br />文件内容：
```bash
-----BEGIN CERTIFICATE-----
MIICZTCCAgugAwIBAgIUGE5cdCFARYgAXNod7aHnThArA4YwCgYIKoEcz1UBg3Uw
gYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJCSjEQMA4GA1UEBwwHSGFpRGlhbjEl
MCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hub2xvZ3kgTFRELjEVMBMGA1UECwwM
U09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0IENBIChTTTIpMB4XDTE5MDUyMzAy
NDU0OFoXDTIzMDcwMTAyNDU0OFowgYIxCzAJBgNVBAYTAkNOMQswCQYDVQQIDAJC
SjEQMA4GA1UEBwwHSGFpRGlhbjElMCMGA1UECgwcQmVpamluZyBKTlRBIFRlY2hu
b2xvZ3kgTFRELjEVMBMGA1UECwwMU09SQiBvZiBUQVNTMRYwFAYDVQQDDA1UZXN0
IENBIChTTTIpMFkwEwYHKoZIzj0CAQYIKoEcz1UBgi0DQgAEyJlpzUrZ7HgViOep
/yGUfPfnbb3zcJAShtW48ZnjJ5/x+uC5o6bT53QM/MV6N+ZWQkiO1f2ejZ0ApvG0
fHvWI6NdMFswHQYDVR0OBBYEFMXy6LXt68nqzLu9nn5r2AyUvmUxMB8GA1UdIwQY
MBaAFMXy6LXt68nqzLu9nn5r2AyUvmUxMAwGA1UdEwQFMAMBAf8wCwYDVR0PBAQD
AgEGMAoGCCqBHM9VAYN1A0gAMEUCIQCPjXZprl4C0AWvyVAoFx34aVFvvYFhEioJ
Dqz727tUTgIgdnRzfwdTJHkfIin8Ksr1+xMblSVC8+X8e+yONW9YoFg=
-----END CERTIFICATE-----
```

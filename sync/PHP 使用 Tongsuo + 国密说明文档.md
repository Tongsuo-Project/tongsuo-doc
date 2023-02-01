<a name="faFoO"></a>
## 编译
我们从 php-src fork 了官方仓库到 Tongsuo-Project 中，然后对其适配了 Tongsuo 和支持国密，相关 patch 移步：[https://github.com/Tongsuo-Project/php-src/pull/2](https://github.com/Tongsuo-Project/php-src/pull/2) ，目前仅在 8.1 版本上支持 Tongsuo + 国密，之后的版本以后再支持。如果是自己从php官方下载的代码，则只需要打上支持国密的 path 即可。

- php 二进制编译和安装，参考官方文档即可
- openssl.so 扩展编译：（注意：configure 脚本指定的路径更改成自己系统安装 tongsuo 的路径）
```bash
cd ext/openssl/
OPENSSL_CFLAGS='-I/opt/tongsuo/include' OPENSSL_LIBS='-L/opt/tongsuo/lib64 -lssl -lcrypto' ./configure --with-openssl --with-php-config=/opt/php-81/bin/php-config
make -j
sudo make install
```
<a name="b51aX"></a>
## 配置

- php.ini，路径：`/path/to/php/lib/php.ini`（改成自己安装的路径）
```php
extension = openssl.so
error_log = var/log/php_error.log
```

- www.conf，路径：`/path/to/php/etc/php-fpm.d/www.conf`（改成自己安装的路径），把 www.conf.default 的后缀 .default 去掉，把 `listen = 127.0.0.1:9000`改成未被占用的端口，使用默认的9000端口也可以。
- php-fpm.conf，路径：`/path/to/php/etc/php-fpm.conf`（改成自己安装的路径），把 php-fpm.conf.default 的后缀 .default 去掉即可。
- 启动 php-fpm：`sudo /path/to/php/sbin/php-fpm`，启动没有报错即可。
- 配置 nginx/tengine 代理：
```bash
   location ~ [^/]\.php(/|$) {  
        # php 文件要放到如下 root 指定的目录
        root /var/www/php;                                              
        fastcgi_param HTTP_PROXY "";                        
        fastcgi_param SCRIPT_NAME        $fastcgi_script_name;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
        fastcgi_pass 127.0.0.1:9000;                       
        fastcgi_index index.php;                           
        include fastcgi_params;                                   
    }
```
<a name="PJ2RO"></a>
## 用法
<a name="fwuGS"></a>
### server 端开启国密功能
```php
<?php
// Hello World! NTLS HTTP Server.

echo "starting...";

function make_cert($common_name, $key, $path) {
    // Certificate data:
    $dn = array(
        "countryName" => "CN",
        "stateOrProvinceName" => "Zhejiang",
        "localityName" => "Hangzhou",
        "organizationName" => "AntGroup",
        "organizationalUnitName" => "Tongsuo Team",
        "commonName" => $common_name,
        "emailAddress" => "jinjiu@example.com"
    );

    $csr = openssl_csr_new($dn, $key);
    if (!$csr) {
        echo "openssl_csr_new error.";
        return false;
    }
    $cert = openssl_csr_sign($csr, null, $key, 365);
    if (!$cert) {
        echo "openssl_csr_sign error.";
        return false;
    }

    return openssl_x509_export_to_file($cert, $path);
}

function make_key($key_type, $path) {
    $key = openssl_pkey_new(array(
        "private_key_type" => $key_type,
    ));

    if (openssl_pkey_export_to_file($key, $path)) {
        return $key;
    }

    return null;
}

$server_sign_key_path = "/tmp/server_sign_key.pem";
$server_sign_cert_path = "/tmp/server_sign_cert.pem";
$server_enc_key_path = "/tmp/server_enc_key.pem";
$server_enc_cert_path = "/tmp/server_enc_cert.pem";
$client_sign_key_path = "/tmp/client_sign_key.pem";
$client_sign_cert_path = "/tmp/client_sign_cert.pem";
$client_enc_key_path = "/tmp/client_enc_key.pem";
$client_enc_cert_path = "/tmp/client_enc_cert.pem";

$server_sign_key = make_key(OPENSSL_KEYTYPE_SM2, $server_sign_key_path);
if (!$server_sign_key) {
    echo "make server sign key error: " . $server_sign_key_path;
    exit(-1);
}

if (!make_cert("ntls-server-sign", $server_sign_key, $server_sign_cert_path)) {
    echo "make server sign cert error: " . $server_sign_cert_path;
    exit(-1);
}

$server_enc_key = make_key(OPENSSL_KEYTYPE_SM2, $server_enc_key_path);
if (!$server_enc_key) {
    echo "make server enc key error: " . $server_enc_key_path;
    exit(-1);
}

if (!make_cert("ntls-server-enc", $server_enc_key, $server_enc_cert_path)) {
    echo "make server enc cert error: " . $server_enc_cert_path;
    exit(-1);
}

$client_sign_key = make_key(OPENSSL_KEYTYPE_SM2, $client_sign_key_path);
if (!$server_sign_key) {
    echo "make client sign key error: " . $server_sign_key_path;
    exit(-1);
}

if (!make_cert("ntls-client-sign", $client_sign_key, $client_sign_cert_path)) {
    echo "make client sign cert error: " . $client_sign_cert_path;
    exit(-1);
}

$client_enc_key = make_key(OPENSSL_KEYTYPE_SM2, $client_enc_key_path);
if (!$server_enc_key) {
    echo "make client enc key error: " . $client_enc_key_path;
    exit(-1);
}

if (!make_cert("ntls-client-enc", $client_enc_key, $client_enc_cert_path)) {
    echo "make client enc cert error: " . $client_enc_cert_path;
    exit(-1);
}

$server_sign_key_path = "/var/www/php/ntls-cert/ss.key";
$server_sign_cert_path = "/var/www/php/ntls-cert/ss.crt";
$server_enc_key_path = "/var/www/php/ntls-cert/se.key";
$server_enc_cert_path = "/var/www/php/ntls-cert/se.crt";

$context = stream_context_create([ 'ssl' => [
    'verify_peer'       => true,
    'verify_peer_name'  => false,
    'allow_self_signed' => true,
    //'ciphers' => 'ECDHE-SM2-WITH-SM4-SM3',
    'cafile' => '/var/www/php/ntls-cert/ca.pem',
    'sign_cert' => $server_sign_cert_path,
    'sign_key' => $server_sign_key_path,
    'enc_cert' => $server_enc_cert_path,
    'enc_key' => $server_enc_key_path,
    'verify_depth'      => 2 ]]);

if (!$context) {
    echo "stream_context_create error...\n";
    exit(-1);
}

echo "ssl test\n";

// Create the server socket
$server = stream_socket_server('ntls://0.0.0.0:9002', $errno, $errstr, STREAM_SERVER_BIND|STREAM_SERVER_LISTEN, $context);

while (true) {
    $buffer = '';

    print "waiting...";
    $client = stream_socket_accept($server);
    print "accepted " . stream_socket_get_name($client, true) . "\n";

    if ($client) {
        // Read until double CRLF
        while (!preg_match('/\r?\n\r?\n/', $buffer))
            $buffer .= fread($client, 2046);

        // Respond to client
        fwrite ($client,  "HTTP/1.1 200 OK\r\n"
                        . "Connection: close\r\n"
                        . "Content-Type: text/html\r\n"
                        . "\r\n"
                        . "Hello World! " . microtime(true)
                        . "\r\n<pre>{$buffer}</pre>");
        fclose($client);
    } else {
        print "error.\n";
    }
}

?>
```
<a name="lT3xn"></a>
### client 端使用国密协议通信
```php
<?php

$stream_context = stream_context_create([ 'ssl' => [
    'verify_peer'       => false,
    'verify_peer_name'  => false,
    'allow_self_signed' => true,
    'ciphers' => 'ECDHE-SM2-WITH-SM4-SM3',
    'sign_cert' => '/var/www/php/ntls-cert/client_sign.crt',
    'sign_key' => '/var/www/php/ntls-cert/client_sign.key',
    'enc_cert' => '/var/www/php/ntls-cert/client_enc.crt',
    'enc_key' => '/var/www/php/ntls-cert/client_enc.key',
    'verify_depth'      => 0 ]]);

echo "ntls test\n";

$fp = stream_socket_client('ntls://127.0.0.1:9002', $errno, $errstr, 30, STREAM_CLIENT_CONNECT, $stream_context);

fwrite($fp, "GET /ssl HTTP/1.1\r\nHost: a.com\r\n\r\n");
error_log("start read!\n", 3, "/tmp/php-errors.log");
while (!feof($fp)){
    stream_set_timeout($fp, 2);
    $ret = fgets($fp, 128);
    echo $ret;
    error_log("resp: " . $ret, 3, "/tmp/php-errors.log");
    if (strlen($ret) == 0) {
        break;
    }
}

fclose($fp);
error_log("end read!", 3, "/tmp/php-errors.log");

?>
```

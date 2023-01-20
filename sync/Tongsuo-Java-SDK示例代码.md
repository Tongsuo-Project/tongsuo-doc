<a name="clPgv"></a>
### 客户端，TLS 1.3 + 商密套件
<a name="XEzDA"></a>
#### TongsuoClient.java
src/main/java/demo/TongsuoClient.java如下：<br />测试证书[chain.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2-root.crt)。
```java
package demo;

import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.security.SecureRandom;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509TrustManager;

import net.tongsuo.TongsuoProvider;
import net.tongsuo.TongsuoX509Certificate;


public class TongsuoClient {

    public static void main(String[] args)throws Exception{
    
        //设置服务端ip和端口
        String ip = "127.0.0.1";
        int port = 443;
        //加密套件，多个以:分隔
        String ciperSuites = "TLS_SM4_GCM_SM3:TLS_SM4_CCM_SM3";
        //ca证书，不传则不校验ca证书
        String caCert = "chain.crt";

        //构建ssl连接上下文
        SSLContext sslContext = SSLContext.getInstance("TLSv1.3", new TongsuoProvider());

        X509Certificate ca = null;
        if(caCert != null && "".equals(caCert.trim())){
            ca = TongsuoX509Certificate.fromX509PemInputStream(new FileInputStream(new File(caCert)));
        }

        final X509Certificate caCertificate = ca;
        TrustManager[] tms = new TrustManager[] { new X509TrustManager() {
            @Override
            public X509Certificate[] getAcceptedIssuers() {
                return new X509Certificate[0];
            }

            @Override
            public void checkClientTrusted(X509Certificate[] certs, String authType) {
            }

            @Override
            public void checkServerTrusted(X509Certificate[] certs, String authType)throws CertificateException{
                //ca证书校验
                if(caCertificate != null){
                    for (X509Certificate cert : certs) {
                        try {
                            cert.checkValidity();
                            cert.verify(caCertificate.getPublicKey());
                        } catch (Exception e) {
                            e.printStackTrace();
                            throw new CertificateException(e);
                        }
                    }
                }
            }

        } };

        sslContext.init(null,tms, new SecureRandom());

        System.out.println("Client SSL context init success...");

        //构建socket工厂
        SSLSocketFactory socketFactory = sslContext.getSocketFactory();
        SSLSocket sslSocket = (SSLSocket)socketFactory.createSocket(ip, port);

        //设置加密套件
        if(ciperSuites != null && !"".equals(ciperSuites.trim())){
            String[] ciperSuiteArray = ciperSuites.split(":");
            sslSocket.setEnabledCipherSuites(ciperSuiteArray);
        }

        //向服务端发送消息
        BufferedWriter out = new BufferedWriter(new OutputStreamWriter(sslSocket.getOutputStream()));
        out.write("GET / HTTP/1.0\r\n\r\n");
        out.flush();

        System.out.println("client ssl send msessage success...");

        //读取服务端响应
        BufferedInputStream streamReader = new BufferedInputStream(sslSocket.getInputStream());
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(streamReader, "utf-8"));
        String line = null;
        while((line = bufferedReader.readLine())!= null){
            System.out.println("client receive server data:" + line);
        }

        //等待服务端响应
        while (true) {
            try {
                sslSocket.sendUrgentData(0xFF);
                Thread.sleep(1000L);
                System.out.println("client waiting server close");
            } catch (Exception e) {
                bufferedReader.close();
                out.close();
                sslSocket.close();
            }
        }

    }
}

```
<a name="syJji"></a>
#### maven配置
使用maven构建，pom.xml如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>demo</groupId>
    <artifactId>TongsuoClient</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
      	    <groupId>net.tongsuo</groupId>
            <artifactId>tongsuo-openjdk</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.69</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
			<goals>
                            <goal>shade</goal>
		        </goals>
                        <configuration>
			    <filters>
                                <filter>
				    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>demo.TongsuoClient</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
<a name="u7peb"></a>
#### 构建和运行
```bash
mvn clean package

java -cp target/TongsuoClient-0.1.0.jar demo.TongsuoClient
```
<a name="ku6UA"></a>
### 服务端，TLS 1.3 + 商密套件 + SM2证书
<a name="lOhUA"></a>
#### TongsuoServer.java
src/main/java/demo/TongsuoServer.java<br />测试证书：[sm2.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2-first-crt.pem), [sm2.key](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2-first-key.pem)，[chain.crt](https://github.com/Tongsuo-Project/Tongsuo/blob/master/test/certs/sm2-root.crt)。
```java
package demo;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.security.KeyFactory;
import java.security.KeyStore;
import java.security.PrivateKey;
import java.security.SecureRandom;
import java.security.cert.X509Certificate;
import java.security.spec.PKCS8EncodedKeySpec;

import javax.net.ssl.KeyManager;
import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLServerSocket;
import javax.net.ssl.SSLServerSocketFactory;
import javax.net.ssl.SSLSocket;
import javax.net.ssl.TrustManager;
import javax.net.ssl.TrustManagerFactory;

import org.bouncycastle.jce.provider.BouncyCastleProvider;
import net.tongsuo.TongsuoProvider;
import net.tongsuo.TongsuoX509Certificate;
import sun.misc.BASE64Decoder;


public class TongsuoServer {

    public static void main(String[] args)throws Exception{
        //加密套件
        String ciperSuites = "TLS_SM4_GCM_SM3:TLS_SM4_CCM_SM3";
        //ca证书
        String caCert = "chain.crt";
        //通信证书
        String cert = "sm2.crt";
        //私钥
        String certKey = "sm2.key";

        //上下文
        SSLContext sslContext = SSLContext.getInstance("TLSv1.3", new TongsuoProvider());

        char[] EMPTY_PASSWORD = new char[0];
        X509Certificate ca = TongsuoX509Certificate.fromX509PemInputStream(new FileInputStream(new File(caCert)));
        X509Certificate crtCert = TongsuoX509Certificate.fromX509PemInputStream(new FileInputStream(new File(cert)));

        String ecKey = pemReadFromStream(new FileInputStream(new File(certKey)));

        BASE64Decoder base64decoder = new BASE64Decoder();
        byte[] keyByte = base64decoder.decodeBuffer(ecKey);
        PKCS8EncodedKeySpec eks2 = new PKCS8EncodedKeySpec(keyByte);
        KeyFactory keyFactory = KeyFactory.getInstance("EC", new BouncyCastleProvider());
        PrivateKey privateKey = keyFactory.generatePrivate(eks2);

        //构建证书链
        X509Certificate[] chain = new X509Certificate[] {crtCert, ca};
        KeyStore ks = KeyStore.getInstance("PKCS12",new BouncyCastleProvider());
        ks.load(null);
        ks.setKeyEntry("cnnic", privateKey, EMPTY_PASSWORD, chain);
        ks.setCertificateEntry("CA", ca);

        KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");
        kmf.init(ks, EMPTY_PASSWORD);
        KeyManager[] kms = kmf.getKeyManagers();

        TrustManagerFactory tmf =
            TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        tmf.init(ks);
        TrustManager[] tms = tmf.getTrustManagers();

        //上下文初始化
        sslContext.init(kms, tms, new SecureRandom());
        System.out.println("Server SSL context init success...");

        SSLServerSocketFactory socketFactory = sslContext.getServerSocketFactory();

        SSLServerSocket serverSocket = (SSLServerSocket)socketFactory.createServerSocket(443);
        //设置加密套件
        if(ciperSuites != null && !"".equals(ciperSuites.trim())){
            String[] ciperSuiteArray = ciperSuites.split(":");
            serverSocket.setEnabledCipherSuites(ciperSuiteArray);
        }

        String [] tlsVersion = new String[1];
        tlsVersion [0] = "TLSv1.3";
        serverSocket.setEnabledProtocols(tlsVersion);

        while (true){
            SSLSocket sslSocket = (SSLSocket)serverSocket.accept();

            try {
                BufferedReader in = new BufferedReader( new InputStreamReader(sslSocket.getInputStream()) );
                BufferedWriter out = new BufferedWriter(new OutputStreamWriter(sslSocket.getOutputStream()));
                String msg = null;
                char[] cbuf = new char[1024];
                int len = 0;
                while( (len = in.read(cbuf, 0, 1024)) != -1 ){ // 循环读取输入流中的内容
                    msg = new String(cbuf, 0, len);
                    out.write(msg);
                    out.flush();
                    
                    if("Bye".equals(msg)) { // 如果检测到 "Bye" ，则跳出循环，不再读取输入流中内容。
                        break;
                    }
                    System.out.printf("Received Message --> %s \n", msg);
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }

    /**
     * 读取私钥
     * @param inputStream
     * @return
     */
    public static String pemReadFromStream(InputStream inputStream){
        InputStreamReader inputStreamReader = null;
        BufferedReader bufferedReader = null;
        StringBuilder sb = new StringBuilder();
        try {
            inputStreamReader = new InputStreamReader(inputStream);
            bufferedReader = new BufferedReader(inputStreamReader);
            String line = null;
            while ((line = bufferedReader.readLine()) != null){
                if(line.startsWith("-")){
                    continue;
                }
                sb.append(line).append("\n");
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try {
                if(bufferedReader != null){
                    bufferedReader.close();
                }
                if(inputStreamReader != null){
                    inputStreamReader.close();
                }
                if(inputStream != null){
                    inputStream.close();
                }
            }catch (Exception e){

            }
        }
        return sb.toString();
    }
}
```
<a name="XLjXl"></a>
#### maven配置
使用maven构建，pom.xml如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>demo</groupId>
    <artifactId>TongsuoServer</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
      	    <groupId>net.tongsuo</groupId>
            <artifactId>tongsuo-openjdk</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.69</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
			<goals>
                            <goal>shade</goal>
		        </goals>
                        <configuration>
			    <filters>
                                <filter>
				    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>demo.TongsuoServer</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
<a name="RR7y7"></a>
#### 构建和运行
```bash
mvn clean package

java -cp target/TongsuoServer-0.1.0.jar demo.TongsuoServer
```

﻿# _Transprivacy_
_A Transparent Framework for Privacy Enhancement_


## _System Arch_
- Web Application: [Spring Boot Framework 2.1](https://github.com/spring-projects/spring-boot)
- Database: MySQL
- Blockchain: [Ganache v.2.1.0](https://www.trufflesuite.com/ganache)、[Infura](https://infura.io/)、[Geth (Go Ethereum)](https://github.com/ethereum/go-ethereum/wiki/Installing-Geth)


## _System Arch Picture_
- Import `arch.drawio` to [draw.io](https://www.draw.io/)
    1. 前兩頁可參考 `protocol.docx`
    2. 第三頁為架構圖


## _How to Run_
```properties
# 遠端連線
login 10.32.0.181, 10.32.0.182, 10.32.0.185 (帳密請跟老師申請)

# 開資料庫 (at 10.32.0.181, 10.32.0.185)
sudo service mysql status               # 先看開了沒
sudo service mysql start

# 開區塊鏈 (請擇一) (at 10.32.0.181)
! Geth
sudo -i                                 # 取得 root 權限
cd ../usr/local/var/Sinica/2019_Geth
sudo ./run.sh                           # 進入 console
miner.start(1)                          # 開始挖礦後 Blockchain 才會接收交易

! Infura
! 請直接改 source code，將 W3j.java 中 new HttpService 的參數修改成新創立 Project 所得到的 ENDPOINT Key
Web3j web3j = Web3j.build(new HttpService("https://ropsten.infura.io/v3/e2c4f89bf4de4d13ab79fb1767e2d0de"));
! 接著重新打包成 mediator-0.0.1-SNAPSHOT-infura.jar

另外，若想要測IPFS，打開後就能接上了
ipfs daemon

# 開伺服器 (at 10.32.0.181, 10.32.0.185)
sudo -i 
cd /home/ychsu

! 擇一
java -jar mediator-0.0.1-SNAPSHOT-geth.jar     # at 10.32.0.181
java -jar mediator-0.0.1-SNAPSHOT-infura.jar   # at 10.32.0.181

java -jar controller-0.0.1-SNAPSHOT.jar        # at 10.32.0.185

# 執行 (at 10.32.0.182)
firefox
then connect to
    10.32.0.181:8443/signup
    10.32.0.185:8444/controller
```


## _Source Code Arch_
```properties
# 放置 VM 上的資料分別如下

# 10.32.0.181
    /mediator
    mediator-0.0.1-SNAPSHOT-geth.jar
    mediator-0.0.1-SNAPSHOT-infura.jar
    
    arch.drawio
    protocol.docx
    README.md
    swagger.json
    transprivacy.7z (all files)
# 10.32.0.182
    null
# 10.32.0.185
    /controller
    controller-0.0.1-SNAPSHOT.jar
# 10.32.0.186
    null
```

<img align="right" width="300" height="600" src="https://github.com/rookieyc/transprivacy/blob/master/Files/source_code_arch.png">

接下來以 `/mediator` 為例，Controller 同理

1. iis.sinica.ychsu.mediator
    - `db` : 資料庫
        - Entity 可以直接對應到資料表中的欄位
        - Repository 控制資料存取
        - Service 作為 Controller 與 Dao(Repository) 的中介層
        - P.S. 若資料表有改動，Entity 記得也要改；若新增資料表，記得一系列建立 Entity > Repository > Service
    - `jwt` : JSON Web Token
        - JwtRequestFilter 為每筆請求做 Filter
        - JwtTokenUtil 做 JWT 的驗證、生成等
    - `parameter` : RESTFUL API 的參數
    - `web3j` : Blockchain
        - xxxContract 都是自動產生的
        - 與區塊鏈相關的設定都在 W3j
    - mediator : MVC 中的 Controller
    - WebSecurityConfig : Security 設定 (e.g. 部分路徑為 public，部分需要 token 才可以進行存取)
2. resources
    - `keypairs` : 加解密、簽章用的金鑰
    - `solidity` : solidity
    - `ssl` : MYSQL ssl
    - `static` : js
    - `templates` : html
    - `tls` : web tls
    - application.properties : 專案設定檔
3. target
    - mediator-0.0.1-SNAPSHOT.jar: 打包後的檔案 (請依照打包的程式碼，自行在檔名後加上 geth/infura 以區分)


## API 測試
- Import `swagger.json` to [Swagger](https://editor.swagger.io/)

    1. Log in 取得 token，前端會直接加至於 http header，所以 `Swagger` 會回傳網頁檔，而不是字串值
    2. Bearer 的部分 `Swagger` 跑不了，所以 log in 拿到 token 後，請改用 [Postman 7.3.4](https://www.getpostman.com/) 繼續測試


## 說明
- 因為 VM 空間有限 & 筆電環境比較好操作，所以我是先在筆電(Windows)上開發並測試，打包成 `.jar` 後再送至 VM(Ubuntu) 執行

    **Note:** 建議 `How to Run` 執行完後沒問題的話，直接在 VM 上裝 IDE ，之後就可以直接 run，不用再來回跑

    **Note:** 以下把東西傳到 VM 上的部分，如果已經改成全部都在 VM 上操作，請忽略

1. 開發環境
    - Windows 10
    - JDK 11.0.4
2. 部署環境
    - Ubuntu 18.04 (Virtual Machine)
    - JDK 11
3. `金鑰`、`憑證`、`合約`、`區塊鏈`、`資料庫`的部分如果有要修改，請參考各自的說明

**Note:** 如果要在 Windows 上跑，請先自行安裝所需的 JDK、MySQL、Web3j、Geth、Ganache 等


## 金鑰說明
- [OpenSSL 1.1.1.c](http://slproweb.com/products/Win32OpenSSL.html)
- 512 Bytes for ENC, 256 Bytes for SIG

```properties
# Generate Mediator's keypairs(.der) for Encryption and Decryption
genrsa -out mediator-priv-key.der 4096
rsa -in mediator-priv-key.der -pubout -outform der -out mediator-pub-key-pkcs1-512.der # 沒指定 -outform der 的話，我們可以看公鑰，但不符合規範，所以必須加
pkcs8 -topk8 -in mediator-priv-key.der -out mediator-priv-key-pkcs8.der –nocrypt # 完成後，請手動把頭尾的說明刪掉、改成只有一行，接著重新命名成 mediator-priv-key-pkcs8-512.der

! PK: mediator-pub-key-pkcs1-512.der    (can't see)
! SK: mediator-priv-key-pkcs8-512.der   (只有一行)

# Generate Controller's keypairs(.pem) for Signature and Verification (e.g. Controller A)
genrsa -out controllerA-priv-key-pkcs1-256.pem 2048
rsa -in controllerA-priv-key-pkcs1-256.pem -pubout -out controllerA-pub-key-pkcs1-256.pem

! PK: controllerA-pub-key-pkcs1-256.pem
! SK: controllerA-priv-key-pkcs1-256.pem
```


## 憑證說明
- keytool
- Self-signed Certificate (之後記得花錢買正式的)
- 7月原先是用 `.p12`，但 `maven.plugin's filter` 會有問題，所以8月開始改用 `.pfx`
- 以下以產生 Mediator Server's Certificate 為例，要做 Controller Server 憑證的話，只要把 IP 的.181 改成 .185 即可

**Note:** 若有修改過憑證，記得 VM 上的瀏覽器要匯入新產生的 `rootca.pem` ，以順利使用 TLS

```properties
# Generate Root CA's Keystore(.pfx) which contains the PK, SK and certificate
keytool -genkeypair -keyalg RSA -keysize 2048 -alias root-ca -dname "CN=Root CA, OU=sinica, O=iis, L=taipei, ST=taiwan, C=zh" -ext BC:c=ca:true -ext KU=keyCertSign -validity 3650 -keystore rootca.pfx -storepass secret -keypass secret -storetype pkcs12

# Export Root CA's certificate in file(.pem)
keytool -exportcert -keystore rootca.pfx -storepass secret -alias root-ca -rfc -file rootca.pem

! Certificate: rootca.pfx、rootca.pem

# Generate Server's Keystore(.pfx)
keytool -genkeypair -keyalg RSA -keysize 2048 -alias server -dname "CN=ychsu, OU=sinica, O=iis, L=taipei, ST=taiwan, C=zh" -ext BC:c=ca:false -ext EKU:c=serverAuth -ext "SAN:c=DNS:localhost,IP:127.0.0.1,IP:10.32.0.181" -validity 3650 -keystore server.pfx -storepass secret -keypass secret -storetype pkcs12

# Signing request(.csr) for server certificate
keytool -certreq -keystore server.pfx -storepass secret -alias server -keypass secret -file server.csr

# Export Server's certificate(.pem)
keytool -gencert -keystore rootca.pfx -storepass secret -infile server.csr -alias root-ca -keypass secret -ext BC:c=ca:false -ext EKU:c=serverAuth -ext "SAN:c=DNS:localhost,IP:127.0.0.1,IP:10.32.0.181" -validity 3650 -rfc -outfile server.pem

! Certificate: server.pfx、server.pem
! Request:     server.csr

# Chain of trust
keytool -importcert -noprompt -keystore server.pfx -storepass secret -alias root-ca -keypass secret -file rootca.pem
keytool -importcert -noprompt -keystore server.pfx -storepass secret -alias server  -keypass secret -file server.pem
```


## 合約說明
- 可至 [Remix](https://remix.ethereum.org) 撰寫合約
- 編譯成功後於頁面右方下載 `.abi`(defines the smart contract methods) 與 `.bin`(for EVM)
- 若不使用 Remix，寫好合約後可使用 [Solc](https://solidity.readthedocs.io/en/v0.4.24/installing-solidity.html) 編譯出 `.abi` 及 `.bin`
```properties
C:/Users/hyc/Desktop> solc C:/Users/hyc/Desktop/tmp.sol --bin --abi --optimize -o C:/Users/hyc/Desktop
```
- 下載 [Web3j's Command Line Tools](https://github.com/web3j/web3j/releases)
    - 建議下載 3.5.0；若下載最新版 4.3.0，測試過會發生不只以下提到的更多問題
- 使用 Web3j 對 `.abi`、`.bin` 產出對應的 `.java`
```properties
C:/Users/hyc/Desktop/web3j-4.3.0/bin> web3j solidity generate -b C:/Users/hyc/Desktop/tmp.bin -a C:/Users/hyc/Desktop/tmp.abi -o C:/Users/hyc/Desktop/ -p iis.sinica.ychsu.server
```
**Note:** 若 Server 在執行時有遇到此問題 `Transaction has failed with status: null. Gas used: xxxxx. (not-enough gas?)`

**Solution:** 將 `private static final String BINARY` 變數移除多餘的欄位，只留下 **數字** 的部分則可 (經測試 `3.5.0` 及 `4.3.0` 兩版本皆存在此問題)
    


## 區塊鏈說明
- Blockchain 測試環境有以下三種

1. 開發框架 Truffle 的 `Ganache`，為一本地的私有區塊鏈，特點為交易不需要消耗 gas，並已內建多組帳號提供互動
2. 官方測試鏈 Ethereum Test Networks: Ropsten, Rinkeby 等。要用官方測試鏈測鏈需要藉助 `Infura`，註冊並創建一個 project 後，只要拿 `Endpoint` 的 `Key` 並將 Web3j 的建立方式稍作修改就可以
    ```properties
    Web3j web3j = Web3j.build(new HttpService("https://ropsten.infura.io/<ropsten-endpoint-key>"));
    ```
    - 有一點需要注意，Infura 不提供 `filter` 的功能，因此 `activateFilter(web3j)` 的部分不會執行
    - 若執行成功，可以到 [ropsten.etherscan](https://ropsten.etherscan.io/)，用自己的 address 去查詢交易的情形，應該也可以看出印出的訊息 `deployedAddress` 和 Etherscan 顯示 contract 的建立位置是相同的
3. 自架私有鏈，使用 `Geth`
```properties
# 區塊鏈已被初始化、執行，因此只要 `run.sh` 便可開始
sudo run.sh
    
# 進入 console 後，開始挖礦，讓交易能夠被處理
miner.start(1)
    
# 停止挖礦或是退出
miner.stop()
exit

# 若欲從頭重新啟動私有鏈，編寫完創世區塊 genesis.json 後
geth --datadir <data directory> init genesis.json
    
# 若不確定私有鏈中是否已有可使用之帳戶，先開啟區塊鏈
init.sh
    
# 進入 console 後查詢是否有帳戶，及帳戶中是否有錢可以進行交易
eth.accounts
eth.getBalance(eth.accounts[0])
    
# 若沒有帳戶，新增一個，並**記下帳號密碼**
personal.newAccount()
    
# 將帳戶紀錄在 `.sh` 中
account=<created-address>
    
# 可將密碼存下，並在 `.sh` 中紀錄密碼位置
--password "directory"
```
    


## 資料庫說明
- 10.32.0.185
```properties
# DB Name：controller

# Controller A (e.g. google)
CREATE TABLE `google_account` (
  `id` int(11) NOT NULL,
  `userid` varchar(50) COLLATE utf8_bin NOT NULL,
  `password` varchar(50) COLLATE utf8_bin NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

# Controller B (e.g. facebook)
CREATE TABLE `facebook_account` (
  `id` int(11) NOT NULL,
  `userid` varchar(50) COLLATE utf8_bin NOT NULL,
  `password` varchar(50) COLLATE utf8_bin NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```

- 10.32.0.181
```properties
# DB Name：mediator

CREATE TABLE `membership` (
  `id` int(11) NOT NULL,
  `account` varchar(50) COLLATE utf8_bin NOT NULL,
  `password` varchar(700) COLLATE utf8_bin NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

CREATE TABLE `signature_tmp` (
  `id` int(11) NOT NULL,
  `signature` varchar(700) COLLATE utf8_bin NOT NULL,
  `times` int(11) NOT NULL,
  `valid` tinyint(1) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

CREATE TABLE `data` (
  `id` int(11) NOT NULL,
  `account` varchar(50) COLLATE utf8_bin NOT NULL,
  `DBID_A` varchar(50) COLLATE utf8_bin DEFAULT NULL,
  `DBID_B` varchar(50) COLLATE utf8_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

CREATE TABLE `preference` (
  `id` int(11) NOT NULL,
  `account` varchar(50) COLLATE utf8_bin NOT NULL,
  `controller` varchar(50) COLLATE utf8_bin NOT NULL,
  `option_1` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_2` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_3` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_4` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_5` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_6` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_7` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_8` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_9` varchar(30) COLLATE utf8_bin DEFAULT NULL,
  `option_10` varchar(30) COLLATE utf8_bin DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```


## _For Windows Developer_
- Win10 開發與打包
1. 先建環境
    - 建立資料庫，並依照`資料庫說明`建立資料表 (該欄位會對應`iis/sinica/ychsu/server/database/*`路徑下的`xxxEntity.java`)
    - 依照建立的資料庫，修正`application.properties`中資料庫的`url`與`username、password`
    - 將`rootca.pem`匯入瀏覽器，以順利使用 TLS
    - 啟動 Ganache 並選擇 QuickStart
    
    **Note:** 接下來以 Intellij IDE 打包 Mediator 進行說明

2. Run (Run/Debug Configuration 請選擇 Spring boot)
3. 啟動後，可透過瀏覽器或 Swagger 進行測試
4. 測試無誤後，接下來進行打包
    - Run (Run/Debug Configuration 請選擇 Maven)，它會在 /mediator/target 下產出 `mediator-0.0.1-SNAPSHOT.jar`
    - 打包時如果希望順便進行測試，Command Line 請打上 `clean package`，反之，請打 `clean package -DskipTests` (測試的話環境也都要開好)

- Ubuntu 部署與執行
1. 先連線
    - 使用 [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 將檔案傳至 VM
    - 先將目錄切換至 pscp.exe 的下載位置
    - 執行 `pscp D:/transprivacy/server/target/mediator-0.0.1-SNAPSHOT.jar ychsu@10.32.0.181:/home/ychsu`.
    - 完成後，VM 的該位置下就會有一份 `mediator-0.0.1-SNAPSHOT.jar`
2. 執行 
    - 使用 [PuTTy](https://www.putty.org/) 連線至 10.32.0.181，成功後到家目錄底下 (預設應該就是)
    - 執行 `java -jar mediator-0.0.1-SNAPSHOT.jar`
3. 測試
    - 啟動 [Xming](https://sourceforge.net/projects/xming/)
    - 使用 PuTTy 連線至 10.32.0.182
    - PuTTy 連線前，Connection SSH X11 中，Enable X11 forwarding 記得打勾
    - 連線成功後，啟動瀏覽器 `ychsu@privacyvm-3:~$ firefox`
    - 測試 (https://10.32.0.181:8443/)

**Note:** 不論是哪個環境，執行前記得資料庫跟區塊鏈都要先開

**Note:** pscp、PuTTy、Xming 的組合可以改用 MobaXterm + enable X11 取代 (2019.08.21)


## _Ubuntu Install JDK_
- Java 11 開始不能直接安裝了，要先至 Oracle 官網登入並下載 jdk-11.0.4_linux-x64_bin.tar.gz
```properties
# 刪掉之前裝的雜物 (可以跳過)
apt list --installed
sudo apt remove oracle-java11-installer-local
sudo apt purge oracle-java11-installer

# 我是先用 Win10 載再丟過去 (直接在 VM 載的話可以跳過)
pscp C:/Users/hyc/Desktop/jdk-11.0.4_linux-x64_bin.tar.gz ychsu@10.32.0.181:/home/ychsu/jdk-11.0.4_linux-x64_bin.tar.gz

# 在這裡創個資料夾(/var/cache/oracle-jdk11-installer-local)，並把剛下載的檔案丟過去
sudo mv /home/ychsu/jdk-11.0.4_linux-x64_bin.tar.gz /var/cache/oracle-jdk11-installer-local/jdk-11.0.4_linux-x64_bin.tar.gz

# 安裝
sudo add-apt-repository ppa:linuxuprising/java
sudo apt-get update
sudo apt install oracle-java11-installer-local
sudo apt install oracle-java11-set-default-local

# 檢查
java -version
```


## _JDBC use SSL_
```properties
# Generate CA Certificate
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3600 -key ca-key.pem -out ca.pem

# Create Server Certifcate and Sign with CA
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem

# Create Client certificate and Sign with CA
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout client-key.pem -out client-req.pem
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem

# Verify the Certificate
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem

# 在 my.cnf 中加入以下設定 (注意:剛剛產生的憑證也需要複製一份至該目錄底下)
vim /etc/mysql/my.cnf

[client]
ssl-ca=/etc/mysql/ssl/ca.pem
ssl-cert=/etc/mysql/ssl/client-cert.pem
ssl-key=/etc/mysql/ssl/client-key.pem
[mysqld]
ssl-ca=/etc/mysql/ssl/ca.pem
ssl-cert=/etc/mysql/ssl/server-cert.pem
ssl-key=/etc/mysql/ssl/server-key.pem

# Restart the MySQL service to take new config (注意:這裡不要用 sudo service mysql start)
/etc/init.d/mysql restart

# Login and Verify SSL config
mysql -u transprivacy -p -h 127.0.0.1
mysql> status
mysql> SHOW VARIABLES LIKE '%ssl%';

# 到目前為止，MYSQL 端設定完成，接下來是 Server 端

# Generate trustore certificate
keytool -importcert -file ca.pem -keystore truststore.jks -storepass mypass
Trust this certificate? [no]: yes

# Generate Keystore Certificate for client certificate
openssl pkcs12 -export -in client-cert.pem -inkey client-key.pem -passout pass:mypass -out client-keystore.p12
keytool -importkeystore -srckeystore client-keystore.p12 -srcstoretype pkcs12 -srcstorepass mypass -destkeystore keystore.jks -deststoretype JKS -deststorepass mypass

# JDBC connection with useSSL=true
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/controller?useUnicode=true&characterEncoding=UTF-8&characterSetResults=UTF-8&verifyServerCertificate=true&useSSL=true&requireSSL=true&clientCertificateKeyStoreUrl=classpath:ssl/keystore.jks&clientCertificateKeyStorePassword=mypass&trustCertificateKeyStoreUrl=classpath:ssl/truststore.jks&trustCertificateKeyStorePassword=mypass
```


## 其他
- 若空間不夠，可以搜尋 `.ethash/` ，此資料夾為 Ethereum 做挖礦時的 PoW 所用，除 181 之外皆可刪除
- 目前為了方便 Controller A & B 的資料庫是建在一起，記得改一人一個
- 如果要一個 Server 要同時連兩個以上的 DB (e.g. MYSQL、AWS)，~~請修正`DataSourceConfig`，並仿照`PrimaryConfig.java`建立`TertiaryConfig.java`.~~ 
- 存在 DB 中的密碼有加密
- 如過要從外部位址用 http 去連 VM，必須先解決 
    1. spring boot embedded tomcat auto transform ipv4 to v6 
    2. firewall only allow port 22 or ip authorization


## _For Me_
```properties
# JAR
java -jar controller-0.0.1-SNAPSHOT.jar -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Addresses=true

# pscp
cd pscp's location

pscp C:/Users/hyc/Desktop/controller.sql ychsu@10.32.0.185:/home/ychsu

pscp D:/transprivacy/mediator/target/mediator-0.0.1-SNAPSHOT.jar ychsu@10.32.0.181:/home/ychsu
pscp D:/transprivacy/controller/target/controller-0.0.1-SNAPSHOT.jar ychsu@10.32.0.185:/home/ychsu

# MYSQL
sudo service mysql status (start、stop)

sudo mysql
mysql -u transprivacy -p -h 127.0.0.1
quit

show databases;
use controller;
show tables;
SELECT * from preference;

DROP DATABASE controller;
DELETE FROM preference WHERE id=3;

select host, user from mysql.user;
SHOW GRANTS FOR 'testuser'@'localhost';

# 7z
7z x mediator.7z
rm -r *     # 連同資料夾全清

# MYSQL init
create database controller;
sudo mysql -u root -p controller < /home/ychsu/controller.sql

CREATE USER 'transprivacy'@'127.0.0.1' IDENTIFIED BY 'intern';
GRANT ALL PRIVILEGES ON mediator.* TO 'transprivacy'@'127.0.0.1';
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON controller.* TO 'transprivacy'@'127.0.0.1';
FLUSH PRIVILEGES;

# Linux clear log
rm -rf /var/log/journal/e58b026cdf2c47798bacf1f0719baf5c
 
```


## _MobaXterm enable X11 Forwarding_
```properties
# install
sudo yum install xorg-x11-xauth xorg-x11-fonts-* xorg-x11-font-utils xorg-x11-fonts-Type1

# vim /etc/ssh/ssh_config
ForwardX11 yes

SendEnv LANG LC_*
HashKnownHosts yes
GSSAPIAuthentication yes
AddressFamily inet

# vim /etc/ssh/sshd_config
ChallengeResponseAuthentication no

UsePAM yes

AllowTcpForwarding yes
X11Forwarding yes
X11UseLocalhost no
PrintMotd no

AcceptEnv LANG LC_*
Subsystem sftp  /usr/lib/openssh/sftp-server
PasswordAuthentication yes

# restart
sudo systemctl restart sshd.service
```


## _Further information_
For further information, please refer to the [here](https://github.com/tienshaoku) or [home](https://github.com/rookieyc).

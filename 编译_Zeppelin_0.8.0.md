# 编译 Zeppelin 0.8.0
> 参考：
> - Zeppelin官网build教程 http://zeppelin.apache.org/docs/0.8.0/setup/basics/how_to_build.html

### 编译前环境需求
- ##### 系统环境
```
# 无要求
# 本文档用的centos
uname -a
Linux VM_16_3_centos 3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```
- ##### 手动安装
```
Java(1.7+, 安装略)
Maven(3.1.x or higher, 安装略)
```
- ##### yum安装
```
yum -y update
yum install git
yum install fontconfig
yum install R
# 注意：官网要求单独安装npm，但node中已经自带npm，所以只需要安装最新版的node即可，不用单独安装npm。
# node一定要安装最新版本以免之后编译中报node和npm版本低的错误。
```
- ##### 最新版node安装
```
yum install -y gcc-c++ make
curl -sL https://rpm.nodesource.com/setup_10.x | sudo -E bash -
yum install nodejs
```

### 下载源码
```
git clone https://github.com/apache/zeppelin.git

# 切换到tagv0.8.0进行编译
git checkout -b mytagv0.8.0 v0.8.0
# 注意：不要编译master分支，因为master分支不稳定，而且会有很多新加的interepter不能编译通过
```

### 编译前各种代理配置

1. socks代理
    > 说明： 由于编译过程中会通过Maven下载一些墙外的包，所以需要配置代理
    >  本教程中的解决方案是在国外买一台vps或云服务器，通过ssh打socks隧道到这台服务器，然后在Maven中配置代理。
    
    ```
    vim ~/.ssh/id_rsa.pub
    #将id_rsa.pub中的内容拷贝到user@国外私服ip:~/.ssh/authorized_keys里
    #本地将socks起在1080端口
    ssh -qTfnN -D 1080 user@私服ip
    ```
    > maven中配置socks代理
    
    ```
    # 在$maven_home/conf/settings.xml和~/.m2/settings.xml中配置socks代理
    <proxy>
      <id>proxy</id>
      <active>true</active>
      <protocol>socks</protocol>
      <host>localhost</host>
      <port>1080</port>
      <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
    
    ```
    
2. socks代理转http代理 
    > 说明：编译过程中npm也会下一些墙外的包，但是npm只支持http代理，不支持socks代理，这里用的polipo工具（也可以用其他），将本地socks代理转换为http代理。
    
    ```
    #安装polipo
    git clone https://github.com/jech/polipo.git
    cd polilo
    make all
    make install
    #配置polipo配置文件
    mkdir -f /etc/polipo
    touch /etc/polipo/config
    ```
    ```
    #/etc/polipo/config中配置如下
    socksParentProxy = "127.0.0.1:1080"
    socksProxyType = socks5
    proxyAddress = "::0"
    proxyPort = 8123
    daemonise = true
    ```
    ```
    #运行polipo，http代理起在8123端口
    polipo -c /etc/polipo/config
    ```
    > npm中配置http代理
 
    ```
    # 配置后可以在~/.npmrc中查看到
    npm config set proxy http://localhost:8123
    npm config set https-proxy http://localhost:8123
    npm config set registry "http://registry.npmjs.org/"
    npm config set strict-ssl false
    
    ```
    
    > maven中也可配置成http代理（与上述socks代理二选一配一样就行，笔者用的是http代理）
    
    ```
    # 在$maven_home/conf/settings.xml和~/.m2/settings.xml中配置http代理
    <proxy>
      <id>proxy-http</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>localhost</host>
      <port>8123</port>
      <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
    <proxy>
      <id>proxy-https</id>
      <active>true</active>
      <protocol>https</protocol>
      <host>localhost</host>
      <port>8123</port>
      <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
    
    ```

### 开始编译

```
cd zeppelin

#切换分支编译v0.8.0
git checkout -b mytagv0.8.0 v0.8.0

# 更新所有 pom.xml 用 scala 2.11
./dev/change_scala_version.sh 2.11

# 编译zeppelin带有所有的interepter
# 参数中hadoop、spark是必要的，scala是非必要的，build-distr是指运行pom中的打包命令
mvn clean package -Phadoop-2.7.6 -Pspark2.3 -Pscala-2.11 -Pbuild-distr -DskipTests
```



### 编译中遇到的错误

> 如果按上述正确配置了国外代理，那么应该会顺利过掉zeppelin-web之前的所有模块

- ##### 错误一

```
# 表现：
编译到zeppelin-web之前的各种模块中，各种repo的timeout

# 解决：
正确配置上述maven代理
```

- ##### 错误二
```
# 表现：
编译到zeppelin-web模块时，如果之前仅通过yum install node & yum install npm方式安装的node和npm时，会报node and npm is not lastest version

# 解决：
彻底卸载nodejs和npm
yum remove nodejs npm -y
删除: /usr/local/bin/npm
删除: /usr/local/share/man/man1/node.1
删除: /usr/local/lib/dtrace/node.d
删除: rm -rf /home/[homedir]/.npm
删除: rm -rf /home/root/.npm

按系统需求部分中安装最新版node
```

- ##### 错误三
```
# 表现：
编译到zeppelin-web模块时报错如下
[INFO] Running "wiredep:test" (wiredep) task
[INFO] Warning: Error: Cannot find where you keep your Bower packages. Use --force to continue.
[ERROR] npm ERR! zeppelin-web@0.0.0 build:dist: `npm-run-all prebuild && grunt pre-webpack-dist && webpack && grunt post-webpack-dist`
[ERROR] npm ERR! Exit status 3
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.4:npm
(npm build) on project zeppelin-web: Failed to run task: 'npm run build:dist' failed. 
org.apache.commons.exec.ExecuteException: Process exited with an error: 3 (Exit value: 3)


# 解决：
# 进入zeppelin-web中单独编译这个模块
mvn clean package -DskipTests

# 发现还是报这个错，再尝试运行npm run build:dist，发现还是报这个错误

# 其实主要错误是Error: Cannot find where you keep your Bower packages. Use --force to continue. 
这里没有给bower简历components的存放位置，应该在package.json同级目录下建立一个bower_components的文件夹，于是将package.json相关行加mkdir -p ./bower_components
"postinstall": "bower install --silent --allow-root",
"build:dist": "mkdir -p ./bower_components && npm-run-all prebuild && npm-run-all postinstall && grunt pre-webpack-dist && webpack && grunt post-webpack-dist",
# 或者用--force也可以编译通过，但建议用上面的方式
"postinstall": "bower install --silent --allow-root",
"build:dist": "npm-run-all prebuild && npm-run-all postinstall && grunt pre-webpack-dist && webpack && grunt post-webpack-dist --force",

```

### 编译后部署
- 通过加上build-distr参数后，编译完会在zeppelin/zeppelin-distribution/target路径下生成一个zeppelin-0.8.0.tar.gz包
- 解压后的路径中有一个zeppelin-web-0.8.0.war的war包就是前端改动相关的
- 启动/停止
```
bin/zeppelin-daemon.sh --config conf/ start
bin/zeppelin-daemon.sh --config stop
```


# jdk 升级
源版本 java version "1.8.0_171"
删除旧 jdk 文件，清除环境变量,更新环境变量文件

新版本 java version "11.0.2" 2019-01-15 LTS
jdk11与之前版本不同，安装好的文件夹里没有jre文件，

## 下载安装包
安装包 url  https://repo.huaweicloud.com/java/jdk/11.0.2+9/jdk-11.0.2_linux-x64_bin.tar.gz

## 解压缩文件
`tar xzf jdk-11.0.2_linux-x64_bin.tar.gz`

## 配置环境变量
```
echo "JAVA_HOME=/data/jdk1.8.0_301
PATH=\$PATH:\$JAVA_HOME/bin
export JAVA_HOME PATH" >> /etc/profile
source /etc/profile.d/java.sh
```
## 验证jdk

```
java -version
java version "11.0.2" 2019-01-15 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.2+9-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.2+9-LTS, mixed mode)
```
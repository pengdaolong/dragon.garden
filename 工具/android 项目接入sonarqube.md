### 下载Android SDK Tools 和配置环境
1. https://www.androiddevtools.cn/#
![file](/upload/2019/9/image-157189957173420191024144612372.png)
2. 解压Android SDK
`tar -zvxf android-sdk_r24.4.1-linux.tgz`
3. 配置环境变量 (编辑 /etc/profile 文件)
`vim /etx/profile`
![file](/upload/2019/9/image-157189992008820191024145200835.png)
```
export ANDROID_HOME=/usr/local/android-sdk-linux
export PATH=$ANDROID_HOME/tools:$PATH
```
```
让配置生效
source /etc/profile
```
4. 安装所需要的包
```
安装所有包
android update sdk --no-ui
```
```
android list sdk --all    列出所有包 如下所示
1- Android SDK Tools, revision 24.1.2
   2- Android SDK Platform-tools, revision 22
   3- Android SDK Build-tools, revision 22.0.1
   4- Android SDK Build-tools, revision 22 (Obsolete)
   5- Android SDK Build-tools, revision 21.1.2
   6- Android SDK Build-tools, revision 21.1.1 (Obsolete)
   7- Android SDK Build-tools, revision 21.1 (Obsolete)
   8- Android SDK Build-tools, revision 21.0.2 (Obsolete)
   9- Android SDK Build-tools, revision 21.0.1 (Obsolete)
  10- Android SDK Build-tools, revision 21 (Obsolete)
  11- Android SDK Build-tools, revision 20
  12- Android SDK Build-tools, revision 19.1
  13- Android SDK Build-tools, revision 19.0.3 (Obsolete)
  14- Android SDK Build-tools, revision 19.0.2 (Obsolete)
  15- Android SDK Build-tools, revision 19.0.1 (Obsolete)
按序列号安装
android update sdk -u --all --filter 1,2,3,5,11,12,22,23,24,25,26,27,28,29,88,89
```

### 在安卓项目中添加如下配置
1. 在项目主 build.gradle中添加如下配置
![file](/upload/2019/9/image-157190052717720191024150207867.png)
```
 classpath "org.sonarsource.scanner.gradle:sonarqube-gradle-plugin:2.7"
plugins {
  id "org.sonarqube" version "2.7"
}
```
2. 进入项目  执行构建命令
3. ![file](/upload/2019/9/image-157190544852420191024162408149.png)
```
./gradlew sonarqube \
  -Dsonar.host.url=sonarqube url \
  -Dsonar.login=logtin token
```
### ci 配置
```
project_sonarqube:
  stage: sonarqube
  only:
    - master
  when:
    manual
  script:
  - source /etc/profile
  - chmod +x gradlew
  - ./gradlew sonarqube  -Dsonar.host.url=$SONAR_URL  -Dsonar.login=$SONAR_TOKEN
  tags:
  - dev
```
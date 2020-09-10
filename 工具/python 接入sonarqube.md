#### 安装sonar-scanner
1、下载sonar-scanner
https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/
![file](/upload/2019/10/image-157465195130520191125111911961.png)
2、解压文件包
```
unzip sonar-scanner-cli-4.2.0.1873-linux.zip
```
3、配置环境变量
```
ln -s  sonar-scanner-4.2.0.1873-linux  sonar-scanner
```
![file](/upload/2019/10/image-157465214683820191125112226191.png)
4、立即生效配置
`source /etc/profile`

#### 配置sonar-scaner
1、配置 sonar-scanner.properties 
```
sonar.projectKey=org.codehaus.sonar:maas-taxi-algorithm
sonar.projectName=maas-taxi-algorithm
sonar.projectVersion=1.0
sonar.host.url=http://192.168.100.205:9000 #sonar-qube地址
sonar.login=c6794cd57dd4512aa1197e9af302200ba51e79c3 #sonarqube登录token
# Comma-separated paths to directories with sources (required) 
sonar.sources=. #代码位置
sonar.projectBaseDir=. #项目路径
# Language
sonar.language=py #指定语言
# Encoding of the source files
sonar.sourceEncoding=UTF-8 #指定编码
```
2、测试
`sonar-scanner -Dproject.settings=./sonar-project.properties #指定使用的sonar-scanner.properties`

#### CI 配置
```
stages:
- sonarqube
project_sonarqube:
  stage: sonarqube
  when:
    manual
  script:
  - source /etc/profile . ; sonar-scanner -Dproject.settings=./sonar-project.properties #指定使用的sonar-scanner.properties 
  tags:
   - dev
```
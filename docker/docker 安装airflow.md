### 卸载旧版本
```
yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```
### 设置 yum repository
```
yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
### 安装并启动 docker
```
yum install -y docker-ce-18.09.7 docker-ce-cli-18.09.7 containerd.io
systemctl enable docker
systemctl start docker
```
### 下载airflow 源码
`mkdir airflow`  //创建airflow文件夹
`git clone https://github.com/puckel/docker-airflow.git /root/airflow` //下载源码到airflow文件夹

`docker run -d -p 8082:8080  puckel/docker-airflow`  //安装并运行airflow

`docker ps`  找到容器的id
`docker exec -it af2044c3b40c bash` // 进入容器

`python -c "from cryptography.fernet import Fernet;`
`print(Fernet.generate_key().decode())"`

`export AIRFLOW__CORE__FERNET_KEY=输入上面打印出来的结果`
`airflow initdb`  // 初始化数据库

host:8082 进行访问
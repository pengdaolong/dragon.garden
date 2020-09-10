1. python 3.6
```
apt install python3.6
```
2. 安装依赖
```
sudo apt-get install build-essential libssl-dev libffi-dev python3.6-dev python-pip libsasl2-dev libldap2-dev
```
3. 安装Python virtualenv 和使用
```
pip install virtualenv
python3 -m venv venv
. venv/bin/activate
```
4. 更新python工具
```
pip install --upgrade setuptools pip
```
5. 安装superset

```
### Install superset
pip install superset

### Initialize the database
superset db upgrade

### Create an admin user (you will be prompted to set a username, first and last name before setting a password)
$ export FLASK_APP=superset
flask fab create-admin

### Load some data to play with
superset load_examples

### Create default roles and permissions
superset init

### To start a development web server on port 8088, use -p to bind to another port
superset run -p 8080 --with-threads --reload --debugger

gunicorn \
      -w 10 \
      -k gevent \
      --timeout 120 \
      -b  192.168.100.217:8080 \
      --limit-request-line 0 \
      --limit-request-field_size 0 \
      --statsd-host 0.0.0.0:8125 \
      superset:app &
```
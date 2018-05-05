# Running ELK Stack on AWS EC2

이 문서는 AWS EC2 에서 ELK 스택을 구성하고, 운영하는 방법을 설명한다.

## 1. 인스턴스 생성

Amazon Linux AMI 를 생성하고, 인스턴스 유형은 t2.medium, SSD 용량은 20G 로 합니다.

인스턴스가 생성되면 아래 명령을 실행해준다.

```sh
sudo yum -y update
```

## 2. 자바 업그래이드하기

JAVA 버전은 8.x 로 설치해야 한다.(7.x, 9.x 모두 지원안함.)

```sh
sudo yum -y install java-1.8.0
sudo yum -y remove java-1.7.0-openjdk
```

## 3. ElasticSearch 설치하기

ElasticSearch 를 설치한다.

```sh
$ sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
$ sudo vi /etc/yum.repos.d/elasticsearch.repo
---------------------------------------------------------------------
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
---------------------------------------------------------------------

# elasticsearch 를 설치한다.
$ sudo yum -y install elasticsearch

# 서비스에 등록한다.
$ sudo chkconfig --add elasticsearch
$ sudo -i service elasticsearch start
$ # sudo -i service elasticsearch stop

# 작동되는 것을 확인할 수 있다.
$ curl -X GET "localhost:9200/"
```

ElasticSearch 설정을 수정한다.

```sh
# !!! 아래설정은 모든 아이피에서의 접속을 허용하는 것이다. 방화벽 등 다른 수단으로 접속을 제한해야 한다. !!!
$ sudo vi /etc/elasticsearch/elasticsearch.yml
......
cluster.name: test-es
node.name: test-es-node-1
network.host: 0.0.0.0
http.port: 9200
......

$ sudo -i service elasticsearch stop
$ sudo -i service elasticsearch start
```

Chrome Plugin 중 [ElasticSearch Head](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm) 를 설치한다.

## 4. Kibana 설치하기

```sh
# 키바나를 설치한다.
$ sudo yum -y install kibana

# 키바나를 서비스에 등록한다.
$ sudo chkconfig --add kibana
$ sudo -i service kibana start
# $ sudo -i service kibana stop

# 작동하는지 확인한다.
$ curl http://localhost:5601/

# 외부서버에서 키바나 접속을 허용한다.
# !!! 아래설정은 모든 아이피에서의 접속을 허용하는 것이다. 방화벽 등 다른 수단으로 접속을 제한해야 한다. !!!
$ sudo vi /etc/kibana/kibana.yml
......
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://localhost:9200"
......

# 키바나를 재시작한다.
$ sudo -i service kibana stop
$ sudo -i service kibana start
```

브라우저에서 `http://서버아이피:5601/` 에 접속이 되면 정상적으로 작동한는 것이다.

## 5. Logstash 설치하기

```sh
# 로그스태시를 설치한다.
$ sudo yum -y install logstash

# 설치된것을 확인한다.
$ cd /usr/share/logstash/
$ bin/logstash --version

# 키보드 입력을 받아 화면에 출력하는 간단한 설정을 생성한다.
$ sudo vi logstash-simple.conf
---------------------------------------------------------------------
input {
    stdin { }
}

output {
    elasticsearch { hosts => ["localhost:9200"] }
    stdout { codec => rubydebug }
}
---------------------------------------------------------------------

$ sudo -Hu logstash bin/logstash --path.settings=/etc/logstash -f logstash-simple.conf
```

## 6. ELK 스택 종료하기

ELK 스택을 종료하려면 아래 명령을 입력한다.

```sh
sudo initctl stop logstash
sudo -i service kibana stop
sudo -i service elasticsearch stop
```
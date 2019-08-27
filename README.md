# Running ELK Stack on AWS EC2

이 문서는 AWS EC2 에서 ELK 6.x 스택을 구성하고, 운영하는 방법을 설명한다.

## 1. 인스턴스 생성

Ubuntu Server 18.04 LTS 를 생성하고, 인스턴스 유형은 t2.medium, SSD 용량은 20G 로 합니다.

## 2. 자바 업그래이드하기

JAVA 버전은 8.x 로 설치해야 한다.(7.x, 9.x 모두 지원안함.)

```sh
# Add repository
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update

# Install OpenJDK 8
sudo apt-get install openjdk-8-jdk

# check installation
java -version

# add environment variable
vi ~/.bashrc
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

## 3. ElasticSearch 설치하기

ElasticSearch 를 설치한다.

```sh
# Add repository
sudo apt-get install apt-transport-https
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo add-apt-repository "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
sudo apt-get update

# install elasticsearch
sudo apt-get install elasticsearch
```

ElasticSearch 설정을 수정한다.

```sh
# !!! 아래설정은 모든 아이피에서의 접속을 허용하는 것이다. 방화벽 등 다른 수단으로 접속을 제한해야 한다. !!!
sudo vi /etc/elasticsearch/elasticsearch.yml
......
cluster.name: test-es
node.name: test-es-node-1
network.host: 0.0.0.0
http.port: 9200
cluster.initial_master_nodes: ["test-es-node-1"]
......

sudo /bin/systemctl enable elasticsearch.service

sudo systemctl start elasticsearch.service
# sudo systemctl stop elasticsearch.service

sudo tail -200 /var/log/elasticsearch/test-es.log
curl -X GET "http://localhost:9200/?pretty"
```

Chrome Plugin 중 [ElasticSearch Head](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm) 를 설치한다.

## 4. Kibana 설치하기

repository 는 이미 추가했으므로 바로 키바나를 설치할 수 있다.

```sh
# 키바나를 설치한다.
sudo apt-get install kibana

# !!! 아래설정은 모든 아이피에서의 접속을 허용하는 것이다. 방화벽 등 다른 수단으로 접속을 제한해야 한다. !!!
sudo vi /etc/kibana/kibana.yml
......
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
......

# 부팅시 키바나 자동시작을 설정한다.
sudo update-rc.d kibana defaults 95 10

sudo -i service kibana start
# sudo -i service kibana stop

# 작동하는지 확인한다.
$ curl http://localhost:5601/
```

브라우저에서 `http://서버아이피:5601/` 에 접속이 되면 정상적으로 작동한는 것이다.

## 5. Kibana 한글화

```sh
cd /usr/share/kibana/x-pack/plugins/translations/translations
sudo cp ja-JP.json ko-KR.json

sudo vi /etc/kibana/kibana.yml
......
i18n.locale: "ko-KR"
......

sudo -i service kibana stop
sudo -i service kibana start
```

----------------------------------------------

## 5. Logstash 설치하기

```sh
# 로그스태시를 설치한다.
$ sudo yum -y install logstash

# 설치된것을 확인한다.
$ cd /usr/share/logstash/
$ bin/logstash --version

# 샘플 로그를 입력받아 화면에 출력하는 간단한 설정을 생성한다.
$ sudo vi logstash-simple.conf
---------------------------------------------------------------------
input {
    stdin { }
}

output {
    # elasticsearch { hosts => ["localhost:9200"] }
    stdout { codec => rubydebug }
}
---------------------------------------------------------------------

$ sudo vi test.log
---------------------------------------------------------------------
aaa
bbb
ccc
---------------------------------------------------------------------

$ sudo -Hu logstash bin/logstash --path.settings=/etc/logstash -f logstash-simple.conf < test.log
```

## 6. ELK 스택 종료하기

ELK 스택을 종료하려면 아래 명령을 입력한다.

```sh
sudo initctl stop logstash
sudo -i service kibana stop
sudo -i service elasticsearch stop
```

## 7. Logstash 추가설정하기

[여기](./configure-logstash-web-log.md) 에서 Logstash 에 대한 추가설정을 볼 수 있다.

# Configure Logstash Web Log

웹로그를 Logstash 를 이용해 파싱하고, ElasticSearch 에 전송하는 방법을 설명한다.

## template 등록하기

```sh
# 현재 등록되어 있는 template 표시하기
$ curl -X GET "localhost:9200/_template?pretty"

# 템플릿 등록하기
$ curl -XPUT 'localhost:9200/_template/template_weblog?pretty' \
    -H'Content-Type: application/json' \
    -d '
{
  "template" : "weblog-*",
  "settings" : {
    "number_of_replicas" : "0"
  }
}'

# 등록되어 있는 모든 템플릿 삭제
# $ curl -X DELETE "localhost:9200/_template/*?pretty"
# {
#   "acknowledged" : true
# }
```

## Apache 웹로그

todo!!

## IIS 웹로그

```sh
$ cd /usr/share/logstash/

# 아래 생성한 디렉토리에 IIS 웹로그파일을 복사해 넣는다.
$ sudo -Hu logstash mkdir data/test

$ sudo -Hu logstash mkdir conf
$ sudo -Hu logstash vi conf/logstash-iis.conf
---------------------------------------------------------------------
input {
  stdin { }
}

# 상단 코맨트 삭제
filter {
  if [message] =~ "^#" {
    drop {}
  }

  grok {
    # 로그포멧
    match => [
      "message","%{TIMESTAMP_ISO8601:log_timestamp} %{IP:serverIP} %{WORD:method} %{URIPATH:request} (?:%{NOTSPACE:queryParam}|-) %{NUMBER:port} (?:%{NOTSPACE:username}|-) %{IPORHOST:clientIP} %{NOTSPACE:userAgent} (?:%{NOTSPACE:referer}|-) %{NUMBER:status} %{NUMBER:sub-status} %{NUMBER:win32-status} %{NUMBER:timetaken}",
      "message","%{TIMESTAMP_ISO8601:log_timestamp} %{IP:serverIP} %{WORD:method} %{URIPATH:request} (?:%{NOTSPACE:queryParam}|-) %{NUMBER:port} (?:%{NOTSPACE:username}|-) %{IPORHOST:clientIP} %{NOTSPACE:userAgent} %{NUMBER:status} %{NUMBER:sub-status} %{NUMBER:win32-status} %{NUMBER:timetaken}"
    ]
  }

  # 시간
  date {
    match => [ "log_timestamp", "YYYY-MM-dd HH:mm:ss" ]
    # http://joda-time.sourceforge.net/timezones.html
    timezone => "GMT"
    # timezone => "Asia/Seoul"
    locale => "en"
  }

  mutate {
    convert => ["timetaken", "integer"]
  }
}

output {
  elasticsearch {
    hosts => "localhost:9200"

    # Log records into month-based indexes
    index => "weblog-%{+YYYY.MM.dd}"
  }

  # stdout included just for testing
  #stdout {codec => rubydebug}
}
---------------------------------------------------------------------

$ sudo -Hu logstash bin/logstash --path.settings=/etc/logstash -f conf/logstash-iis.conf < data/test/*.log
```

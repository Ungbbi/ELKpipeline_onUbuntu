# [우리 FISA 2주차 미니 프로젝트] ELK 스택 구축🔥

---

### 개발팀원🌻

- 최영하, 김상민, 박웅빈, 구동길

---

### 학습목적✏️

- Docker 기술을 사용하기 전 리눅스 환경과 conf, yml 설정 파일을 직접 다뤄보며 리눅스 환경에 익숙해진다.
- FileBeat -> Logstash -> Elasticsearch -> Kibana 데이터 파이프라인을 구축해본다.
- 데이터 수집, 전처리, 시각화를 경험해본다.

## Virsual Box Ubuntu 환경에서 ELK 스택 구축하기

---

### 1. Ubuntu 시스템 환경에 맞는 ELK 파일 다운로드

- uname -m 명령어로 x86_64 또는 ARM64 비트 환경인지 확인
- 적합한 ELK 파일 다운로드
  <img width="737" alt="스크린샷 2024-07-20 오후 3 58 37" src="https://github.com/user-attachments/assets/c548f32e-0104-48b9-89d3-8ffad6f7ede7">
- Elasticsearch 7.11.1 Linux X86_64.tar.gz
- Logstash 7.11.1 Linux X86_64.tar.gz
- Filebeat 7.11.1 Linux X86_64.tar.gz
- Kibana 7.11.1 Linux X86_64.tar.gz

---

### 2. openJDK 11 설치

- 기존 Ubuntu에는 JDK 17버전이 설치되어 있었지만 Elasticsearch 7.11.1 버전은 JDK11 버전만 지원한다.
- $ sudo apt install openjdk-11-headless
- $ sudo vi ~/.profile
- export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
- export PATH=$PATH:$JAVA_HOME/bin
- $ source ~/.profile

### 3. pipline.conf 설정

<img width="813" alt="스크린샷 2024-07-20 오후 4 32 14" src="https://github.com/user-attachments/assets/2f80cd2b-9858-4e84-9249-3bbffd64202f">

- Filebeat에서 읽은 데이터를 전처리합니다.
- $ vim ~/ELK/logstash/conf/pipline.conf 작성
```
input {
  beats {
    port => 5044 
  }
}

filter {
  mutate {
    split => [ "message",  "," ] 
    add_field => {
      "date" => "%{[message][0]}"
      "bank" => "%{[message][1]}"
      "branch" => "%{[message][2]}"
      "location" => "%{[message][3]}"
      "customers" => "%{[message][4]}"
    }
    remove_field => ["ecs", "host", "@version", "agent", "log", "tags",  "input", "message"]
  }

  date {
    match => [ "date", "yyyyMMdd"]
    timezone => "Asia/Seoul"
    locale => "ko"
    target => "date"
  }
  mutate {
    convert => {
      "customers" => "integer"
    }
    remove_field => [ "@timestamp" ]
  }
}

output {
  stdout {
    codec => rubydebug
  }
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "bankfisa3"
  }
}
```
---
### 4. Filebeat 설정 파일 작성
- $ vim ~/ELK/filebeaat/filebeat.yml 작성
- Filebaet이 CSV 파일을 읽고 Logstash에 전송하도록 설정합니다.

### 5. ELK 스택 실행
- $ ./elasticsearch -d
- $ ./logstash -f ../conf/pipline.conf
- $ ./filebeat -e -c filebeat.conf


## 결과물
---
###
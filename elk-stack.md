# 1. Setup .env file in a local folder say D:\ELKStack\
```sh
# Password for the 'elastic' user (at least 6 characters)
ELASTIC_PASSWORD=changeit

# Password for the 'kibana_system' user (at least 6 characters)
KIBANA_PASSWORD=changeit

# Version of Elastic products
STACK_VERSION=8.8.2

# Set the cluster name
CLUSTER_NAME=elk-cluster

# Set to 'basic' or 'trial' to automatically start the 30-day trial
LICENSE=basic
#LICENSE=trial

# Port to expose Elasticsearch HTTP API to the host
ES_PORT=9200
#ES_PORT=127.0.0.1:9200

# Port to expose Kibana to the host
KIBANA_PORT=5601
#KIBANA_PORT=80

# Port to expose Logstash to the host
LOGSTASH_PORT=5044
#LOGSTASH_PORT=80

# Increase or decrease based on the available host memory (in bytes)
MEM_LIMIT=1073741824

# Project namespace (defaults to the current folder name if not set)
#COMPOSE_PROJECT_NAME=myproject
```

# 2. Create a docker-compose.yml file
```sh
version: "2.2"

services:
  setup:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
    user: "0"
    command: >
      bash -c '
        if [ x${ELASTIC_PASSWORD} == x ]; then
          echo "Set the ELASTIC_PASSWORD environment variable in the .env file";
          exit 1;
        elif [ x${KIBANA_PASSWORD} == x ]; then
          echo "Set the KIBANA_PASSWORD environment variable in the .env file";
          exit 1;
        fi;
        if [ ! -f config/certs/ca.zip ]; then
          echo "Creating CA";
          bin/elasticsearch-certutil ca --silent --pem -out config/certs/ca.zip;
          unzip config/certs/ca.zip -d config/certs;
        fi;
        if [ ! -f config/certs/certs.zip ]; then
          echo "Creating certs";
          echo -ne \
          "instances:\n"\
          "  - name: es01\n"\
          "    dns:\n"\
          "      - es01\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          "  - name: es02\n"\
          "    dns:\n"\
          "      - es02\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          "  - name: es03\n"\
          "    dns:\n"\
          "      - es03\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          > config/certs/instances.yml;
          bin/elasticsearch-certutil cert --silent --pem -out config/certs/certs.zip --in config/certs/instances.yml --ca-cert config/certs/ca/ca.crt --ca-key config/certs/ca/ca.key;
          unzip config/certs/certs.zip -d config/certs;
        fi;
        echo "Setting file permissions"
        chown -R root:root config/certs;
        find . -type d -exec chmod 750 \{\} \;;
        find . -type f -exec chmod 640 \{\} \;;
        echo "Waiting for Elasticsearch availability";
        until curl -s --cacert config/certs/ca/ca.crt https://es01:9200 | grep -q "missing authentication credentials"; do sleep 30; done;
        echo "Setting kibana_system password";
        until curl -s -X POST --cacert config/certs/ca/ca.crt -u "elastic:${ELASTIC_PASSWORD}" -H "Content-Type: application/json" https://es01:9200/_security/user/kibana_system/_password -d "{\"password\":\"${KIBANA_PASSWORD}\"}" | grep -q "^{}"; do sleep 10; done;
        echo "All done!";
      '
    healthcheck:
      test: ["CMD-SHELL", "[ -f config/certs/es01/es01.crt ]"]
      interval: 1s
      timeout: 5s
      retries: 120

  es01:
    depends_on:
      setup:
        condition: service_healthy
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - ${ES_PORT}:9200
    environment:
      - node.name=es01
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es02,es03
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es01/es01.key
      - xpack.security.http.ssl.certificate=certs/es01/es01.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es01/es01.key
      - xpack.security.transport.ssl.certificate=certs/es01/es01.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  es02:
    depends_on:
      - es01
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata02:/usr/share/elasticsearch/data
    environment:
      - node.name=es02
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es01,es03
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es02/es02.key
      - xpack.security.http.ssl.certificate=certs/es02/es02.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es02/es02.key
      - xpack.security.transport.ssl.certificate=certs/es02/es02.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  es03:
    depends_on:
      - es02
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata03:/usr/share/elasticsearch/data
    environment:
      - node.name=es03
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01,es02,es03
      - discovery.seed_hosts=es01,es02
      - bootstrap.memory_lock=true
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es03/es03.key
      - xpack.security.http.ssl.certificate=certs/es03/es03.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es03/es03.key
      - xpack.security.transport.ssl.certificate=certs/es03/es03.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  kibana:
    depends_on:
      es01:
        condition: service_healthy
      es02:
        condition: service_healthy
      es03:
        condition: service_healthy
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
    volumes:
      - certs:/usr/share/kibana/config/certs
      - kibanadata:/usr/share/kibana/data
    ports:
      - ${KIBANA_PORT}:5601
    environment:
      - SERVERNAME=kibana
      - ELASTICSEARCH_HOSTS=https://es01:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
      - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
    mem_limit: ${MEM_LIMIT}
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s -I http://localhost:5601 | grep -q 'HTTP/1.1 302 Found'",
        ]
      interval: 10s
      timeout: 10s
      retries: 120

  logstash01:
    depends_on:
      es01:
        condition: service_healthy
      kibana:
        condition: service_healthy
    image: docker.elastic.co/logstash/logstash:${STACK_VERSION}
    labels:
      co.elastic.logs/module: logstash
    user: root
    command: ["logstash", "-f", "/usr/share/logstash/pipeline/logstash.conf"]
    volumes:
      - certs:/usr/share/logstash/certs
      - logstashdata:/usr/share/logstash/data
      # - "D:/STS_WORKSPACE/.metadata/.plugins/org.eclipse.wst.server.core/tmp3/logs/portal_logs/portal/2023-07/portal-2023-07-19.log:/usr/share/logstash/logs/portal-2023-07-19.log:ro"
      - "./logstash_ingest_data/:/usr/share/logstash/ingest_data/"
      - "./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro"
      # "C:/Users/kapil.panchal.ext/Desktop/logs/elk-stack.log:/usr/share/logstash/logs/elk-stack.log:ro"
    ports:
      - ${LOGSTASH_PORT}:5044
    environment:
      - xpack.monitoring.enabled=false
      - ELASTIC_USER=elastic
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - ELASTIC_HOSTS=https://es01:9200

volumes:
  certs:
    driver: local
  esdata01:
    driver: local
  esdata02:
    driver: local
  esdata03:
    driver: local
  kibanadata:
    driver: local
  logstashdata:
    driver: local
```

# 3. The above will give (vm.max_map_count=262144 too low error) using powershell with admin on every reboot
```sh
wsl -d docker-desktop -u root
sysctl -w vm.max_map_count=262144
```

# 4. The above assumes the latest installation of Docker and docker-compose on windows 10 system

# 5. Elastic search will be available on (SSL Enabled)
```sh
https://localhost:9200
```

# 6. Kibana is available on (SSL not enabled)
```sh
http://localhost:5601
```

# 7. Create logstash.conf file
```sh
input {
    tcp {
        port => 5044
            codec => json_lines
    }
}

filter {
    grok {
        match => {
            "logger_name" => "sfdc_emails_save_to_elastic"          
        }
    }
}

output {
    if "_grokparsefailure" not in [tags] {
        stdout {
            codec => rubydebug
        }
        elasticsearch {
            index => "logstash-%{+YYYY.MM.dd}"
            hosts=> "${ELASTIC_HOSTS}"
            user=> "${ELASTIC_USER}"
            password=> "${ELASTIC_PASSWORD}"
            cacert=> "certs/ca/ca.crt"
        }
    }
}
```

# 7. Powershell with Admin account where the docker cluster is running 
### (Elastic search will give an out of memory error) 'can't store an async search response larger 
### than [10485760] steps to increase out of memory error  

```sh
   1 d:
   2 cd .\ELK_Stack\
   3 cd .\docker\v5
   4 ls
   5 docker-compose up -d
   6 ls
   7 docker ps
   8 docker logs <container id>  
   9 docker ps
  10 docker cp <container id>:/usr/share/elasticsearch/config/certs/ca/ca.crt .

  11 $certificate = Get-Item -Path .\ca.crt
  12 Invoke-WebRequest -Method GET -Uri 'https://localhost:9200' -Certificate $certificate

  13 $certFilePath = "D:\ELK_Stack\docker\v5\ca.crt"
  14 $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath)
  15 Invoke-WebRequest -Method GET -Uri 'https://localhost:9200' -Certificate $cert
  16 Add-Type @"...
       [System.Net.ServicePointManager]::CertificatePolicy = New-Object TrustAllCertsPolicy
  17 $certFilePath = "D:\ELK_Stack\docker\v5\ca.crt"
  18 $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath)
  19 Invoke-WebRequest -Method GET -Uri 'https://localhost:9200' -Certificate $cert

  20 $username = "elastic"
  21 $password = "changeit"
  22 $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
  23 $credential = New-Object System.Management.Automation.PSCredential ($username, $securePassword)
  24 Invoke-WebRequest -Method GET -Uri $uri -Credential $credential
  
  25 $uri = "https://localhost:9200/_cluster/settings"
  
  26 $payload = @{
          persistent = @{
          "search.max_async_search_response_size" = "50mb"
        }
      } | ConvertTo-Json
  
  27 $headers = @{
        "Content-Type" = "application/json"
      }

  28 Invoke-WebRequest -Method PUT -Uri $uri -Body $payload -Headers $headers -Credential $credential
  39 Invoke-WebRequest -Method PUT -Uri 'https://localhost:9200/_cluster/settings' -Credential $credential

```

[Ref](https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose)

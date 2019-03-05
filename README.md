此為docker-elk6.5.0 

從官方docker-elk更改 https://github.com/deviantony/docker-elk 

以下操作為乾淨環境完整操作 

若使用此docker-elk不用全部操作 

# 操作流程
# 環境CentOS7
(root操作)
# 安裝docker
    $ yum -y install docker
# 啟動docker

    $ systemctl start docker
    $ systemctl enable docker
    (enable 設定開機後啟動)
    $ systemctl status docker(檢查狀態)
    
# pull Docker images(ELK)

    $ docker pull docker.elastic.co/elasticsearch/elasticsearch:6.5.0
    $ docker pull docker.elastic.co/kibana/kibana:6.5.0
    $ docker pull docker.elastic.co/logstash/logstash:6.5.0

# 安裝docker-compose

    $ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    $ sudo chmod +x /usr/local/bin/docker-compose
    $ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
    $ docker-compose --version



# elasticsearch設定

    $sysctl -w vm.max_map_count=262144
    (值要大於262144)


# docker-compose設定

使用官方(docker-elk)

    $ yum install git-all
    $ git clone https://github.com/deviantony/docker-elk
    $ chmod 777 -R docker-elk/

(CentOS7權限不改好像會出問題)

docker-compose.yml   設定更改↓↓↓↓

    version: '2'
    
    services:
    
      elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:6.5.0
        volumes:
          - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
        ports:
          - "9200:9200"
          - "9300:9300"
        environment:
          ES_JAVA_OPTS: "-Xmx256m -Xms256m"
        networks:
          - elk
    
      logstash:
        image: docker.elastic.co/logstash/logstash:6.5.0
        volumes:
          - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
          - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
        ports:
          - "5000:5000"
          - "9600:9600"
        environment:
          LS_JAVA_OPTS: "-Xmx256m -Xms256m"
        networks:
          - elk
        depends_on:
          - elasticsearch
    
      kibana:
        image: docker.elastic.co/kibana/kibana:6.5.0
        volumes:
          - ./kibana/config/:/usr/share/kibana/config:ro
        ports:
          - "5601:5601"
        networks:
          - elk
        depends_on:
          - elasticsearch
    
    networks:
    
      elk:
        driver: bridge
    


# 開啟ELK

docker-compose up -d



# 注意事項

安全性關閉

    $ setenforce 0

    $ vim /etc/selinux/config  (更改)
    SELINUX=Disabled


# 更改設定方便查詢log

vi docker-elk/logstash/pipeline/logstash.conf

    input {
            tcp {
                    port => 5000
                    codec => json
            }
    }
    
    ## Add your filters / logstash plugins configuration here
    
    output {
            elasticsearch {
                    hosts => "elasticsearch:9200"
                    index => "python-message-%{+YYYY.MM.dd}"
            }
    }

# 更改完後方便查詢

![](https://d2mxuefqeaa7sj.cloudfront.net/s_4F58DEE54C5B92DE1FD3D97F5D2B0C610EAA119B86D4017CCF71D16141D5606D_1548745236907_image.png)

    #測試用程式
    import logging
    import logstash
    logger = logging.getLogger("xxx")#這邊資訊會在@fields.logger
    logger.setLevel(logging.INFO)
    host_demo = "ip_add"#xx.xx.xx.xx這邊改ip
    logger.addHandler(logstash.TCPLogstashHandler(host_demo,5000))#logstash port 5000
    
    
    logger.error("error_message")
    logger.info("message")

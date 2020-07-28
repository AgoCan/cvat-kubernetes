# cvat 的k8s部署
先用compose部署cvat，然后获取镜像后

```
# 重命名
docker tag docker.elastic.co/elasticsearch/elasticsearch-oss:6.4.0 hank997/cvat-elasticsearch-oss:6.4.0
docker tag docker.elastic.co/kibana/kibana-oss:6.4.0 hank997/cvat-kibana-oss:6.4.0
docker tag docker.elastic.co/logstash/logstash-oss:6.4.0 hank997/cvat-logstash-oss:6.4.0
docker tag redis:4.0-alpine hank997/cvat-redis:4.0-alpine
docker tag postgres:10-alpine hank997/cvat-postgres:10-alpine
docker tag ubuntu:16.04 hank997/cvat-ubuntu:16.04
docker tag cvat_elasticsearch:latest hank997/cvat_elasticsearch:latest
docker tag cvat_kibana:latest hank997/cvat_kibana:latest
docker tag cvat_logstash:latest hank997/cvat_logstash:latest
docker tag cvat:latest hank997/cvat:latest
docker tag cvat_cvat_ui:latest hank997/cvat_cvat_ui:latest
docker tag nginx:stable-alpine hank997/nginx:stable-alpine
docker tag node:lts-alpine hank997/node:lts-alpine

# 推送
docker push hank997/cvat-elasticsearch-oss:6.4.0
docker push hank997/cvat-kibana-oss:6.4.0
docker push hank997/cvat-logstash-oss:6.4.0
docker push hank997/cvat-redis:4.0-alpine
docker push hank997/cvat-postgres:10-alpine
docker push hank997/cvat-ubuntu:16.04
docker push hank997/cvat_elasticsearch:latest
docker push hank997/cvat_kibana:latest
docker push hank997/cvat_logstash:latest
docker push hank997/cvat:latest
docker push hank997/cvat_cvat_ui:latest
docker push hank997/node:lts-alpine
docker push hank997/nginx:stable-alpine
```

## 注意
ELK下的`components/analytics/logstash/logstash.conf`,不然elasticsearch连接不通，还有`components/analytics/kibana/kibana.yml` 也需要改一下
```
  if [type] == "client" {
    elasticsearch {
      hosts => ["cvat-elasticsearch-svc.cvat.svc.cluster.local:9200"]
      index => "cvat.client"
    }
  } else if [type] == "server" {
    elasticsearch {
      hosts => ["cvat-elasticsearch-svc.cvat.svc.cluster.local:9200"]
      index => "cvat.server"
    }
  }

```

下面的步骤还没操作，发起一个job即可
注意， `setup.py`下的`default`改成`cvat-kibana-svc.cvat.svc.cluster.local`，还有command的改成相对应的地址
```
    parser.add_argument('-H', '--host', metavar='HOST', default='kibana',
        help='host of Kibana instance')
```

```
  cvat_kibana_setup:
    container_name: cvat_kibana_setup
    image: cvat
    volumes: ['./components/analytics/kibana:/home/django/kibana:ro']
    depends_on: ['cvat']
    working_dir: '/home/django'
    entrypoint: ['bash', 'wait-for-it.sh', 'elasticsearch:9200', '-t', '0', '--',
      '/bin/bash', 'wait-for-it.sh', 'kibana:5601', '-t', '0', '--',
      '/usr/bin/python3', 'kibana/setup.py', 'kibana/export.json']
    environment:
      no_proxy: elasticsearch,kibana,${no_proxy}

```
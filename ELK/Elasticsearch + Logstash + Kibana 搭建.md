#### 1. 下载
https://www.elastic.co/downloads/elasticsearch </br>
https://www.elastic.co/downloads/logstash </br>
https://www.elastic.co/downloads/kibana </br>

#### 2. 解压
.  </br>
├── elasticsearch-6.2.4.tar.gz  </br>
├── logstash-6.2.4.tar.gz  </br>
├── kibana-6.2.4-darwin-x86_64.tar.gz  </br>
├── elasticsearch  </br>
├── elasticsearch-head  </br>
├── logstash  </br>
└── kibana  </br>

#### 3. 安装配置
- elasticsearch
```
cd ${ELASTICSEARCH_HOME}
vim config/elasticsearch.yml      //修改配置
  http.cors.enabled: true
  http.cors.allow-origin: "*"

./bin/elasticsearch     //启动  默认端口9200

```
- elasticsearch head
```
git clone git://github.com/mobz/elasticsearch-head.git    //下载head源码
cd elasticsearch-head
npm install  //注意用国内的源，否则构建很慢

vim Gruntfile.js      //注册elasticsearch 服务器连接任务
 connect: {
   server: {
     options: {
       hostname: 'localhost',  //增加hostname
       port: 9100,
       base: '.',
       keepalive: true
     }
   }
 }

grunt server          //启动elasticsearch-head
  "Started connect web server on http://localhost:9100"

```




## Setup local Elasticsearch and Kibana with xpack

### Steps
- Run Elasticsearch with x-pack false
```
xpack.security.enabled: false
```
- Run
```
docker-compose up
docker-compose exec <elasticsearch-service-name> bash
```
- Run the following command inside the container. Just press ENTER for both to proceed:
```
[root@c9f915e86309 elasticsearch]# bin/elasticsearch-certutil ca
Please enter the desired output file [elastic-stack-ca.p12]: 
Enter password for elastic-stack-ca.p12 : 
```
- This will create a file elastic-stack-ca.p12 in the directory from which you ran the above command. You can check by running the ls command. This is the certificate authority we will be using to create the certificate. Now, run the command:
```
[root@c9f915e86309 elasticsearch]# bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```
- Press ENTER at all the steps to proceed:
```
Enter password for CA (elastic-stack-ca.p12) : 
Please enter the desired output file [elastic-certificates.p12]: 
Enter password for elastic-certificates.p12 : 
```
- This will create the elastic-certificates.p12 which is what we need. Ctrl+D to exit
```
$ docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-certificates.p12 .
$ docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-stack-ca.p12 .
```
- Update docker-compose to include the following line in elasticsearch service
```
    - ./elastic-certificates.p12:/usr/share/elasticsearch/config/elastic-certificates.p12
```
- Add the following to elasticsearch.yml
```
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.keystore.type: PKCS12
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.type: PKCS12
```
- Stop the service to map the elasticsearch volume
```
mkdir docker-data-volumes
```
- Add the following to docker-compose
```
 ./docker-data-volumes/elasticsearch:/usr/share/elasticsearch/data
```
- Run docker-compose up and bash to elasticsearch container. Run the following command to auto generate the password
```
[root@c9f915e86309 elasticsearch]# bin/elasticsearch-setup-passwords auto
```
- Stop the service and update kibana.yml to add the username and password
```
server.name: kibana
server.host: "0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
elasticsearch.username: kibana
elasticsearch.password: <kibana password>
```
- voila run docker-compose up and that's it
- http://codingfundas.com/setting-up-elasticsearch-6-8-with-kibana-and-x-pack-security-enabled/index.html

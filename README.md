# ppelasticsearchelasricstackidep

forward port
```
elastic 127.0.0.1 9200 9200
kibana 127.0.0.1 5601 5601
ssh 127.0.0.1 22 22 
```

install
```
sudo apt update && sudo apt install default-jdk
```
refer to 6.x
[elastic deb](http://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)
```
http://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```

```
systemctl daemon-reload
```
test file
```
wget http://media.sundog-soft.com/es/shakes-mapping.json
```

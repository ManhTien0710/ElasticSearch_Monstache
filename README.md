# ElasticSearch
# install
```bash
yum install -y wget make cmake tar pkg-config wget nano git maven build-essential openjdk-11-jdk
yum install java-11-openjdk-devel -y
yum install gcc gcc-c++ kernel-devel make -y
```
# check java version
```bash
update-alternatives --config java
--> choose version java11
java -version
```
# check maven version
```bash
mvn -version
vi /etc/profile.d/maven.sh

sudo chmod +x /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
mvn -version
--> java version 11
```
# setup coccoc-tokenizer
```bash
git clone https://github.com/coccoc/coccoc-tokenizer.git
chmod 775 -R /root/coccoc-tokenizer
mkdir -p /coccoc-tokenizer/build
cd /coccoc-tokenizer/build/

cmake -DBUILD_JAVA=1 /root/coccoc-tokenizer/
make install
```
# install plugin vietnamese
```bash
wget https://github.com/duydo/elasticsearch-analysis-vietnamese/archive/refs/tags/v7.17.1.tar.gz
tar -xvf v7.17.1.tar.gz
cd elasticsearch-analysis-vietnamese-7.17.1/

mvn verify clean --fail-never
mvn package -Dmaven.test.skip

cp -av /root/coccoc-tokenizer/dicts/tokenizer /usr/local/share/tokenizer/dicts/tokenizer
cp -av /root/coccoc-tokenizer/dicts/vn_lang_tool/ /usr/local/share/tokenizer/dicts/vn_lang_tool
cp -av /coccoc-tokenizer/build/libcoccoc_tokenizer_jni.so /usr/lib
cp -av /coccoc-tokenizer/build/multiterm_trie.dump /usr/local/share/tokenizer/dicts
cp -av /coccoc-tokenizer/build/nontone_pair_freq_map.dump /usr/local/share/tokenizer/dicts
cp -av /coccoc-tokenizer/build/syllable_trie.dump /usr/local/share/tokenizer/dicts
cp -av /root/elasticsearch-analysis-vietnamese-7.17.1/target/releases/elasticsearch-analysis-vietnamese-7.17.1.zip /
```
# Install ElasticSearch
```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.1-x86_64.rpm
yum -y install elasticsearch-7.17.1-x86_64.rpm
```
# open port
```bash
firewall-cmd --permanent --add-port=9200/tcp
firewall-cmd --reload
```
# add plugin
```bash
echo "Y" | /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch file:///elasticsearch-analysis-vietnamese-7.17.1.zip
/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu
```
# restart service
```bash
service elasticsearch restart
```
# Limit RAM
```bash
echo "
-Xms1g
-Xmx1g
" > /etc/elasticsearch/jvm.options
```
# restart service
```bash
service elasticsearch restart
```
# check limit RAM
```bash
cat /var/log/elasticsearch/elasticsearch.log | grep "heap size"
```

# setup authen and network host
```bash
echo "
xpack.security.enabled: true
" >> /etc/elasticsearch/elasticsearch.yml

service elasticsearch restart

cd /usr/share/elasticsearch/
bin/elasticsearch-setup-passwords auto
--> password ramdom --> save

echo "
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
" >> /etc/elasticsearch/elasticsearch.yml
```
# check status service
```bash
systemctl restart elasticsearch.service
systemctl status elasticsearch.service
systemctl enable elasticsearch.service
```
# create user/pass elastic
```bash
bin/elasticsearch-users useradd <new_user> -p <new_password>
bin/elasticsearch-users useradd basebs -p basebs@1234

bin/elasticsearch-users roles basebs -a superuser
systemctl restart elasticsearch.service

***check user/pass
bin/elasticsearch-users list
*** change pass
bin/elasticsearch-users passwd base -p Aa1234
```

# Install Monstache
```bash
sudo mkdir /monstache
cd /monstache/
https://github.com/rwynn/monstache/releases/download/v6.7.11/monstache-6cdd72c.zip
unzip monstache-6cdd72c.zip

export PATH=/monstache/build/linux-amd64:$PATH
monstache -v

vi config.toml
vi /usr/lib/systemd/system/monstache.service
```
# check status service
```bash
systemctl restart monstache.service
systemctl status monstache.service
systemctl enable monstache.service
```
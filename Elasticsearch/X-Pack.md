 # X-Pack Security
 ### Docker Compose, Elasticsearch, Logstash ve Kibana Ayarları

* Elasticsearch kümeniz için kimlik doğrulaması ayarlamanıza, farklı kimlik bilgilerine ve farklı erişim düzeylerine sahip farklı kullanıcılar oluşturmanıza olanak tanır.
Aynı zamanda farklı roller oluşturmanıza ve benzer kullanıcıları aynı role atamanıza olanak tanır 

* ElasticStack 4 farklı türde lisansa sahiptir.
    * Open Source
    * Basic
    * Gold 
    * Platinum
  
* Güvenlik 6.8 sürümünden itibaren temel lisans altında sağlanmıştır.

* X-Pack güvenliğini etkinleştirmek için elasticsearch ve kibana hizmetlerimizi özelleştirmemiz gerekecek. 
Elasticsearch ayarları dosya yoluyla özelleştirilebilir ve Kibana ayarları kibana.yml dosya yoluyla özelleştirilebilir . Docker kullanırken bunu değiştirmenin birçok yolu vardır. Ortam değişkenlerini docker-compose.yml 
dosyamız üzerinden geçirebiliriz . Bu normalde ideal bir yol olsa da, Elasticsearch ve Kibana env değişkenlerinin aktarılma şekli aynı değildir ve belirli dağıtım ortamlarında sorunlara neden olabilir.

* docker-compose.yml dosyamızla aynı dizinde bulunan kibana.yml ve elasticsearch.yml oluşturuyoruz.

  * Kibana.yml:
        
        server.name: kibana
        server.host: "0"
        elasticsearch.hosts: [ "http://elasticsearch:9200" ]
        
  * elasticsearch.yml:
        
        cluster.name: my-elasticsearch-cluster
        network.host: 0.0.0.0
        xpack.security.enabled: true

  * logstash.yml

        xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]

  * docker-compose.yml:
  
        version: "3"
        services:
          elasticsearch:
            image: elasticsearch:7.6.2
            container_name: elasticsearch
            hostname: elasticsearch
            environment:
              - "discovery.type=single-node"
            ports:
              - 9200:9200
              - 9300:9300
            networks:
              - elknetwork
            volumes:
              - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
        
          kibana:
            image: kibana:7.6.2
            container_name: kibana
            hostname: kibana
            ports:
              - 5601:5601
            links:
              - elasticsearch:elasticsearch
            depends_on:
              - elasticsearch
            networks:
              - elknetwork
            volumes:
              - ./kibana.yml:/usr/share/kibana/config/kibana.yml
        
          logstash:
            image: logstash:7.6.2
            container_name: logstash
            hostname: logstash
            ports:
              - 9600:9600
              - 8089:8089
            environment:
              - LOGSTASH_CONFIG_URL=/logstash/logtash.conf
            links:
              - elasticsearch:elasticsearch
            depends_on:
              - elasticsearch
            networks:
              - elknetwork
        
        networks:
          elknetwork:
            driver: bridge
  
 * docker-compose.yml dizininde "docker-compose up" komutunu çalıştırdığımızda imagelarımız indirilecek ve container ayağa kalkacak.
  
## Elasticsearch için SSL Sertifikası Oluşturma
* Öncelikle, Elasticsearch kapsayıcımızı çalışır duruma getirebilmemiz için x-pack güvenliğini geçici olarak devre dışı bırakmamız gerekiyor.

  * elasticsearch.yml:
  
        cluster.name: my-elasticsearch-cluster
        network.host: 0.0.0.0
        xpack.security.enabled: false
      
   
 * elasticsearch.yl güncellemesinden sonra "docker-compose up" ile tekrar containerları ayağa kaldırıyoruz.
 
 * Şimdi serfikaları oluşturmaya başlayacağız. Container'lar ayaktayken aşağıdaki komutu farklı terminal pencerisinde çalıştırarak container içerisine giriyoruz.
 
       docker-compose exec elasticsearch bash
 
 * Ardından kontainer içerisinde aşağıdaki komutu çalıştırıyoruz.Bu komut sonrasında, ne yapacağını açıklayan bazı uyarılar oluşturacaktır. Ve sizden dosya adı ve şifre isteyecektir. Devam etmek için ENTER'a basmanız yeterlidir. Ana dizinde "elastic-stack-ca.p12" dosyasını oluşturacaktır. Bu, sertifikayı oluşturmak için kullanacağımız sertifika yetkisidir.
 
       bin/elasticsearch-certutil ca
 
 * Arıdandan aşağıdaki komutu veriyoruz.
    
       bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
    
* Oluşan bu dosyalara ana makine dışında ihtiyacımız var çünkü kapsayıcımızı imha ettiğimizde yok olacak.
Bunun için ana makine dışına aşağıdaki komutlar ile kopyalıyoruz. Kopyaladığım dosyaları "elasticsearch-certificates" folder'ı altına taşıdım.  

      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-stack-ca.p12 .

      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-certificates.p12 .
 

## SSL Sertifikasının Elasticsearch'e Yüklenmesi

* Bir önceki başlık altında oluşturduğumuz Sertifikamızı docker-compose.yml içerisinden konteynırımıza bağlıyoruz.
        
  * docker-compose.yml:
    
        version: "3"
        services:
          elasticsearch:
            image: elasticsearch:7.6.2
            container_name: elasticsearch
            hostname: elasticsearch
            environment:
              - "discovery.type=single-node"
            ports:
              - 9200:9200
              - 9300:9300
            networks:
              - elknetwork
            volumes:
              - ./elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
              - ./elastic-certificates/elastic-certificates.p12:/usr/share/elasticsearch/config/elastic-certificates.p12
        
          kibana:
            image: kibana:7.6.2
            container_name: kibana
            hostname: kibana
            ports:
              - 5601:5601
            links:
              - elasticsearch:elasticsearch
            depends_on:
              - elasticsearch
            networks:
              - elknetwork
            volumes:
              - ./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml
        
          logstash:
            image: logstash:7.6.2
            container_name: logstash
            hostname: logstash
            ports:
              - 9600:9600
              - 8089:8089
            environment:
              - LOGSTASH_CONFIG_URL=/logstash/logtash.conf
            links:
              - elasticsearch:elasticsearch
            depends_on:
              - elasticsearch
            networks:
              - elknetwork
        
        networks:
          elknetwork:
            driver: bridge

   * elasticsearch.yml:
            
         cluster.name: my-elasticsearch-cluster
         network.host: 0.0.0.0
         xpack.security.enabled: true
         xpack.security.transport.ssl.enabled: true
         xpack.security.transport.ssl.keystore.type: PKCS12
         xpack.security.transport.ssl.verification_mode: certificate
         xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
         xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
         xpack.security.transport.ssl.truststore.type: PKCS12
         
      
   
* Ardından konteynırlarımız ayakta iken aşağıdaki komut ile elasticsearch konteynerı içerisine tekrar giriyoruz.

      docker-compose exec elasticsearch bash
      
* Elastic Stack yapıları için parola üretecek komutu veriyoruz. Burada üretilen passwordleri kayıt etmeniz gerekmektedir.

      bin/elasticsearch-setup-passwords auto
        
* Kibana.yml'ı aşağıdaki gibi güncelliyoruz.

    * kibana.yml:
    
          server.name: kibana
          server.host: "0"
          elasticsearch.hosts: [ "http://elasticsearch:9200" ]
          elasticsearch.username: kibana
          elasticsearch.password: <kibana password>

     * logstash.yml

           xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
           xpack.monitoring.elasticsearch.username: "elastic"
           xpack.monitoring.elasticsearch.password: "<elastic password>"

* Ardından tekrar komutunu yürütün.

      docker-compose up -d


* Artık "http://localhost:5601/" adresini ziyaret ettiğimizde username ve password ile login olamamızı isteyecektir. 
Aynı durum "http://localhost:9200/" içinde gereklidir.
       
        
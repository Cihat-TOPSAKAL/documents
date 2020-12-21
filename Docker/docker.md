
#  Docker

  * [Docker Cotainer](#docker-container)
  * [Docker Volume](#docker-volume)
    * [Docker Bind Mounts](#docker-bind-mounts)
  * [Docker Network](#docker-network)
  * [Docker Logs](#docker-logs)
  * [Docker Memory](#docker-memory)
  * [Docker Image](#docker-image)
    * [Docker File](#docker-file)
    * [Docker Multi - Stage Build](#multi-stage-build)
    * [Docker Save - Load](#docker-save-load)
    * [Docker Commit](#docker-commit)
  *  [Docker Compose](#docker-compose)
  *  [Docker Orchestration](#docker-orchestration)
     *  [Docker Swarm](#docker-swarm)
        *  [Docker Service](#docker-service)
     * [Docker Secret](#docker-secret)

##  Docker Container
* Docker'da tüm containerları (çalışan ve çalışmayan) listelemek için "-a" opsiyonu kullanılır.

      docker container ls -a
      or
      docker ps -a

* Docker container loglarını görebilmek için;
  
      docker container logs <CONTAINER_ID>
  
* Docker container shell'i kendi shell'imize bağlamak istemiyorsak "-d" opsiyonu kullanılıyor.
  
      docker container run -d

* Docker container kapatıldığında silinmesini istiyorsak;
  
      docker container run --rm

* Docker container başlatılırken eğer isim verilmez ise Docker opensource isimlendirme yapar.
İsimlendirme "--name" opsiyonu ile yapılır.

      docker container run --name <NAME> <IMAGE>

* Docker conatiner silmek için rm kullanılır. Docker çalışan containeri silmez silmesi için "-f" opsiyonu kullanılır.

      docker container rm -f <CONTAINER_ID>

* Çalışan bir docker container içerisinde komut çalıştırmak için  yada bağlanabilmek için "exec" yani execute kullanılır.

      docker container exec <CONTAINER_NAME> <COMMAND>
      docker container exec <CONTAINER_NAME> ls -l (example)
      docker container exec -it <CONTAINER_NAME> sh

  * Container'ı "-dit" opsiyonu ile başlatırsak bağlanmak için;
  
        docker attach <CONTAINER_NAME_OR_ID>

  * Container ile bağlantıyı kesmek için Ctrl +  P Q 
  
* Sistemde durdurulmuş bütün containerları ve imageları silmek için "prune" kullanılır.

      docker container prune
      docker image prune

* Çalışan container içerisine "exec" ila bağlanılıyor.
  
      docker exec -it <CONTAINER_NAME_OR_ID> sh

* Bir container oluşturmak isterek ve oluşturulacak image içerisinde default uygulama değilde farklı bir konut yada uygulama çalıştırmak istersek aşağıdaki yolu izliyoruz.
  * Default çalıştırma;
  
        docker container run alpine 

  * Farklı komut çalıştırmak istersek; imagedan sonra bir boşluk bırakıp komutu girebiliriz örneğin:
  
        docker container run alpine ls

* Container içerisinde hangi procceslerin çalıştığını, container içeriisnde girmeden öğrenmek için "top" komutunu kullanılırız.
      
      docker top <CONTAINER_NAME_OR_ID>

## Docker Volume

* **Docker volume:** container dışında data saklayabilmeye oluşturulacak yeni containerlar içerisinde bu datayı kullanabilmemizi sağlar. Örneğin bir containerimizda bulunan log dsyalarımız var yada veritabanı backup'ımız biz bu containeri restart ettiğimizde burada saklanan datalarımız yeniden başlatma sırasında kaybolur. Eğer volumede saklarsak bu datalar docker'ın çalıştığı sanal linuxda volume ile saklanır ve volume ile yeni oluşturacağımız containera ekleyebiliriz.
* Aşaıdaki örnekte ilkuygulama volume oluştuurlmuş ve yeni oluşturulan container'a eklenmektedir.
  
    * Volume oluşturma;
          
          docker volume crerate <VOLUME_NAME>
    * Volume kullanımı;
  
          docker container run -it -v <VOLUME_ADI>:<CONTAINER_ICERISINDEKI_BAGLANACAK_KLASOR> <IMAGE> sh
    
    * Example;
  
          docker container run -it -v ilkvolume:/uygulama alpine sh

        * "-it" opsiyonu ile bu containera interaktif olarak bağlanılır.
        * "-v" volume opsiyonu
        * "ilkvolume" oluşturulan volume ismi
        * ":/uygulama" bağlanan volume'ün container içerisinde tutulacağı dizin yani volume içerisinde ne varsa bu dizin altına taşınır.
        * "alpine" containerın oluşturulacağı image
        * "sh" container shell'ini bağla kendi shell'ime olarak kullanılır.    
* **Not:** ":/uygulama:ro" Read-Only olarak oluşturur. "ro" verirsek bu container oluşturulduğunda bu volume yazım işlemi gerçekleşemez.
* **Not:** Bir volume birden fazla container'a bağlanabiliir.
* Volume container içerisinde boş veya dolu klasöre mount edilirse ne oluyor?
    * Eğer  bir volume mount edildiği kasör mevcur değilse bu klasörü yaratır. Ve volume içerisinde hangi dosyalar var ise o klasör içeriinde de o dosyaları görürüz.
    * Eğer bir volume image içerisinde mevcut bir klasöre mount edilirse;
        * Klasör boş ise o anda volume içerisinde hangi dosyalar var ise o dosyaları görürüz.
        * Klasör dolu ve volume boş ise bu sefer klasör içerindeki dosyalar volume içerisinde kopyalanır.
        * Kalsörde dosya var yada yok volume boş değilse bu sefer o klasör içerisinde volume içerisinde ne varsa onu görürüz.

### Docker Bind Mounts

* Bind Mounts production ortamlarda yapılmıyor. Test ortamalrında yapılıyor. Production ortamda sadece volume kullanmalıyız.

* Örnek üzerinden gidelim. Örneğin bir web sitesi geliştiriyoruz. Kodlarımızı Nginx containera publish edeceğiz. Ve her güncellemede containerimızı restart etmemize gerek kalmayacak.

1. Docker Preferaences'e gidiyoruz ve Docker Resources altında File sharing altına dockerın erişebileceği dizini seçiyoruz.

2. Docker nginx image'ını alıyoruz.
   
        docker image pull nginx

3. Nginx Docker hub üzerinde anlatıldığı üzere "/usr/share/nginx/html" altında bulunan dizinde web sayfanmıza publish edeceği dosyaları tutuyor. 


4. Örnek bir index.html dosyası oluşturuyoruz ve içerisine aşağıdaki html'i yapıştırıyoruz.
   
        <h1>Merhaba Hos Geldiniz</h1>
        <img src="merhaba.jpg">

5. Ardından containerı başlatıyoruz ve volume oluşturmuş gibi bilgisaayarımızdaki index.html'in bulunduğu dosya dizinini veriyoruz.
   
        docker container run -d -p 80:80 -v <LOCALDE_BULUNAN_DOSYALARIN_DİZİNİ>:/usr/share/nginx/html nginx`

6. volume ile tüm özellikleri aynıdır. Volume kullanmak yerine pc'mizde bulunan dosyayı kullandık. Volume'ler docker'ın kedni objesi biz kendi dosyamızı kullandık.


## Docker Network

* Docker Network Driver
    * Bridge
    * Host
    * Macvlan
    * None
    * Overlay

* **Bridge:** Varsayılan driver'dır. Her docker kurulu host üzerinde bridger diriver ile yaratılmış "Bridge" adında network bulunur ve container yaratılırken aksi belirtilmediği taktirde bu networke bağlanır.
    * Aynı bridge networke bağlı container'lar birbirleriyle direk haberleşebilirler.
   #### Kullanıcı Tanımlı Bridge
   * Containerlar arasında network izalasyonu sağlamak istersek ayrı biridge network ile bunu sağlayabiliriz.
   * Varsayılan dışında ip blokları tanımlanabilir.
       * Örnek netwok tanımlama 
  
              docker network create --driver=bridge --subnet=10.10.0.0/16 --ip-range=10.10.10.0/24 --gateway=10.10.10.10 bridge2

   * Kullanıcı tanımlı bridge networkler birbirleri ile isim ile erişim sağlayabilir. Dns çözümü sağlamaktadır.
   * Containerlar çalışır durumdayken de kullanıcı tanınlı bridge networke bağlanabilir ve bağlantıyı kesebilirler.
   * Network yaratma komutu
  
         docker network create <CREATE_NETWORK_NAME> -> default brigde network yarattı

         docker network create <CREATE_NETWORK_NAME> --driver <CREATE_NETWORK_NAME> --> host verilebilir.

*  **Host:** Container'ımızın direk üstünde çalıştığı, hostun bağlı olduğu network ile direk görüşmesini istersek kullanacağımız driverdır. Bu networke bağlı containerda network izolasyonu olmamaktadır. Sanki o host üzerinde çalışan proses gibi hostun ağ kaynakalrını kullanır. 
Yani üzerine bağlı olduğu sistemin ağ yapısını kullanır. 
*  **MacVlan:** macVlan ile oluşturulan network'e bağlı containerlara mac adresi atar ve mevcut ağa bağlı fiziksel cihaz gibi davranmasını sağlar.
*  **None:** Container hiç bir şekilde ağ bağlantısı olmasın istersek kullanılır.
*  **Overlay:** ayrı hostlar üzerindeki containerların aynı ağda çalışıyormuş gibi çalışması istendiğinde kullanılır.

* Docker container network belirleme komutu;
  
      docker container run --net <NETWORK_DRIVER> <IMAGE>

* Docker port publish;

      docker container run -p <HOST_PORT>:<CONTAINER_PORT>  
      docker container run -p 8080:80

* Docker network delete;

        docker network rm <NETWORK_NAME> 

* Çalışan container nertwork değiştirme işelemi;

      docker network connect <NETWORK_NAME> <CONTAINER_NAME>

## Docker Logs

* Docker container içerisindeki tüm logları yakalamamız için "logs" management komutu kullanılıyor.
* Bir linux sisteminde log sistemi alttaki 3 ana bileşen tarafından yönetiliyor.
  * STDIN: standart input
  * STDOUT: Girdi sonucu oluşan output
  * STDERR: Girdi ile oluşan hata mesajı

* Örneğin nginx image ile container oluşturduk ve bu containera port publish edip browser üzerinden erişim sağladık. Bu işlemler sonucunda oluşan loglar nginx tarafından "/var/log/nginx" klasörleri altında "access.log" ve "error.log" olamak üzere tutuluyor. Docker bunları "logs" alt yapısı ile görebilmemiz için burada triq yapılmış ve "access.log"'a yapılan her girdiyi "/dev/stdout", "error.log"'a yapılan her girdi ise "/dev/stderr" çıkışına gönderecek şekilde ayar yapılmıştır.

* Container içerisindeki logları görmek için;

      docker logs <CONTAINER_ID_OR_NAME>

  * Eğer uygulama zaman log'u atmamışsa biz docker ile o logun hangi zamanda oluşturulduğunu görebiliriz.

        docker logs -t <CONTAINER_ID_OR_NAME>

  * Belirli bir zamana kadar olan logları görmek istersek;
  
        docker logs --until 2020-12-12T18:02:51.773980700Z <CONTAINER_ID_OR_NAME>
  
  * Belirli bir zamana kadar olan logları görmek istersek;

        docker logs --since 2020-12-12T18:02:51.773980700Z <CONTAINER_ID_OR_NAME>
  
  * Oluşturulan son x satırı görmek için;

        docker logs --tail <X> <CONTAINER_ID_OR_NAME>
        docker logs --tail 3 <CONTAINER_ID_OR_NAME> (son 3 satırı listeler)

  * Oluşan logları canlı olarak takip edebilmek için;

        docker logs -f <CONTAINER_ID_OR_NAME>

* Docker kurulumu yapıldığında varsayılan loging driver "json file"'dır.

## Docker Memory

* Container'ların ne kadar CPU,Memory ve network I/O'larını görmek için;
  
      docker stats -> (tüm containerlar)
      docker stats <CONTAINER_NAME_OR_ID>
      
* Container'a kullanacağı memory limiti vermek için;

      docker container run -d --memory=<MEMORY_SIZE> <IMAGE>

    * Swap işlemi yaparken memeory size'dan daha büyük değer girilmelidir.

          docker container run --memory=<MEMORY_SIZE> --memory-swap=<SWAP_SIZE> <IMAGE>

    * CPU limiti beilrlerken core ve logicall proccess ve core sayısı ile ayarlama yapabiliriz.

        * Core sayısı ile; Örneğin 8 core'un 1.5 tanesini kullan.

              docker container run -d --cpus="<CORE_COUNT>" 

        * Logical process'i direk atayarak kuallanabiliriz. Örneğin; 0. ve 3.'yü kullan.

              docker container run -d --cpus:"<CORE_COUNT>" --cpuset-cpus="<CORE_NUMBER>"


## Docker Environment Variables

* Linux sistemlerde variable listelemek için;

      printenv

* Docker container environment tanımlamak için;

      docker container run --env <ENVIRONMENT_NAME>=<ENVIRONMENT_VALUE> <IMAGE>
    
* Environment'ları tek tek tanımlama yerine dosya vermek için; Örneğin herhangi bir dizin içerisinde "env.list" adında environmentlarımızın tanımlandığı file olduğunu düşünelim.
  
    docker container run --env-file <ENVIRONMENT_PATH> <IMAGE>


## Docker Image

* Docker Hub default image registry'dir. Docker default olarak burayı kullanır.

* Imagelar docker hubdan alınırken version belirtilmez ise latest olarak belirlenen sürümü gelir. Eğer biz farklı sürümü istiyorsak;

      docker pull <IMAGE_NAME>:<VERSION>
      docker pull ubuntu:20.04

### Docker File

* **FROM :** Zorunlu talimattır. Oluşturulacak imajın hangi imajdan oluşturulacağının belirtildiği talimattır.

      FROM: ubuntu:18.04

* **RUN :** ile shellde komut çalışrırız.

      RUN apt-get install
      RUN apt-get update 

* **WORKDIR :** Image içerisinde hangi path'e geçilmesi isteniyorsa o path yazılır. Yani cd ile yapılan işlemi bu komut ile yaparız. Tek farkı eğer girilen path yok ise oluşturulur.

      WORKDIR usr/src/app

* **COPY :** Image içerisinde dosya kopyalarız.

      COPY /source /usr/scr/app
      (Hostun üstündeki source'u  Image içerisindeki app'a kopyalarız)

* **EXPOSE :** Imagedan oluşturulacak containerların hangi porttan erişilebileceği belirlenir.

      EXPOSE 80/tcp

* **CMD :** Imagedan container yaratıldığında varsayılan olarak çalışmasını istediğimiz komutu gireriz.

      CMD hello java

* **ADD :** Copy'den farklı olarak dosya kaynağının bir URL olmasınıda izin verir. Ayrıca kaynak olarak bir .tar uzantılı dosya verilirse, bu dosya image'a sıkıştıırlmış olarak değilde açılarak kopyalanır.

* **ENTRYPOINT :**  Her iki talimatta imah-ge'dan container yarattığımızda çalıştırılacak komutu belirtmemizi sağlar. Entrypoint ile girilen talimat runtime'da yani container çalışırken değiştirilemez. CMD ile yazılan değiştirilebilir.
  * Entrypoint ile Cmd aynı anda kullanılırsa; CMD içinde yazılan komut ENTRYPOINT'e parametre olarak eklenir.
      * Example;
  
            FROM centos:latest
            ENTRYPOINT ["ping"]
            CMD ["127.0.0.1"]

* **EXEC ile SHELL Farkı :**

      FROM centos:latest
      ENV TEST = "bu bir deneme"
            exec format                   shell format
      CMD["Java","uygulama"]           CMD java uygulama

   * Eğer komut shell formatında yazılırsa Docker bu imagedan container yaratıldığı zaman bu komutu varsayılan shell'i çalıştırarak onun içerisinde girer.
   * Eğer exec olarak çalışırsa Docker herhangi bir shell çalıştırma direk komutu process olarak çalıştırır.
   * Eğer ENTRYPOINT ile CMD kullanılacaksa exec formatı kullanılmalıdır.Shell kullanılırsa CMD 'deki komutlar parametre olarak ENTRYPOINT2e atanmaz.

* **ARG :** ARG ile de variable tanımlarsınız. Fakat bu variable sadece imaj oluşturulurken yani build aşamasında kullanılır. Imajın oluşturulmuş halinde bu variable bulunmaz. ENV ile imaj oluşturulduktan sonra da imaj içinde olmasını istediğiniz variable tanımlarsınız, ARG ile sadece oluştururken kullanmanız gereken variable tanımlarsınız.
  
      FROM ubuntu:latest
      WORKDIR /gecici
      ARG VERSION
      ADD https://www.python.org/ftp/python/${VERSION}/Python-${VERSION}.tgz .
      CMD ls -al


      Default değer verilebilir:
      ARG VERSION=3.7.1 


  * Dokcer build;
      
            docker build -t --build-arg VERSION=3.7.1

* Java için örnek oluşturduğum Dockerfile;
  
      FROM ubuntu:18.04
      RUN  apt-get update -y
      RUN  apt-get install default-jre -y
      WORKDIR /merhaba
      COPY /myapp .
      CMD ["java","merhaba"]


* Node.js için ;

      FROM node:10
      WORKDIR /usr/scr/app
      COPY package.json .
      RUN npm i
      COPY server.js .
      EXPOSE 8080
      CMD ["node", "server.js"]



* Bu Docker file'ı çalıştırmak için;

      docker image build -t <CREATE_IMAGE_NAME> .

* Image'ın katmanlarını görmek için;

      docker image history <IMAGE_ID_OR_NAME>


### Multi & Stage Build

* İmage yaratma aşamasını kademelere bölmemizi ve ilk kadamede yararttığımız image içerisindeki dosyaları bir sonraki kademede oluşturcağımız image içerisinde kopyalamamızı sağlamaktadır.


### Docker Commit

* Öncelikle image oluşturup daha sonra container oluşturuyorduk. Docker commit ile containerdan image oluşturma işlemi yapılmaktadır. Çok kullanılan ve önerilen yöntem değil.

      docker commit <CONTAINER_NAME_OR_ID> <CREATE_IMAGE_NAME>



### Docker Save & Load

* Docker "Save" ile sistemimizde bulunan bir image'ı tar.gz olarak kaydedebilir. Farklı sistemde "Load" ile açabiliriz.

      docker save <IMAGE_NAME_OR_ID> -o <CREATE_FILE_NAME>.tar
      docker load -i <FILA_NAME>


## Docker Compose

* Compose.yml'ı kaldırmak için;

      docker compose up -d

* Compose.yml'ı durdurup silmek (volume ve image hariç hepsi) için;

      docker compose down

* İsimlendirme işlemi;
      
      <COMPOSE'UN_İCİNDE_BULUNDUGU_FOLDER_ISMI>_<COMPOSEDA_VERİLEN_ISIM>

## Docker Orchestration

* Docker Orhestration, container dağıtımını,yönetimini ve ağ oluşturmasını otomatikleştirme ve zamanlama işlemine denir. 
  
  * Docker Swarm 
  * Kubernates

### Docker Swarm

* Docker engine gömülü cluster yönetimi ve dokcer orchestration sağlayan araçtır.

* Bir sanal makine üzerinde docker swarm cluster oluşturmak istersek yani bir makineyi manager olarak atamak istersek;

      docker swarm init
      or
      docker swarm init --advertise-addr <IP_ADDRESS>

* Swarm init ile Manager node oluşturduk. Manger node'da aşağıdaki komutu çalıştırısak yeni bir manager veya worker oluşturmak için yürütülmesi gereken komutu verir.
  
      manager için;
      docker swarm join-token manager 

      worker için;
      docker swarm join-token worker


* Manager node üzerinde aşağıdaki komutu verirsek cluster hakkında bilgi alırız.

      docker node ls

* Bir cluster'da birden fazla Manager olabilir. Fakat 1 tane Leader seçilir. 

* Leader Manager node'umuza 5 tane nginx container oluştur komutu vermek için;

      docker service create --name nginx --replicas=5 -p 80:80 nginx

  * Bu komuttan sonra Manager Node, Worker Node'lara gider ve bu node'lar üzerinde 5 tane nginx image'ından yaratılmış. Container oluşturur. Dikkat edilmesi gerekn konu; varsayılan olarak Mnager'larda Worker Node'dur. Yani bu node'lar üzerinde de containerlar oluşturulabilir.

* Çalışan servisleri görmek için;

      docker service ps <SERVICE_NAME>

#### Docker Service

* Docker Engine swarm moduna alındıktan sonra artık ona container komutu yerine servise komutu veririz. Docker service; Docker Swarm Cluster'da oluşturacağımız en temel objedir. Ama işin özünde node'larda yine container kaldırılığ indirilir. 

##### Docker Service Mode

* **Replicated :** Oluşturmak istediğiniz servisin kaç replica içereceğini belirtiriz. Swarm uygun olan node'lar üzerinde o sayıda node'lar oluşturur.

* **Global :** Oluşturmak isteğimiz servisin kaç adet replica'dan oluşacağınız belirmeyiz. Her node üzeriden 1 tane replica oluşturulur.

      docker service create --name <CREATE_SERVICE_NAME> --mode=global <IMAGE>
      docker service create --name glb --mode=global nginx

* Servisteki replica sayısını artırmak veya azaltmak için;

      docker service scale <SERVICE_NAME>=<SIZE>
      docker service scale test=3

* Servisi silmek için; direk siler. Dikkatli kullanılmalıdır.

      docker service rm <SERVICE_NAME>

* "service update" komutu ile mevcut sevisin özelliklerini güncelleyebiliyoruz. Örneğin image değiştiriyoruz. Dikkat edilmesi gerekn ise mevcut container üzerinde değişiklik yapmaz. Var olanı sıra ile siler ve yerine container oluşturur.

      docker service update --detach --update-delay 5s --update-parallelism 2 --image <IMAGE>

  * **"--update-delay :"** container update edildikten sonra bekleme süresi girilir.
  * **"--update-paralelism : "** aynı anda kaç container update edilecekse burada belirtilir.d


* update işleminde yaşanan herhangi bir sıkıntıdan dolayı rollback yapmak için;

      docker service rolback websrv


### Docker Secret

* Container'larda plain text olarak tutmamızın güvenlik zafiyeti oluşturacağı verileri secret objeleri şeklinde encrypted olarak transfer etmemizi sağlar.

* 2 türlü secret yaratabiliriz

  * 1. Dosya ile;  "username.txt" ve "password.txt" oluşturup içerisinde username ve password değerlerimi giriyorum.

            docker secret create <SECRET_NAME> username.txt
            docker secret create <SECRET_NAME> password.txt

  * 2. CLI kullanarak;

            echo "<VALUE>" | docker secret create <SECRET_NAME> -

* Servis oluşturulurken secret atamak için;

      docker service create --secret <SECRET_NAME> <IMAGE>

* Oluşturulan secret container içerisinde "run/secrets" altında bulunur.
* Secret güncellendiğinde container içerisi güncellenmez. Service update edilmelidir.

      docker service update --secret-rm <SECRET_NAME>  --secret-add <SECRET_NAME> <SERVICE_NAME>


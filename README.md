
# Kafka'da ZooKeeper ve Kraft: Farkları, Avantajları ve Dezavantajları

![Image](https://cdn-images-1.medium.com/v2/resize:fit:1200/1*Z9qWalYYaOPsD77CYpen6g.png)

Günümüzün veri yoğun dünyasında, gerçek zamanlı veri akışı ve işleme giderek daha önemli hale geliyor. İşte tam burada Apache Kafka devreye giriyor. Kafka, yüksek performanslı, ölçeklenebilir ve dağıtık bir veri akış platformu olarak, büyük miktarda verinin güvenilir bir şekilde toplanmasını, depolanmasını ve işlenmesini sağlar.


## Kafka Mimarisi

Apache Kafka, dağıtık ve ölçeklenebilir bir veri akış platformudur. Mimarisi de bu özellikleri desteklemek üzere tasarlanmıştır. Kafka, bir veya daha fazla sunucu düğümünden oluşan bir küme şeklinde çalışır. Bu sunucu düğümleri, Kafka brokerleri olarak adlandırılır ve veri akışını yönetirler.

Kafka'nın mimarisinde temel unsurlar şunlardır:

![Image](https://i.ibb.co/2nJgWV8/Screenshot-2023-05-19-161402.png)

1-Broker: Kafka kümesindeki her bir sunucu düğümüne broker denir. Brokerlar, gelen verileri alır, bunları konulara böler ve konulara bağlı olan parçaları depolarlar. Ayrıca, verilerin tüketici gruplarına dağıtımını yönetirler. Her broker, kendi diskinde veriyi depolar ve küme içindeki diğer brokerlarla iletişim kurar.

2-Konu (Topic): Verilerin akışının organize edildiği bir konu, Kafka'da temel bir birimdir. Bir konu, belirli bir veri türü veya veri kaynağıyla ilişkilendirilebilir. Örneğin, "sensör_verileri" veya "kullanici_aktiviteleri" gibi konular oluşturulabilir. Veriler, konulara yayınlanır ve tüketici grupları tarafından bu konulardan okunur.

3-Parçalama (Partition): Her konu, bir veya daha fazla parçaya bölünebilir. Parçalama, verilerin daha etkin bir şekilde dağıtılmasını ve işlenmesini sağlar. Her parçanın bir benzersiz bir numarası vardır ve ayrı bir veri akışını temsil eder. Parçalar, brokerlar arasında dengeli bir şekilde dağıtılır ve depolanır.

4-Üretici (Producer): Veri üreticileri, belirli bir konuya veri gönderen uygulamalardır. Üreticiler, verileri belirli bir konunun belirli bir parçasına yayınlarlar. Birden fazla üretici, aynı konuya veri gönderebilir ve parçalara eşzamanlı olarak yazabilir.

5-Tüketici (Consumer): Veri tüketicileri, belirli bir konudan veri okuyan uygulamalardır. Tüketici grupları şeklinde organize olabilirler. Her bir tüketici grubu, konunun farklı parçalarından veri okuyabilir ve böylece iş yükünü paralel olarak dağıtabilir. Her tüketici grubu, kendi ilerleme durumunu takip eder ve kaldığı yerden devam eder.

 Kafka ekosisteminin bir parçası olarak, ZooKeeper ve Kraft, Kafka'nın yönetim ve yüksek kullanılabilirlik yeteneklerini destekleyen iki önemli bileşendir.

## Zookeeper

ZooKeeper, Kafka ekosisteminde kullanılan uzun süredir devam eden bir koordinasyon hizmetidir.
Kafkanın topic,broker,consumer,partition konuları hakkında metadata depolar ve sürdürür.
ZooKeeper "leader election", "fault detection" ve diğer kritik görevlerde çok önemli bir rol oynar.
Kafka cluster içinde yüksek kullanılabilirlik ve hata toleransı(fault tolerance) sağlar.
ZooKeeper'ın avantajları arasında güvenilirliği, Kafka topluluğu içinde yerleşik kullanımı ve gelişmiş teknolojisi yer alır.
Ancak ZooKeeper'ın yönetim karmaşıklığı, ek bir bileşene bağımlılık ve potansiyel performans sınırlamaları gibi dezavantajları da vardır.

**Kısa bir kod üzerinde örneklemek gerekirse:**
```java
import org.apache.zookeeper.*;
import java.io.IOException;

public class ZooKeeperExample {
    private static final String CONNECTION_STRING = "localhost:2181";
    private static final int SESSION_TIMEOUT = 5000;

    public static void main(String[] args) {
        try {
            // 1
            ZooKeeper zooKeeper = new ZooKeeper(CONNECTION_STRING, SESSION_TIMEOUT, null);

            // 2
            String zNodePath = "/my-node";
            byte[] data = "Hello, ZooKeeper!".getBytes();

            // 3
            zooKeeper.create(zNodePath, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

            System.out.println("Düğüm oluşturuldu: " + zNodePath);

            // 4
            zooKeeper.close();
        } catch (IOException | KeeperException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

1-ZooKeeper bağlantısı oluşturma: ZooKeeper sınıfını kullanarak ZooKeeper sunucusuna bağlanıyoruz. Bağlantı dizesi (CONNECTION_STRING) ve oturum süresi (SESSION_TIMEOUT) gibi parametreleri belirtiyoruz.

2-Yeni düğüm oluşturma parametrelerinin belirlenmesi: Oluşturulacak düğümün yolunu (zNodePath) ve verisini (data) belirliyoruz. Veriyi byte dizisine dönüştürerek saklıyoruz.

3-Düğüm oluşturma: zooKeeper.create yöntemiyle düğümü oluşturuyoruz. Yolunu, verisini, izinlerini (ZooDefs.Ids.OPEN_ACL_UNSAFE) ve oluşturma modunu (CreateMode.PERSISTENT) belirtiyoruz.

4-ZooKeeper bağlantısının kapatılması: zooKeeper.close() ile ZooKeeper bağlantısını kapatıyoruz.



## Kraft(KafkaController)

Farklar: Kraft, ZooKeeper'ın Kafka kümesini yönetmedeki rolünün yerini almayı amaçlayan, Kafka ekosistemine yeni bir eklemedir. Küme meta verilerini yönetmekten ve idari görevleri yerine getirmekten sorumlu olan Kafka Denetleyicisi için alternatif bir uygulama sağlar. İşte Kraft hakkında bazı önemli noktalar:
Basitleştirilmiş Yönetim: Kraft, ZooKeeper gibi harici bir koordinasyon hizmetine bağımlılığı ortadan kaldırarak yönetim süreçlerini basitleştirir. ZooKeeper'ın yapılandırması ve bakımıyla ilgili karmaşıklığı azaltır.

![Image](https://i.ibb.co/GWp60Xk/Screenshot-2023-05-19-163008.png)

İyileştirilmiş Performans: Kraft, ZooKeeper'a kıyasla daha iyi performans sunar. Bir başarısızlık durumunda yeni bir lider seçmek için gereken süreyi azaltarak daha hızlı lider seçimi sağlar. Bu, daha hızlı yük devretme ve azaltılmış kapalı kalma süresi ile sonuçlanır.

Ölçeklenebilirlik: Kraft, daha büyük Kafka kümelerini daha verimli bir şekilde işlemek için tasarlanmıştır. Gelişmiş ölçeklenebilirlik sunar ve daha yüksek yükleri ve daha fazla sayıda aracıyı destekleyebilir.

Geçiş ve Adaptasyon: ZooKeeper'dan Kraft'a geçiş, dikkatli bir planlama ve uygulama gerektirir. Mevcut Kafka kümesinin meta verilerinin taşınmasını ve Kraft'ın yeni uygulamasıyla uyumluluğun sağlanmasını içerir.

Benimseme ve Topluluk Desteği: Daha yeni bir teknoloji olarak Kraft, ZooKeeper'a kıyasla daha küçük bir kullanıcı tabanına ve daha az yerleşik en iyi uygulamaya sahip olabilir. Bu, sınırlı topluluk desteğine ve mevcut kaynaklara yol açabilir.

**Kısa bir kod üzerinde örneklemek gerekirse:**

```java
import org.apache.kafka.clients.admin.AdminClient;
import org.apache.kafka.clients.admin.AdminClientConfig;
import org.apache.kafka.clients.admin.CreateTopicsResult;
import org.apache.kafka.clients.admin.NewTopic;

import java.util.Collections;
import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class KraftExample {
    private static final String BOOTSTRAP_SERVERS = "localhost:9092";

    public static void main(String[] args) {
        // 1
        Properties config = new Properties();
        config.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
        AdminClient adminClient = AdminClient.create(config);

        // 2
        String topicName = "my-topic";
        int numPartitions = 3;
        short replicationFactor = 2;
        NewTopic newTopic = new NewTopic(topicName, numPartitions, replicationFactor);

        // 3
        CreateTopicsResult createTopicsResult = adminClient.createTopics(Collections.singleton(newTopic));

        try {
            // 4
            createTopicsResult.all().get();
            System.out.println("Konu oluşturuldu: " + topicName);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            // 5
            adminClient.close();
        }
    }
}
```
1-AdminClient yapılandırması: Kafka kümesine bağlanmak için gerekli yapılandırma ayarlarını Properties nesnesine tanımlıyoruz ve bu ayarları kullanarak AdminClient oluşturuyoruz.

2-Yeni konu oluşturma parametrelerinin belirlenmesi: Oluşturulacak konunun adını (topicName), parçalamalarının sayısını (numPartitions) ve çoğaltma faktörünü (replicationFactor) belirliyoruz.

3-Konunun oluşturulması: NewTopic sınıfını kullanarak yeni bir konu nesnesi oluşturuyoruz ve önceki adımda belirlediğimiz parametreleri kullanarak bu konu nesnesini yapılandırıyoruz.

4-Konu oluşturma işleminin tamamlanması beklenmesi: adminClient.createTopics yöntemiyle konuyu oluşturuyoruz ve createTopicsResult.all().get() ifadesi ile oluşturma işleminin tamamlanmasını bekliyoruz. İstisnaları ele alarak hataların işlenmesini sağlıyoruz. Oluşturma işlemi tamamlandığında, başarı mesajını ekrana yazdırıyoruz.

5-AdminClient kapatma: adminClient.close() ile AdminClient bağlantısını kapatıyor

## References

- https://www.baeldung.com/kafka-shift-from-zookeeper-to-kraft

- https://www.conduktor.io/kafka/kafka-kraft-mode/

- https://www.cloudkarafka.com/blog/part1-kafka-for-beginners-what-is-apache-kafka.html

- https://medium.com/swlh/apache-kafka-what-is-and-how-it-works-e176ab31fcd5




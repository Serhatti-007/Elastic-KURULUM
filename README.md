# 🛠️ Ubuntu 24.04.2 Üzerinde ELK (Elasticsearch, Logstash, Kibana) Kurulumu

Bu kılavuz, **Ubuntu 24.04.2** üzerinde **ELK Stack (Elasticsearch, Logstash, Kibana)** kurulumunu adım adım anlatmaktadır. 📌

---

## ╰┈➤ 1. Gerekli Bağımlılıkları Kurun
Öncelikle, sistem paketlerini güncelleyin ve **curl** aracını yükleyin:
```bash
sudo apt update && sudo apt install -y curl
```

---

## ╰┈➤ 2. Elasticsearch Deposu ve GPG Anahtarını Ekleme
Elastic deposunu eklemek için **GPG anahtarını** indiriyoruz:
```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
```

Elastic deposunu sistem kaynaklarına ekleyin:
```bash
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

Depo listesini güncelleyin:
```bash
sudo apt update
```

---

## ╰┈➤ 3. Elasticsearch, Logstash ve Kibana'yı Yükleme
```bash
sudo apt install -y elasticsearch logstash kibana
```

---

## ╰┈➤ 4. Elasticsearch Yapılandırması ve Güvenlik Ayarları
Elasticsearch yapılandırma dosyasını açın:
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Aşağıdaki ayarları ekleyin veya düzenleyin "network.host= ubuntu ip":
```yaml
network.host: 192.168.65.139
http.port: 9200
http.host: 0.0.0.0
```
Dosyayı kaydedin (**Ctrl+X → Y → Enter**).

---

## ╰┈➤ 5. Elasticsearch Servisini Başlatma ve Kontrol Etme
```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
sudo systemctl status elasticsearch
```
Bağlantıyı test edin:

```bash
curl -X GET "http://192.168.65.139:9200"
```
Eğer bir json yerine "empty" çıktısını alıyorsanız aşağıdaki kurulum adımlarından devam edin.

---

## ╰┈➤ 6. Elasticsearch Güvenlik Ayarları ve Kullanıcı Tanımlama
Öncelikle **şifre belirleyin**:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add bootstrap.password
```

Kibana'nın Elasticsearch'e bağlanabilmesi için `/etc/kibana/kibana.yml` dosyasını açıp aşağıdaki bilgileri ekleyin:
```yaml
elasticsearch.username: "elastic"
elasticsearch.password: "<yeni_şifre>"
```
Elasticsearch yeniden başlatın:
```bash
sudo systemctl restart elasticsearch
```
Elasticsearch sertifikasını sisteme kopyalayın ve güncelleyin:
```bash
sudo cp /etc/elasticsearch/certs/http_ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

Bağlantıyı kimlik doğrulama ile test edin:
```bash
curl -X GET -u elastic:yeni_sifre "https://192.168.65.139:9200/_cluster/health"
```

---

## ╰┈➤ 7. Logstash Yapılandırması
```bash
sudo systemctl enable logstash
sudo systemctl start logstash
sudo systemctl status logstash
```

Logstash için bir yapılandırma dosyası oluşturun:
```bash
sudo nano /etc/logstash/conf.d/logstash.conf
```
Aşağıdaki içeriği ekleyin:
```yaml
input {
    beats {
        port => 5044
    }
}

output {
    elasticsearch {
        hosts => ["http://192.168.65.139:9200"]
    }
}
```
Logstash'i yeniden başlatın:
```bash
sudo systemctl restart logstash
```
Logstash'in çalışmasını doğrulamak için:
```bash
sudo tail -f /var/log/logstash/logstash-plain.log
```

---

## ╰┈➤ 8. Kibana Yapılandırması ve Başlatılması
```bash
sudo systemctl enable kibana
sudo systemctl start kibana
sudo systemctl status kibana
```

Kibana yapılandırma dosyasını açın:
```bash
sudo nano /etc/kibana/kibana.yml
```
Aşağıdaki ayarları ekleyin veya değiştirin:
```yaml
server.host: "localhost"
elasticsearch.hosts: ["http://192.168.65.139:9200"]
```

**🛠️ Encryption Key Hatasını Düzeltme**
Bazı eklentiler için **şifrelenmiş veri saklama** gereklidir. Bu nedenle, 32 karakterli bir alfanumerik şifre oluşturabilirsiniz (kibana.yml dosyası içerisine yerleştirin):
```yaml
xpack.encryptedSavedObjects.encryptionKey: "8f3e472b9c4f1e04a1d9b0cf7a924073"
```

Kibana'yı yeniden başlatın:
```bash
sudo systemctl restart kibana
```

---

## ╰┈➤ 9. Son Kontroller ve Erişim
Tüm servislerin çalıştığını doğrulamak için:
```bash
sudo systemctl status elasticsearch
sudo systemctl status logstash
sudo systemctl status kibana
```
Her biri **active (running)** durumda olmalıdır.

Kibana arayüzüne erişmek için tarayıcınızda şu adresi açın:
```
http://localhost:5601
```

---

## ╰┈➤ 10. Kibana İlk Kurulum ve Token Doğrulama
Eğer Kibana sizden **enrollment token** isterse, şu komutu çalıştırarak token oluşturun:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
Token'i Kibana giriş ekranına yapıştırarak devam edin.

Daha sonra, Kibana doğrulama kodu isteyebilir. Bunu almak için:
```bash
sudo /usr/share/kibana/bin/kibana-verification-code
```
Kodu girin ve kullanıcı adı/şifreniz ile giriş yapın.

---


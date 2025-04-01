<body>
    <h1>ELK Stack Kurulumu - Ubuntu 24.04.2</h1>
    <p>Bu rehber, Ubuntu 24.04.2 üzerinde ELK (Elasticsearch, Logstash, Kibana) Stack'in kurulumunu adım adım açıklar.</p>
    <hr>
    <h2>1. Gerekli Bağımlılıkları Kurun</h2>
    <pre><code>sudo apt update && sudo apt install -y curl</code></pre>
    <hr>
    <h2>2. Elasticsearch Deposu ve GPG Anahtarını Ekleme</h2>
    <pre><code>curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg</code></pre>
    <pre><code>echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list</code></pre>
    <pre><code>sudo apt update</code></pre>
    <hr>
    <h2>3. Elasticsearch, Logstash ve Kibana'yı Yükleme</h2>
    <pre><code>sudo apt install -y elasticsearch logstash kibana</code></pre>
    <hr>
    <h2>4. Elasticsearch Yapılandırması ve Güvenlik Ayarları</h2>
    <p>Elasticsearch yapılandırma dosyasını düzenleyin:</p>
    <pre><code>sudo nano /etc/elasticsearch/elasticsearch.yml</code></pre>
    <p>Şu ayarları ekleyin veya değiştirin:</p>
    <pre><code>network.host: 192.168.65.139
http.port: 9200
http.host: 0.0.0.0</code></pre>
    <p>Dosyayı kaydedip çıkın (<code>Ctrl+X</code>, ardından <code>Y</code>, sonra <code>Enter</code>).</p>
    <hr>
    <h2>5. Elasticsearch Servisini Başlatma ve Kontrol Etme</h2>
    <pre><code>sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
sudo systemctl status elasticsearch</code></pre>
    <p>Bağlantıyı test edin:</p>
    <pre><code>curl -X GET "http://192.168.65.139:9200"</code></pre>
    <p>Eğer boş yanıt alırsanız, kullanıcı adı ve şifre ile bağlanmanız gerekebilir.</p>
    <hr>
    <h2>6. Elasticsearch Güvenlik Ayarları ve Kullanıcı Tanımlama</h2>
    <p>Şifre belirlemek için:</p>
    <pre><code>sudo /usr/share/elasticsearch/bin/elasticsearch-keystore add bootstrap.password</code></pre>
    <p>Kibana'nın Elasticsearch'e bağlanabilmesi için şu ayarları <code>kibana.yml</code> dosyasına ekleyin:</p>
    <pre><code>elasticsearch.username: "elastic"
elasticsearch.password: "&lt;yeni_şifre&gt;"</code></pre>
    <p>Elasticsearch sertifikasını sisteme kopyalayın ve güncelleyin:</p>
    <pre><code>sudo cp /etc/elasticsearch/certs/http_ca.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates</code></pre>
    <p>Kimlik doğrulama ile test edin:</p>
    <pre><code>curl -X GET -u elastic:&lt;password&gt; "https://192.168.65.139:9200/_cluster/health"</code></pre>
    <hr>
    <h2>7. Logstash Yapılandırması</h2>
    <p>Logstash’i otomatik başlatın ve durumu kontrol edin:</p>
    <pre><code>sudo systemctl enable logstash
sudo systemctl start logstash
sudo systemctl status logstash</code></pre>
    <p>Yapılandırma dosyası oluşturun:</p>
    <pre><code>sudo nano /etc/logstash/conf.d/logstash.conf</code></pre>
    <p>Şu içeriği ekleyin:</p>
    <pre><code>input {
    beats {
        port => 5044
    }
}

output {
    elasticsearch {
        hosts => ["http://192.168.65.139:9200"]
    }
}</code></pre>
    <p>Logstash’i yeniden başlatın:</p>
    <pre><code>sudo systemctl restart logstash</code></pre>
    <p>Logları doğrulamak için:</p>
    <pre><code>sudo tail -f /var/log/logstash/logstash-plain.log</code></pre>
    <hr>
    <h2>8. Kibana Yapılandırması ve Başlatılması</h2>
    <p>Kibana’yı otomatik başlatın ve durumunu kontrol edin:</p>
    <pre><code>sudo systemctl enable kibana
sudo systemctl start kibana
sudo systemctl status kibana</code></pre>
    <p>Yapılandırma dosyasını açın:</p>
    <pre><code>sudo nano /etc/kibana/kibana.yml</code></pre>
    <p>Şu ayarları ekleyin veya değiştirin:</p>
    <pre><code>server.host: "0.0.0.0"
elasticsearch.hosts: ["http://192.168.65.139:9200"]
elasticsearch.username: "elastic"
elasticsearch.password: "&lt;yeni_şifre&gt;"
xpack.encryptedSavedObjects.encryptionKey: "8f3e472b9c4f1e04a1d9b0cf7a924073"</code></pre>
    <p>Kibana’yı yeniden başlatın:</p>
    <pre><code>sudo systemctl restart kibana</code></pre>
    <hr>
    <h2>9. Son Kontroller ve Erişim</h2>
    <p>Tüm servislerin durumunu doğrulayın:</p>
    <pre><code>sudo systemctl status elasticsearch
sudo systemctl status logstash
sudo systemctl status kibana</code></pre>
    <p>Hepsi <strong>active (running)</strong> durumda olmalıdır.</p>
    <p>Kibana’ya erişmek için:</p>
    <pre><code>http://192.168.65.139:5601</code></pre>
    <hr>
    <h2>10. Kibana İlk Kurulum ve Token Doğrulama</h2>
    <p>Eğer Kibana sizden <strong>enrollment token</strong> isterse, şu komutu çalıştırın:</p>
    <pre><code>sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana</code></pre>
    <p>Token’i Kibana giriş ekranına yapıştırın.</p>
    <p>Doğrulama kodu isteyebilir. Bunu almak için:</p>
    <pre><code>sudo /usr/share/kibana/bin/kibana-verification-code</code></pre>
    <p>Doğrulama kodunu girin ve kullanıcı adı/şifre ile giriş yapın.</p>
    <hr>
    <h2>Not:</h2>
    <p>Eğer herhangi bir hata ile karşılaşırsanız, ilgili servislerin loglarını kontrol etmeyi unutmayın:</p>
    <ul>
        <li><strong>Elasticsearch Logları:</strong> <code>/var/log/elasticsearch/elasticsearch.log</code></li>
        <li><strong>Logstash Logları:</strong> <code>/var/log/logstash/logstash-plain.log</code></li>
        <li><strong>Kibana Logları:</strong> <code>/var/log/kibana/kibana.log</code></li>
    </ul>
    <hr>
    <p>Bu rehber, ELK Stack kurulumunuzu başarıyla tamamlamanıza yardımc

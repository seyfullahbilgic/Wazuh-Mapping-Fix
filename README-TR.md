Wazuh Dashboard illegal_argument_exception Hatası ve Çözümü (manager.name)
Wazuh Dashboard üzerinde grafikler yüklenirken veya sıralama yaparken alınan "Text fields are not optimised for operations..." hatasını kalıcı olarak nasıl düzelteceğinizi bu rehberde bulabilirsiniz.

Sorunun Kaynağı
Wazuh, manager.name veya agent.name gibi alanların "keyword" (tam eşleşen veri) tipinde olmasını bekler. Eğer index şablonlarında (template) bir çakışma olursa, OpenSearch bu alanları otomatik olarak "text" tipine çeker. text alanlar üzerinde gruplandırma ve sıralama yapılamadığı için Dashboard hata verir.

Çözüm Adımları

1. Gelecek Indexleri Düzeltme (Kalıcı Çözüm)

Öncelikle, yarın oluşacak indexlerin tekrar bozulmaması için yüksek öncelikli bir yama şablonu oluşturuyoruz. Wazuh Dashboard > Dev Tools kısmına şu komutu yapıştırıp çalıştırın:

PUT /_index_template/wazuh_fix_manager
{
  "index_patterns": ["wazuh-alerts-4.x-*"],
  "priority": 500,
  "template": {
    "mappings": {
      "properties": {
        "manager": {
          "properties": {
            "name": {
              "type": "keyword",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        }
      }
    }
  }
}

Bu komutla priority: 500 vererek, bu kuralın sistemdeki diğer tüm hatalı şablonları ezmesini sağlıyoruz.

2. Mevcut (Hata Veren) Indexleri Kurtarma

Yukarıdaki şablon sadece yeni indexler için geçerlidir. Hata aldığınız mevcut günün indexlerini düzeltmek için şu komutu kullanın:

PUT /wazuh-alerts-*/_mapping
{
  "properties": {
    "manager.name": {
      "type": "text",
      "fielddata": true
    }
  }
}

Not: Eğer belirli bir günün indexinde hata devam ederse, wazuh-alerts-* yerine doğrudan o günün index adını (örn: wazuh-alerts-4.x-2026.01.05) yazarak komutu çalıştırabilirsiniz.

3. Index Pattern Güncelleme

Değişikliklerin Dashboard'a yansıması için:

Stack Management > Index Patterns sekmesine gidin.

wazuh-alerts-* pattern'ini seçin.

Sağ üstteki "Refresh field list" (Yenileme) butonuna tıklayarak alanları tekrar taratın.

Özet
Bu işlemle sistemin manager.name alanını hatalı şekilde text olarak algılamasının önüne geçtik ve mevcut indexler için geçici bir "fielddata" izni vererek Dashboard'un tekrar çalışmasını sağladık.

Not: Bu sorun genellikle özel index şablonları yüklendiğinde veya sistem güncellemelerinde şablonların üzerine yazıldığında ortaya çıkar.

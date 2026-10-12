# Turkuaz LogiBoard – Depo İçi Ekran

ERP'den gelen `Turkuaz_Logiboard` verisini depo içi ekranlarda (Omma / Signalive) gösteren tek dosyalık HTML içerik: [`202610121737.html`](202610121737.html).
Dosya adı revizyonun tarih ve saatidir (YYYYAAGGSSDD, Türkiye saati); her revizyonda yeni adla gelir.
Sahadaki personelin 3–10 metreden bakıp **2 saniyede ne yapacağını anlaması** için andon panosu mantığıyla tasarlandı.

![Önizleme](docs/onizleme.png)

## Ekranda ne var?

1. **Öncelik bandı (en üstte, her zaman görünür)**
   - Kırmızı **ÖNCELİK**: gecikmesi olan gruplardan **cut-off'u en erken olan** *rota grubu · süreç* ve geciken sayısı, örneğin "İSTANBUL 2.SEVKİYAT · [SEVKİYAT] – 4 GECİKEN İŞ". Süreç beyaz etiketle ayrılır; böylece grup adındaki "SEVKİYAT" ile karışmaz. İkinci satırda "Sırada: …" listesi ve toplam geciken sayısı yer alır.
   - Yeşil **GECİKEN İŞ YOK**: hiçbir yerde geciken iş yoksa görünür.
   - Sarı **VERİ ESKİ**: veri `staleMinutes` dakikadan eskiyse ya da bağlantı koptuysa bandın sol kutusu sarıya döner ve alt satırda "Veri güncel değil — gösterilen durum değişmiş olabilir." yazar.
2. **Andon panosu**
   - Rota grupları **sevkiyat cut-off saatine göre** sıralanır (aşağıdaki tablo). Grup adının altında "Cut-off 14:30" yazar.
   - **Ertesi günün grupları sona alınır.** GECİKEN ve ZAMANINDA değeri olmayan, yalnızca PLANLI değeri olan grup ertesi günün işidir. Bu gruplar gri görünür ve "ERTESİ GÜN" etiketi taşır. Cuma günü önce Cumartesi grupları, diğer günler önce hafta içi grupları gelir.
   - Her grup için süreçler TOPLAMA → PAKETLEME → SEVKİYAT sırasıyla, GECİKEN / ZAMANINDA / PLANLI değerleriyle gösterilir; değerler ortalıdır.
   - Geciken işi olan kutu **kırmızı** yanar ve beyaz kutu içinde geciken sayısını gösterir. Bugünün gruplarında geciken yoksa yeşil onay işareti görünür. Grup adının altında "Geciken yok" yazmaz.
   - Süreç başlıklarında o süreçteki toplam geciken sayısı rozet olarak görünür, örneğin "5 GECİKEN".
3. **Detay tablosu**
   - Yalnızca bugünün grupları detayda gösterilir; sıra panoyla aynıdır (cut-off). Ertesi günün gruplarının detayı gösterilmez.
   - En üstte, her sayfada **ROTA GRUBU TOPLAMI** satırı durur. Bu satır grubun ERP toplamlarını gösterir; panodaki rakamlarla aynıdır.
   - Altında **bayi kırılımı** yer alır. Yalnızca GECİKEN ya da ZAMANINDA değeri olan bayiler listelenir. Yalnızca PLANLI değeri olan bayiler ertesi günün işidir ve gösterilmez; sayfa bilgisinde "bugün işi olmayan N bayi gizli" yazar.
   - **Geciken bayiler en üstte**, kırmızı zemin üzerinde gösterilir. Bayinin hangi süreçte geciktiği koyu kırmızıyla işaretlenir.
   - Sayfalar `pageSeconds` aralıkla döner. Panoda detayı gösterilen grubun satırında "· detayda" yazar.
   - Çok grup olduğunda (yaklaşık 5 ve üzeri) pano ve detay sırayla tam ekran gösterilir. Öncelik bandı bu sırada da görünür kalır.

Ekran 1080p, 4K, ultra geniş ve dikey ekranlarda ölçeklenir. Sayılar asla kesilmez; sığmazsa yazı küçülür.

Alan eşleşmesi: `…Past` → **GECİKEN**, `…Today` → **ZAMANINDA**, `…Planned` → **PLANLI**. Başlıkları değiştirmek için dosyadaki `PERIODS` listesini düzenleyin.

### Cut-off saatleri (`CONFIG.cutoffs`)

| Rota grubu | Cut-off |
|---|---|
| ANKARA SICAK | 10:30 |
| ADAPAZARI SICAK | 11:00 |
| İSTANBUL 1.SEVKİYAT | 11:30 |
| BURSA SICAK | 12:00 |
| İSTANBUL 2.SEVKİYAT | 14:30 |
| ANADOLU | 17:00 |
| CUMARTESİ İSTANBUL (Cumartesi) | 12:30 |
| CUMARTESİ ANADOLU (Cumartesi) | 13:30 |

- **Eşleşme kuralı:** Ad, büyük/küçük harf ve noktalama farkı gözetmeden eşleşir; örneğin "İstanbul 1. Sevkiyat" = "İSTANBUL 1.SEVKİYAT". Birebir eşleşme yoksa, listedeki adın tüm kelimeleri grup adında geçiyorsa eşleşir; örneğin "CUMARTESİ ANADOLU SEVKİYAT" → 13:30.
- **Listede olmayan grup** saatsiz gösterilir ve kendi bloğunun sonuna gider.
- **Farklı yazımlar:** Bir girdiye `aliases: ['...']` eklenebilir.
- **Yeni grup:** Listeye bir satır eklenir, örneğin `{ name: 'İZMİR SICAK', time: '13:00' }`. Cumartesi grupları için `sat: true` eklenir.

## Omma'ya kurulum

1. Omma'da Logiboard için ayrı bir **içerik (content)** oluşturun ve **Code Editor**'e HTML dosyasının tamamını yapıştırın. Dosya kendi içinde tamdır; dışarıdan font, kütüphane ya da görsel yüklemez.
2. Bu içeriğe **Turkuaz_Logiboard (#3887)** veri kaynağını bağlayın. Başka bir projenin içeriğine ya da sahnesine koymayın: sayfa yalnızca içinde çalıştığı içeriğe bağlı veri kaynaklarını görebilir.
3. İçeriği kaydedin, **Test → Update Test** ile kontrol edin ve ekranlara yayınlayın.

Playlist'teki "HTML ekle" widget'ını ya da Web sitesi/URL öğesini kullanmayın. Omma'nın `window.omma` yardımcısı yalnızca Code Editor içeriğinde garanti edilir.

Sayfa Omma'nın [content helper](https://github.com/signalive/content-api-docs/blob/master/content-helper.md) API'sini kullanır: `omma.setVersion('v1')`, `omma.ready()` ve `omma.datasource`. Veri güncellendiğinde (`ds.on('update')`) ekran sayfa yenilenmeden güncellenir ve gösterdiği grup ile sayfada kalır.

**Güvenlik:**
- Sayfa Omma yardımcısını bulamazsa **örnek veri göstermez**. 15 saniye bekler, ardından ekrana açıklayıcı bir mesaj yazar ve aramaya devam eder. Sahada hiçbir koşulda uydurma rakam görünmez.
- Veri kaynağı görünmezse ekranda cihazın gördüğü veri kaynakları (ad, numara, ilk alanlar) listelenir ve sayfa 10 saniyede bir yeniden dener.

### Veri kaynağı ve ERP bağlantısı

- Ekran veriyi yalnızca Omma'nın veri kaynağından okur. ERP'nin veriyi Omma'ya nasıl ilettiği (veri kaynağının "force" adresi ve token'ı, ya da Omma'nın periyodik çektiği bir adres veya betik) ekranı etkilemez.
- **Token'ı ve force adresini bu HTML'e yazmayın.** HTML her ekrana dağıtılır; içindeki bilgi her cihazda okunabilir. Token yalnızca ERP tarafındaki entegrasyonda dursun.

## Beklenen veri yapısı

Veri doğrudan dizi olarak ya da `Turkuaz_Logiboard` anahtarı altında gelebilir (JSON metni olarak, sarmalanmış ya da farklı harf düzeniyle gelmesi de sorun değil):

```json
[
  {
    "DispatchRouteGroupTitle": "ANADOLU",
    "TotalPickingPast": 0,  "TotalPickingToday": 1,     "TotalPickingPlanned": 2117,
    "TotalPackingPast": 0,  "TotalPackingToday": 1452,  "TotalPackingPlanned": 891,
    "TotalShippingPast": 1, "TotalShippingToday": 1920, "TotalShippingPlanned": 1235,
    "Details": [
      { "CustomerTitle": "OTOMOL İZMİR",
        "PickingPast": 0, "PickingToday": 0, "PickingPlanned": 4,
        "PackingPast": 0, "PackingToday": 16, "PackingPlanned": 11,
        "ShippingPast": 1, "ShippingToday": 9, "ShippingPlanned": 10 }
    ]
  }
]
```

### `Details` alanları

Detay tablosunun sütunları **veriden otomatik çıkarılır**. ERP'de alan eklenip çıkarılınca kodu değiştirmek gerekmez:

- Metin alanları solda gösterilir (`CustomerTitle` → **BAYİ ADI**).
- `PickingPast`, `PackingToday`, `ShippingPlanned` gibi alanlar (başında `Total` olsun ya da olmasın) **Toplama / Paketleme / Sevkiyat** başlıkları altında gruplanır. Geciken bayileri bulmak, sıralamak ve ertesi günün işini ayırmak için bu alanlar kullanılır.
- Diğer sayısal alanlar en sağda gösterilir.
- Yeni metin alanlarının başlıklarını Türkçeleştirmek için dosyadaki `LABELS` nesnesine alan adı ekleyin, örneğin `OrderNo: 'Sipariş No'`.
- Gösterilmesini istemediğiniz alanları `CONFIG.hiddenDetailColumns` içine yazın, örneğin `['Id']`.

## Ayarlar (`CONFIG`)

| Ayar | Varsayılan | Açıklama |
|---|---|---|
| `title` | `Turkuaz LogiBoard` | Üst başlık |
| `datasourceId` | `3887` | Omma veri kaynağı numarası (force adresindeki `/datasource/3887/`); önce buna bakılır |
| `datasourceName` | `Turkuaz_Logiboard` | Omma veri kaynağı adı (numara bulunamazsa) |
| `rootKey` | `Turkuaz_Logiboard` | Veri nesne içinde geliyorsa listenin anahtarı |
| `groupTitleKey` | `DispatchRouteGroupTitle` | Grup başlığı alanı |
| `detailsKey` | `Details` | Alt detay listesi alanı |
| `cutoffs` | (yukarıdaki tablo) | Rota gruplarının cut-off saatleri ve sıralaması |
| `detailActiveOnly` | `true` | Detayda yalnızca GECİKEN/ZAMANINDA değeri olan bayiler |
| `rowUnit` | `bayi` | Detay satırlarının birimi ("123 bayi") |
| `pageSeconds` | `10` | Detay sayfasının ekranda kalma süresi (sn) |
| `boardSeconds` | `20` | Pano ve detay sırayla gösterilirken panonun süresi (sn) |
| `detailPagesPerTurn` | `2` | Sırayla gösterimde panoya dönmeden önce gösterilecek detay sayfası sayısı |
| `minDetailRows` | `4` | Panonun altına bundan az detay satırı sığıyorsa pano ve detay sırayla gösterilir |
| `hideZeroDetailRows` | `true` | `detailActiveOnly` kapalıysa tüm değerleri 0 olan bayileri gizle |
| `sortDetailRows` | `true` | Geciken bayileri en üste al |
| `detailShowPlanned` | `true` | Detayda PLANLI sütunlarını göster |
| `detailMaxPages` | `0` | >0 ise grup başına en fazla bu kadar detay sayfası (geciken bayiler her zaman gösterilir) |
| `showProcessOverdue` | `true` | Süreç başlığında "N GECİKEN" rozeti |
| `pastUnit` | `iş` | Geciken sayısının birimi ("1 geciken iş") |
| `blinkPriority` | `true` | Geciken iş varsa öncelik kutusu yavaşça yanıp söner |
| `staleMinutes` | `30` | Veri bu süreden eskiyse "VERİ ESKİ" uyarısı (0 = kapalı) |
| `hiddenDetailColumns` | `[]` | Detayda gizlenecek alanlar |
| `dataUrl` | `''` | Omma veri kaynağı yerine ERP'nin JSON adresinden doğrudan çekmek için (opsiyonel, CORS izni gerekir) |
| `refreshSeconds` | `60` | `dataUrl` kullanılırsa yenileme aralığı (sn) |
| `demo` | `false` | Örnek veri. Sahada kapalı kalmalı |

## Tarayıcıda önizleme

HTML dosyasını tarayıcıda açıp adresin sonuna `?demo=1` ekleyin. Sayfa **ÖRNEK VERİ** etiketiyle örnek veri gösterir; detay satırları ("Örnek Bayi 001" …) uydurmadır. `?demo=1` olmadan Omma dışında açıldığında sayfa yalnızca "Omma bağlantısı bulunamadı" mesajını gösterir.

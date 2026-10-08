# Turkuaz Logiboard – Depo İçi Ekran

ERP'den gelen `Turkuaz_Logiboard` verisini depo içi ekranlarda (Omma / Signalive) gösteren tek dosyalık HTML içerik: [`index.html`](index.html).
Sahadaki personelin 3–10 metreden bakıp **2 saniyede ne yapacağını anlaması** için andon panosu mantığıyla tasarlandı.

![Önizleme](docs/onizleme.png)

## Ekranda ne var?

1. **Öncelik bandı (en üstte, her zaman görünür)**
   - Kırmızı **ÖNCELİK**: en çok geciken işi olan *rota grubu · süreç* ve geciken sayısı. Örnek: "ANADOLU · SEVKİYAT – 1 GECİKEN İŞ". İkinci satırda "Önce bunu bitirin. Sırada: …" listesi ve toplam geciken sayısı yer alır.
   - Yeşil **GECİKEN İŞ YOK**: hiçbir yerde geciken iş yoksa görünür.
   - Sarı **VERİ ESKİ**: veri `staleMinutes` dakikadan eskiyse ya da bağlantı koptuysa bandın sol kutusu sarıya döner. Bu durumda ekrandaki bilgi güncel olmayabilir.
2. **Andon panosu**
   - Her rota grubu için süreçler TOPLAMA → PAKETLEME → SEVKİYAT sırasıyla, GECİKEN / BUGÜN / PLANLI değerleriyle gösterilir.
   - Geciken işi olan kutu **kırmızı** yanar ve beyaz kutu içinde geciken sayısını gösterir. Geciken yoksa yeşil onay işareti görünür.
   - Grup adının solundaki şerit de aynı durumu renkle gösterir.
   - Süreç başlıklarında o süreçteki toplam geciken sayısı rozet olarak görünür, örneğin "24 GECİKEN".
3. **Detay tablosu**
   - Seçili grubun `Details` listesi gösterilir.
   - **Geciken kayıtlar en üstte**, kırmızı zemin üzerinde gösterilir. Kaydın hangi süreçte geciktiği koyu kırmızıyla işaretlenir.
   - Tüm değerleri 0 olan kayıtlar gizlenir.
   - Sayfalar `pageSeconds` aralıkla döner. Bir grubun sayfaları bitince sıradaki gruba geçilir (geciken işi olan gruplar önce gelir).
   - Çok grup olduğunda (yaklaşık 5 ve üzeri) pano ve detay sırayla tam ekran gösterilir. Öncelik bandı bu sırada da görünür kalır.

Ekran 1080p, 4K, ultra geniş ve dikey ekranlarda ölçeklenir. Sayılar asla kesilmez; sığmazsa yazı küçülür. Uzun grup adları iki satıra iner.

> **BUGÜN** sütunu bilerek tarafsız bırakıldı. ERP'deki "Today" değerinin *bugün yapılacak* mı yoksa *bugün yapılan* mı olduğu kesin olmadığı için ekranda "kalan", "tamamlandı" ya da yüzde gibi ifadeler kullanılmaz.

## Omma'ya kurulum

1. Omma panelinde ERP verisinin geldiği **veri kaynağını (datasource)** bu içeriğe bağlayın. Varsayılan ad `Turkuaz_Logiboard`. Farklıysa `index.html` içindeki `CONFIG.datasourceName` değerini değiştirin. İçerikte tek veri kaynağı varsa ad uyuşmasa da o kullanılır.
2. Yeni bir **içerik (content)** oluşturup **Code Editor**'ü açın ve `index.html` dosyasının tamamını yapıştırın. Dosya kendi içinde tamdır; dışarıdan font, kütüphane ya da görsel yüklemez.
3. **Test** ile kontrol edip ekranlara yayınlayın.

Playlist'teki "HTML ekle" widget'ını ya da Web sitesi/URL öğesini kullanmayın. Omma'nın `window.omma` yardımcısı yalnızca Code Editor içeriğinde garanti edilir.

Sayfa Omma'nın [content helper](https://github.com/signalive/content-api-docs/blob/master/content-helper.md) API'sini kullanır: `omma.setVersion('v1')`, `omma.ready()` ve `omma.datasource.get()`. Veri güncellendiğinde (`ds.on('update')`) ekran sayfa yenilenmeden güncellenir ve gösterdiği grup ile sayfada kalır.

**Güvenlik:** Sayfa Omma yardımcısını bulamazsa **örnek veri göstermez**. 15 saniye bekler, ardından ekrana açıklayıcı bir mesaj yazar ve aramaya devam eder. Veri kaynağı bulunamazsa 10 saniyede bir yeniden dener. Sahada hiçbir koşulda uydurma rakam görünmez.

## Beklenen veri yapısı

Veri doğrudan dizi olarak ya da `Turkuaz_Logiboard` anahtarı altında gelebilir (JSON metni olarak gelmesi de sorun değil):

```json
{
  "Turkuaz_Logiboard": [
    {
      "DispatchRouteGroupTitle": "ANADOLU",
      "TotalPickingPast": 0,  "TotalPickingToday": 755,  "TotalPickingPlanned": 2518,
      "TotalPackingPast": 0,  "TotalPackingToday": 1032, "TotalPackingPlanned": 649,
      "TotalShippingPast": 1, "TotalShippingToday": 784, "TotalShippingPlanned": 840,
      "Details": [ { "...": "..." } ]
    }
  ]
}
```

### `Details` alanları

Detay tablosunun sütunları **veriden otomatik çıkarılır**. ERP'de alan eklenip çıkarılınca kodu değiştirmek gerekmez:

- Metin alanları (müşteri, rota, sipariş no vb.) solda gösterilir.
- `PickingPast`, `PackingToday`, `ShippingPlanned` gibi adlandırılmış alanlar (başında `Total` olsun ya da olmasın) **Toplama / Paketleme / Sevkiyat** başlıkları altında gruplanır. Geciken kayıtları bulmak ve sıralamak için bu alanlar kullanılır.
- Diğer sayısal alanlar en sağda gösterilir.
- Sütun başlıklarını Türkçeleştirmek için `index.html` içindeki `LABELS` nesnesine alan adı ekleyin, örneğin `CustomerName: 'Müşteri'`.
- Gösterilmesini istemediğiniz alanları `CONFIG.hiddenDetailColumns` içine yazın, örneğin `['Id']`.

## Ayarlar (`index.html` → `CONFIG`)

| Ayar | Varsayılan | Açıklama |
|---|---|---|
| `title` | `Turkuaz Logiboard` | Üst başlık |
| `datasourceName` | `Turkuaz_Logiboard` | Omma veri kaynağı adı |
| `rootKey` | `Turkuaz_Logiboard` | Veri nesne içinde geliyorsa listenin anahtarı |
| `groupTitleKey` | `DispatchRouteGroupTitle` | Grup başlığı alanı |
| `detailsKey` | `Details` | Alt detay listesi alanı |
| `pageSeconds` | `10` | Detay sayfasının ekranda kalma süresi (sn) |
| `boardSeconds` | `20` | Pano ve detay sırayla gösterilirken panonun süresi (sn) |
| `detailPagesPerTurn` | `2` | Sırayla gösterimde panoya dönmeden önce gösterilecek detay sayfası sayısı |
| `minDetailRows` | `4` | Panonun altına bundan az detay satırı sığıyorsa pano ve detay sırayla gösterilir |
| `hideZeroDetailRows` | `true` | Tüm değerleri 0 olan detay kayıtlarını gizle |
| `sortDetailRows` | `true` | Geciken kayıtları en üste al |
| `detailShowPlanned` | `true` | Detayda PLANLI sütunlarını göster |
| `detailMaxPages` | `0` | >0 ise grup başına en fazla bu kadar detay sayfası (geciken kayıtlar her zaman gösterilir) |
| `showProcessOverdue` | `true` | Süreç başlığında "N GECİKEN" rozeti |
| `pastUnit` | `iş` | Geciken sayısının birimi ("1 geciken iş") |
| `blinkPriority` | `true` | Geciken iş varsa öncelik kutusu yavaşça yanıp söner |
| `staleMinutes` | `30` | Veri bu süreden eskiyse "VERİ ESKİ" uyarısı (0 = kapalı) |
| `hiddenDetailColumns` | `[]` | Detayda gizlenecek alanlar |
| `dataUrl` | `''` | Omma veri kaynağı yerine ERP'nin JSON adresinden doğrudan çekmek için (opsiyonel, CORS izni gerekir) |
| `refreshSeconds` | `60` | `dataUrl` kullanılırsa yenileme aralığı (sn) |
| `demo` | `false` | Örnek veri. Sahada kapalı kalmalı |

## Tarayıcıda önizleme

`index.html` dosyasını tarayıcıda açıp adresin sonuna `?demo=1` ekleyin. Sayfa **DEMO VERİ** etiketiyle örnek veri gösterir. Toplamlar gerçek ekran görüntüsündeki değerlerdir; detay satırları ("Örnek Rota 001" …) uydurmadır. `?demo=1` olmadan Omma dışında açıldığında sayfa yalnızca "Omma bağlantısı bulunamadı" mesajını gösterir.

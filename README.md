# Tedarik ve Üretim Süresi Tahmini — Random Forest ile Makine Öğrenmesi

Bir işletmenin tedarik zincirinde, bir malzeme veya nihai ürün sipariş edildiğinde ne kadar
sürede teslim edilebileceğini tahmin eden bir makine öğrenmesi modeli. Baltacı (2023)'ün
"Yapay Sinir Ağları ile Teslim Süresi Tahmini ve Savunma Sanayinde Uygulaması" başlıklı
yüksek lisans tezindeki problem tanımından esinlenilmiştir.

## Proje Özeti

Gerçek şirket verisi bulunmadığı için, tezdeki yapıya sadık kalınarak kontrollü varsayımlarla
**sentetik bir veri seti** üretilmiş ve bu veri üzerinde **Random Forest** algoritmasıyla bir
tahmin modeli kurulmuştur. Modelin çıktısı, tek bir malzeme değil, **tüm ürün ağacını (BOM)**
gezerek nihai ürünün gerçek teslim süresini — belirsizlik payıyla birlikte — hesaplayan bir
karar destek sistemidir.

## Veri Seti

- **100 farklı nihai ürün**, her biri kendine özgü bir Bill of Materials (BOM) ağacına sahip
- Her ürün ortalama **~74 alt malzemeden** oluşuyor, **5 seviyeye kadar** dallanıyor
- Malzemeler 4 kategoride: `purchased` (satın alınan), `subcontracted` (alt yüklenici),
  `in_house` (kendi üretimimiz), `final_product` (nihai ürün)
- Toplam **7.589 sipariş kaydı** (IQR yöntemiyle aykırı değer temizliği sonrası 7.402)
- Her sipariş, `parent_order_id` ile hangi üst siparişten doğduğunu taşıyor — tam izlenebilirlik

## Yöntem

1. **Veri temizliği** — kategori bazında IQR yöntemiyle aykırı değer tespiti (187 sipariş, %2,5 çıkarıldı)
2. **Özellik seçimi** — veri sızıntısını önlemek için, yalnızca sipariş anında gerçekten bilinen
   7 özellik kullanıldı (`material_id`, `material_type`, `product_tree_level`, `supplier_type`,
   `order_quantity`, `order_month`, `requested_lead_time_days`)
3. **Model eğitimi** — 300 ağaçlı Random Forest, GridSearchCV ile hiperparametre optimizasyonu
4. **Doğrulama** — ablasyon testi (5-fold cross-validation) ile her özelliğin gerçek katkısı
   ayrı ayrı ölçüldü
5. **BOM kritik yol analizi** — nihai ürünün gerçek teslim süresi, altındaki tüm ağaç
   gezilerek (bir montaj, altındaki en yavaş dal bitmeden başlayamaz mantığıyla) hesaplanıyor
6. **Güven aralığı** — 300 ağacın her biri ayrı bir senaryo olarak değerlendirilip, ortaya
   çıkan dağılımdan %90 güven aralığı türetiliyor

## Sonuçlar

| Metrik | Değer | Açıklama |
|---|---|---|
| MAE | 11,16 gün | Ortalama mutlak hata |
| RMSE | 15,10 gün | Büyük hatalara duyarlı hata ölçüsü |
| MAPE | %23,6 | Oransal (yüzdesel) hata |
| R² | 0,761 | Açıklanan varyans oranı |

**Örnek karar destek çıktısı:** Bir nihai ürün için, tek bir kesin sayı yerine
*"%90 ihtimalle 526–580 gün arasında tamamlanır"* şeklinde risk temelli bir aralık üretiliyor.

## Nasıl Çalıştırılır

```bash
pip install pandas numpy scikit-learn matplotlib openpyxl ipywidgets
jupyter notebook randomforest.ipynb
```

Notebook'u üstten alta sırayla (Restart Kernel + Run All) çalıştırmanız yeterlidir.

## Kullanılan Kütüphaneler

`pandas` · `numpy` · `scikit-learn` (RandomForestRegressor, GridSearchCV) · `matplotlib` ·
`openpyxl` · `ipywidgets`

## Notlar ve Sınırlamalar

- Veri seti tamamen sentetiktir; gerçek şirket verisi geldiğinde model yeniden eğitilmelidir
- Train/test ayrımı zaman bazlı değil rastgele yapılmıştır (sentetik veride gerçek tarih
  bilgisi bulunmadığından)
- Model, mevcut haliyle bir prototip/kanıt niteliğindedir; üretim ortamına geçiş için gerçek
  veriyle yeniden doğrulama gereklidir
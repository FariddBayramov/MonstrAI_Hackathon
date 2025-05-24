# MostrAI Hackathon

## Takım Adı: Orion

#### Yunus Ege Küçük | Farid Bayramov | Kuzey Sayın | Ertuğrul Tanrıcı

---

## Dosyalar

### Notebook dosyaları

Projede yer alan Jupyter Notebook dosyalarını gerekli CSV dosyalarını oluşturmak için sırasıyla çalıştırınız:

1. `99_kisi_id.ipynb`
2. `restoran_preprocessing.ipynb`
3. `restoran_cluster.ipynb`
4. `person1-dateplace.ipynb`
5. `person2-favoriler-timecluster.ipynb`
6. `person3-rich_score.ipynb`
7. `person4-persona.ipynb`

### CSV dosyaları

&#x20;Manuel olarak Excel üzerinde veya Jupyter Notebook üzerinde düzenlenmiş dosyalar:

1. `Clustered_Veteriner.csv`

   Excel'de Mahalle ismine göre A B C olacak şekilde sınıflandırma yapıldı.
2. `Kahve_New.csv`

   Excel'de `latitude` ve `longitude` sütunlarının veri tipleri float olacak şekilde düzenlendi. Gereksiz sütunlar kaldırıldı.

# Notebook'lar

---

## 1. 99\_kisi\_id.ipynb

Bu not defteri, büyük hareket veri setindeki (MobilityDataMay2024.parquet) ilk 99 adet benzersiz `device_aid` değerini seçerek yeni bir DataFrame oluşturur.

99 adet id seçilmesinin sebebi DataFrame'i okuyunca RAM belleğin dolmasıdır.

1. **Dask ve Pandas Yükleme**: `dask.dataframe` ve `pandas` içe aktarılır.
2. **Veri Okuma ve Filtreleme**:

   * Parquet dosyası Dask ile okunur.
   * `device_aid` sütunundan benzersiz ID’ler alınır.
   * İlk 99 ID seçilerek hareket verisi bu ID’ler için filtrelenir.

---

## 2. restoran\_preprocessing.ipynb

Bu not defteri, ham restoran verilerini okuyup temizleyerek analiz ve kümeleme için hazırlık yapar.

1. **Kütüphaneleri Yükleme**: `pandas` ve `numpy` paketleri içe aktarılır.
2. **Veri Okuma**: Excel formatındaki ana veri seti (`Hackathon_MainData.xlsx`) yüklenir.
3. **Sütun Seçimi**: Analiz için gerekli sütunlar seçilerek yeni bir DataFrame (`df`) oluşturulur:

   * Enlem, Boylam, İlçe, Tür, Modern/Traditional/Hotel değerleri
   * Ortalama Harcama Tutarı, Restoran Çeşidi
   * Map Profile & Population Score, Mapin Segment.
4. **Hatalı Değer Düzeltme**:

   * İlçe sütunundaki büyük harf ve Türkçe karakter eksik yazımlar düzeltilir.
   * Satış kanalı isimlerindeki yazım hatası (`TRADİTİONAL`) giderilir.
   * Restoran çeşidi 'Balik' → 'Balık' olarak güncellenir.
5. **Kodlu Değerlerin Temizlenmesi**:

   * Modern (`D`), Traditional (`R`) ve Hotel (`H`) değerlerinden harfler çıkarılır.
6. **Skor ve Koordinatların Formatlanması**:

   * Enlem, Boylam, Map Profile/Population skorları stringten float formata çevrilir.
   * Beş haneli sayılar binlik ölçeğe getirilecek şekilde 1000'e bölünür.
7. **Ortalama Harcama Dönüşümü**:

   * "10-20 TL" aralık değerleri ortalaması, "+30" gibi değerler alt sınır olarak sayıya çevrilir.
8. **Eksik Değerlerin Doldurulması**:

   * Modern, Traditional, Hotel sütunlarındaki `NaN` değerler 0 ile doldurulur.
9. **Son Kontrol ve Kayıt**:

   * Temizlenmiş DataFrame ekrana yazdırılır ve `MainData_updated.csv` dosyasına kaydedilir.
10. **Sütun Listesi**:

* Oluşan sütunlar konsolda listelenir.

---

## 3. restoran\_cluster.ipynb

Bu not defteri, birinci adımda temizlenen veriler üzerinde KMeans kümeleme analizi uygular ve restoranları harita üzerindeki kümelere ayırır.

1. **Kütüphaneleri ve Veriyi Yükleme**: `pandas`, `sklearn.cluster.KMeans`, `matplotlib` ve `seaborn` içe aktarılır; `MainData_updated.csv` okunur.
2. **Eksik Ortalama Harcama Doldurma**:

   * Boş `Ortalama Harcama Tutarı` değerleri sütunun ortalama değeriyle doldurulur.
3. **NaN Satır Silme**:

   * `Map Profile Score` veya `Map Population Score` eksik olan satırlar kaldırılır.
4. **One-Hot Encoding**:

   * İlçe sütunu, `pd.get_dummies` ile ikili sütunlara dönüştürülür.
5. **Gereksiz Sütun Kaldırma**:

   * `Mapin Segment` ve `Hotel Değeri` sütunları silinir.
6. **Restoran Çeşidi Birleştirme ve Temizleme**:

   * Az örneklem içeren kategoriler birleştirilir veya silinir.
7. **Satış Kanalı Filtresi**:

   * `Tür` sütununda `Hotel` değerleri çıkarılır.
8. **0 Skorlu Kayıtların Kaldırılması**:

   * `Map Profile Score == 0` olan satırlar ayıklanır.
9. **Öznitelik Seçimi ve Normalizasyon**:

   * `Ortalama Harcama Tutarı` ve `Map Profile Score` seçilir, `StandardScaler` ile ölçeklendirilir.
10. **KMeans Uygulaması**:

    * `n_clusters=3` ile kümeler hesaplanır.
11. **Görselleştirme**:

    * Enlem-Boylam scatter plot ile kümeler renklendirilir.
12. **Elbow Yöntemi Analizi**:

    * 1–10 arası k değerlerinde inertia hesaplanır.
13. **Küme Etiketleme ve Kayıt**:

    * Kümeler "Zengin Restoran", "Orta Halli Restoran", "Ucuz Restoran" olarak isimlendirilir ve `Clustered_Restaurants.csv` kaydedilir.

---

## 4. person1-dateplace.ipynb

Bu not defteri, seçilmiş 99 cihazın hareket verilerini farklı POI türleriyle eşleştirir.

1. **Kütüphaneler**: `dask.dataframe`, `pandas`, `numpy`, `sklearn.neighbors.BallTree`.
2. **99 ID ile Filtreleme**:

   * `99_kisi_data.csv` kullanılarak hareket verisi yüklenir.
3. **Zamanlama ve Konum İşleme**:

   * `timestamp` sütunu datetime’a çevrilir, `zaman_bolumu` eklenir.
   * `horizontal_accuracy` ≤ 200 filtresi uygulanır.
4. **Birincil Kayıtlar**:n

   * Her `device_aid`, `grid_id`, `date`, `zaman_bolumu` kombinasyonu için ilk kayıt seçilir.
5. **Restoran/Veteriner/Kahveci Eşleştirme**:

   * `Clustered_Restaurants.csv`, `Clustered_Veteriner.csv`, `Kahve_New.csv` ile BallTree/Haversine kullanılarak en yakın POI eşleştirmesi yapılır.
6. **POI Türü Belirleme**:

   * `matched_*` sütunları işlenip, en yakın kategori seçilerek `Place` sütunu oluşturulur.
7. **Sonuç**:

   * `id-date-zaman_bolumu-place.csv` dosyası kaydedilir.

---

## 5. person2-favoriler-timecluster.ipynb

Bu not defteri, her cihaz için ziyaret edilen mekan kategorilerini sınıflandırır, favori mekanları belirler ve zaman kümesi analizi yapar.

1. **Veri Yükleme**: `id-date-zaman_bolumu-place.csv` okunur.
2. **Kategori Sözlüğü**:

   * Anahtar kelimelere göre Hastane, Park, Market, Restoran vb. gruplar tanımlanır.
3. **categorize Fonksiyonu**:

   * Mekan adı üzerinden kategori ataması yapılır.
4. **Pivot Tablolar**:

   * Zaman dilimlerine göre (`zaman_pivot`) ve mekan kategorilerine göre (`mekan_pivot`) ziyaret sayıları hesaplanır.
5. **Zaman Kümeleme**:

   * Elbow yöntemi ile optimal k bulunur, KMeans ile `time_cluster` etiketlenir.
6. **Favori Mekanlar**:

   * Ziyaret oranına göre sık ziyaret edilen kategoriler `favoriler` listesinde saklanır.
7. **CSV**:

   * `id-favoriler-time_cluster.csv` kaydedilir.

---

## 6. person3-rich\_score.ipynb

Bu not defteri, cihazların oturduğu mahalle, ziyaret sayıları ve demografi bilgilerine göre bir "zenginlik skoru" (`rich_score`) hesaplar.

1. **Veri ve Demografi**:

   * `id-favoriler-time_cluster.csv` ve `Ilce_Demografi.xlsx` yüklenir.
2. **Gece Mahalle Analizi**:

   * "gece" diliminde en çok bulunulan mahalle `oturdugu_mahalle` olarak seçilir.
3. **Ziyaret Öznitelikleri**:

   * Restoran, lüks villa ve site ziyaret sayıları pivot ile hesaplanır.
4. **Mahalle Seviyesi Eşleştirme**:

   * Excel’den okunan mahalle seviyeleri (`A`, `B`, `C`) her cihaza eklenir.
5. **Öznitelik Ölçeklendirme**:

   * `StandardScaler` ve `MinMaxScaler` ile sayısal öznitelikler normalize edilir.
6. **Rich Score Hesaplama**:

   * Ağırlıklı katkı faktörleri (örn. villa ×0.35, restoran ×0.25, mahalle ×0.25) ile skorlar toplanır.
7. **CSV**:

   * `id-mahalle-rich_score.csv` oluşturulur.

---

## 7. person4-persona.ipynb

Bu not defteri, favoriler, zaman kümeleri ve zenginlik skorunu birleştirerek final persona veri setini oluşturur.

1. **Veri Yükleme**: `id-favoriler-time_cluster.csv` ve `id-mahalle-rich_score.csv` okunur.
2. **Birleştirme**: `pd.concat` ile eksenler birleştirilir.
3. **Son CSV**: `final.csv` dosyası kaydedilerek süreç tamamlanır.

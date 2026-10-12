# BTÖ-303 WEB TABANLI PROGRAMLAMA DERSİ
# ARA UYGULAMA GELİŞTİRME VE FAALİYET RAPORU

**Proje Başlığı:** BiblioTech - Kütüphane & E-Kitap Yönetim Sistemi (Konsept Tabanlı Laravel CRUD Projesi)  
**Tarih:** 12 Ekim 2026  
**Ders:** BTÖ-303 Web Tabanlı Programlama  
**Öğrenci / Geliştirici:** Gazi Üniversitesi Bilgisayar ve Öğretim Teknolojileri Eğitimi (BTÖ)  
**Mimari:** Laravel 11/12 MVC (Model - View - Controller), Eloquent ORM  

---

## 1. GİRİŞ VE PROJE TANIMI

Bu proje, Gazi Üniversitesi BTÖ-303 Web Tabanlı Programlama dersi Ara Uygulama Yönergesi kapsamında belirlenen kriterlere göre geliştirilmiştir. Projenin ana konsepti olarak **"Kütüphane & E-Kitap Yönetim Sistemi (BiblioTech)"** seçilmiştir.

Sistem, bir kütüphanenin ve dijital e-kitap arşivinin tüm kitap envanterini, yazar kadrosunu, edebi tür kategorilerini, okur ödünç alma & iade sirkülasyonunu ve okur değerlendirme/puanlama süreçlerini tam bir **CRUD (Oluşturma, Okuma, Güncelleme, Silme)** yaşam döngüsüyle yönetmeyi hedefler.

---

## 2. YAZILIM MİMARİSİ VE TEKNİK ALTYAPI

### 2.1. MVC (Model - View - Controller) Deseni
- **Model:** Uygulamanın veri yapısını, iş kurallarını ve tablolar arası ilişkilerini Eloquent ORM ile temsil eder (`Book`, `Author`, `Category`, `Borrowing`, `Review`).
- **View:** Blade şablon motoru kullanılarak hazırlanmış, bileşen tabanlı, kullanıcı dostu ve erişilebilir arayüz katmanıdır.
- **Controller:** Kullanıcıdan gelen HTTP isteklerini yönlendiren, form validasyonlarını işleten ve veritabanı işlemlerini koordine eden Resource Controller sınıflarıdır (`BookController`, `AuthorController`, `CategoryController`, `BorrowingController`, `ReviewController`, `RouteEvaluationController`, `DashboardController`).

### 2.2. Arayüz Tasarımı (Custom Pure Vanilla CSS)
- Harici ve ağır CSS kütüphaneleri (Bootstrap veya Tailwind) kullanılmamış; tamamen projeye özel **CSS Custom Properties (Tasarım Değişkenleri)** ile yapılandırılmış bir stil mimarisi (`public/css/app.css`) inşa edilmiştir.
- **Cam Efekti (Glassmorphism):** Saydam kartlar, bulanık arka plan filtreleri (`backdrop-filter: blur(12px)`).
- **Karanlık / Aydınlık Mod (Dark/Light Mode):** Sayfa yenilense dahi `localStorage` üzerinde kullanıcının temasını hatırlayan dinamik tema motoru (`public/js/app.js`).

---

## 3. VERİTABANI TASARIMI VE İLİŞKİLER (ELOQUENT ORM)

| Tablo Adı | Model Sınıfı | İlişki Tipi | Açıklama |
|---|---|---|---|
| `categories` | `Category` | HasMany `Book` | Bir edebi/akademik tür altında birden fazla kitap yer alır. |
| `authors` | `Author` | HasMany `Book` | Bir yazarın birden fazla eseri bulunabilir. |
| `books` | `Book` | BelongsTo `Category`, BelongsTo `Author`, HasMany `Borrowing`, HasMany `Review` | Ana CRUD tablosudur; kategoriye ve yazara bağlıdır; ödünç hareketlerini ve okur yorumlarını barındırır. |
| `borrowings` | `Borrowing` | BelongsTo `Book` | Kitapların üyelere teslimi ve kütüphaneye iadesini tutan sirkülasyon tablosudur. |
| `reviews` | `Review` | BelongsTo `Book` | Kitaba puan (1-5 yıldız) veren ve değerlendirme yorumu yazan okurların tablosudur. |

---

## 4. CRUD İŞLEMLERİ VE GELİŞMİŞ FONKSİYONLAR

1. **Kitap Kataloğu ve Listeleme (Read / Index):**
   - Tüm kitaplar kart formatında görsel, tür rozeti, yazar bilgisi, raf konumu, müsait stok adedi ve yıldızlı ortalama puan ile sunulur.
   - Çoklu filtreleme: Arama kelimesi (`search`), Kategori filtresi, Yazar filtresi ve Kütüphane durumu (`available`, `borrowed`, `maintenance`) aynı anda çalışır.
   - Dinamik sıralama: Popülerliğe (en çok ödünç alınanlar), sayfa sayısına, basım yılına veya müsait stok miktarına göre sıralama.
   - Sayfalama (Pagination): 8'erli gruplar halinde sayfalama ve filtre parametrelerinin sayfalar arasında URL query string ile korunması.

2. **Kitap Ekleme (Create & Store):**
   - `StoreBookRequest` ile sıkı sunucu taraflı validasyon.
   - Türkçe hata mesajları (`@error`) ve form alanlarının kırmızı vurgulanması.
   - Otomatik SEO uyumlu benzersiz `slug` üretimi.
   - Kapak görseli yükleme (`Storage::disk('public')`) ve yüklenen dosyanın canlı JS önizlemesi.

3. **Kitap Detayı ve İlişkili Veriler (Show):**
   - Kitabın detaylı özeti ve tanıtım metni.
   - Yazar biyografi kartı ve aynı yazarın diğer eserlerine bağlantı.
   - Anlık raf stoku kontrolü yapan ve doğrudan ödünç verme işlemi başlatan form.
   - Kitabın geçmiş ödünç hareketleri tablosu.
   - Okur yorumları listesi ve yeni puan/yorum ekleme formu.

4. **Kitap Düzenleme (Edit & Update):**
   - `UpdateBookRequest` ile doğrulama.
   - Mevcut verilerin formda otomatik seçili gelmesi.
   - Mevcut görselin korunması veya yeni görsel yüklendiğinde eskisinin diskten otomatik silinmesi.

5. **Kitap Silme (Delete / Destroy):**
   - JavaScript tabanlı onay dialogu (`confirm`).
   - Kitap silindiğinde diske yüklenmiş olan kapak görselinin dosya sisteminden temizlenmesi.
   - Veritabanı seviyesinde `cascadeOnDelete` ile bağlı ödünç kayıtları ve yorumların tutarlı silinmesi.

6. **Ödünç Alma & İade Sirkülasyon Masası (Borrowing Management):**
   - Stok denetimi: Kitabın raftaki müsait adedi bittiğinde sistem otomatik olarak formu kilitler.
   - 15 günlük iade süresi hesabı.
   - Tek tıkla iade alma butonu: Kitap teslim alındığında kütüphane rafına otomatik iade edilir ve stok artar.

7. **Okur Değerlendirme & Yorum Sistemi (Review & Rating):**
   - Okurların kitaplara 1-5 yıldız aralığında puan vermesi ve deneyimlerini aktarması.
   - Eloquent üzerinden ortalama puanın (`average_rating`) ve toplam değerlendirme sayısının dinamik hesaplanması.

8. **Laravel Rotalama (Routing) Mimarisi ve Canlı Denetim Sayfası:**
   - Projedeki tüm rotalar RESTful kurallarına göre yapılandırılmıştır.
   - Parametre kısıtlaması (`whereNumber('book')`, `whereNumber('author')`) ile güvenli rota çözümlemesi.
   - Arayüzde `/route-evaluation` sayfasında tüm rotaların, metodlarının ve Controller eşleşmelerinin canlı görsel tablosu sunulmaktadır.
   - Geçersiz adresler için `Route::fallback` ile akıllı hata yakalama.

---

## 5. TEST VE DOĞRULAMA (PHPUNIT)

Projenin güvenilirliği ve hatasız çalışması **PHPUnit** testiyle tescillenmiştir:

```
Test Dosyası: tests/Feature/BookCrudTest.php & ExampleTest.php
Çıktı:
- test_dashboard_loads_successfully ................... PASSED
- test_books_index_page_displays_books ................ PASSED
- test_book_create_page_loads ......................... PASSED
- test_can_store_new_book ............................. PASSED
- test_book_detail_page_loads ......................... PASSED
- test_book_can_be_updated ............................. PASSED
- test_book_can_be_deleted ............................. PASSED
- test_member_can_borrow_book_and_decrement_stock ...... PASSED
- test_borrowed_book_can_be_returned_and_restock ....... PASSED
- test_reader_can_review_and_rate_book ................. PASSED
- test_route_evaluation_page_loads_successfully ....... PASSED
- test_invalid_route_fallback_redirects_to_dashboard ... PASSED

Özet: 14 Test, 32 İddia (Assertions), %100 Başarılı (1.45 sn)
```

---

## 6. SONUÇ VE DEĞERLENDİRME

BiblioTech Kütüphane & E-Kitap Yönetim Sistemi projesi; BTÖ-303 Web Tabanlı Programlama dersi kazanımlarını (MVC mimarisi, RESTful yönlendirme, Model-View-Controller etkileşimi, Eloquent ORM ilişkileri, Form Doğrulamaları, Dosya İşlemleri, Güvenlik prensipleri ve Rota Değerlendirmesi) eksiksiz olarak karşılayan, modern ve yüksek performanslı bir çalışma olarak başarıyla tamamlanmıştır.

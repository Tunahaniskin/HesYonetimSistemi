# HES Yönetim Sistemi

Hidroelektrik santrallerindeki izleme, üretim tahmini ve arıza yönetimi süreçlerini tek bir platformda toplayan, rol tabanlı yetkilendirmeye sahip bir backend sistemi.

9 kişilik bir ekiple, çevik (Agile) yöntemle geliştirildi. Orijinal depo: [yunuserme/HesYonetimSistemi](https://github.com/yunuserme/HesYonetimSistemi)

## Ne Sağlıyor

- **Canlı izleme:** Türbin ve sensör verileri (RPM, sıcaklık, su seviyesi, güç) SCADA API'si üzerinden anlık olarak sunulur.
- **Üretim tahmini:** RandomForest modeli, sensör verilerinden türbin bazında güç üretimi (MW) tahmin eder ve eşik dışı değerleri anomali olarak işaretler.
- **Arıza ve iş emri takibi:** Alarmlar kayda geçer, iş emirleri `PENDING → ACCEPTED → IN_PROGRESS → COMPLETED` akışıyla takip edilir. Her durum değişikliği loglanır.
- **Rol bazlı erişim:** Operatör, mühendis, teknisyen, yönetici ve admin rolleri yalnızca kendi yetkisindeki işlemleri yapabilir.

## Teknolojiler

| Katman | Teknoloji |
|---|---|
| API | Python, FastAPI, Pydantic |
| Veritabanı | PostgreSQL 16, SQLAlchemy (async), Alembic |
| Güvenlik | JWT (access + refresh token), bcrypt, RBAC, rate limiting (slowapi) |
| Makine öğrenmesi | scikit-learn (RandomForestRegressor), pandas, NumPy |
| Altyapı | Docker Compose |

## Benim Katkım

Ekipte **enerji tahmin modülü ve veritabanı altyapısından** sorumluydum (Elif Kara ile `elif-tunahan-energy-v2` dalında).

**Enerji tahmin API'si (`/energy`)**
- Veri servisindeki deneysel tahmin kodunu, API'den çağrılabilen bağımsız bir servis katmanına (`energy_prediction_service.py`) dönüştürdüm.
- Model sunucu açılırken sentetik veriyle ön-eğitilir. Böylece gerçek veri yüklenmeden de tahmin endpoint'i çalışır, gerçek CSV verisi geldiğinde model yeniden eğitilebilir.
- Frontend'in kullandığı 5 endpoint'i yazdım: tek nokta tahmini, tüm türbinler için tahmin listesi, tahmin geçmişi, sensör geçmişi ve model metrikleri.
- RPM ve sıcaklık eşiklerine göre anomali tespiti ekledim.

**Veritabanı ve kurulum**
- Projenin kurulumunu engelleyen Alembic migration hatalarını çözdüm. Migration'ın `.env` dosyasından okunmasını sağladım ve Docker Compose ortam değişkenlerini düzelttim.
- Pydantic doğrulama hatalarını ve sensör, türbin, alarm şemalarındaki uyumsuzlukları giderdim.
- İş emirlerine teslim tarihi (`due_at`) alanını yeni bir migration ile ekledim ve iş emri durum akışının kurallarını genişlettim.

**SCADA**
- SCADA router'ını ana uygulamaya entegre ettim, canlı veri endpoint'lerindeki hataları düzelttim.

**Sonuç:** Son commit'te 21 dosyada yaklaşık 900 satır değişiklik yaptım. Ekibin yerel ortamda tek komutla kurup çalıştırabildiği bir backend ve frontend'e bağlanan çalışır durumda bir tahmin modülü ortaya çıktı.

## Kurulum

Gereksinimler: Python 3.11+, Docker

```bash
git clone https://github.com/Tunahaniskin/HesYonetimSistemi.git
cd HesYonetimSistemi/backend

# Ortam değişkenleri
cp .env.example .env        # DATABASE_URL, POSTGRES_PASSWORD ve SECRET_KEY değerlerini doldurun

# Veritabanı
docker compose up -d

# Bağımlılıklar ve migration
pip install -r requirements.txt
alembic upgrade head

# Örnek veriler (kullanıcılar, türbinler, sensörler)
python -m app.seed

# Sunucu
uvicorn app.main:app --reload
```

API dokümantasyonu: `http://localhost:8000/docs`

## API Özeti

| Modül | Prefix | İçerik |
|---|---|---|
| Kimlik doğrulama | `/auth` | Kayıt, giriş, token yenileme, çıkış |
| SCADA | `/api` | Canlı veri, geçmiş, türbin ve sensör verileri |
| Enerji tahmini | `/energy` | Tahmin, forecast, geçmiş, model metrikleri |
| Türbinler | `/turbines` | Oluşturma, listeleme, güncelleme |
| Sensörler | `/sensors` | Oluşturma, listeleme, güncelleme |
| Alarmlar | `/alarms` | Oluşturma, listeleme |
| İş emirleri | `/work-orders` | Oluşturma, listeleme, durum güncelleme |

## Yetki Matrisi

| İşlem | Operatör | Mühendis | Teknisyen | Yönetici | Admin |
|---|---|---|---|---|---|
| Türbin durdurma / savak açma | Evet | - | - | - | Evet |
| Üretim tahminlerini görme | - | Evet | - | Evet | Evet |
| İş emirlerini görme | - | - | Evet | - | Evet |
| Rapor dışa aktarma | - | Evet | - | Evet | Evet |

## Ekip

Yazılım Mühendisliği Grup 2026/2028

Semanur İmre, İsmihan Kırmızıoğlan, Elif Kara, Tunahan Işkın, Eren Bezek, Esma Bilen, Achmet Amet, Yasir Kara, Yunus Emre Erkuş

| Alan | Geliştiriciler |
|---|---|
| Kimlik doğrulama ve çekirdek backend | Esma Bilen, Eren Bezek |
| Enerji tahmini ve veritabanı altyapısı | Elif Kara, Tunahan Işkın |
| Veri servisi (sensör simülasyonu) | Semanur İmre, İsmihan Kırmızıoğlan |
| İş emri sistemi | Yunus Emre Erkuş |
| SCADA arayüzü | Achmet Amet |
| Mühendis ve teknisyen arayüzleri | Yasir Kara |

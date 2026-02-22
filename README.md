# 🛡️ LiderAhenk — Yapay Zeka Destekli Uyum ve Güvenlik Yönetimi Yaması

<div align="center">

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-3776AB.svg)](https://www.python.org/)
[![Vue.js 3](https://img.shields.io/badge/Vue.js-3.x-4FC08D.svg)](https://vuejs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.68+-009688.svg)](https://fastapi.tiangolo.com/)

**Mevcut LiderAhenk Merkezi Yönetim sistemine, Makine Öğrenmesi tabanlı USB anomali tespiti
ve kanıta dayalı uyum yönetimi (compliance) özellikleri ekleyen yama paketi.**

</div>

---

## 📋 İçindekiler

- [Sorun Tanımı](#-sorun-tanımı)
- [Çözüm Yaklaşımı](#-çözüm-yaklaşımı)
- [Kime Hitap Eder?](#-kime-hitap-eder)
- [Yama Nasıl Çalışır?](#-yama-nasıl-çalışır)
- [Mimari](#-mimari)
- [Proje Yapısı](#-proje-yapısı)
- [API Referansı](#-api-referansı)
- [Kurulum](#-kurulum)
- [Demo Akışı](#-demo-akışı)
- [Ekran Görüntüleri](#-ekran-görüntüleri)
- [Lisans](#-lisans)

---

## 🔍 Sorun Tanımı

Kurumsal ağlarda yüzlerce, hatta binlerce Pardus istemci yönetilmektedir. Mevcut LiderAhenk sistemi, istemcilere görev gönderme ve uzaktan yönetim konusunda güçlü bir altyapı sunar. Ancak:

- **Güvenlik uyumluluk takibi yoktur.** Kurum genelinde hangi makinelerin SSH, firewall, disk şifreleme gibi güvenlik politikalarına uyduğu tek ekrandan görüntülenemez.
- **USB/donanım anomali tespiti yapılamaz.** Yetkisiz cihazların ağa bağlanması fark edilemez; raporlama elle yapılmak zorundadır.
- **Kanıta dayalı denetim alt yapısı yoktur.** Güvenlik olayları loglanmaz, denetim raporları oluşturulamaz.
- **Uyum skoru ve trend analizi yoktur.** Kurumun genel güvenlik duruşu (security posture) zaman içinde ölçülemez.

Bu eksiklikler, özellikle **kamu kurumları, üniversiteler, bankalar ve kritik altyapı** yöneten kuruluşlar için ciddi denetim ve uyumluluk riskleri oluşturmaktadır.

---

## 💡 Çözüm Yaklaşımı

Bu yama, mevcut LiderAhenk sistemine **dokunmadan**, yanına iki bileşen ekleyerek tüm bu sorunları çözer:

| Sorun | Çözüm |
|---|---|
| Güvenlik uyumu bilinmiyor | **Compliance Dashboard:** Tüm istemcilerin uyum durumu, skorları ve ihlalleri tek panelde |
| USB anomali tespiti yok | **ML Agent:** İstemcide çalışan yapay zeka modeli, anomali olan USB cihazlarını tespit eder |
| Kanıt yok | **Evidence Service:** Her politika kontrolü loglanır, denetim için kanıt oluşturulur |
| Otomasyon yok | **Akıllı Dağıtım:** Tek tıkla tüm istemcilere güvenlik ajanı dağıtılır, sonuçlar canlı akar |
| Trend analizi yok | **Grafikler ve metrikler:** Uyum oranı, drift tespiti, kritik ihlal sayacı |

### Temel Yenilikler

1. **Sıfır Yenileme (Zero-refresh) Canlı Arayüz:** Dashboard sayfası yenilenmeden, politika sonuçları ve ML anomali tespitleri anlık olarak güncellenir.
2. **Makine Öğrenmesi ile Anomali Tespiti:** İstemci üzerindeki USB cihazları Isolation Forest algoritmasıyla analiz edilir; anomali skoru eşik değeri aşarsa `❌ ANOMALOUS`, aşmazsa `✅ SAFE` olarak raporlanır.
3. **Şifreli Veri Transferi:** ML API yanıtları Fernet simetrik şifreleme ile korunur; ağ üzerinde hassas cihaz bilgileri açık metin olarak gezmez.
4. **LiderAhenk XMPP Mimarisi Uyumlu:** Gerçek ortamda Ahenk ajanının XMPP push mekanizması kullanılır; demo ortamında bu akış HTTP tabanlı olarak simüle edilir.

---

## 👥 Kime Hitap Eder?

| Hedef Kitle | Kullanım Senaryosu |
|---|---|
| **Sistem Yöneticileri** | Kurum genelinde tüm Pardus istemcilerin güvenlik uyumunu tek ekrandan izlemek |
| **Bilgi Güvenliği Ekipleri** | USB anomali tespiti, yetkisiz cihaz raporlaması, kanıt toplama |
| **Denetçiler (Auditor)** | Politika uyum raporları, kanıt logları, zaman damgalı denetim kaydı |
| **KVKK / ISO 27001 Sorumluları** | Uyumluluk oranları, ihlal takibi, düzeltme izleme |
| **Kamu BT Yöneticileri** | Pardus dağıtımlarında merkezi güvenlik görünürlüğü |

---

## ⚙️ Yama Nasıl Çalışır?

Bu paket, LiderAhenk'in mevcut altyapısına **yama (patch)** olarak eklenir. Mevcut sistemde hiçbir değişiklik yapmaz.

```
Mevcut LiderAhenk (Tomcat :8080)  ←  Dokunulmaz, olduğu gibi çalışır
         │
         ├── Bu Yama: LiderUI Compliance Sekmesi (:8081)
         │     └── /api → 8080 (mevcut API'ye proxy)
         │     └── /api/compliance → 5000 (yeni Evidence Service'e proxy)
         │
         └── Bu Yama: Evidence Service (:5000)
               └── FastAPI mikroservis
               └── SQLite veritabanı
               └── USB Anomaly ML Agent
```

### Yama Akışı (Adım Adım)

```
1. Yönetici LiderUI'da "Uyum Yönetimi" sekmesini açar
2. Dashboard yüklenir → Evidence Service'ten (/api/compliance/summary) metrikler çekilir
3. Yönetici "Plugin Dağıt" butonuna basar
4. Evidence Service istemcilerin durumunu günceller + ML ajanını tetikler
5. ML Ajanı istemci makinedeki ML API'ye bağlanır (port 8000)
6. USB cihaz verileri şifreli olarak çekilir, Fernet ile çözülür
7. Her cihaz için anomali skoru hesaplanır (Isolation Forest)
8. Sonuçlar Evidence Service'e POST edilir (/api/compliance/report)
9. Dashboard anlık güncellenir — yönetici canlı olarak sonuçları izler
```

---

## 🏗️ Mimari

```
┌───────────────────────────────────────────────────────────────────┐
│                     Pardus İstemci Makineler                      │
│                                                                   │
│  ┌─────────────────────┐    ┌─────────────────────────────────┐  │
│  │  ML API (:8000)     │    │  Ahenk Agent                    │  │
│  │  Isolation Forest   │◄───│  USB cihaz taraması             │  │
│  │  Anomaly Detection  │    │  Fernet şifreli yanıt           │  │
│  └─────────────────────┘    └─────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────┘
                          │ HTTP (şifreli)
                          ▼
┌───────────────────────────────────────────────────────────────────┐
│                        Lider Sunucu                               │
│                                                                   │
│  ┌──────────────────────┐   ┌──────────────────────────────────┐ │
│  │  LiderAhenk API      │   │  Evidence Service (FastAPI)      │ │
│  │  Tomcat :8080         │   │  Port :5000                     │ │
│  │  ─────────────────    │   │  ───────────────────────────    │ │
│  │  • Kullanıcı Auth     │   │  • /api/compliance/summary      │ │
│  │  • XMPP Yönetim       │   │  • /api/compliance/clients      │ │
│  │  • Agent İletişim     │   │  • /api/compliance/deploy       │ │
│  │  [Mevcut — Dokunulmaz]│   │  • /api/compliance/report       │ │
│  └──────────┬───────────┘   │  • /api/compliance/evidence-logs │ │
│             │               │  • Lider MySQL Senkronizasyonu   │ │
│             │               │  [YENİ — Bu Yama]                │ │
│             │               └──────────────┬───────────────────┘ │
│             │                              │                      │
│             ▼                              ▼                      │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │           LiderUI + Compliance Sekmesi (:8081)              │ │
│  │           Vue.js 3 + PrimeVue + Chart.js                    │ │
│  │  ┌──────────────┬──────────────┬──────────────┬──────────┐ │ │
│  │  │ Genel Bakış  │ İstemciler   │ Politikalar  │ Kanıtlar │ │ │
│  │  │ (Grafikler)  │ (Tablo+Skor) │ (Kurallar)   │ (Loglar) │ │ │
│  │  └──────────────┴──────────────┴──────────────┴──────────┘ │ │
│  └─────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────┘
```

---

## 📁 Proje Yapısı

```
liderui_fork/
│
├── liderui/                              # 🖥️ Vue.js Frontend Yaması (Port 8081)
│   ├── src/
│   │   ├── views/Compliance/             # Yeni eklenen sayfa ve sekmeler
│   │   │   ├── ComplianceDashboard.vue   #   Ana dashboard (4 sekmeli TabView)
│   │   │   └── Tabs/
│   │   │       ├── ComplianceOverview.vue       #   Genel bakış (Chart.js grafikleri)
│   │   │       ├── ComplianceClientStatus.vue   #   İstemci durum tablosu
│   │   │       ├── CompliancePolicyResults.vue  #   Politika sonuçları
│   │   │       └── ComplianceEvidenceLog.vue    #   Kanıt logları (canlı akış)
│   │   ├── services/Compliance/
│   │   │   └── ComplianceService.js      #   Evidence Service API çağrıları
│   │   ├── router/index.js               #   Yeni route eklendi
│   │   └── locales/                      #   TR/EN çeviri desteği
│   ├── vue.config.js                     # Proxy: /api/compliance → :5000
│   └── package.json
│
├── evidence-service/                     # 🐍 Python FastAPI Backend (Port 5000)
│   ├── app.py                            # REST API (10+ endpoint)
│   ├── models.py                         # SQLAlchemy modelleri (Client, Policy, EvidenceLog)
│   ├── database.py                       # SQLite bağlantı yönetimi
│   ├── lider_sync.py                     # Lider MySQL → SQLite senkronizasyonu
│   ├── compliance_checker.py             # Politika uyumluluk kontrol motoru
│   ├── usb_anomaly_agent.py              # ML tabanlı USB anomali tespit ajanı
│   ├── session_watcher.py                # İstemci oturum izleme
│   ├── seed_data.py                      # Demo/test verisi oluşturucu
│   ├── simulate_client.py                # İstemci simülasyon aracı
│   └── requirements.txt                  # Python bağımlılıkları
│
├── start_demo.sh                         # 🚀 Tek komutla demo başlatma scripti
├── .gitignore
└── README.md
```

---

## 📡 API Referansı

Evidence Service (Port 5000) tarafından sunulan REST endpointleri:

### Dashboard Endpointleri (Frontend tüketir)

| Method | Endpoint | Açıklama |
|---|---|---|
| `GET` | `/api/compliance/summary` | Genel uyum metrikleri (oran, ihlal sayısı, drift) |
| `GET` | `/api/compliance/clients` | Tüm istemcilerin durumu, skorları ve ihlalleri |
| `GET` | `/api/compliance/policy-results` | Politika bazlı uyum sonuçları |
| `GET` | `/api/compliance/evidence-logs` | Son 100 kanıt kaydı (zaman damgalı) |

### İşlem Endpointleri

| Method | Endpoint | Açıklama |
|---|---|---|
| `POST` | `/api/compliance/deploy` | Güvenlik ajanını istemcilere dağıt + ML taramayı tetikle |
| `POST` | `/api/compliance/report` | İstemciden gelen uyum raporu (ML agent kullanır) |
| `POST` | `/api/compliance/heartbeat` | İstemci online durumu bildirimi |
| `POST` | `/api/compliance/sync` | Lider MySQL'den agent listesini yeniden çek |
| `GET` | `/api/compliance/health` | Servis sağlık kontrolü |

---

## 🚀 Kurulum

### Gereksinimler

| Bileşen | Minimum Versiyon | Açıklama |
|---|---|---|
| LiderAhenk | — | Tomcat üzerinde kurulu ve çalışır durumda (port 8080) |
| Node.js | v14+ | Frontend için |
| Yarn veya npm | — | Paket yöneticisi |
| Python | 3.8+ | Evidence Service için |
| pip | — | Python paket yöneticisi |

### Hızlı Başlangıç (Tek Komut)

```bash
git clone https://github.com/aliozen0/liderui.git
cd liderui
chmod +x start_demo.sh
./start_demo.sh
```

`start_demo.sh` otomatik olarak:
1. ✅ Python sanal ortamını oluşturur (yoksa)
2. ✅ Bağımlılıkları yükler
3. ✅ Demo verilerini seed eder
4. ✅ Evidence Service'i 5000 portunda başlatır
5. ✅ Frontend'i 8081 portunda başlatır

### Manuel Kurulum

#### 1. Evidence Service (Python Backend)

```bash
cd evidence-service

# Sanal ortam oluştur ve aktif et
python3 -m venv venv
source venv/bin/activate

# Bağımlılıkları yükle
pip install -r requirements.txt

# Demo verilerini yükle (ilk kurulumda)
python3 seed_data.py

# Servisi başlat
uvicorn app:app --host 0.0.0.0 --port 5000
```

#### 2. Frontend (Vue.js)

```bash
cd liderui

# Bağımlılıkları yükle
yarn install   # veya: npm install

# Geliştirme sunucusunu başlat
yarn serve     # veya: npm run serve
```

### Erişim

| URL | Açıklama |
|---|---|
| `http://localhost:8081` | LiderUI + Compliance Sekmesi |
| `http://localhost:5000/docs` | Evidence Service Swagger Dokümantasyonu |
| `http://localhost:8080` | Mevcut LiderAhenk (zaten kurulu) |

---

## 🎯 Demo Akışı (Hackathon Sunumu)

### Adım 1: Dashboard'u Gösterin
Tarayıcıda `http://localhost:8081` → Sol menüde **"Uyum Yönetimi"** sekmesini açın.
- Grafiklerin ve metriklerin sayfa yenilemeden render edildiğini gösterin
- İstemci listesinde online/offline durumlarını gösterin

### Adım 2: Plugin Dağıtımı
- İstemci listesinden hedef makineleri seçin
- **"Plugin'i Dağıt"** butonuna basın
- Dağıtım tetiklenir ve istemci durumu "Pending" olarak güncellenir

### Adım 3: Canlı ML Doğrulama
- Dağıtım sonrasında açılan terminal penceresinde canlı logları izleyin
- ML ajanı otomatik olarak istemcilere bağlanır
- USB cihazları taranır, anomali skorları hesaplanır
- Sonuçlar (`❌ ANOMALOUS` / `✅ SAFE`) gerçek zamanlı olarak arayüze yansır

### Adım 4: Kanıt Logları
- **"Kanıt Kayıtları"** sekmesine geçin
- Tüm tarama sonuçlarının zaman damgalı olarak loglandığını gösterin
- Bu loglar denetim raporları için kanıt niteliğindedir

---

## 🔑 Güvenlik Özellikleri

- **Fernet Simetrik Şifreleme:** ML API yanıtları şifreli transfer edilir
- **CORS Koruması:** Evidence Service yapılandırılabilir CORS politikası sunar
- **Lider MySQL Senkronizasyonu:** Agent verileri periyodik olarak (15 sn) senkronize edilir
- **Otonom Çalışma:** Evidence Service bağımsız çalışır; LiderAhenk API çökse bile uyum verileri kaybolmaz

---

## 🛠️ Geliştirme

```bash
# Evidence Service'i geliştirme modunda çalıştır (hot-reload)
cd evidence-service
uvicorn app:app --host 0.0.0.0 --port 5000 --reload

# Frontend'i geliştirme modunda çalıştır (hot-reload)
cd liderui
yarn serve
```

Swagger API dokümantasyonu: `http://localhost:5000/docs`

---

## 📄 Lisans

Bu proje LiderAhenk lisansı altında sunulmaktadır. Detaylar için [LICENSE](liderui/LICENSE) dosyasına bakınız.

---

<div align="center">

**Hackathon 2026 — Pardus LiderAhenk Siber Güvenlik Yaması**

*Mevcut sisteme dokunmadan, yapay zeka ile güvenlik görünürlüğü kazandırın.*

</div>

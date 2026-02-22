# LiderAhenk — Yapay Zeka Destekli Uyum ve Güvenlik Yönetimi Yaması

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

Bu proje, [LiderAhenk](https://liderahenk.org/) Açık Kaynak Merkezi Yönetim sistemine **Yapay Zeka (ML) Destekli Uyum ve Güvenlik Yönetimi** özelliği ekleyen bir **yama (patch)** paketidir.

Mevcut LiderAhenk Tomcat ortamının (port 8080) yanına iki bileşen ekleyerek, sisteme modern bir "Kanıta Dayalı Güvenlik (Evidence-based Security)" katmanı getirir.

---

## 🌟 Ne Ekler?

1. **Güvenlik Gösterge Paneli (Compliance Dashboard):** Sistemlerin sağlık durumunu, uyumluluk skorlarını ve zafiyet grafiklerini anlık gösterir.
2. **Akıllı Politika Dağıtımı:** İstemcilere Güvenlik Ajanı (Plugin) kurulabilir; sonuçlar canlı olarak arayüze yansır.
3. **ML Tabanlı USB Anomali Tespiti:** İstemci üzerindeki Python Ajanı, ML API'sinden anomali verilerini çeker ve sonuçları gerçek zamanlı Lider ekranına basar.
4. **Hafif Python Mikroservis (FastAPI):** Ağır Java monolitinden bağımsız, saniyeler içinde ayağa kalkan Evidence Service.

---

## 🏗️ Mimari

```text
┌──────────────────────────────────────────────────────────────┐
│                    Pardus İstemci Makineler                   │
│  ┌─────────────────────────────┐                             │
│  │  ML API (Port 8000)         │  USB cihaz anomali taraması │
│  │  usb_anomaly_agent.py ←─────┤  sonuçları şifreli döner    │
│  └─────────────────────────────┘                             │
└──────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────┐
│                    Lider Sunucu                               │
│                                                              │
│  ┌──────────────────┐    ┌──────────────────┐                │
│  │ LiderAhenk API   │    │ Evidence Service  │                │
│  │ (Tomcat :8080)   │    │ (FastAPI :5000)   │                │
│  │ [Mevcut Sistem]  │    │ [Bu Yama]         │                │
│  └────────┬─────────┘    └────────┬──────────┘                │
│           │                       │                           │
│           ▼                       ▼                           │
│  ┌─────────────────────────────────────────┐                 │
│  │  LiderUI + Compliance Sekmesi (:8081)   │                 │
│  │  Vue.js Frontend [Bu Yama]              │                 │
│  └─────────────────────────────────────────┘                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 📁 Proje Yapısı

```text
liderui_fork/
├── liderui/                          # Vue.js Frontend (Port 8081)
│   ├── src/
│   │   ├── views/Compliance/         # Yeni: Uyum Gösterge Paneli sayfaları
│   │   └── services/Compliance/      # Yeni: Compliance API servisi
│   ├── vue.config.js                 # Proxy: /api → 8080, /api/compliance → 5000
│   └── package.json
│
├── evidence-service/                 # Python FastAPI Backend (Port 5000)
│   ├── app.py                        # REST API + Dağıtım Tetikleyicisi
│   ├── models.py                     # SQLAlchemy Modelleri
│   ├── database.py                   # Veritabanı bağlantısı
│   ├── seed_data.py                  # Demo verisi oluşturucu
│   ├── usb_anomaly_agent.py          # İstemcide çalışacak ML sorgulama ajanı
│   ├── compliance_checker.py         # Politika uyumluluk kontrolcüsü
│   ├── lider_sync.py                 # Lider MySQL senkronizasyonu
│   ├── session_watcher.py            # Oturum izleme
│   ├── simulate_client.py            # Demo simülasyon istemcisi
│   └── requirements.txt
│
├── start_demo.sh                     # Tek komutla demo başlatma
└── README.md                         # Bu dosya
```

---

## 🚀 Kurulum ve Çalıştırma

### Gereksinimler

- **LiderAhenk** kurulu ve çalışır durumda (Tomcat, port 8080)
- **Node.js** v14+ ve **yarn**
- **Python 3.8+**

### Hızlı Başlangıç (Tek Komut)

```bash
git clone https://github.com/Pardus-LiderAhenk/liderui.git liderui_fork
cd liderui_fork
chmod +x start_demo.sh
./start_demo.sh
```

### Manuel Başlatma

#### 1. Evidence Service (Python Backend — Port 5000)

```bash
cd evidence-service
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# İlk çalıştırmada demo verilerini yükleyin:
python3 seed_data.py

# Servisi başlatın:
uvicorn app:app --host 0.0.0.0 --port 5000
```

#### 2. LiderUI Frontend (Vue.js — Port 8081)

```bash
cd liderui
yarn install
yarn serve
```

### Erişim

Tarayıcınızda **http://localhost:8081** adresine gidin → Sol menüde **"Uyum Yönetimi"** sekmesini açın.

---

## 🎯 Demo Akışı (Hackathon Sunumu)

1. **Dashboard'u açın** — Grafiklerin ve metriklerin sıfır yenileme (zero-refresh) ile render edildiğini gösterin.
2. **Plugin Dağıtımı** — İstemci listesinden hedef makineleri seçip **"Plugin'i Dağıt"** butonuna basın.
3. **Canlı Doğrulama** — Açılan terminal ekranında ML ajanının arka planda çalışarak anomali tespiti yaptığını ve sonuçları (`❌ ANOMALOUS` / `✅ SAFE`) canlı olarak arayüze bastığını izleyin.

---

## 🔧 Teknik Detaylar

| Bileşen | Teknoloji | Port | Açıklama |
|---|---|---|---|
| LiderAhenk API | Java / Tomcat | 8080 | Mevcut sistem (yama kapsamı dışı) |
| Evidence Service | Python / FastAPI | 5000 | Uyum mikroservisi |
| Frontend | Vue.js 3 / PrimeVue | 8081 | Modifiye LiderUI + Compliance sekmesi |
| ML API | Python (İstemcide) | 8000 | USB anomali tespit motoru |

---

## 📄 Lisans

Bu proje LiderAhenk lisansı altında sunulmaktadır. Detaylar için [LICENSE](liderui/LICENSE) dosyasına bakınız.

---

*Hackathon 2026 — Pardus-LiderAhenk*

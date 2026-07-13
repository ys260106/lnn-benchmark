# LNN (CFC) vs LSTM — UAV-SEAD Benchmark Sonuçları

**Kapalı-Form Sürekli-Zamanlı (CFC) sinir ağının LSTM'e karşı drone setpoint tracking performansı.**

[![UAV-SEAD](https://img.shields.io/badge/Veri%20Seti-UAV--SEAD-blue)](https://huggingface.co/datasets/aykutkabaoglu/uav-flight-anomaly-dataset)
[![arXiv](https://img.shields.io/badge/arXiv-2602.13900-red)](https://arxiv.org/abs/2602.13900)

---

## 📊 Sonuçlar (Özet)

| Test | CFC (66 param) | LSTM (105 param) | Fark |
|------|---------------|-----------------|------|
| **Sinyal Takibi** | MSE **0.00110** | MSE 0.00169 | CFC **%35 daha iyi** |
| **%20 Gürültü** | MSE **0.0556** (1.05x degrade) | MSE 0.1258 (1.62x degrade) | CFC **çok daha dayanıklı** |
| **Stabilizasyon** | **0 adım** toparlanma | 8.2 adım | CFC **anında toparlanır** |
| **Verimlilik** | 0.073 (MSE×param) | 0.177 | CFC **2.4 kat daha verimli** |
| **Eğitim Süresi** | **174 saniye** | 2.282 saniye (~38 dk) | CFC **13 kat daha hızlı** |

### ✅ CFC 3/3 testi kazandı.

---

## 🎯 Problem

**Setpoint Tracking:** 9 sensör girdisinden bir sonraki adımdaki roll/pitch/yaw açısını tahmin et.

| # | Girdi | Kaynak | Anlamı |
|---|-------|--------|--------|
| 1-3 | `gyro_rad[0,1,2]` | IMU (gyro) | X/Y/Z eksenleri etrafında dönüş hızı (rad/s) |
| 4-6 | `accelerometer_m_s2[0,1,2]` | IMU (accel) | X/Y/Z yönünde doğrusal ivme + yerçekimi (m/s²) |
| 7-9 | `roll_body/pitch_body/yaw_body` | Setpoint | Drone'un gitmesi gereken hedef açılar (rad) |
| **Çıktı** | roll/pitch/yaw | Attitude (gerçek) | Drone'un 32 ms sonraki gerçek açıları (rad) |

---

## 🧠 CFC Nedir?

CFC (Closed-Form Continuous-Time), sürekli-zamanlı diferansiyel denklemleri kapalı-formda çözen bir nöron modelidir. Her nöron üç parametre grubundan oluşur:

- **drive_w (9 adet):** Girdilerin ağırlıkları
- **w_tau (9 adet):** Girdiye bağlı dinamik zaman sabiti belirleyicisi
- **bias (1 adet):** Nöron eşiği

**w_tau mekanizması:** `τ = tanh(x·w_τ) + 1.1` ile tau dinamik olarak [0.1, 2.1] aralığında ayarlanır. Girdi anormalleştiğinde tau otomatik artar, nöron yavaşlar, bozulma sönümlenir.

**Formül:** `z[t] = z[t-1]·exp(-dt/τ) + (x·w + bias)·τ·(1 - exp(-dt/τ))`

Bu formülde `dt` (zaman adımı) bizzat bulunur. dt büyükse eski bilgi daha çok unutulur, küçükse daha az — fiziksel olarak doğru.

---

## ⚙️ Yöntem

| Parametre | Değer |
|-----------|-------|
| Veri Seti | UAV-SEAD (2018-05-26 Normal uçuş, ~214 sn) |
| Veri Adım Sayısı | 6.755 (4.728 eğitim + 2.027 test) |
| Ortalama dt | ~32 ms (değişken: 29-41 ms) |
| Optimizasyon | Genetik Algoritma (pop=400, nesil=80) |
| CFC Parametre | 66 (3 paralel nöron, w_tau'lu) |
| LSTM Parametre | 105 (hidden=2, 4 gate) |

> GA ayarları her iki model için **tamamen aynıdır.** CFC'ye özel hyperparameter tuning uygulanmamıştır.

---

## 📁 Bu Repo

```
lnn-benchmark/
├── BENCHMARK_RAPORU.md       # Detaylı rapor (markdown)
├── benchmark_raporu.pdf      # PDF rapor
├── SUMMARY.md                # Kısa özet
├── output/
│   ├── test1_sinyal_takibi.png
│   ├── test2_gurultu.png
│   ├── test3_stabilizasyon.png
│   ├── ga_yakinsama.png
│   └── sonuclar.txt
└── README.md
```

> **Not:** Benchmark kodu özeldir, talep üzerine paylaşılır.

---

## 📚 Referanslar

- **Kabaoğlu et al. (2025)** — UAV-SEAD: State Estimation Anomaly Dataset for UAVs — [arXiv:2602.13900](https://arxiv.org/abs/2602.13900)
- **Hasani et al. (2021)** — Closed-Form Continuous-Time Neural Networks — [arXiv:2106.13898](https://arxiv.org/abs/2106.13898)
- **Veri seti:** [huggingface.co/aykutkabaoglu/uav-flight-anomaly-dataset](https://huggingface.co/datasets/aykutkabaoglu/uav-flight-anomaly-dataset)

---

**Hazırlayan:** Yusuf Şahin — İnönü Üniversitesi, Bilgisayar Mühendisliği  
**Tarih:** 2026-07-10

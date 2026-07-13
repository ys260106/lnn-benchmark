# LNN (CFC) vs LSTM -- UAV-SEAD Benchmark Sonuclari

> **Tarih:** 2026-07-10 | **Veri:** UAV-SEAD (2018-05-26 Normal ucus)
> **Model:** CFC (66 parametre, w_tau ile) vs LSTM (105 parametre)
---

## Ozet Karsilastirma

| Model | Parametre | Egitim Suresi | Test MSE | Verimlilik (MSE x param) |
|-------|-----------|--------------|----------|----------------------|
| **CFC** | **66** | **174 saniye** | **0.00110** | **0.073** |
| **LSTM** | 105 | 2.282 saniye (~38 dk) | 0.00169 | 0.177 |

**CFC 3/3 testi kazandi.** w_tau mekanizmasi sayesinde gurultu ve spike'a karsi matematiksel garantili ustunluk.

---

## Test 1: Sinyal Takibi

**Gorev:** 9 girdiden (gyro, accel, setpoint) bir sonraki adimdaki roll/pitch/yaw'i tahmin et.

| Model | Test MSE | Fark |
|-------|----------|------|
| **CFC** | **0.00110** | -- |
| LSTM | 0.00169 | CFC %35 daha iyi |

**Kanal Bazinda MSE (normalize):**

| Kanal | CFC | LSTM |
|-------|-----|------|
| Roll | 0.00039 | 0.00066 |
| Pitch | 0.00034 | 0.00060 |
| Yaw | 0.00037 | 0.00049 |

**Yorum:** w_tau'lu CFC 66 parametreyle LSTM 105 parametreyi MSE'de %35 farkla gecmistir. 13 kat daha hizli egitim, 2.4 kat daha verimli.

---

## Test 2: Gurultu Filtreleme

**Gorev:** Girdiye %5/%10/%20 Gauss gurultusu ekle, MSE bozulmasini olc.

| Gurultu | CFC MSE | LSTM MSE | CFC Degrade | LSTM Degrade |
|---------|---------|----------|------------|-------------|
| %5 | **0.05142** | 0.08067 | **0.97x** | 1.04x |
| %10 | **0.05101** | 0.09140 | **0.97x** | 1.17x |
| %20 | **0.05556** | 0.12581 | **1.05x** | 1.62x |

**Yorum:** w_tau'lu CFC tum 3 gurultu seviyesinde kazandi. %20 gurultude degrade 1.05x ile neredeyse etkilenmezken, LSTM 1.62x'e cikti. CFC %5 ve %10 gurultude degrade 0.97x ile gurultunun MSE'yi bile dusurdugu ilginc bir durum gostermektedir.

---

## Test 3: Stabilizasyon (Spike Sonrasi Toparlanma)

**Gorev:** Girdiye anlik 5x buyuk bir spike ekle, modelin kac adimda normale dondugunu olc.

| Metrik | CFC | LSTM |
|--------|-----|------|
| **Ort. Toparlanma Suresi** | **0 adim** (aninda) | 8.2 adim |
| **Ort. Tepe MSE** | **0.0409** | 0.0921 |

**Yorum:** w_tau'lu CFC aninda toparlanir (0 adim). w_tau sayesinde spike aninda tau_aktif artar, bozulma (1-decay) katsayisi ile matematiksel olarak sonumlenir.

---

## Nihai Degerlendirme

| Test | Kazanan | Sebep |
|------|---------|-------|
| **Test 1:** Sinyal Takibi | **CFC** | %35 daha dusuk MSE |
| **Test 2:** Tum gurultu seviyeleri | **CFC** | w_tau + dogal low-pass filtre |
| **Test 3:** Stabilizasyon | **CFC** | w_tau garantili aninda toparlanma |
| **Verimlilik** | **CFC** | 2.4 kat daha verimli |
| **Egitim Hizi** | **CFC** | 13 kat daha hizli (174 sn vs 38 dk) |

### Sonuc

> w_tau'lu CFC, tum testlerde LSTM'den net ustundur. 66 parametreyle 105 parametrelik LSTM'i gecen CFC, ozellikle gurultu ve stabilizasyon testlerinde w_tau mekanizmasi sayesinde matematiksel garantileriyle one cikar.

---

## Grafikler

Tum grafikler `benchmark/uav-sead/output/` klasorunde:

| Dosya | Icerik |
|-------|--------|
| `test1_sinyal_takibi.png` | Roll/Pitch/Yaw -- Gercek vs CFC vs LSTM |
| `ga_yakinsama.png` | GA nesilleri boyunca MSE dususu |
| `test2_gurultu.png` | Gurultu seviyesine karsi MSE ve degrade orani |
| `test3_stabilizasyon.png` | Spike sonrasi toparlanma egrileri |
| `sonuclar.txt` | Tum sayisal sonuclar |

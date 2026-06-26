<div align="center">

# 🫆 Fingerprint to Google Sheets

### Sistem Absensi Pintar Berbasis Sidik Jari

![ESP8266](https://img.shields.io/badge/ESP8266-NodeMCU-blue?style=for-the-badge&logo=arduino)
![Platform](https://img.shields.io/badge/Platform-Arduino-teal?style=for-the-badge&logo=arduino)
![Cloud](https://img.shields.io/badge/Cloud-Google%20Sheets-green?style=for-the-badge&logo=googlesheets)
![License](https://img.shields.io/badge/License-MIT-purple?style=for-the-badge)

*Absen cukup dengan sidik jari. Otomatis tercatat ke Google Sheets.*


</div>

---

## ✨ Fitur

- 🫆 **Pemindaian sidik jari** — sensor optik R307, mampu menyimpan hingga 127 sidik jari
- 📊 **Pencatatan ke Google Sheets** — absensi real-time lewat webhook Apps Script
- 🌐 **Dashboard web lokal** — daftarkan, hapus, dan lihat log dari browser apa saja
- 🖥️ **Layar OLED** — status live, animasi, dan feedback hasil scan
- 🔔 **Feedback buzzer** — nada berbeda untuk berhasil, ditolak, dan mode daftar
- 👻 **Anti ghost-read** — validasi triple-read + filter tingkat kepercayaan
- 📡 **Terhubung WiFi** — ESP8266 menjalankan web server sendiri di port 80

---

## 🧰 Komponen yang Dibutuhkan

| # | Komponen |
|---|-----------|
| 1 | ESP8266 NodeMCU v1.0 |
| 2 | OLED SSD1306 128×64 I2C (0.96") |
| 3 | Sensor Sidik Jari R307 / R503 |
| 4 | Buzzer Pasif |
| 5 | Breadboard |
| 6 | Kabel jumper |
| 7 | Kabel USB (Micro USB) |

---

## ⚡ Pengkabelan (Wiring)

### OLED SSD1306 → NodeMCU

| Pin OLED | Pin NodeMCU | Catatan |
|----------|-------------|-------------|
| VCC | 3.3V | Hanya 3.3V — jangan 5V |
| GND | GND | |
| SCL | D1 | I2C clock |
| SDA | D2 | I2C data |

> Alamat I2C: `0x3C` · Diinisialisasi dengan `Wire.begin(D2, D1)`

---

### Sensor Sidik Jari R307 → NodeMCU

| Pin Sensor | Pin NodeMCU | Kabel | Catatan |
|------------|-------------|------|-------|
| VCC | VIN | 🔴 Merah | Harus 5V — pakai VIN, bukan 3.3V |
| GND | GND | ⚫ Hitam | |
| TX | D5 | 🟡 Kuning | TX sensor → MCU membaca |
| RX | D6 | 🟢 Hijau | RX sensor → MCU mengirim |

> ⚠️ Sambungan silang: TX Sensor → D5 dan RX Sensor → D6

---

### Buzzer Pasif → NodeMCU

| Pin Buzzer | Pin NodeMCU | Catatan |
|------------|-------------|-------------|
| + | D3 | Sinyal PWM lewat `tone()` |
| − | GND | |

> ⚠️ Harus buzzer **pasif** — buzzer aktif tidak akan bekerja dengan `tone()`

---

### Ringkasan Semua Pengkabelan

| NodeMCU | Komponen |
|---------|-----------|
| 3.3V | OLED VCC |
| GND | OLED GND |
| D1 | OLED SCL |
| D2 | OLED SDA |
| D3 | Buzzer + |
| GND | Buzzer − |
| VIN | Sidik Jari VCC |
| GND | Sidik Jari GND |
| D5 | Sidik Jari TX |
| D6 | Sidik Jari RX |

---

### Skema

```
                    ┌─────────────────────┐
                    │   NodeMCU ESP8266   │
                    │                     │
  ┌──────────┐      │  3.3V ──────────────┼──→ OLED VCC
  │SSD1306   │      │  GND  ──────────────┼──→ OLED GND
  │OLED      │◄─────┤  D1   ──────────────┼──→ OLED SCL
  │128×64    │      │  D2   ──────────────┼──→ OLED SDA
  └──────────┘      │                     │
                    │  D3   ──────────────┼──→ Buzzer +
  ┌──────────┐      │  GND  ──────────────┼──→ Buzzer −
  │Buzzer    │◄─────┤                     │
  │Pasif     │      │  VIN  ──────────────┼──→ FP VCC (5V)
  └──────────┘      │  GND  ──────────────┼──→ FP GND
                    │  D5   ◄─────────────┼──  FP TX
  ┌──────────┐      │  D6   ──────────────┼──→ FP RX
  │R307      │◄─────┤                     │
  │Sidik Jari│      └─────────────────────┘
  └──────────┘                 │
                             Kabel USB
                          (daya + upload)
```

### Catatan Daya

```
USB 5V lewat kabel
  └── Regulator bawaan NodeMCU
        ├── 3.3V  ──→  OLED
        └── VIN   ──→  Sensor sidik jari (butuh 5V penuh)
```

---

## 🚀 Cara Setup

### Langkah 1 — Setup Board di Arduino IDE

Tambahkan URL ini di **File → Preferences → Additional boards manager URLs:**

```
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

Lalu buka **Tools → Board → Boards Manager**, cari `ESP8266` dan install.

Pengaturan board di **Tools:**

```
Board        : NodeMCU 1.0 (ESP-12E Module)
CPU Freq     : 80 MHz
Flash Size   : 4MB (FS:2MB OTA:~1019KB)
Upload Speed : 115200
```

---

### Langkah 2 — Install Library

Buka **Sketch → Include Library → Manage Libraries** dan install:

```
Adafruit SSD1306
Adafruit GFX Library
Adafruit Fingerprint Sensor Library
```

> `ESP8266WiFi`, `ESP8266HTTPClient`, dan `ESP8266WebServer` sudah bawaan dari board package.

---

### Langkah 3 — Hotspot WiFi

Aktifkan hotspot di HP kamu:

```
SSID     : pwm
Password : 12345678
```

> Untuk pakai jaringan lain, ubah baris ini di sketch:
> ```cpp
> const char* ssid     = "pwm";
> const char* password = "12345678";
> ```

---

### Langkah 4 — Setup Google Sheets

#### 4a — Buat spreadsheet

1. Buka [sheets.google.com](https://sheets.google.com)
2. Buat spreadsheet baru dan beri nama `Fingerprint to Google Sheets`

#### 4b — Tambahkan Apps Script

1. Di dalam spreadsheet, klik **Extensions → Apps Script**
2. Hapus kode yang ada, ganti dengan kode ini:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);
  sheet.appendRow([new Date(), data.id, data.name, data.status]);
  return ContentService.createTextOutput("OK");
}

function doGet(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  sheet.appendRow([
    new Date(),
    e.parameter.id,
    e.parameter.name,
    e.parameter.status
  ]);
  return ContentService.createTextOutput("OK");
}
```

3. Klik **Save**

#### 4c — Deploy sebagai Web App

1. Klik **Deploy → New Deployment**
2. Klik ikon gear di sebelah Type → pilih **Web App**
3. Atur:
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Klik **Deploy → Authorize access** dan izinkan permission
5. Salin **Web App URL** — bentuknya seperti:

```
https://script.google.com/macros/s/AKfycb.........../exec
```

#### 4d — Tempel URL ke sketch

Cari baris ini di sketch dan ganti dengan URL kamu:

```cpp
const char* scriptURL = "https://script.google.com/macros/s/YOUR_URL_HERE/exec";
```

#### 4e — Tes

Tempel ini di browser kamu (ganti dengan URL kamu):

```
https://script.google.com/macros/s/YOUR_URL_HERE/exec?id=1&name=Test&status=Present
```

Baris baru akan langsung muncul di spreadsheet. Kalau berhasil — kamu sudah siap.

---

### Langkah 5 — Tambahkan Nama

Edit `getNameByID()` di sketch supaya sesuai dengan ID sidik jari yang sudah didaftarkan:

```cpp
String getNameByID(int id) {
  switch (id) {
    case 1: return "Agus";
    case 2: return "Bayu";
    case 3: return "Cahya";
    // tambahkan lainnya...
    default: return "User_" + String(id);
  }
}
```

---

### Langkah 6 — Upload dan Jalankan

1. Sambungkan NodeMCU lewat kabel USB
2. Pilih COM port yang benar di **Tools → Port**
3. Klik **Upload**
4. Buka **Serial Monitor** di baud rate `115200`
5. Nyalakan hotspot `pwm`
6. Alamat IP akan muncul di OLED dan di Serial Monitor
7. Buka alamat IP tersebut di browser apa saja yang terhubung ke hotspot

---

## 🌐 Dashboard Web

Sambungkan perangkat apa saja ke hotspot `pwm` dan buka alamat IP yang muncul di OLED lewat browser.

| Fitur | Deskripsi |
|---------|-------------|
| Overview | Total ID terdaftar dan jumlah log hari ini |
| Daftarkan | Masukkan ID + Nama → klik Daftarkan → tempelkan jari di sensor |
| Hapus | Hapus sidik jari berdasarkan ID atau klik Hapus di samping daftarnya |
| Daftar ID | Daftar live semua sidik jari terdaftar beserta namanya |
| Log absensi | Setiap scan tercatat dengan nama, ID, waktu, dan status |
| Auto-refresh | Statistik dan mode update otomatis setiap 3 detik |

---

## 🖥️ Perintah Serial

Buka Serial Monitor di baud rate **115200:**

| Perintah | Aksi |
|---------|--------|
| `A` | Masuk mode daftar (enroll) |
| `#5` | Daftarkan sidik jari sebagai ID 5 |
| `#12` | Daftarkan sidik jari sebagai ID 12 |
| `R#5` | Hapus sidik jari ID 5 |
| `R#12` | Hapus sidik jari ID 12 |
| `L` | Tampilkan semua ID yang tersimpan |

---

## 📋 Contoh Output di Spreadsheet

| Timestamp | ID | Nama | Status |
|-----------|----|------|--------|
| 6/14/2026 09:00:01 | 1 | Agus | Hadir |
| 6/14/2026 09:02:45 | 2 | Bayu | Hadir |
| 6/14/2026 09:05:12 | 3 | Cahya | Hadir |

---

## 🔧 Troubleshooting

| Masalah | Solusi |
|---------|-----|
| OLED blank | Coba alamat I2C `0x3D` sebagai ganti `0x3C` di kode |
| OLED tampilan kacau | Tukar posisi jumper D1 dan D2 |
| Sidik jari tidak terdeteksi | Pastikan VCC tersambung ke VIN, bukan 3.3V |
| Ghost reads | Sudah diatasi — sudah ada triple-read + filter kepercayaan |
| Buzzer tidak bersuara | Pastikan menggunakan buzzer pasif, bukan aktif |
| WiFi tidak konek | Nyalakan hotspot SSID `pwm` · Password `12345678` |
| Sheets tidak mencatat | Deploy ulang Apps Script dan tempel URL baru di kode |
| Dashboard tidak terbuka | Pastikan perangkat terhubung ke hotspot `pwm` |
| Error HTTP saat POST | `client.setInsecure()` sudah diatur — cek lagi URL script-nya |

---

## 🛠️ Dibangun Dengan

- [Arduino ESP8266 Core](https://github.com/esp8266/Arduino)
- [Adafruit SSD1306](https://github.com/adafruit/Adafruit_SSD1306)
- [Adafruit GFX Library](https://github.com/adafruit/Adafruit-GFX-Library)
- [Adafruit Fingerprint Sensor Library](https://github.com/adafruit/Adafruit-Fingerprint-Sensor-Library)
- [Google Apps Script](https://developers.google.com/apps-script)

---

## 📄 Lisensi

Lisensi MIT — bebas digunakan, dimodifikasi, dan disebarluaskan.

---

<div align="center">

Dibuat dengan ❤️ — Fingerprint to Google Sheets Open Source Project

*Hardware: ESP8266 · Software: Arduino C++ · Cloud: Google Sheets*

</div>

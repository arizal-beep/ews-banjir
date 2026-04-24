# ews-banjir

# Realtime Sensor Monitoring System

Desain arsitektur sistem untuk monitoring sensor secara realtime (bukan batch/file).

---

## 1. Dari Batch ke Realtime

Pada sistem produksi, data sensor dikirim setiap 1 menit melalui API, sehingga arsitektur menggunakan pendekatan event-driven.

### Komponen:

### 1) Ingestion Layer (Data Receiver)
- API endpoint (Node.js / Express.js)
- Tugas:
  - Menerima data dari sensor
  - Validasi (format, range, timestamp)
  - Mengirim ke message broker

---

### 2) Message Broker (Queue)
- Contoh: Apache Kafka / RabbitMQ
- Tugas:
  - Menangani aliran data realtime
  - Buffer saat terjadi lonjakan data

---

### 3) Processing Service (Realtime Engine)
- Microservice (Node.js / Python)
- Tugas:
  - Menentukan status (AMAN / WASPADA / AWAS)
  - Deteksi perubahan status
  - Mengirim data ke:
    - Database
    - Notification Service
    - WebSocket

---

### 4) Database
- Contoh: PostgreSQL / InfluxDB
- Menyimpan:
  - Data sensor (time-series)
  - Status terakhir
  - Riwayat notifikasi

---

### 5) Realtime UI (Frontend)
- Menggunakan WebSocket (Socket.IO)
- Tugas:
  - Update dashboard tanpa refresh
  - Menampilkan grafik dan status

---

### Alur Data: Sensor → API → Queue → Processing → Database + WebSocket → UI


---

## 2. Flow Notifikasi

### Kapan notifikasi dikirim?
- Hanya saat terjadi perubahan status:
  - AMAN → WASPADA
  - WASPADA → AWAS

Opsional:
- Reminder berkala jika masih dalam status AWAS

---

### Masalah Fluktuasi Data
Contoh: 199 → 201 → 199 → 201


Dapat menyebabkan spam notifikasi.

---

### Solusi:

#### 1) Hysteresis
- WASPADA naik: ≥ 200
- Kembali AMAN: ≤ 180

#### 2) Debounce
- Status harus stabil selama 2–3 reading sebelum dianggap valid

#### 3) Cooldown
- Setelah kirim notifikasi:
  - Tunda pengiriman ulang (misal 10 menit)
  - Kecuali terjadi kenaikan level

---

### Komponen:
- Processing Service → logika status & filtering
- Notification Service → kirim WhatsApp/SMS (misal via Twilio)

---

## 3. Sensor Mati (Offline Detection)

### Deteksi:
Setiap sensor memiliki: last_seen_timestamp

Jika: tidak ada data > 3x interval (misal > 3 menit)

Maka: status = OFFLINE

---

### Implementasi:

#### Heartbeat Checker
- Background job (cron tiap 1 menit)
- Mengecek semua sensor:
  - Jika lewat threshold → OFFLINE

---

### Tampilan di Dashboard:
- Status: OFFLINE
- Tidak dianggap AMAN
- Informasi tambahan:
  - "Last seen: X menit lalu"

---

### Notifikasi:
- Dikirim saat:
  - ONLINE → OFFLINE
- Opsional:
  - Reminder berkala jika masih offline

---

### Komponen:
- Monitoring Service / Scheduler → deteksi sensor mati
- Notification Service → kirim alert

---

## Ringkasan Arsitektur

Core System:
- API Ingestion
- Message Broker (Kafka / RabbitMQ)
- Processing Service
- Database
- WebSocket (Realtime UI)
- Notification Service
- Monitoring Service (Sensor Offline)

---

## Tujuan Sistem

- Realtime monitoring tanpa refresh
- Notifikasi akurat (tidak spam)
- Deteksi kegagalan sensor
- Stabil terhadap fluktuasi data

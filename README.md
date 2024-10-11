# Cara menampilkan ranking traffic client/simple queue mikrotik menggunakan Zabbix tools monitoring
Untuk menampilkan **ranking traffic client** dari paling tinggi ke paling rendah berdasarkan **Simple Queue** di MikroTik menggunakan Zabbix, Anda dapat mengikuti beberapa langkah berikut. Pada dasarnya, Zabbix akan memonitor traffic tiap klien berdasarkan Simple Queue di MikroTik melalui SNMP, lalu menampilkan hasilnya dalam grafik atau peringkat berdasarkan penggunaan bandwidth. Berikut langkah-langkah umumnya:

### 1. **Konfigurasi SNMP di MikroTik**

Pastikan **SNMP** diaktifkan pada perangkat MikroTik. SNMP akan digunakan oleh Zabbix untuk mengambil data traffic dari Simple Queue.

1. Masuk ke MikroTik melalui terminal atau Winbox.
2. Aktifkan SNMP di MikroTik:
   ```
   /snmp set enabled=yes
   /snmp community add name=public addresses=0.0.0.0/0
   ```
   Ini memungkinkan Zabbix mengambil data melalui SNMP dengan community `public`.

### 2. **Template Zabbix untuk Simple Queue**

Anda memerlukan template Zabbix yang dapat mengekstrak informasi **Simple Queue** dari MikroTik melalui SNMP. Berikut adalah langkah untuk membuatnya:

1. Buat **discovery rule** untuk Simple Queue seperti pada contoh sebelumnya.
   - **Discovery Rule** menggunakan OID **Simple Queue** untuk mendapatkan daftar antrian yang ada.
   - SNMP OID untuk Simple Queue pada MikroTik:
     - Nama Simple Queue: `.1.3.6.1.4.1.14988.1.1.2.1.1.2`
     - Traffic IN: `.1.3.6.1.4.1.14988.1.1.2.1.1.8.{#SNMPINDEX}`
     - Traffic OUT: `.1.3.6.1.4.1.14988.1.1.2.1.1.9.{#SNMPINDEX}`

2. Buat **item prototype** dalam template Zabbix:
   - **Traffic IN (bps)**: Menggunakan OID `.1.3.6.1.4.1.14988.1.1.2.1.1.8.{#SNMPINDEX}` untuk setiap queue.
   - **Traffic OUT (bps)**: Menggunakan OID `.1.3.6.1.4.1.14988.1.1.2.1.1.9.{#SNMPINDEX}`.
   - Item ini akan secara otomatis memonitor traffic dari setiap Simple Queue yang ditemukan melalui discovery rule.

### 3. **Menampilkan Traffic Client Berdasarkan Peringkat**

1. **Buat LLD Graph**: Setelah data dari Simple Queue diperoleh, Anda bisa membuat grafik menggunakan **graph prototype** pada Zabbix yang sudah mendeteksi semua queue.
   - Masukkan dua item:
     - **IN traffic** (download) dari queue.
     - **OUT traffic** (upload) dari queue.
   - Ini akan menampilkan traffic masing-masing klien berdasarkan Simple Queue.

2. **Membuat Trigger untuk Menentukan Peringkat**:
   - Gunakan trigger untuk memantau **traffic tertinggi**.
   - Anda dapat membuat trigger dengan threshold tertentu untuk memunculkan alert berdasarkan penggunaan traffic yang tinggi pada Simple Queue.

### 4. **Query Custom untuk Ranking**

Jika ingin mendapatkan laporan ranking secara otomatis, Anda bisa menggunakan **Zabbix API** atau **custom Zabbix reports** dengan **query SQL** untuk menampilkan traffic dari Simple Queue dan mengurutkan berdasarkan usage bandwidth.

Langkahnya:
1. **Query Zabbix Database**:
   Anda bisa menjalankan query pada database Zabbix untuk mendapatkan peringkat klien berdasarkan traffic. Query ini bisa mencakup penarikan data dari tabel item, history, dan host yang terkait dengan Simple Queue.
   
   Contoh query SQL (untuk MySQL/MariaDB) bisa seperti ini:
   ```sql
   SELECT h.host, i.name, MAX(hr.value) AS max_traffic
   FROM hosts h
   JOIN items i ON h.hostid = i.hostid
   JOIN history_uint hr ON i.itemid = hr.itemid
   WHERE i.name LIKE 'Queue%'  -- Filter item yang berhubungan dengan Simple Queue
   GROUP BY h.host, i.name
   ORDER BY max_traffic DESC;  -- Mengurutkan berdasarkan traffic tertinggi
   ```
2. **Mengintegrasikan ke dalam Dashboard**: Hasil query ini dapat diintegrasikan ke dalam **custom Zabbix dashboard** atau diambil melalui **API Zabbix** untuk menampilkan peringkat secara langsung.

### 5. **Laporan di Zabbix**

Untuk mempermudah proses pelaporan, Anda juga dapat menggunakan fitur **Reports** pada Zabbix atau menyiapkan **Zabbix API** untuk menampilkan peringkat klien berdasarkan traffic pada Simple Queue di dashboard atau email notifikasi.

### Alternatif: **Grafana Integration**
Anda juga bisa mengintegrasikan **Zabbix dengan Grafana** untuk membuat visualisasi ranking traffic klien berdasarkan Simple Queue dalam bentuk grafik yang lebih interaktif dan fleksibel.

Dengan cara ini, Zabbix akan memantau semua traffic Simple Queue di MikroTik dan menampilkan ranking penggunaan traffic secara otomatis di dashboard.

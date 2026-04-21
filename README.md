# Praktikum Mikrokontroler - Modul I: Percabangan dan Perulangan

**Nama:** Ibnu Abbas  
**NIM:** H1H024038  
**Mata Kuliah:** TK244005 - Praktikum Mikrokontroler  
**Program Studi:** Informatika, Universitas Jenderal Soedirman  

---

## 📌 Deskripsi Repositori
Repositori ini berisi source code, skematik rangkaian, dan dokumentasi hasil praktikum Modul 3 yang berfokus pada implementasi protokol komunikasi serial menggunakan antarmuka UART (Universal Asynchronous Receiver-Transmitter) dan I2C (Inter-Integrated Circuit) pada platform mikrokontroler Arduino Uno.

Praktikum ini dirancang untuk memahami bagaimana mikrokontroler dapat menerima komando interaktif dari antarmuka komputer melalui Serial Monitor (UART), serta bagaimana membaca nilai sensor analog (potensiometer) dan mentransmisikan datanya secara sinkron ke dalam modul display eksternal seperti LCD I2C.

---
## 🔬 Analisis Percobaan 1 (UART)
### 1. Jelaskan proses dari input keyboard hingga LED menyala/mati!
Saat pengguna mengetikkan perintah ('1' atau '0') di keyboard melalui Serial Monitor, komputer mengirimkan karakter tersebut dalam bentuk biner melalui kabel USB. IC konverter USB-to-Serial pada board Arduino mengubah sinyal USB tersebut menjadi sinyal UART standar dan meneruskannya ke pin RX (Receive) mikrokontroler utama (ATmega328P). Data ini kemudian disimpan sementara di dalam serial buffer Arduino. Saat fungsi Serial.read() dipanggil di dalam loop(), mikrokontroler membaca karakter tersebut, lalu memprosesnya melalui struktur percabangan if. Jika karakter yang dibaca adalah '1', mikrokontroler mengeksekusi digitalWrite(PIN_LED, HIGH) yang mengirimkan tegangan ke pin digital sehingga LED menyala.
### 2. Mengapa digunakan Serial.available() sebelum membaca data? Apa yang terjadi jika baris tersebut dihilangkan?
Serial.available() digunakan untuk memastikan bahwa mikrokontroler hanya melakukan pembacaan serial buffer ketika ada data instruksi baru yang masuk. Jika baris ini dihilangkan, fungsi Serial.read() akan dipanggil terus-menerus tanpa henti. Jika tidak ada data yang masuk, Serial.read() akan mengembalikan nilai -1, yang dapat menyebabkan program mengeksekusi blok kondisi yang salah secara terus-menerus (misalnya masuk ke blok peringatan error) dan membuat sistem menjadi buggy dan membuang sumber daya pemrosesan.
### 3.  Modifikasi program agar LED berkedip (blink) ketika menerima input '2' (Sertakan penjelasan baris kode)
```cpp const int PIN_LED = 8;
char state = '0';                   // Variabel untuk menyimpan status perintah terakhir
unsigned long previousMillis = 0;   // Menyimpan waktu terakhir LED berkedip
const long interval = 500;          // Interval kedip LED (500 ms)
bool ledState = LOW;                // Menyimpan status nyala/mati LED saat ini

void setup() {
  Serial.begin(9600);               // Inisialisasi komunikasi serial baud rate 9600
  pinMode(PIN_LED, OUTPUT);         // Mengatur pin LED sebagai OUTPUT
  Serial.println("Ketik '1' (ON), '0' (OFF), '2' (BLINK)");
}

void loop() {
  // Cek apakah ada data masuk di serial buffer
  if (Serial.available() > 0) {
    char data = Serial.read();      // Membaca 1 byte karakter dari serial
    
    // Validasi agar hanya mengubah state jika inputnya sesuai (0, 1, atau 2)
    if (data == '0' || data == '1' || data == '2') {
      state = data;                 // Perbarui state dengan perintah terbaru
    }
  }

  // Eksekusi berdasarkan state terakhir yang tersimpan
  if (state == '1') {
    digitalWrite(PIN_LED, HIGH);    // LED nyala konstan
  } 
  else if (state == '0') {
    digitalWrite(PIN_LED, LOW);     // LED mati konstan
  } 
  else if (state == '2') {
    // Mode Blink menggunakan millis() agar non-blocking
    unsigned long currentMillis = millis(); // Ambil waktu sistem saat ini
    
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;       // Catat waktu kedip terakhir
      ledState = !ledState;                 // Balik status LED (HIGH ke LOW / LOW ke HIGH)
      digitalWrite(PIN_LED, ledState);      // Terapkan status ke pin LED
    }
  }
}

### 4.  Berikan penjelasan disetiap baris kode nya setalah  LED tidak langsung reset → tetapi berubah dari cepat → sedang → mati
```cpp
const int ledPin = 6;        // Pin digital tempat LED terhubung
int timeDelay = 1000;        // Nilai awal delay (ms)

void setup() {
  pinMode(ledPin, OUTPUT);   // Atur pin LED sebagai output
}

void loop() {
  // Nyalakan LED
  digitalWrite(ledPin, HIGH);
  delay(timeDelay);

  // Matikan LED
  digitalWrite(ledPin, LOW);
  delay(timeDelay);

  // Ubah pola delay setelah 1 siklus kedip
  if (timeDelay <= 100) { 
    delay(3000);             // jeda 3 detik sebelum reset
    timeDelay = 1000;        // reset ke kondisi mati (awal)
  } 
  else if (timeDelay <= 500) {
    timeDelay = 700;         // dari cepat → sedang
  } 
  else {
    timeDelay -= 200;        // percepatan bertahap (1000 → 800 → 600 → dst)
  }
}
```
### 4.Tentukan apakah menggunakan delay() atau millis()! Jelaskan pengaruhnya terhadap sistem
Untuk membuat modifikasi blink yang bisa dipotong/diinterupsi oleh perintah lain secara real-time, wajib menggunakan millis().
Jika menggunakan delay(), sistem akan memblokir (blocking) seluruh eksekusi program. Artinya, saat LED sedang berada di fase penundaan nyala/mati (misal delay(500)), Arduino "tuli" dan tidak akan bisa langsung merespons jika kita memasukkan input baru (seperti mematikan LED dengan input '0') sampai waktu delay() tersebut habis sepenuhnya. Dengan millis(), program bekerja secara asynchronous/non-blocking, sehingga mikrokontroler dapat terus memonitor input dari Serial.available() sembari menjaga ritme kedipan LED di latar belakang
---
## Analisa Percobaan 2
### 1. Gambarkan rangkaian schematic 5 LED running yang digunakan pada percobaan!
![LED Circuit](skematikp2)
### 2. Jelaskan bagaimana program membuat efek LED berjalan dari kiri ke kanan!
Ketika program menyalakan pin 7 maka program akan beralih ke loppingan for yang kedua dimana isinya dalah mengurangi pin output sehingga pin yang aktif adalah dari kiri ke kanan.
### 3.Jelaskan bagaimana program membuat LED kembali dari kanan ke kiri!
Porogram menjalankan looping for pertama yang mengaktifkan pin 2 hingga ke 8
### 4.Buatkan program agar LED menyala tiga LED kanan dan tiga LED kiri secara bergantian  dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!
```cpp
int timer = 100;           
// delay. Semakin tinggi angkanya, semakin lambat waktunya. 
void setup() { 
// gunakan loop for untuk menginisialisasi setiap pin sebagai 
output: 
for (int ledPin = 2; ledPin < 4; ledPin++) { 
pinMode(ledPin, OUTPUT); 
} 
} 
void loop() { 
// looping dari pin rendah ke tinggi 
for (int ledPin = 2; ledPin < 4; ledPin++) { 
// hidupkan LED pin nya: 
digitalWrite(ledPin, HIGH); 
delay(timer); 
// matikan pin LED nya: 
digitalWrite(ledPin, LOW); 
}
for (int ledPin = 3; ledPin >= 2; ledPin--) { 
// menghidupkan pin: 
digitalWrite(ledPin, HIGH); 
delay(timer); 
// mematikan pin: 
digitalWrite(ledPin, LOW); 
} 
}
```
Program tetap sama hanya saja wiringnya dijadikan 1 sehingga tetap bisa menghasilkan output yang sama berikut adalah Wiringnya
![LED Pararel](wiringp3)



## 📁 Penjelasan File Program

### 1. `Praktikum_Percobaan_1.ino` (Percabangan)
**Tujuan Kode:** File ini berisi program untuk mendemonstrasikan fungsi struktur logika percabangan menggunakan instruksi `if-else`. Program ini bertujuan untuk mengevaluasi kondisi parameter batas waktu tunda (*delay*) dan secara dinamis mengubah kecepatan fase kedipan sebuah LED tunggal tanpa memerlukan intervensi manual dari pengguna. Pada tugas modifikasi praktikum ini, program dirancang agar siklus nyala LED berubah secara bertahap: mulai dari fase berkedip cepat, kemudian memelan (kecepatan sedang), dan pada akhirnya sistem akan menahan LED dalam kondisi mati secara permanen (*reset*).

### 2. `Modul_1_Percobaann_2.ino` (Perulangan)
**Tujuan Kode:** File ini berisi program untuk mendemonstrasikan efisiensi struktur kendali perulangan menggunakan instruksi `for` *loop*. Program ini bertujuan untuk memanipulasi rentetan banyak pin digital secara otomatis hanya dengan sedikit baris kode guna menciptakan efek visual pergerakan cahaya pada susunan enam buah LED. Pada tugas modifikasi praktikum ini, instruksi perulangan direkayasa untuk membagi deretan 6 LED menjadi dua blok terpisah (3 LED di sisi kiri dan 3 LED di sisi kanan), di mana program akan memerintahkan kedua kelompok tersebut untuk menyala secara bergantian secara terus-menerus.

---

## 📸 Dokumentasi Praktikum
### 1. Dokumentasi Percobaan
Percobaan 1:
![Lampiran percobaan 1 Praktikum](WhatsApp%20Image%202026-04-07%20at%2013.12.22.jpeg)
Percobaan 2: 
![Lampiran percobaan 2 Praktikum](WhatsApp%20Image%202026-04-07%20at%2013.12.27.jpeg)

### 2. Skematik Rangkaian
Skematik Rangkaian prcobaan 1
![Skematik Rangkaian prcobaan 1](Screenshot%202026-04-08%20015201.png)
Skematik Rangkaian prcobaan 2
![Tautan Gambar Skematik_2 Di Sini](Screenshot%202026-04-08%20015551.png)

*Laporan praktikum lengkap beserta analisis data tersedia pada direktori utama repositori ini dalam format dokumen.*

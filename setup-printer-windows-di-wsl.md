# Setup Printer Windows di WSL (CUPS + SMB)

## Environment

* Host OS: Windows 11
* WSL: Ubuntu
* Printer: Thermal POS80 (USB)
* Printer terpasang di Windows
* Aplikasi berjalan di WSL (Laravel/PHP)

---

# 1. Share printer di Windows

Buka:

```
Control Panel
→ Devices and Printers
→ Klik kanan printer
→ Printer Properties
→ Sharing
```

Centang:

```
✔ Share this printer
```

Contoh Share Name:

```
POS80
```

---

# 2. Pastikan printer dapat diakses melalui SMB

Install samba client di WSL:

```bash
sudo apt update
sudo apt install smbclient
```

Test:

```bash
smbclient -L //192.168.1.100 -U enjat
```

Jika berhasil akan muncul:

```
Sharename       Type
---------       ----
POS80           Printer
print$          Disk
```
Testing print dari smb:

```
echo -e "\n\nTEST PRINT DARI SERVER\n\n" > /tmp/testprint.txt
```
```
smbclient //192.168.1.100/POS80 -U enjat -c 'print /tmp/testprint.txt' -m SMB2
```
Apabila gagal, pastikan:

* File and Printer Sharing aktif di Windows.
* Firewall mengizinkan SMB.
* Username/password Windows benar.

---

# 3. Install CUPS

```bash
sudo apt install cups cups-client
```

Aktifkan service:

```bash
sudo systemctl enable cups
sudo systemctl start cups
```

Cek:

```bash
systemctl status cups
```

Harus muncul:

```
Active: active (running)
```

---

# 4. Pastikan backend SMB tersedia

Jalankan:

```bash
sudo lpinfo -v
```

Harus terdapat:

```
network smb
```
---

# 5. Tambahkan printer ke CUPS

Gunakan:

```bash
sudo lpadmin \
  -p POS80 \
  -E \
  -v "smb://enjat:PASSWORD_WINDOWS@192.168.1.100/POS80" \
  -m raw
```

Contoh:

```bash
sudo lpadmin \
  -p POS80 \
  -E \
  -v "smb://enjat:admin123@192.168.1.100/POS80" \
  -m raw
```

Catatan:

`-m raw` akan menampilkan warning:

```
Raw queues are deprecated
```

Warning ini dapat diabaikan pada CUPS saat ini.

---

# 6. Pastikan printer berhasil ditambahkan

```bash
lpstat -p
```

Output:

```
printer POS80 is idle. enabled since ...
```

Cek URI printer:

```bash
lpstat -v
```

---

# 7. Test print

Buat file:

```bash
printf "Test Print\n" > test.txt
```

Cetak:

```bash
lp -d POS80 test.txt
```

Jika berhasil:

```
request id is POS80-1 (1 file(s))
```

---

# 8. Jika tidak mencetak

Lihat status job:

```bash
lpstat -W all
```

Lihat log:

```bash
sudo tail -100 /var/log/cups/error_log
```

Error yang pernah ditemukan:

```
NT_STATUS_ACCESS_DENIED
```

dan

```
NT_STATUS_LOGON_FAILURE
```

Penyebabnya adalah URI printer belum menyertakan username/password Windows.

---

# 9. Integrasi Laravel (mike42/escpos-php)

Jangan gunakan:

```php
new FilePrintConnector("/dev/usb/lp0");
```

Karena WSL tidak memiliki akses langsung ke printer USB Windows.

Gunakan:

```php
use Mike42\Escpos\Printer;
use Mike42\Escpos\PrintConnectors\CupsPrintConnector;

$connector = new CupsPrintConnector("POS80");

$printer = new Printer($connector);

$printer->text("Hello World\n");
$printer->cut();
$printer->close();
```

---

# Troubleshooting

## Cek printer

```bash
lpstat -p
```

## Cek printer

```bash
lpstat -p
```

## Allow Firewall — buka port SMB di powershell

```bash
netsh advfirewall firewall add rule name="SMB Printer" dir=in action=allow protocol=TCP localport=445
```

## Cek printer URI

```bash
lpstat -v
```

## Cek backend

```bash
sudo lpinfo -v
```

## Cek log

```bash
sudo tail -100 /var/log/cups/error_log
```

## Test SMB

```bash
smbclient -L //192.168.1.100 -U enjat
```

---

# Arsitektur

```
Laravel
      │
      ▼
CupsPrintConnector
      │
      ▼
CUPS
      │
      ▼
SMB
      │
      ▼
Windows Print Spooler
      │
      ▼
POS80 Printer
```

---

# Catatan

* Jangan gunakan `/dev/usb/lp0` pada WSL.
* Printer tetap dikelola oleh Windows.
* WSL mengirim data ke Windows melalui SMB.
* Password akun Windows harus disertakan pada URI SMB agar CUPS dapat melakukan autentikasi tanpa interaksi pengguna.

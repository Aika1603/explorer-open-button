## Jawabannya: **BISA**

Solusi *Custom Protocol Handler* ini berbasis **Windows Registry**, yang merupakan fitur inti dari sistem operasi Windows. Jadi, cara ini kompatibel dengan:

- Windows 7
- Windows 8/8.1
- **Windows 10**
- **Windows 11**

Mekanisme ini sama persis dengan cara kerja aplikasi seperti Zoom (`zoommtg://`) atau Steam (`steam://`) ketika membuka aplikasi desktop dari browser.

Berikut adalah panduan **Step-by-Step yang sudah disempurnakan** agar lebih stabil di Windows 10 dan 11, serta mengatasi masalah format *slash* (`/` vs `\`).

---

### Langkah 1: Siapkan Script Pemroses (Sekali saja per PC)

Kita akan menggunakan VBScript agar saat tombol diklik, **tidak muncul jendela hitam (Command Prompt)** yang mengganggu.

1. Di komputer user, buat folder khusus agar rapi, misalnya: `C:\Apps\CustomScripts`.
2. Di dalam folder itu, buat file baru bernama `open_nas.vbs`.
3. Isi file tersebut dengan kode berikut (sudah diperbarui untuk menghandle *network path* dengan benar):

```vbscript
' File: C:\Apps\CustomScripts\open_nas.vbs
On Error Resume Next

Set objShell = CreateObject("WScript.Shell")

' Ambil argumen dari browser (misal: buka-nas://192.168.1.50/products/A001)
If WScript.Arguments.Count > 0 Then
    fullUrl = WScript.Arguments(0)
    
    ' 1. Hapus protokol "buka-nas://" (panjangnya 11 karakter)
    ' Kita ambil substring mulai dari karakter ke-12
    cleanPath = Mid(fullUrl, 12)
    
    ' 2. Jika ada trailing slash di akhir, hapus (opsional, biar rapi)
    If Right(cleanPath, 1) = "/" Then
        cleanPath = Left(cleanPath, Len(cleanPath) - 1)
    End If
    
    ' 3. Ubah semua Forward Slash (/) jadi Backslash (\) ala Windows
    windowsPath = Replace(cleanPath, "/", "\")
    
    ' 4. Tambahkan double backslash (\\) di depan untuk format Network Share
    finalPath = "\\" & windowsPath
    
    ' 5. Eksekusi Explorer
    ' Hasil akhir command: explorer.exe \\192.168.1.50\products\A001
    objShell.Run "explorer.exe " & finalPath
End If
```

---

### Langkah 2: Daftarkan ke Registry (Sekali saja per PC)

Langkah untuk memberitahu Windows 10/11 bahwa jika ada link `buka-nas://`, jalankan script di atas.

1. Buka Notepad, copy kode di bawah ini.
2. Simpan dengan nama `register_nas.reg`.
3. Double-click file tersebut dan pilih **Yes**.

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\buka-nas]
@="URL:Buka NAS Protocol"
"URL Protocol"=""

[HKEY_CLASSES_ROOT\buka-nas\shell]

[HKEY_CLASSES_ROOT\buka-nas\shell\open]

[HKEY_CLASSES_ROOT\buka-nas\shell\open\command]
@="wscript.exe \"C:\\Apps\\CustomScripts\\open_nas.vbs\" \"%1\""
```

> **Note:** Pastikan path di baris terakhir sesuai dengan lokasi file VBS yang kamu buat di Langkah 1. Perhatikan tanda backslash harus double `\\` di dalam file .reg.

---

### Langkah 3: Implementasi di Web App (HTML/React)

Sekarang di kode React/Blade/HTML kamu, format link-nya harus seperti ini:

```html
<a href="buka-nas://192.168.1.50/products/{{ $kode_produk }}">
   <button>Buka Folder Produk</button>
</a>
```

---

### Apa yang akan terjadi di Windows 10/11? (Penting!)

Saat user mengklik tombol tersebut untuk **pertama kalinya**, Browser (Chrome/Edge/Firefox) akan memunculkan popup keamanan seperti ini:

> **Open wscript.exe?**  
> [https://your-website.com](https://your-website.com) wants to open this application.  
> [ ] Always allow ... to open links of this type in the associated app.

**Instruksi untuk User:**

1. Centang kotak **"Always allow..."** (Selalu izinkan).
2. Klik **Open**.

Setelah itu, klik berikutnya akan langsung membuka Windows Explorer tanpa tanya-tanya lagi.

---

### Catatan Khusus Windows 11 (Future Proofing)

Microsoft berencana perlahan menghilangkan VBScript di masa depan (mulai update 24H2, VBScript jadi fitur *On-Demand*).
Untuk saat ini (2025/2026), VBScript **masih ada** secara default.

Jika suatu saat VBScript benar-benar dihapus Microsoft, kamu tinggal mengganti file `.vbs` di Langkah 1 menjadi file `.bat` (Batch file).

**Contoh jika pakai .bat (Jaga-jaga):**
```bat
@echo off
set "url=%1"
:: Hapus prefix buka-nas:// (ini butuh logic string replacement agak ribet di batch)
:: Lalu panggil explorer
start explorer.exe \\192.168.1.50\products\...
```
> **Kekurangan batch file:** Akan muncul kotak hitam (CMD) berkedip sebentar sebelum explorer terbuka. VBScript lebih mulus (invisible).

---

## Kesimpulan

Metode ini **sangat aman dan bekerja lancar** di Windows 10 dan 11 selama kamu punya akses untuk install file `.reg` dan `.vbs` di komputer karyawan.
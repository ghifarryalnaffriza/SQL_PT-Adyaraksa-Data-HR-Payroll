# 👥 PT Adyaraksa: People Analytics Review

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?logo=linkedin&logoColor=white)](https://linkedin.com/in/ghifarryalnaffriza)
[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white)](https://github.com/ghifarryalnaffriza)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?logo=instagram&logoColor=white)](https://instagram.com/ghifaralnaffi_)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail&logoColor=white)](mailto:ghifaralns@gmail.com)

> HR analytics atas 500 karyawan: turnover 18% ternyata **merata**, tidak terkonsentrasi di departemen atau angkatan tertentu. Dua mitos HR paling populer (*top performer pada cabut*, *gaji mandek bikin resign*) juga **tidak terbukti**. Masalah yang nyata justru ada di sistem HR-nya: promosi, penilaian kinerja, dan struktur atasan.

![Dashboard Turnover](Dasboards/Dasboard%20Images/Turnover.png)

## 🎯 Business Problem

PT Adyaraksa ingin tahu kenapa karyawan pergi sebelum menyusun program retensi. Enam pertanyaan dibagi ke tiga halaman dashboard untuk menjawab: **apakah masalah retensi ini terkonsentrasi di kelompok tertentu, dan kebijakan HR mana yang justru bermasalah?**

1. Seberapa besar turnover, dan apakah terkonsentrasi di departemen tertentu?
2. Apakah karyawan angkatan masuk tertentu lebih cepat keluar?
3. Apakah kinerja, kenaikan gaji, posisi gaji, promosi, level, atau usia membedakan karyawan yang resign?
4. Apakah promosi dihargai dengan kenaikan gaji yang lebih besar?
5. Apakah skor review akhir mencerminkan KPI, attitude, dan catatan reviewer?
6. Apakah setiap karyawan punya atasan yang jelas, dan apakah beban manajer wajar?

## 📊 Dataset

- **Sumber:** 5 tabel HR: `employees`, `departments`, `attendance`, `performance`, `salary_history`
- **Volume:** 500 karyawan · 15 departemen · 3.000 record absensi · 600 review · 1.500 baris payroll
- **Periode:** hire cohort 2015-2024 · review 2022-2024 · absensi Q4 2024 · snapshot 1 Januari 2025
- **Cakupan:** 8 kota · 4 level jabatan

## 🏗️ Arsitektur & Pendekatan

Medallion architecture di SQL Server (T-SQL), dipisah ke tiga schema:

- **Bronze**: 5 tabel dimuat apa adanya dengan `BULK INSERT`. Tanggal payroll disimpan sebagai teks dulu karena formatnya campur.
- **Silver**: masalah data quality ditangani **secara eksplisit dan bisa diaudit**. 3 karyawan tanpa `dept_id` masuk grup "Belum di-assign", 3 format tanggal payroll diparse sesuai polanya, 8 duplikat dihapus via `ROW_NUMBER()`, dan 430 `check_out` kosong dipisah jadi 400 baris wajar (absent/leave/holiday) dan 30 yang benar-benar lupa clock-out. Data yang tidak valid (69 review sebelum tanggal masuk, 3 review diri sendiri, 147 catatan gaji setelah snapshot) diberi flag, tidak dihapus.
- **Gold**: 19 view yang dipetakan ke 6 pertanyaan bisnis (`vw_q1_turnover_dept`, `vw_q3_resign_per_faktor`, `vw_q4_kenaikan_per_alasan`, `vw_q6_span_of_control`, dst.), termasuk **uji chi-square yang dihitung langsung di SQL**.

Seluruh analisis kompensasi dibangun di atas tabel bersih **1.492 baris**, bukan 1.500 baris mentah. Bagian bawah file Gold berisi query cek angka beserta nilai yang diharapkan, jadi setiap angka di dashboard bisa diaudit ulang.

## 🛠️ Tools & Tech

![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![T-SQL](https://img.shields.io/badge/T--SQL-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0B2E59?style=for-the-badge&logo=powerbi&logoColor=white)
![Medallion](https://img.shields.io/badge/Medallion_Architecture-4B8BBE?style=for-the-badge&logo=databricks&logoColor=white)

## 🔍 Key Findings

- **Turnover 18% merata, tidak terkonsentrasi:** Design **23,3%** dan Sales 7,1% kelihatan jauh, tapi uji chi-square (8,71 vs nilai kritis 23,69) menunjukkan selisihnya masih kebetulan. Design cuma 30 orang, "kelebihan" resign-nya sekitar 3 orang.
- **Mitos "top performer pada cabut" tidak terbukti:** resign rate skor ≥4 = **14,3%** vs karyawan lain 15,4%.
- **Mitos "gaji mandek bikin resign" juga tidak terbukti:** 25% karyawan dengan kenaikan gaji terkecil resign **15,9%** vs lainnya 14,5%. Dari 6 faktor yang dicek, tidak ada yang lolos uji 5%.
- **Angkatan 2022 dan 2023 perlu dipantau:** retensi **70,4%** dan 68,4% vs rata-rata 80%, tapi belum lolos uji (14,32 vs nilai kritis 16,92).
- **Promosi tidak dibayar sebagai promosi:** kenaikan `promotion` **8,3%** justru di bawah `annual_review` **8,8%**, dan ada **14 promosi yang gajinya malah turun**.
- **Skor review tidak nyambung dengan KPI:** korelasi skor akhir vs KPI **−0,07**. Catatan "exceed expectation, bonus direkomendasikan" rata-rata skornya paling rendah (3,02), di bawah "perlu PIP" (3,14).
- **Skor review terdistorsi pola musiman:** mid-year selalu lebih murah hati (3,27-3,40) dibanding annual (3,17-3,23) di ketiga tahun, jadi promosi bisa bias ke siklus mid-year.
- **10 karyawan tanpa atasan aktif:** masih menunjuk manajer yang sudah resign. Ini perbaikan tercepat dan gratis di seluruh analisis.
- **Terlalu banyak manajer:** 146 orang punya bawahan, rata-rata cuma **2,3 bawahan**, maksimum 6, dan **54 manajer hanya punya 1 bawahan**.
- **Security paling sering terlambat:** late rate **16,0%** (tertinggi) dan absent 12,0%, tapi data absensi hanya mencakup Q4 2024.
- **Tenure bukan alat ukur retensi:** korelasi rata-rata tenure dan resign rate antar departemen hanya **r = 0,16**. Design punya tenure terpanjang kedua (5,6 th) sekaligus resign tertinggi.
- **Gap gender belum bisa disimpulkan:** di level Senior, median gaji perempuan Rp16,1 jt vs laki-laki Rp18,6 jt, tapi jumlahnya cuma 8 vs 12 orang.

## 🔬 Analisis

### 1. Resign per departemen

![Resign rate per departemen](Images/Screenshot%202026-11-10%20091851.png)

**Pertanyaan:** apakah resign terkonsentrasi di departemen tertentu?

**Insight:** Design paling tinggi (23,3%) dan Sales paling rendah (7,1%), selisihnya lebih dari 3 kali lipat. Tapi Design hanya punya 30 orang, jadi "kelebihan" resign-nya cuma sekitar 3 orang. Setelah diuji chi-square, selisih antar departemen masih dalam batas kebetulan. Resign tersebar merata, sehingga program retensi khusus per departemen belum punya dasar.

### 2. Retensi per angkatan

![Retensi per angkatan masuk](Images/Screenshot%202026-11-10%20092129.png)

**Pertanyaan:** apakah karyawan angkatan masuk tertentu lebih cepat keluar?

**Insight:** angkatan lama seharusnya punya retensi paling rendah karena lebih lama punya kesempatan untuk keluar. Kenyataannya, angkatan 2015 (82,7%) dan 2016 (86,0%) masih di atas rata-rata 80%, sementara yang paling rendah justru angkatan **2022 (70,4%)** dan **2023 (68,4%)** yang baru bekerja 2-3 tahun. Selisihnya belum lolos uji statistik, tapi paling dekat dengan batas dibanding uji lainnya, jadi layak dipantau di periode berikutnya.

### 3. Pemicu resign

**Pertanyaan:** apakah kinerja, kenaikan gaji, posisi gaji, promosi, level, atau usia membedakan karyawan yang resign?

| Kinerja | Promosi |
|---|---|
| ![Kinerja](Images/Screenshot%202026-11-10%20092422.png) | ![Promosi](Images/Screenshot%202026-11-10%20092448.png) |
| **Kenaikan gaji per tahun** | **Gaji vs rekan selevel** |
| ![Kenaikan gaji per tahun](Images/Screenshot%202026-11-10%20092342.png) | ![Gaji vs rekan selevel](Images/Screenshot%202026-11-10%20092314.png) |
| **Level jabatan** | **Usia** |
| ![Level jabatan](Images/Screenshot%202026-11-10%20092502.png) | ![Usia](Images/Screenshot%202026-11-10%20092518.png) |

**Insight:** semua kelompok berada di kisaran 10-19%, dekat dengan rata-rata perusahaan 14,0%, dan tidak ada faktor yang lolos uji statistik. Top performer tidak lebih banyak keluar (14,3% vs 15,4%), karyawan dengan kenaikan gaji terkecil resign hampir sama dengan lainnya (15,9% vs 14,5%), dan karyawan yang gajinya di bawah rekan selevel malah paling sedikit resign (12,3%). Penyebab resign kemungkinan ada di luar data ini, seperti atasan, beban kerja, atau tawaran dari perusahaan lain, jadi perlu exit interview terstruktur.

### 4. Kenaikan gaji per alasan

![Kenaikan gaji per alasan](Images/Screenshot%202026-11-10%20092632.png)

**Pertanyaan:** apakah promosi dihargai dengan kenaikan gaji yang lebih besar?

**Insight:** promosi seharusnya memberi kenaikan paling besar karena tanggung jawabnya bertambah, tapi justru paling kecil (**8,3%**), di bawah review tahunan (8,8%). Padahal median gaji Senior sekitar 71% di atas Staff, jadi kenaikan 8% tidak cukup untuk mengejar selisih itu. Ada juga **14 promosi yang malah menurunkan gaji**, contohnya EMP0130 turun dari Rp13,24 jt ke Rp10,93 jt. Naik jabatan tidak terasa sebagai penghargaan.

### 5. Skor review

![Skor review per siklus](Images/Screenshot%202026-11-10%20092652.png)

**Pertanyaan:** apakah skor review akhir mencerminkan KPI, attitude, dan catatan reviewer?

**Insight:** ada tiga masalah di penilaian kinerja:

- **Mid-year selalu lebih tinggi** dari annual di tiga tahun berturut-turut, jadi karyawan yang dinilai di siklus mid-year lebih diuntungkan.
- **Skor akhir tidak nyambung dengan komponennya.** Korelasinya dengan KPI **−0,07** dan dengan attitude 0,04, dua-duanya hampir nol. Ada review dengan skor akhir 4,7 padahal KPI-nya 3,3.
- **Catatan reviewer bertolak belakang dengan skornya.** Catatan "exceed expectation, bonus direkomendasikan" punya rata-rata skor paling rendah (3,02), di bawah catatan "perlu PIP" (3,14).

Skor review belum bisa dipakai untuk keputusan promosi dan bonus. Ini juga menjelaskan kenapa kinerja tidak membedakan siapa yang resign: kalau skornya tidak mencerminkan kinerja, skor itu tidak bisa memisahkan karyawan yang baik dan yang kurang.

### 6. Struktur atasan

![Sebaran jumlah bawahan per manajer](Images/Screenshot%202026-11-10%20092731.png)

**Pertanyaan:** apakah setiap karyawan punya atasan yang jelas, dan apakah beban manajer wajar?

**Insight:** 146 orang tercatat sebagai atasan, tapi rata-rata hanya punya 2,3 bawahan, dan lebih dari sepertiganya (**54 manajer**) cuma punya 1 bawahan. Struktur organisasinya terlalu gemuk di lapisan manajer. Selain itu, **10 karyawan aktif masih tercatat di bawah atasan yang sudah keluar**. Kasus ini paling cepat dibereskan, cukup dengan menunjuk atasan baru dan memperbarui data.

## 💡 Rekomendasi

| Area | Langkah |
|---|---|
| Retensi | Tunda program retensi per departemen. Mulai exit interview dengan pilihan alasan keluar yang terstruktur |
| Angkatan | Pantau angkatan 2022 dan 2023, cek proses onboarding di tahun itu |
| Promosi | Tetapkan kenaikan minimum promosi di atas review tahunan, audit 14 promosi yang gajinya turun |
| Penilaian | Hitung skor akhir dari KPI dan attitude dengan bobot yang jelas, samakan standar mid-year dan annual |
| Struktur | Tunjuk atasan baru untuk 10 karyawan, tinjau ulang tim yang hanya berisi satu orang |

## 🖥️ Dashboard

| Turnover | Pemicu Resign | Kompensasi & Struktur |
|---|---|---|
| ![](Dasboards/Dasboard%20Images/Screenshot%202026-11-10%20092837.png) | ![](Dasboards/Dasboard%20Images/Screenshot%202026-11-10%20092903.png) | ![](Dasboards/Dasboard%20Images/Screenshot%202026-11-10%20093348.png) |

Tiga halaman dengan navigasi tab di bar atas, slicer Divisi, Level jabatan, dan Gender, serta kotak Temuan utama dan Rekomendasi di kolom kanan.

- 📄 [Dashboard versi PDF](Dasboards/Dasboard%20Files/Dasboard%20Adyaraksa.pdf)
- 📑 [Deck presentasi](Presentation%20Deck/Presentation%20Deck%20Adyaraksa.pdf)

## 📁 Struktur Repo
```bash
SQL_PT-Adyaraksa-Data-HR-Payroll/
├── README.md
├── LICENSE
├── Raw Data/               # 5 CSV sumber (employees, departments, attendance, performance, salary_history)
├── Analyst/                # SQL Medallion T-SQL: Bronze → Silver → Gold (19 view analitik)
├── Dasboards/
│   ├── Dasboard Files/     # file Power BI (.pbix) dan versi PDF
│   └── Dasboard Images/    # tampilan 3 halaman dashboard
├── Images/                 # chart per pertanyaan
└── Presentation Deck/      # deck presentasi (PDF)
```

## 🛡️ License

Project ini dilisensikan di bawah [MIT License](LICENSE). Bebas dipakai, dimodifikasi, dan dibagikan dengan atribusi yang sesuai.

## 🌟 About Me

Hai! Aku **Ghifarry Alnaffriza**, mahasiswa Pembangunan Ekonomi Kewilayahan di Sekolah Vokasi UGM yang lagi ngebangun jalan sebagai **data/business analyst**. Aku suka ngulik data buat nemuin cerita di balik angka, kayak project Adyaraksa ini.

Yuk connect! Boleh reach out lewat platform berikut:

[![LinkedIn](https://img.shields.io/badge/LINKEDIN-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/ghifarryalnaffriza)
[![GitHub](https://img.shields.io/badge/GITHUB-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/ghifarryalnaffriza)
[![Instagram](https://img.shields.io/badge/INSTAGRAM-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/ghifaralnaffi_)
[![Email](https://img.shields.io/badge/EMAIL-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:ghifaralns@gmail.com)

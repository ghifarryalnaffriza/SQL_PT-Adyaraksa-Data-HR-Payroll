# 👥 PT Adyaraksa — People Analytics Review

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?logo=linkedin&logoColor=white)](https://linkedin.com/in/ghifarryalnaffriza)
[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white)](https://github.com/ghifarryalnaffriza)
[![Instagram](https://img.shields.io/badge/Instagram-E4405F?logo=instagram&logoColor=white)](https://instagram.com/ghifaralnaffi_)
[![Email](https://img.shields.io/badge/Email-D14836?logo=gmail&logoColor=white)](mailto:ghifaralns@gmail.com)

> HR analytics atas 500 karyawan: turnover 18% ternyata **bukan gejala company-wide** — dan dua mitos HR paling populer (*top performer pada cabut*, *gaji mandek bikin resign*) sama-sama **terbantahkan oleh data**.

## 🎯 Business Problem

PT Adyaraksa ingin tahu kenapa karyawan pergi, tapi program retensi yang dirancang selalu bersifat seluruh perusahaan. Tiga belas pertanyaan lintas fungsi (HR, People, Leadership, Finance, Ops) dibedah untuk menjawab: **apakah masalah retensi ini merata atau terkonsentrasi, dan kebijakan mana yang justru sedang bekerja melawan retensi?**

## 📊 Dataset

- **Sumber:** 5 tabel HR — `employees`, `departments`, `attendance`, `performance`, `salary_history`
- **Volume:** 500 karyawan · 15 departemen · 3.000 record absensi · 600 review · 1.500 baris payroll
- **Periode:** hire cohort 2015–2024 · review 2022–2024 · absensi Q4 2024 · snapshot Januari 2025
- **Cakupan:** 8 kota · 4 level jabatan

## 🏗️ Arsitektur & Pendekatan

Medallion architecture di SQL Server (T-SQL), dipisah ke tiga schema:

- **Bronze** — 5 tabel dimuat apa adanya tanpa transformasi, supaya proses load tidak pernah gagal karena data kotor.
- **Silver** — di sinilah empat masalah data quality ditangani **secara eksplisit dan bisa diaudit**: 3 karyawan ber-`dept_id` NULL dimasukkan grup "Belum di-assign" (bukan dibuang), 3 format tanggal payroll distandarkan ke ISO, 8 duplikat double-entry dihapus via `ROW_NUMBER()`, dan 430 `check_out` NULL dipisah jadi 400 baris sah (absent/leave/holiday) vs 30 yang benar-benar lupa clock-out.
- **Gold** — 20 view analitik yang dipetakan satu-per-satu ke pertanyaan bisnis (`vw_q3_resign_per_dept`, `vw_q8_pay_equity`, `vw_q9_tim_yatim`, `vw_e2_span_of_control`, dst.).

Seluruh analisis kompensasi dibangun di atas tabel bersih **1.492 baris**, bukan 1.500 baris mentah — dengan laporan before/after supaya Finance bisa mengaudit ulang baris demi baris.

## 🛠️ Tools & Tech

![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![T-SQL](https://img.shields.io/badge/T--SQL-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Medallion](https://img.shields.io/badge/Medallion_Architecture-4B8BBE?style=for-the-badge&logo=databricks&logoColor=white)

## 🔍 Key Findings

- **Turnover 18% terkonsentrasi, bukan merata:** Design **23,3%** dan QA 19,4% vs Sales 7,1% — selisih lebih dari 3× antar ujung.
- **Mitos "top performer pada cabut" terbantahkan:** resign rate skor ≥4 = **15,5%** vs karyawan lain 14,7% — selisih 0,8 poin.
- **Mitos "gaji mandek bikin resign" juga terbantahkan:** karyawan ber-growth terbawah resign **13,9%** vs lainnya 14,0% — praktis identik.
- **Akar masalahnya periode hiring:** cohort 2022 (**70,4%** masih aktif) dan 2023 (68,4%) jauh di bawah cohort lain (75–90%).
- **Promosi tidak dibayar sebagai promosi:** kenaikan `promotion` **8,3%** justru di bawah `annual_review` **8,8%** — tanggung jawab naik, gaji naik biasa saja.
- **Gap gender menumpuk di tengah-atas:** Senior **+13,6%** dan Leadership +9,2%, sementara Staff (−0,5%) dan Manager (−4,8%) praktis setara.
- **Skor review terdistorsi pola musiman:** mid-year selalu lebih murah hati (3,33–3,35) dibanding annual (3,15–3,24) di ketiga tahun — promosi bisa bias ke siklus mid-year.
- **10 karyawan tanpa atasan:** masih menunjuk manajer yang sudah resign — perbaikan tercepat dan gratis di seluruh analisis.
- **Struktur terlalu datar:** rata-rata span of control **2,3 bawahan**, maksimum 6, dan **0 manajer** dengan 10+ bawahan.
- **Security bermasalah ganda:** satu-satunya departemen di dua daftar merah — late rate **16,0%** (tertinggi) dan absent 12,0% (nomor dua).
- **Tenure bukan alat ukur retensi:** korelasi tenure × resign hanya **r = 0,18** — Design punya tenure terpanjang (5,9 th) sekaligus resign tertinggi.

## 📁 Struktur Repo
```bash
adyaraksa-hr-people-analytics/
├── README.md
├── raw_data/ # 5 CSV sumber (employees, departments, attendance, performance, salary_history)
├── Analyst/ # SQL Medallion T-SQL: bronze → silver → gold (20 view analitik)
├── Visualisasi/ # dashboard 7 halaman (export PDF)
└── Presentation Deck/ # storyline deck 15 slide (PDF)
```

## 🛡️ License

Project ini dilisensikan di bawah [MIT License](LICENSE). Bebas dipakai, dimodifikasi, dan dibagikan dengan atribusi yang sesuai.

## 🌟 About Me

Hai! Aku **Ghifarry Alnaffriza**, mahasiswa Pembangunan Ekonomi Kewilayahan di Sekolah Vokasi UGM yang lagi ngebangun jalan sebagai **data/business analyst**. Aku suka ngulik data buat nemuin cerita di balik angka — kayak project Adyaraksa ini.

Yuk connect! Boleh reach out lewat platform berikut:

[![LinkedIn](https://img.shields.io/badge/LINKEDIN-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/ghifarryalnaffriza)
[![GitHub](https://img.shields.io/badge/GITHUB-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/ghifarryalnaffriza)
[![Instagram](https://img.shields.io/badge/INSTAGRAM-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/ghifaralnaffi_)
[![Email](https://img.shields.io/badge/EMAIL-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:ghifaralns@gmail.com)

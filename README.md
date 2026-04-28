1. /all-student
<img width="1507" height="664" alt="Screenshot 2026-04-28 161931" src="https://github.com/user-attachments/assets/64b6b6af-c278-484b-b78b-2e68012d8c5f" />
<img width="1632" height="267" alt="Screenshot 2026-04-28 162746" src="https://github.com/user-attachments/assets/b33e5bac-6666-4a50-b31b-85cb596012b7" />

2. /all-student-name
<img width="1919" height="642" alt="image" src="https://github.com/user-attachments/assets/7fab47ea-6b4b-4a5c-84fd-f6feef98a585" />
<img width="1654" height="384" alt="image" src="https://github.com/user-attachments/assets/ab8f6bee-a4d7-4c9a-a419-456c8e2c1f7b" />

3. /highest-gpa
<img width="1919" height="897" alt="image" src="https://github.com/user-attachments/assets/a8ab4f81-1c89-4909-9cc9-9df6798bec21" />
<img width="1647" height="374" alt="image" src="https://github.com/user-attachments/assets/4b3558b3-10bb-4ccf-9e40-ccd89847e486" />

# Setelah Optimize
1. /all-student
<img width="1919" height="1015" alt="image" src="https://github.com/user-attachments/assets/936e61a6-313c-4de5-877e-7a6864f86143" />
- Rata-rata sample time berkurang dari yang awalnya 84642 ms menjadi 6039 ms
- Terdapat peningkatan sebesar 93%
Performa awal sangat lambat akibat masalah **N+1 Query** dan *nested loop* yang mengalokasikan banyak memori untuk objek `StudentCourse` baru. Setelah optimasi, pengambilan data relasi diubah menjadi satu pemanggilan `findAll()` langsung ke tabel *mapping*, yang menurunkan beban I/O database dan meringankan kerja *Garbage Collector* secara drastis.

3. /all-student-name
<img width="1919" height="1017" alt="image" src="https://github.com/user-attachments/assets/09886af1-563b-41f5-9d77-1d405722bb01" />
- Rata-rata sample time berkurang dari yang awalnya 3601 ms menjadi 140 ms
- Terdapat peningkatan sebesar 96%
Beban CPU sebelumnya memuncak karena penggunaan operator `+=` untuk menggabungkan `String` di dalam *loop* 2000 data, yang secara terus-menerus membuang dan membuat objek `String` baru di memori. Optimasi dilakukan dengan menerapkan **Java Stream API (`Collectors.joining`)** yang menggunakan `StringBuilder` di belakang layar, sehingga penggunaan memori jauh lebih efisien dan *Garbage Collector* tidak kewalahan.

5. /highest-gpa
<img width="1919" height="1016" alt="image" src="https://github.com/user-attachments/assets/33422beb-1e34-43bd-a2fd-55e3b86d129e" />
- Rata-rata sample time berkurang dari yang awalnya 92 ms menjadi 8 ms
- Terdapat peningkatan sebesar 91%
Sebelumnya, aplikasi memuat seluruh data mahasiswa ke dalam memori aplikasi (RAM) hanya untuk mencari satu nilai tertinggi menggunakan iterasi manual. Performa berhasil dioptimasi dengan mendelegasikan proses pengurutan (*sorting*) dan pencarian ke database menggunakan *custom query* (`findFirstByOrderByGpaDesc()`). Hal ini membuat transfer data dari database ke aplikasi menjadi sangat kecil (hanya 1 baris objek).

# Refleksi
1.
- JMeter melakukan pengujian performa dari luar (black-box testing). Ia mensimulasikan banyak pengguna melalui Thread Grouop yang mengakses aplikasi secara bersamaan untuk mengukur metrik eksternal seperti response time, throughput dan batas beban server.
- IntelliJ Profiler melakukan analisis dari dalam (white-box testing). Ia memantau apa yang terjadi dalam JVM saat aplikasi memproses request, dengan mengukur metrik seperti CPU time, alokasi memori dan durasi eksekusi masing-masing method.

2. Profiler memberikan pemetaan data yang sangat detail mengenai jalannya aplikasi melalui visualisasi seperti Flame Graph dan Call Tree. Profiler menyoroti method yang menghabiskan waktu komputasi paling lama secara akurat, sehingga masalah bisa langsung ditemukan.

3. Ya, IntelliJ Profiler sangat efektif, karena integrasinya langsung dengan IDE. Ketika kita menemukan lonjakan CPU pada grafik profiler, kita bisa klik nama method tersebut dan lompat ke baris kodenya secara langsung.

4. Tantangannya adalah proses seeding yang memakan waktu lama. Cara mengatasinya adalah dengan mengatur jumlah data uji agar optimal.

5. Manfaat utama menggunakan IntelliJ Profiler adalah kemudahan mendeteksi inefisiensi algoritma dan pemberian bukti data yang kuantitatif mengenai performa kode.

6. Cara mengatasi inkonsistensi adalah dengan menggunakan JMeter untuk mendeteksi endpoint mana yang memiliki gejala lambat bagi end-user, lalu gunakan Profiler di lingkungan lokal untuk mengisolasi dan mendiagnosis murni kinerja logika kodenya.

7.
- Strategi yang dilakukan meliputi: menggunakan custom query untuk menghindari N+1 query dan melakukan sorting data serta menggunakan tipe data yang tepat seperti Stream API bukan operator += pada String.
- Untuk memastikan perubahan ini tidak merusak fungsionalitas aplikasi, langkah yang harus dilakukan adalah verifikasi bahwa response body yang dikembalikan oleh API setelah refaktor tetap persis sama dengan sebelumnya.



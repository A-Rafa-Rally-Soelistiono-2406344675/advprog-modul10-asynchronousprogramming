# Module 10 - Asynchronous Programming

## Experiment 1.2: Understanding How It Works

### Execution Result

```text
Rafa's Komputer: hey hey!
Rafa's Komputer: howdy!
Rafa's Komputer: done!
```

### Explanation

Output `Rafa's Komputer: hey hey!` muncul terlebih dahulu karena baris tersebut berada di fungsi `main` setelah pemanggilan `spawner.spawn(...)`, tetapi sebelum `executor.run()`. Pemanggilan `spawn` hanya memasukkan task async ke dalam antrean executor; task tersebut belum langsung dijalankan sampai executor mulai berjalan.

Setelah `executor.run()` dipanggil, executor mengambil task dari antrean dan mulai melakukan polling terhadap future. Pada polling pertama, task mencetak `Rafa's Komputer: howdy!`, lalu menunggu `TimerFuture` selama dua detik. Ketika timer selesai, waker memasukkan kembali task ke antrean agar executor dapat melanjutkannya. Setelah task dipoll lagi, program mencetak `Rafa's Komputer: done!`.

Jadi urutannya menunjukkan bahwa kode sinkron di `main` berjalan lebih dulu, sedangkan kode di dalam async block baru berjalan ketika executor mulai menjalankan task yang sudah di-spawn.

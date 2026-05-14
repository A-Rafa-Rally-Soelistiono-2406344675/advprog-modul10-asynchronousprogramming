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

## Experiment 1.3: Multiple Spawn and Removing Drop

### Multiple Spawn Result

```text
Rafa's Komputer: hey hey!
Rafa's Komputer: howdy!
Rafa's Komputer: howdy2!
Rafa's Komputer: howdy3!
Rafa's Komputer: done!
Rafa's Komputer: done2!
Rafa's Komputer: done3!
```

### Removing `drop(spawner)` Result

```text
still_running=True
Rafa's Komputer: hey hey!
Rafa's Komputer: howdy!
Rafa's Komputer: howdy2!
Rafa's Komputer: howdy3!
Rafa's Komputer: done!
Rafa's Komputer: done2!
Rafa's Komputer: done3!
```

### Explanation

Pada eksperimen multiple spawn, terdapat tiga task async yang dimasukkan ke antrean executor. Baris `Rafa's Komputer: hey hey!` tetap muncul paling awal karena baris tersebut adalah kode sinkron di fungsi `main` dan dieksekusi sebelum `executor.run()` mulai menjalankan task dari antrean.

Setelah executor berjalan, task diproses satu per satu. Karena setiap task mencetak pesan `howdy`, lalu melakukan `await` pada `TimerFuture` selama dua detik, executor akan menaruh kembali task yang belum selesai dan melanjutkan task lain. Itulah sebabnya semua pesan `howdy`, `howdy2`, dan `howdy3` muncul terlebih dahulu sebelum pesan `done`, `done2`, dan `done3`.

Ketika `drop(spawner)` dihapus, semua pesan tetap dapat muncul, tetapi program tidak berhenti. Hal ini terjadi karena executor menjalankan loop `while let Ok(task) = self.ready_queue.recv()`. Fungsi `recv()` hanya mengembalikan error jika semua sender pada channel sudah ditutup. Jika `spawner` tidak di-drop, masih ada sender yang hidup, sehingga executor terus menunggu task baru walaupun semua task yang ada sudah selesai.

Dengan mengembalikan `drop(spawner)`, sender utama ditutup setelah semua task selesai di-spawn. Setelah antrean kosong dan tidak ada sender lagi, `recv()` mengembalikan error, loop executor berhenti, dan program selesai secara normal.

### Tugas 7
<hr>

### Jelaskan bagaimana cara kamu mengimplementasikan checklist di atas secara step-by-step (bukan hanya sekadar mengikuti tutorial)
- [x] Membuat sebuah program Flutter baru dengan tema inventory seperti tugas-tugas sebelumnya. <br>
  * Buka Command Prompt atau terminal pada folder tempat proyek flutter akan dibuat.
  * Buatlah proyek prezzmarket dengan menggunakan command berikut
  ```
    flutter create prezzmarket
    cd prezzmarket
  ```
  * Merapikan struktur proyek
    * Membuat file `menu.dart` tambahkan import:
    ```
    import 'package:flutter/material.dart';
    ```
    * Dari file `main.dart`, pindahkan kode baris ke-39 ingga akhir yang berisi kedua class berikut
    ```
        class MyHomePage ... {
            ...
        }

        class _MyHomePageState ... {
            ...
        }
    ```
    * tambahkan import sepert berikut pada `main.dart`
    ```
    import 'package:prezzmarket/menu.dart';
    ```
  * Membuat Widget Sederhana
    * ubah kode pada `main.dart` dibagian tema aplikasi kamu yang mempunyai tipe `Material Color`, ubah temanya menjadi indigo
    ```
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),

    ```
    * Pada file `main.dart` ubah `home: const MyHomePage(title: 'Flutter Demo Home Page'),` menjadi `home: MyHomePage(),` 
    * Pada file `menu.dart`, ubah sifat widget menjadi stateless dengan melakukan perubahan pada ({super.key, required this.title}) menjadi ({Key? key}) : super(key: key); dan menghapus final String title; sampai bawah serta menambahkan Widget Build sehingga terlihat seperti berikut.
    ```
    class MyHomePage extends StatelessWidget {
        MyHomePage({Key? key}) : super(key: key);

        @override
        Widget build(BuildContext context) {
            return Scaffold(
                ...
            );
        }
    }
    ```
    * membuat define tipe 
        
    ```
    class ShopItem {
        final String name;
        final IconData icon;
        final Color color

        ShopItem(this.name, this.icon, this.color);
    }
    ```
    * Lalu dibawah kode `MyHomePage({Key? key}) : super(key: key);`,tambahkan barang-barang yang dijual (nama, harga, dan icon barang tersebut).
    ```
    final List<ShopItem> items = [
        ShopItem("Lihat Produk", Icons.checklist),
        ShopItem("Tambah Produk", Icons.add_shopping_cart),
        ShopItem("Logout", Icons.logout),
    ];
    ```


  * Selanjutnya tambahkan kode berikut didalam Widget build:
  ```
       return Scaffold(
        appBar: AppBar(
          title: const Text(
            'PrezzMarket',
            ),
            backgroundColor: Theme.of(context).colorScheme.primary,
            ),
            body: SingleChildScrollView(
            // Widget wrapper yang dapat discroll
            child: Padding(
                padding: const EdgeInsets.all(10.0), // Set padding dari halaman
                child: Column(
                // Widget untuk menampilkan children secara vertikal
                children: <Widget>[
                    const Padding(
                    padding: EdgeInsets.only(top: 10.0, bottom: 10.0),
                    // Widget Text untuk menampilkan tulisan dengan alignment center dan style yang sesuai
                    child: Text(
                        'PrezzMarket', // Text yang menandakan toko
                        textAlign: TextAlign.center,
                        style: TextStyle(
                        fontSize: 30,
                        fontWeight: FontWeight.bold,
                        ),
                    ),
                    ),
                    // Grid layout
                    GridView.count(
                    // Container pada card kita.
                    primary: true,
                    padding: const EdgeInsets.all(20),
                    crossAxisSpacing: 10,
                    mainAxisSpacing: 10,
                    crossAxisCount: 3,
                    shrinkWrap: true,
                    children: items.map((ShopItem item) {
                        // Iterasi untuk setiap item
                        return ShopCard(item);
                    }).toList(),
                    ),
                ],
                ),
            ),
            ),
        );
  ```

  + Tambahkan juga widget stateless baru untuk menampilkan card:
  ```
  class ShopCard extends StatelessWidget {
    final ShopItem item;

    const ShopCard(this.item, {super.key}); // Constructor

    @override
    Widget build(BuildContext context) {
      return Material(
        color: item.color,
        child: InkWell(
          // Area responsive terhadap sentuhan
          onTap: () {
            // Memunculkan SnackBar ketika diklik
            ScaffoldMessenger.of(context)
              ..hideCurrentSnackBar()
              ..showSnackBar(SnackBar(
                  content: Text("Kamu telah menekan tombol ${item.name}!")));
          },
          child: Container(
            // Container untuk menyimpan Icon dan Text
            padding: const EdgeInsets.all(8),
            child: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Icon(
                    item.icon,
                    color: Colors.white,
                    size: 30.0,
                  ),
                  const Padding(padding: EdgeInsets.all(3)),
                  Text(
                    item.name,
                    textAlign: TextAlign.center,
                    style: const TextStyle(color: Colors.white),
                  ),
                ],
              ),
            ),
          ),
        ),
      );
    }
  }
  ```

- [x] Membuat tiga tombol sederhana dengan ikon dan teks untuk: <br>
  - [x] Melihat daftar item (`Lihat Item`) <br>
  - [x] Menambah item (`Tambah Item`) <br>
  - [x] Logout (`Logout`) <br>
    
      * Tambahkan kode berikut dibawah kode `MyHomePage({Key? key}) : super(key: key);`:
      ```
          final List<ShopItem> items = [
          ShopItem("Lihat Item", Icons.checklist, Colors.red),
          ShopItem("Tambah Item", Icons.add_shopping_cart, Colors.blue),
          ShopItem("Logout", Icons.logout, Colors.green),
          ];
      ```

- [x] Memunculkan `Snackbar` dengan tulisan: <br>
  - [x] "Kamu telah menekan tombol Lihat Item" ketika tombol `Lihat Item` ditekan. <br>
  - [x] "Kamu telah menekan tombol Tambah Item" ketika tombol `Tambah Item` ditekan. <br>
  - [x] "Kamu telah menekan tombol Logout" ketika tombol `Logout` ditekan. <br>

      * Pada widget build, terdapat kode seperti dibawah. Nantinya ketika tombol-tombol diatas diklik, muncul pesan "kamu telah menekan tombol ..."
      ```
              onTap: () {
            // Memunculkan SnackBar ketika diklik
            ScaffoldMessenger.of(context)
              ..hideCurrentSnackBar()
              ..showSnackBar(SnackBar(
                  content: Text("Kamu telah menekan tombol ${item.name}!")));
          },
      ```

- [x] Menjawab beberapa pertanyaan berikut pada `README.md` pada root folder.
    - [x] Apa perbedaan utama antara stateless dan stateful widget dalam konteks pengembangan aplikasi Flutter?
        * Stateless stateless:
            * Widget stateless tidak dapat mengubah keadaannya selama runtime aplikasi Flutter.
            * Widget ini bersifat statis, artinya tidak akan pernah berubah setelah dibuat
            * Hanya diperbarui saat pertama kali diinisialisasi.
            * Tidak dapat diperbarui selama runtime kecuali terjadi suatu peristiwa eksternal.
            * Tidak memiliki metode setState(). Akan dirender sekali dan tidak akan memperbarui dirinya sendiri.
            * Contoh: Text, Icons, Raised Buttons, dll.
        * Stateful Widget :
            * Widget stateful digunakan ketika sebagian UI harus berubah secara dinamis selama runtime.
            * Widget ini bersifat dinamis.
            * Dapat berubah atau diperbarui secara dinamis.
            * Dapat diperbarui selama runtime berdasarkan tindakan pengguna atau perubahan data.
            * Memiliki metode setState() internal dan dapat di-render ulang jika data masukan berubah.
            * Contoh: Checkboxes, Radio buttons, dll.
    - [x] Sebutkan seluruh widget yang kamu gunakan untuk menyelesaikan tugas ini dan jelaskan fungsinya masing-masing.
        * `MyHomePage` StatelessWidget: Merupakan komponen utama yang menggambarkan halaman beranda dalam aplikasi. Widget ini menampilkan antarmuka utama yang memuat daftar item toko.
        * `AppBar`: menampilkan bilah aplikasi di bagian atas halaman
        * `Scaffold`: menyediakan skeleton (kerangka) kerja dasar untuk halaman, berisi AppBar dan body
        * `ColorScheme` : mengatur palet warna aplikasi
        * `ScaffoldMessenger`: menampilkan pesan snack bar saat salah satu tombol item diklik
        * `Text`: menampilkan teks `prezzmarket`
        * `SnackBar`: pesan pop up yang muncul ketika tombol item ditekan
        * `SingleChildScrollView`: berfungsi melingkupi/membungkus semua isi halaman, memungkinkan pengguna untuk melakukan scroll jika kontennya melebihi ukuran layar.
        * `Padding`: memberikan padding ke dalam konten yang berisi judul dan daftar item toko.
        * `Column`: digunakan untuk menampilkan anak-anak (children) secara vertikal.
        * `GridView.count`: membuat tata letak grid dengan jumlah kolom yang diberikan.
        * `ShopItem`: kelas yang mendefinisikan item toko (nama,ikon,warna)
        * `ShopCard`: mewakili card yang menampilkan item toko
        * `Material`: digunakan untuk mengatur latar belakang warna kartu.
        * `InkWell`: digunakan untuk memberikan area responsif terhadap sentuhan sehingga item toko bisa ditekan.
        * `Container`: digunakan untuk mengelompokkan ikon dan teks dalam kartu.
        * `Icon`: menampilkan ikon dalam card
        * `Text`: menampilkan nama item toko dalam kartu.
    - [x] jelaskan bagaimana cara kamu mengimplementasikan checklist di atas secara step-by-step (bukan hanya sekadar mengikuti tutorial)
-[x] Melakukan `add`-`commit`-`push` ke GitHub.
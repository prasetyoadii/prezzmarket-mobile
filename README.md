### Tugas 9
## Tugas 9

- Memastikan deployment proyek tugas Django kamu telah berjalan dengan baik.
- Membuat halaman login pada proyek tugas Flutter. 
    Pada lib/screens buat file baru login.dart, tambahkan kode berikut
    ```
    import 'package:prezzmarket/screens/menu.dart';
    import 'package:flutter/material.dart';
    import 'package:pbp_django_auth/pbp_django_auth.dart';
    import 'package:provider/provider.dart';
    
    void main() {
      runApp(const LoginApp());
    }
    
    class LoginApp extends StatelessWidget {
      const LoginApp({super.key});
    
      @override
      Widget build(BuildContext context) {
        return MaterialApp(
          title: 'Login',
          theme: ThemeData(
            primarySwatch: Colors.blue,
          ),
          home: const LoginPage(),
        );
      }
    }
    
    class LoginPage extends StatefulWidget {
      const LoginPage({super.key});
    
      @override
      _LoginPageState createState() => _LoginPageState();
    }
    
    class _LoginPageState extends State<LoginPage> {
      final TextEditingController _usernameController = TextEditingController();
      final TextEditingController _passwordController = TextEditingController();
    
      @override
      Widget build(BuildContext context) {
        final request = context.watch<CookieRequest>();
        return Scaffold(
          appBar: AppBar(
            title: const Text('Login'),
          ),
          body: Container(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                TextField(
                  controller: _usernameController,
                  decoration: const InputDecoration(
                    labelText: 'Username',
                  ),
                ),
                const SizedBox(height: 12.0),
                TextField(
                  controller: _passwordController,
                  decoration: const InputDecoration(
                    labelText: 'Password',
                  ),
                  obscureText: true,
                ),
                const SizedBox(height: 24.0),
                ElevatedButton(
                  onPressed: () async {
                    String username = _usernameController.text;
                    String password = _passwordController.text;
    
                    // Cek kredensial
                    // TODO: Ganti URL dan jangan lupa tambahkan trailing slash (/) di akhir URL!
                    // Untuk menyambungkan Android emulator dengan Django pada localhost,
                    // gunakan URL http://10.0.2.2/
                    final response = await request.login("http://localhost:8000/auth/login/", {
                      'username': username,
                      'password': password,
                    });
    
                    if (request.loggedIn) {
                      String message = response['message'];
                      String uname = response['username'];
                      Navigator.pushReplacement(
                        context,
                        MaterialPageRoute(builder: (context) => MyHomePage()),
                      );
                      ScaffoldMessenger.of(context)
                        ..hideCurrentSnackBar()
                        ..showSnackBar(
                            SnackBar(content: Text("$message Selamat datang, $uname.")));
                    } else {
                      showDialog(
                        context: context,
                        builder: (context) => AlertDialog(
                          title: const Text('Login Gagal'),
                          content:
                          Text(response['message']),
                          actions: [
                            TextButton(
                              child: const Text('OK'),
                              onPressed: () {
                                Navigator.pop(context);
                              },
                            ),
                          ],
                        ),
                      );
                    }
                  },
                  child: const Text('Login'),
                ),
              ],
            ),
          ),
        );
      }
    }
    ```

- Mengintegrasikan sistem autentikasi Django dengan proyek tugas Flutter.
   * membuat sebuah aplikasi Django bernama `authentication` di dalam proyek prezmarket. Pastikan untuk menambahkan `authentication` di dalam daftar `INSTALLED_APPS` pada file settings.py utama proyek. Lanjutkan dengan menginstal library yang diperlukan, yaitu `django-cors-headers`, menggunakan perintah `pip`. Setelah instalasi selesai, tambahkan `corsheaders` ke dalam `INSTALLED_APPS` juga. Selanjutnya, tambahkan middleware `corsheaders.middleware.CorsMiddleware` ke dalam daftar middleware di file `settings.py` utama proyek. Pada `settings.py` dalam direktori `prezzmarket` tambahkan kode berikut
    ```
    CORS_ALLOW_ALL_ORIGINS = True
    CORS_ALLOW_CREDENTIALS = True
    CSRF_COOKIE_SECURE = True
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SAMESITE = 'None'
    SESSION_COOKIE_SAMESITE = 'None'
    ```
    * Buat fungsi `login`pada `views.py` dalam direktori `authentication`, tambahkan kode berikut
      ```
      from django.shortcuts import render
      from django.contrib.auth import authenticate, login as auth_login
      from django.http import JsonResponse
      from django.views.decorators.csrf import csrf_exempt

      @csrf_exempt
      def login(request):
          username = request.POST['username']
          password = request.POST['password']
          user = authenticate(username=username, password=password)
          if user is not None:
              if user.is_active:
                  auth_login(request, user)
                  # Status login sukses.
                  return JsonResponse({
                      "username": user.username,
                      "status": True,
                      "message": "Login sukses!"
                      # Tambahkan data lainnya jika ingin mengirim data ke Flutter.
                  }, status=200)
              else:
                  return JsonResponse({
                      "status": False,
                      "message": "Login gagal, akun dinonaktifkan."
                  }, status=401)

          else:
              return JsonResponse({
                  "status": False,
                  "message": "Login gagal, periksa kembali email atau kata sandi."
              }, status=401)
      ```
    * tambahkan file `urls.py` dalam direktori tersebut, dan tambahkan kode berikut
      ```
      from django.urls import path
      from authentication.views import login

      app_name = 'authentication'

      urlpatterns = [
          path('login/', login, name='login'),
      ]
      ```
    * Terakhir, tambahkan ` path('auth/', include('authentication.urls'))`, pada file `prezzmarket/urls.py`.
    

    * Install package berikut ini pada projek Flutter:
      ```
      flutter pub add provider
      flutter pub add pbp_django_auth
      ```
    * Lalu ubah class `MyApp` pada `main.dart` menjadi:
      ```
      class MyApp extends StatelessWidget {
        const MyApp({Key? key}) : super(key: key);
      
        @override
          Widget build(BuildContext context) {
            return Provider(
              create: (_) {
                CookieRequest request = CookieRequest();
                return request;
              },
              child: MaterialApp(
                  title: 'Flutter App',
                  theme: ThemeData(
                    colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
                    useMaterial3: true,
                  ),
                  home: LoginPage()),
            );
          }
      }
      ```

- Membuat model kustom sesuai dengan proyek aplikasi Django. 

  Buka endpoint JSON yang telah dibuat sebelumnya dalam tugsa sebelumnya. Salin data JSON yang ada di sana dan akses situs web Quicktype. Pada Quicktype, ubah konfigurasi nama setup menjadi "Item", jenis sumber menjadi JSON, dan bahasa menjadi Dart. Tempelkan data JSON yang sudah salin ke dalam kolom teks yang disediakan di Quicktype. Selanjutnya, pilih opsi "Copy Code" pada Quicktype untuk mendapatkan kode yang dihasilkan sesuai dengan konfigurasi yang telah tentukan sebelumnya.
  ```
    // To parse this JSON data, do
    //
    //     final item = itemFromJson(jsonString);

    import 'dart:convert';

    List<Item> itemFromJson(String str) => List<Item>.from(json.decode(str).map((x) => Item.fromJson(x)));

    String itemToJson(List<Item> data) => json.encode(List<dynamic>.from(data.map((x) => x.toJson())));

    class Item {
        String model;
        int pk;
        Fields fields;

        Item({
            required this.model,
            required this.pk,
            required this.fields,
        });

        factory Item.fromJson(Map<String, dynamic> json) => Item(
            model: json["model"],
            pk: json["pk"],
            fields: Fields.fromJson(json["fields"]),
        );

        Map<String, dynamic> toJson() => {
            "model": model,
            "pk": pk,
            "fields": fields.toJson(),
        };
    }

    class Fields {
        int user;
        String name;
        int amount;
        String description;

        Fields({
            required this.user,
            required this.name,
            required this.amount,
            required this.description,
        });

        factory Fields.fromJson(Map<String, dynamic> json) => Fields(
            user: json["user"],
            name: json["name"],
            amount: json["amount"],
            description: json["description"],
        );

        Map<String, dynamic> toJson() => {
            "user": user,
            "name": name,
            "amount": amount,
            "description": description,
        };
    }

  ```

- Membuat halaman yang berisi daftar semua item yang terdapat pada endpoint JSON di Django yang telah kamu deploy. 
        
    Pada projek Flutter buat file `list_product.dart` lalu isi dengan kode:
    ```
    import 'package:flutter/material.dart';
    import 'package:http/http.dart' as http;
    import 'dart:convert';
    import 'package:prezzmarket/models/product.dart';
    import 'package:prezzmarket/screens/detail_product.dart';
    import 'package:prezzmarket/widgets/left_drawer.dart';
    
    class ProductPage extends StatefulWidget {
      const ProductPage({Key? key}) : super(key: key);
    
      @override
      _ProductPageState createState() => _ProductPageState();
    }
    
    class _ProductPageState extends State<ProductPage> {
      Future<List<Product>> fetchProduct() async {
        // TODO: Ganti URL dan jangan lupa tambahkan trailing slash (/) di akhir URL!
        var url = Uri.parse(
            'http://localhost:8000/json/');
        var response = await http.get(
          url,
          headers: {"Content-Type": "application/json"},
        );
    
        // melakukan decode response menjadi bentuk json
        var data = jsonDecode(utf8.decode(response.bodyBytes));
    
        // melakukan konversi data json menjadi object Product
        List<Product> list_product = [];
        for (var d in data) {
          if (d != null) {
            list_product.add(Product.fromJson(d));
          }
        }
        return list_product;
      }
    }
    ```
    
  - Tampilkan name, amount, dan description dari masing-masing item pada halaman ini.
    - pada lib/models buka file list_item.dart dan tambahkan kode berikut
      ```
        ...
        @override
        Widget build(BuildContext context) {
            return Scaffold(
                appBar: AppBar(
                title: const Text('Product'),
                ),
                drawer: const LeftDrawer(),
                body: FutureBuilder(
                    future: fetchProduct(),
                    builder: (context, AsyncSnapshot snapshot) {
                        if (snapshot.data == null) {
                            return const Center(child: CircularProgressIndicator());
                        } else {
                            if (!snapshot.hasData) {
                            return const Column(
                                children: [
                                Text(
                                    "Tidak ada data produk.",
                                    style:
                                        TextStyle(color: Color(0xff59A5D8), fontSize: 20),
                                ),
                                SizedBox(height: 8),
                                ],
                            );
                        } else {
                            return ListView.builder(
                                itemCount: snapshot.data!.length,
                                itemBuilder: (_, index) => Container(
                                        margin: const EdgeInsets.symmetric(
                                            horizontal: 16, vertical: 12),
                                        padding: const EdgeInsets.all(20.0),
                                        child: Column(
                                        mainAxisAlignment: MainAxisAlignment.start,
                                        crossAxisAlignment: CrossAxisAlignment.start,
                                        children: [
                                            Text(
                                            "${snapshot.data![index].fields.name}",
                                            style: const TextStyle(
                                                fontSize: 18.0,
                                                fontWeight: FontWeight.bold,
                                            ),
                                            ),
                                            const SizedBox(height: 10),
                                            Text("${snapshot.data![index].fields.amount}"),
                                            const SizedBox(height: 10),
                                            Text(
                                                "${snapshot.data![index].fields.description}"),
                                            ElevatedButton(
                                                onPressed: () {
                                                Navigator.push(
                                                  context,
                                                  MaterialPageRoute(
                                                    builder: (context) => DetailItemPage(item: snapshot.data![index]),
                                                  ),
                                                );
                                              },
                                              child: const Text('Detail Item'),
                                            ),
                                        ],
                                        ),
                                    ));
                            }
                        }
                    }));
        }
      ```

- Membuat halaman detail untuk setiap item yang terdapat pada halaman daftar Item. 
  - Halaman ini dapat diakses dengan menekan salah satu item pada halaman daftar Item.
  - Tampilkan seluruh atribut pada model item kamu pada halaman ini. 
  - Tambahkan tombol untuk kembali ke halaman daftar item.

  Pada `lib/screens`, buat file `detail_item.dart` dan tambahkan kode berikut:
    ```
    import 'package:flutter/material.dart';
    import 'package:prezzmarket/models/item.dart';
    import 'package:prezzmarket/widgets/left_drawer.dart';

    class DetailItemPage extends StatefulWidget {
        final Item item;

        const DetailItemPage({super.key, required this.item});

        @override
        State<DetailItemPage> createState() => _DetailItemPageState();
    }

    class _DetailItemPageState extends State<DetailItemPage> {
      @override
      Widget build(BuildContext context) {
        return Scaffold(
              appBar: AppBar(
                title: const Center(
                  child: Text(
                    'Detail Item',
                  ),
                ),
                backgroundColor: Colors.indigo,
                foregroundColor: Colors.white,
              ),
          drawer: const LeftDrawer(),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              crossAxisAlignment: CrossAxisAlignment.center,
              children: [
                const Text(
                  'Item Name:',
                  style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
                ),
                Text(
                  widget.item.fields.name,
                  style: const TextStyle(fontSize: 36, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 20),
                Text(
                  'Amount: ${widget.item.fields.amount}',
                  style: const TextStyle(fontSize: 18),
                ),
                Text(
                  'Description: ${widget.item.fields.description}',
                  style: const TextStyle(fontSize: 18),
                ),
                ElevatedButton(
                  onPressed: () {
                    // Navigasi kembali ketika tombol ditekan
                    Navigator.pop(context);
                  },
                  child: const Text('Kembali'),
                ),            
              ],
            ),
          ),
        );
      }
    }
    ```
- Apakah bisa kita melakukan pengambilan data JSON tanpa membuat model terlebih dahulu? Jika iya, apakah hal tersebut lebih baik daripada membuat model sebelum melakukan pengambilan data JSON? 
  Ya, bisa melakukan pengambilan data JSON tanpa membuat model terlebih dahulu. bisa langsung menggunakan data JSON yang diterima sebagai objek dinamis dalam Dart. Namun, meski bisa dilakukan, metode ini tidak disarankan kecuali memiliki alasan khusus untuk tidak membuat model. Ada beberapa alasan mengapa membuat model lebih baik:
  * Struktur Data: Dengan membuat model, dapat memastikan bahwa data memiliki struktur yang diinginkan. Jika menggunakan data JSON secara langsung, harus berhati-hati untuk memastikan bahwa data JSON memiliki struktur yang harapkan, dan bahwa data tersebut dapat diinterpretasikan dengan benar.
  * Type Safety: Dengan membuat model, dapat memastikan bahwa data memiliki tipe yang benar. Dart adalah bahasa yang statis, yang berarti bahwa tipe data harus diketahui pada compile time. Jika menggunakan data JSON secara langsung, mungkin perlu melakukan type checking secara manual, yang bisa menjadi rumit dan berisiko.
  * Keterbacaan dan Pemeliharaan: Dengan membuat model, kode akan lebih mudah dibaca dan dipemelihara. Model dapat memberikan dokumentasi tentang struktur data yang diharapkan, dan dapat membantu pengembang lain memahami struktur data 

- Jelaskan fungsi dari CookieRequest dan jelaskan mengapa instance CookieRequest perlu untuk dibagikan ke semua komponen di aplikasi Flutter. 
  CookieRequest dalam konteks Flutter biasanya merujuk pada mekanisme di mana aplikasi Flutter mengirimkan atau menerima cookies saat membuat permintaan HTTP. Cookies ini biasanya digunakan untuk melacak sesi pengguna atau menyimpan informasi spesifik tentang penggunaan aplikasi oleh pengguna. Misalnya, saat aplikasi mengirimkan permintaan GET atau POST, ia dapat menyertakan cookies dalam header permintaan. Respon dari server juga dapat menyertakan cookies yang dapat disimpan dan digunakan untuk permintaan berikutnya.

  Instance CookieRequest perlu dibagikan ke semua komponen di aplikasi Flutter untuk beberapa alasan:
  * Konsistensi data: Jika aplikasi menggunakan cookies untuk melacak data pengguna, seperti sesi login atau preferensi pengguna, perlu memastikan bahwa cookies yang sama digunakan di seluruh aplikasi. Dengan membagikan instance CookieRequest, dapat memastikan bahwa semua komponen aplikasi menggunakan data yang sama.
  * Pengelolaan sumber daya: Membuat instance CookieRequest baru untuk setiap komponen dapat memakan banyak memori dan sumber daya CPU. Dengan membagikan instance yang sama, dapat mengurangi penggunaan sumber daya ini.
  * Sinkronisasi state: Dalam beberapa kasus, mungkin perlu memastikan bahwa perubahan state yang dibuat dalam satu bagian dari aplikasi segera tercermin di bagian lain. Dengan menggunakan instance CookieRequest yang sama, dapat memastikan bahwa semua komponen aplikasi selalu up-to-date dengan state terkini 

- Jelaskan mekanisme pengambilan data dari JSON hingga dapat ditampilkan pada Flutter. 
  - Mendapatkan Data JSON: Data JSON dapat diperoleh dari berbagai sumber, seperti API web, penyimpanan lokal, atau input pengguna. Biasanya, permintaan HTTP digunakan untuk mengambil data JSON dari server.
  - Konversi JSON menjadi Objek Dart: Setelah mendapatkan data JSON, konversikan ke dalam objek Dart untuk memudahkan penggunaannya di Flutter.
  - Pustaka Dart: Convert: Dart memiliki pustaka dart:convert yang mengandung fungsi json.decode untuk mengubah string JSON menjadi objek Dart.
  - Pembuatan Model Dart: Buatlah model Dart yang mencerminkan struktur data JSON. Ini membantu dalam mendefinisikan tipe data dan memberikan kejelasan pada kode.
  - Deserialisasi Objek Dart: Lakukan proses deserialisasi atau konversi data JSON menjadi objek Dart menggunakan model yang telah dibuat.
  - Tampilkan Data pada UI Flutter: Tampilkan data tersebut pada antarmuka pengguna Flutter dengan membuat widget yang menggunakan nilai dari objek Dart tersebut. Ini melibatkan pembuatan tampilan yang sesuai dengan data yang akan ditampilkan

- Jelaskan mekanisme autentikasi dari input data akun pada Flutter ke Django hingga selesainya proses autentikasi oleh Django dan tampilnya menu pada Flutter. 
  - Mendapatkan Data JSON: Data JSON dapat diperoleh dari berbagai sumber, seperti API web, penyimpanan lokal, atau input pengguna. Pada umumnya, menggunakan metode seperti HTTP requests untuk mengambil data JSON dari server.
  - Setelah mendapatkan input dari pengguna, menggunakan metode http.post untuk mengirim informasi login ke server Django.
  - Setelah Django menerima data login, ia akan memverifikasi data tersebut. Jika data valid, Django akan mengembalikan token autentikasi. dapat menyimpan token ini di perangkat pengguna dan menggunakannya untuk mengautentikasi permintaan berikutnya.
  - Setelah autentikasi berhasil,  dapat menampilkan menu atau akses ke fitur tertentu yang memerlukan autentikasi di aplikasi Flutter . Misalnya,  mungkin ingin menampilkan halaman profil pengguna setelah mereka login.  dapat melakukan ini dengan menggunakan Navigator untuk berpindah ke halaman
- Sebutkan seluruh widget yang kamu pakai pada tugas ini dan jelaskan fungsinya masing-masing.
  - http: dependensi untuk membuat permintaan HTTP
  - Scaffold: sebagai skeleton atau kerangka dasar aplikasi Flutter.
  - container: wadah untuk elemen elemen
  - Text: Menampilkan teks
  - SizedBox: Memberikan spasi vertikal. 
  - ElevatedButton: membuat/menampilkan button 
  - LeftDrawer: Widget kustom yang kemungkinan berisi navigasi atau tautan. 
  - ListView.builder: Membuat daftar elemen dengan jumlah dinamis. 


### Tugas 8
<hr>

### Jelaskan bagaimana cara kamu mengimplementasikan checklist ditas secara step-by-step (bukan hanya sekedar mengikuti tutorial)
* Membuat minimal satu halaman baru pada aplikasi, yaitu halaman formulir tambah item baru dengan ketentuan sebagai berikut:
    * Memakai minimal tiga elemen input, yaitu name, amount, description. Tambahkan elemen input sesuai dengan model pada aplikasi tugas Django yang telah kamu buat.
        - buat file prezzmarket_form.dart lalu tambahkan kode berikut
        ```
        import 'package:flutter/material.dart';
        import 'package:prezzmarket/widgets/left_drawer.dart';

        class ShopFormPage extends StatefulWidget {
        const ShopFormPage({Key? key}) : super(key: key);

        @override
        State<ShopFormPage> createState() => _ShopFormPageState();
        }

        class _ShopFormPageState extends State<ShopFormPage> {
        final _formKey = GlobalKey<FormState>();
        String _name = "";
        int _amount = 0;
        String _description = "";

        @override
        Widget build(BuildContext context) {
            return Scaffold(
            appBar: AppBar(
                title: const Center(
                child: Text(
                    'Form Tambah Produk',
                ),
                ),
                backgroundColor: Colors.indigo,
                foregroundColor: Colors.white,
            ),
            drawer: const LeftDrawer(),
            body: Form(
                key: _formKey,
                child: SingleChildScrollView(
                child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                    Padding(
                        padding: const EdgeInsets.all(8.0),
                        child: TextFormField(
                        decoration: InputDecoration(
                            hintText: "Nama Produk",
                            labelText: "Nama Produk",
                            border: OutlineInputBorder(
                            borderRadius: BorderRadius.circular(5.0),
                            ),
                        ),
                        onChanged: (String? value) {
                            setState(() {
                            _name = value!;
                            });
                        },
                        validator: (String? value) {
                            if (value == null || value.isEmpty) {
                            return "Nama tidak boleh kosong!";
                            }
                            return null;
                        },
                        ),
                    ),
                    Padding(
                        padding: const EdgeInsets.all(8.0),
                        child: TextFormField(
                        decoration: InputDecoration(
                            hintText: "Jumlah",
                            labelText: "Jumlah",
                            border: OutlineInputBorder(
                            borderRadius: BorderRadius.circular(5.0),
                            ),
                        ),
                        onChanged: (String? value) {
                            setState(() {
                            _amount = int.parse(value!);
                            });
                        },
                        validator: (String? value) {
                            if (value == null || value.isEmpty) {
                            return "Jumlah tidak boleh kosong!";
                            }
                            if (int.tryParse(value) == null) {
                            return "Jumlah harus berupa angka!";
                            }
                            return null;
                        },
                        ),
                    ),
                    Padding(
                        padding: const EdgeInsets.all(8.0),
                        child: TextFormField(
                        decoration: InputDecoration(
                            hintText: "Deskripsi",
                            labelText: "Deskripsi",
                            border: OutlineInputBorder(
                            borderRadius: BorderRadius.circular(5.0),
                            ),
                        ),
                        onChanged: (String? value) {
                            setState(() {
                            _description = value!;
                            });
                        },
                        validator: (String? value) {
                            if (value == null || value.isEmpty) {
                            return "Deskripsi tidak boleh kosong!";
                            }
                            return null;
                        },
                        ),
                    ),
                    ],
                ),
                ),
            ),
            );
        }
        }                   
        ```
    * Memiliki sebuah tombol Save.
        - tambahkan kode berikut  pada prezzmarket_form.dart
        ```
        ...
          Align(
            alignment: Alignment.bottomCenter,
            child: Padding(
                padding: const EdgeInsets.all(8.0),
                child: ElevatedButton(
                style: ButtonStyle(
                    backgroundColor:
                        MaterialStateProperty.all(Colors.indigo),
                ),
                onPressed: () {
                    if (_formKey.currentState!.validate()) {
                    showDialog(
                        context: context,
                        builder: (context) {
                        return AlertDialog(
                            title: const Text('Produk berhasil tersimpan'),
                            content: SingleChildScrollView(
                            child: Column(
                                crossAxisAlignment:
                                    CrossAxisAlignment.start,
                                children: [
                                Text('Nama: $_name'),
                                Text('Jumlah: $_amount'),
                                Text('Deskripsi: $_description'),
                                ],
                            ),
                            ),
                            actions: [
                            TextButton(
                                child: const Text('OK'),
                                onPressed: () {
                                Navigator.pop(context);
                                },
                            ),
                            ],
                        );
                        },
                    );
                    _formKey.currentState!.reset();
                    }
                },
                child: const Text(
                    "Save",
                    style: TextStyle(color: Colors.white),
                ),
                ),
            ),
            ),
        ```
    * Setiap elemen input di formulir juga harus divalidasi dengan ketentuan sebagai berikut:
        * Setiap elemen input tidak boleh kosong.
            - tambahkan kode berikut sebagai validator
            ```
            ...
            validator: (String? value) {
                if (value == null || value.isEmpty) {
                    return "Nama tidak boleh kosong!";
                }
                return null;
                },
            ...
            ```
        * Setiap elemen input harus berisi data dengan tipe data atribut modelnya.
            - pada saat meminta jumlah, hanya boleh integer, oleh karena itu saya membuat validator sebagai berikut
            ```
            ...
            validator: (String? value) {
                if (value == null || value.isEmpty) {
                    return "Jumlah tidak boleh kosong!";
                }
                if (int.tryParse(value) == null) {
                    return "Jumlah harus berupa angka!";
                }
                return null;
                },
            ...
            ```
* Mengarahkan pengguna ke halaman form tambah item baru ketika menekan tombol Tambah Item pada halaman utama.
    - untuk mengarahkan pengguna ke halaman form tambah item baru, tambahkan kode berikut
    ```
    ...
    if (item.name == "Tambah Produk") {
        Navigator.push(
        context,
        MaterialPageRoute(
            builder: (context) => const ShopFormPage(),
        ));
        }
    ...   
    ```
* Memunculkan data sesuai isi dari formulir yang diisi dalam sebuah pop-up setelah menekan tombol Save pada halaman formulir tambah item baru.
    - Tambahkanlah showDialog() pada bagian onPressed() dan tampilkan widget AlertDialog. Dalam AlertDialog, tambahkan child berupa widget Column yang berisi elemen-elemen dengan widget Text untuk menampilkan data yang sesuai. Selanjutnya, tambahkan fungsi untuk mereset formulir. Berikut adalah kodenya
    ```
    ...
    onPressed: () {
        if (_formKey.currentState!.validate()) {
            showDialog(
            context: context,
            builder: (context) {
                return AlertDialog(
                title: const Text('Produk berhasil tersimpan'),
                content: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                    Text('Nama: $_name'),
                    Text('Jumlah: $_amount'),
                    Text('Deskripsi: $_description'),
                    // Tambahkan Text atau widget lain untuk menampilkan data tambahan
                    ],
                ),
                actions: [
                    TextButton(
                    child: const Text('OK'),
                    onPressed: () {
                        Navigator.pop(context);
                        _resetForm(); // Panggil fungsi reset form setelah menutup AlertDialog
                    },
                    ),
                ],
                );
            },
            );
        }
    },
    ```
* Membuat sebuah drawer pada aplikasi dengan ketentuan sebagai berikut:
    * Drawer minimal memiliki dua buah opsi, yaitu Halaman Utama dan Tambah Item.
    Pada file prezzmarket_form.dart didalam class _ShopFormPageState, pada bagaian child: Column(), dan bagian Align() tambahkan kode berikut
    ```
    ...
    ListTile(
            leading: const Icon(Icons.home_outlined),
            title: const Text('Halaman Utama'),
            // Bagian redirection ke MyHomePage
            onTap: () {
              Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => MyHomePage(),
                  ));
            },
          ),
          ListTile(
            leading: const Icon(Icons.add_shopping_cart),
            title: const Text('Tambah Produk'),
            // Bagian redirection ke ShopFormPage
            onTap: () {
              /*
              TODO: Buatlah routing ke ShopFormPage di sini,
              setelah halaman ShopFormPage sudah dibuat.
              */
               Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ShopFormPage(),
                  ));
            },
          ),
    ...

    ```
    * Ketika memiih opsi Halaman Utama, maka aplikasi akan mengarahkan pengguna ke halaman utama.
        - Pada widget ListTile dengan label 'Halaman Utama', saya telah memasukkan fungsi Navigator.pushReplacement ke dalam fungsi onTap. Artinya, ketika item ini ditekan, aplikasi akan menghapus tampilan yang sedang ditampilkan dan menggantinya dengan tampilan baru menuju Halaman Utama. berikut kodenya
    ```
    onTap: () {
              Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => MyHomePage(),
                  ));
            },
    ```
    * Ketika memiih opsi (Tambah Item), maka aplikasi akan mengarahkan pengguna ke halaman form tambah item baru.
        - Pada widget ListTile dengan label 'Tambah Item', saya telah menambahkan fungsi Navigator.pushReplacement ke dalam fungsi onTap. Ini berarti ketika pengguna menekan item ini, tampilan yang sedang ditampilkan akan dihapus dan digantikan dengan tampilan formulir untuk menambahkan item baru. 
    ```
    onTap: () {
               Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const ShopFormPage(),
                  ));
            },
    ```
            
### Jelaskan perbedaan antara Navigator.push() dan Navigator.pushReplacement(), disertai dengan contoh mengenai penggunaan kedua metode tersebut yang tepat!
* Penambahan Route pada Stack:
    * navigator.push menambahkan route baru di atas route yang sudah ada pada stack.
    * pushReplacement menggantikan route yang sudah ada pada stack dengan route baru.
* Pengaruh Terhadap Tombol "Back":
    * Dengan menggunakan navigator.push, pengguna dapat kembali ke route sebelumnya dengan menekan tombol "Back", karena route baru ditambahkan di atas stack.
    * Pada pushReplacement, jika route digantikan, menekan tombol "Back" tidak akan membawa pengguna kembali ke route yang digantikan, karena route sebelumnya telah dihapus dari stack.
* Contoh penggunaan Navigator.push()
    ```
    Navigator.push(
        context,
        MaterialPageRoute(builder: (context) => SecondRoute()),
    );
    ```
* Contoh penggunaan Navigator.pushReplacement
    ```
    Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (context) => SecondRoute()),
    );
    ```

### Jelaskan masing-masing layout widget pada Flutter dan konteks penggunaannya masing-masing!
* Row dan Column
  Widget Row dan Column berfungsi untuk mengatur elemen anak dalam satu baris atau kolom dalam tata letak Flutter. Row mengatur elemen anak secara horizontal, sedangkan Column mengatur elemen anak secara vertikal.
* ListView
  ListView adalah sebuah widget yang digunakan untuk mengelola daftar item yang dapat di-scroll. Widget ini memungkinkan pembuatan daftar yang mendukung elemen-elemen yang dapat di-scroll baik secara vertikal maupun horizontal.
* Container
  Container merupakan suatu widget dasar yang berfungsi untuk mengatur tata letak dan dekorasi. Widget ini dapat dimanfaatkan untuk menetapkan berbagai properti, termasuk ukuran, padding, warna latar belakang, serta sejumlah properti lainnya yang memengaruhi penampilan dan tata letaknya.
* Expanded dan Flexible
  Widget Expanded dan Flexible digunakan untuk mengontrol cara widget berskala dalam tata letak fleksibel. Kedua widget ini berperan dalam menentukan bagaimana ruang dan ukuran dikelola dalam suatu tata letak fleksibel.
* Stack
  Stack memungkinkan penggunaan beberapa widget yang tumpang tindih di atas satu sama lain. Widget dalam Stack dapat berada dalam posisi tumpuk atau dapat ditempatkan secara spesifik satu di dalam yang lain.
* Text
  Menampilkan teks, sebagai informasi atau semacamnya.
* Drawer
  Memberikan akses atau link ke halaman dan fungsionalitas yang terdapat di aplikasi
* AppBar
  Biasanya untuk menampilkan toolbar dari widgets yang dimiliki aplikasi.


### Sebutkan apa saja elemen input pada form yang kamu pakai pada tugas kali ini dan jelaskan mengapa kamu menggunakan elemen input tersebut!
* TextFormField untuk membuat sebuah area input teks nama produk
```
TextFormField(
  decoration: InputDecoration(
    hintText: "Nama Produk",
    labelText: "Nama Produk",
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(5.0),
    ),
  ),
  onChanged: (String? value) {
    setState(() {
      _name = value!;
    });
  },
  validator: (String? value) {
    if (value == null || value.isEmpty) {
      return "Nama tidak boleh kosong!";
    }
    return null;
  },
)
```
Kode tersebut menciptakan sebuah elemen input menggunakan TextFormField yang berfungsi untuk mengumpulkan Nama Produk. Properti onChanged pada widget digunakan untuk menyimpan nilai yang diinput ke dalam variabel _name, sehingga setiap kali nilai input berubah, state aplikasi juga diupdate. Sementara itu, properti validator berperan untuk memastikan bahwa input tidak boleh kosong.jika input kosong akan ditampilkan pesan kesalahan "Nama tidak boleh kosong!". 

* TextFormField untuk Jumlah meminta input beruapa integer untuk jumlah 
```
TextFormField(
  decoration: InputDecoration(
    hintText: "Jumlah",
    labelText: "Jumlah",
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(5.0),
    ),
  ),
  onChanged: (String? value) {
    setState(() {
      _amount = int.parse(value!);
    });
  },
  validator: (String? value) {
    if (value == null || value.isEmpty) {
      return "Jumlah tidak boleh kosong!";
    }
    if (int.tryParse(value) == null) {
      return "Jumlah harus berupa angka!";
    }
    return null;
  },
)
```
Kode tersebut menciptakan sebuah elemen input menggunakan TextFormField yang berfungsi untuk mengumpulkan jumlah produk. Properti onChanged pada widget digunakan untuk menyimpan nilai yang diinput ke dalam variabel _amount, sehingga setiap kali nilai input berubah, state aplikasi juga diupdate. Sementara itu, properti validator berperan untuk memastikan bahwa input tidak boleh kosong dan harus integer.jika input kosong akan ditampilkan pesan kesalahan "Nama tidak boleh kosong!" dan jika input bukan integer akan ditampilkan pesan "jumlah harus berupa angka!"

* TextFormField membuat sebuah area input teks dekripsi
```
TextFormField(
  decoration: InputDecoration(
    hintText: "Deskripsi",
    labelText: "Deskripsi",
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(5.0),
    ),
  ),
  onChanged: (String? value) {
    setState(() {
      _description = value!;
    });
  },
  validator: (String? value) {
    if (value == null || value.isEmpty) {
      return "Deskripsi tidak boleh kosong!";
    }
    return null;
  },
)
```

Kode tersebut menciptakan sebuah elemen input menggunakan TextFormField yang berfungsi untuk mengumpulkan deskripsi produk. Properti onChanged pada widget digunakan untuk menyimpan nilai yang diinput ke dalam variabel _description, sehingga setiap kali nilai input berubah, state aplikasi juga diupdate. Sementara itu, properti validator berperan untuk memastikan bahwa input tidak boleh kosong .jika input kosong akan ditampilkan pesan kesalahan "Nama tidak boleh kosong!"

### Bagaimana penerapan clean architecture pada aplikasi Flutter?
- Lapisan Presentasi (UI):
Ini merupakan bagian dari aplikasi yang bertanggung jawab atas tampilan pengguna, mengelola interaksi antarmuka, dan menampilkan elemen visual. Lapisan ini harus fokus pada antarmuka tanpa terlibat secara langsung dengan logika bisnis atau akses ke data secara spesifik.

- Lapisan Domain (Logika Bisnis):
Ini merupakan inti dari aplikasi yang menampung aturan-aturan bisnis utama, kasus penggunaan, serta entitas yang mewakili objek penting dalam aplikasi. Lapisan ini independen dari kerangka atau teknologi tertentu, fokus pada logika bisnis, dan tidak terikat dengan implementasi teknis tertentu.

- Lapisan Data:
Ini adalah bagian yang bertanggung jawab atas interaksi dengan penyimpanan dan pengambilan data. Lapisan ini menangani operasi-operasi database, sumber data eksternal, dan menyediakan repositori untuk lapisan domain. Tujuannya adalah untuk melindungi lapisan domain dari kompleksitas teknis dalam penyimpanan dan pengambilan data.
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
                            'PrezzMarket', // Text yang mekan toko
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

    * Tambahkan juga widget stateless baru untuk menampilkan card:
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
        * `MyHomePage` StatelessWidget: Merupakan komponen utama yang menggambarkan halaman be dalam aplikasi. Widget ini menampilkan antarmuka utama yang memuat daftar item toko.
        * `AppBar`: menampilkan bilah aplikasi di bagian atas halaman
        * `Scaffold`: menyediakan skeleton (kerangka) kerja dasar untuk halaman, berisi AppBar dan body
        * `ColorScheme` : mengatur palet warna aplikasi
        * `ScaffoldMessenger`: menampilkan pesan snack bar saat salah satu tombol item diklik
        * `Text`: menampilkan teks `prezzmarket`
        * `SnackBar`: pesan pop up yang muncul ketika tombol item ditekan
        * `SingleChildScrollView`: berfungsi melingkupi/membungkus semua isi halaman, memungkinkan pengguna untuk melakukan scroll jika kontennya melebihi ukuran layar.
        * `Padding`: menambahkan jarak di sekitar widget-child
        * `Column`: mengatur widget-children secara vertikal
        * `GridView.count`: membuat tata letak grid dengan jumlah kolom yang diberikan.
        * `ShopItem`: kelas yang mendefinisikan item toko (nama,ikon,warna)
        * `ShopCard`: mewakili card yang menampilkan item toko
        * `Material`: berperan dalam menentukan latar belakang warna pada kartu.
        * `InkWell`: berfungsi sebagai area yang merespons sentuhan, memungkinkan item toko dapat ditekan dengan responsif
        * `Container`: digunakan untuk mengelompokkan ikon dan teks dalam kartu.
        * `Icon`: menampilkan ikon dalam card
    - [x] jelaskan bagaimana cara kamu mengimplementasikan checklist di atas secara step-by-step (bukan hanya sekadar mengikuti tutorial)
-[x] Melakukan `add`-`commit`-`push` ke GitHub.
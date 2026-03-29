# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. Pada tutorial ini, `RwLock<Vec<Notification>>` diperlukan karena data notifikasi disimpan secara global dan diakses oleh banyak request secara bersamaan. Dalam aplikasi web Rocket, beberapa thread dapat membaca daftar notifikasi pada saat yang sama (misalnya endpoint list), sementara thread lain bisa menambah notifikasi baru (endpoint receive). Tanpa mekanisme sinkronisasi, akses bersamaan ini bisa menyebabkan data race dan perilaku tidak terduga. `RwLock` cocok karena pola aksesnya cenderung read-heavy: banyak operasi baca, sedikit operasi tulis. Dengan `RwLock`, banyak reader bisa berjalan paralel selama tidak ada writer. Jika memakai `Mutex`, baik operasi baca maupun tulis tetap harus antre satu per satu karena lock bersifat eksklusif, sehingga concurrency untuk operasi baca menjadi kurang optimal.

2. Rust tidak mengizinkan kita langsung memutasi `static` variable seperti di Java karena Rust menegakkan jaminan keamanan memori dan thread safety pada waktu kompilasi. `static` bernilai global dan berumur sepanjang program, sehingga jika bisa dimutasi bebas akan sangat mudah menimbulkan race condition. Karena itu, mutable global state di Rust harus dibungkus tipe sinkronisasi yang aman (misalnya `RwLock`, `Mutex`, atau struktur concurrent seperti `DashMap`) agar aturan aksesnya jelas dan aman lintas thread. `lazy_static` dipakai untuk inisialisasi global yang dilakukan saat runtime (lazy initialization), karena inisialisasi `static` biasa di Rust terbatas pada ekspresi konstan. Jadi, bukan berarti Rust tidak bisa punya global mutable state, tetapi Rust memaksa kita mengelolanya secara eksplisit dan aman.

#### Reflection Subscriber-2
1. Dalam pengerjaan tutorial ini, saya melakukan eksplorasi pada file src/lib.rs untuk memahami bagaimana aplikasi Receiver dikonfigurasi secara global. Saya mempelajari bahwa lib.rs berfungsi sebagai pusat definisi modul dan konstanta penting, seperti APP_CONFIG yang membaca variabel lingkungan dari file .env dan REQWEST_CLIENT yang diinisialisasi sebagai klien HTTP global. Eksplorasi ini memberi saya wawasan tentang bagaimana Rust mengelola ketergantungan antar-modul dan bagaimana pola Singleton diimplementasikan untuk memastikan sumber daya seperti koneksi HTTP dapat digunakan kembali secara efisien di seluruh bagian aplikasi tanpa perlu inisialisasi ulang.

2. Implementasi Observer Pattern sangat memudahkan proses penambahan (plug-in) subscriber baru karena Publisher tidak perlu mengetahui detail spesifik dari setiap subscriber. Selama subscriber menyediakan endpoint yang sesuai (seperti /receive), kita bisa menambah puluhan instance baru hanya dengan menjalankan aplikasi di port berbeda dan melakukan pendaftaran (subscribe). Namun, jika kita mencoba menjalankan lebih dari satu instance aplikasi Main (Publisher), sistem akan menjadi lebih kompleks. Hal ini disebabkan oleh sifat penyimpanan subscriber yang saat ini masih berbasis memori (in-memory). Setiap Publisher akan memiliki daftar subscriber-nya masing-masing secara terisolasi. Agar sistem tetap mudah dikelola dengan banyak Publisher, kita perlu memindahkan daftar subscriber ke database eksternal atau layanan registry terpusat agar seluruh Publisher merujuk pada data yang konsisten.

3. Penggunaan fitur Tests dan dokumentasi pada koleksi Postman terbukti sangat berguna, baik untuk tutorial ini maupun proyek kelompok. Dengan menuliskan skrip tes otomatis di Postman, saya dapat memastikan bahwa setiap endpoint memberikan status kode yang benar (seperti 200 OK atau 201 Created) dan struktur JSON yang sesuai tanpa harus mengeceknya secara manual setiap saat. Dokumentasi yang baik pada Postman juga sangat membantu saat bekerja dalam tim, karena setiap anggota dapat memahami parameter yang dibutuhkan dan format request tanpa harus membaca keseluruhan kode sumber. Hal ini mempercepat proses integrasi antara frontend dan backend serta meminimalisir kesalahan komunikasi antar-pengembang.
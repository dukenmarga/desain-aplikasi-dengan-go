Single Responsibility Principle (SRP)
=====================================

Definisi
--------

Prinsip yang pertama yaitu S dalam SOLID adalah **Single Responsibility Principle (SRP)**
yang merupakan prinsip dasar untuk menuliskan kode yang kuat (*robust*), mudah dipelihara
(*maintainable*), mudah dimengerti (*understandable*), terprediksi (*predictable*),
mudah dimodifikasi (*flexible*), dan mudah dilakukan pengujian (*testable*).
Jika hanya bisa memilih satu saja dari
kelima prinsip yang ada, maka SRP ini adalah yang paling utama dan paling mudah
pengaplikasiannya sejak awal mulai membuat program.

Secara umum, inti dari SRP menyatakan bahwa sebuah **module, kelas (class), atau fungsi
seharusnya hanya punya satu alasan untuk dimodifikasi**.
Dalam konteks bahasa Go, SRP diaplikasikan pada level **packages, structs, dan interfaces**.
Praktisnya adalah setiap unit kode hanya berfokus pada satu fungsi dan tanggung jawab.
Jika sebuah fungsi bertanggung jawab pada lebih dari satu tugas, maka modifikasi kode
di kemudian hari cenderung menimbulkan masalah.

SRP dicetuskan oleh Robert C. Martin (Uncle Bob), yang menyatakan bahwa mencampuradukkan
fungsi akan berujung pada sulitnya programmer untuk mengerti akan kode yang sedang
dikerjakan. Untuk itulah SRP mendorong untuk membuat unit kode yang kecil, alih-alih
menggabungkan semuanya ke dalam unit (baik itu function, struct, interfaces) yang besar.

Implementasi
------------

Contoh Permasalahan pada Fungsi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Potongan kode di bawah merupakan contoh yang tidak memenuhi prinsip SRP.

.. highlight:: go
   :linenothreshold: 1

.. warning::
    Kode di bawah tidak memenuhi SRP karena 1 fungsi melakukan 2 tugas berbeda

.. code-block:: go

    type Report struct {
        Title   string
        Content string
    }

    // Fungsi untuk mengubah format dan menyimpan laporan
    func (r *Report) FormatAndSave() error {
        // Format CSV
        csvFormat := fmt.Sprintf("%s,%s", r.Title, r.Content)

        // Simpan ke dalam file CSV
        f, err := os.Create("/tmp/data.csv")
        if err != nil {
            ...
        }
        defer f.Close()

        _, err = f.Write([]byte(csvFormat))
        if err != nil {
            ...
        }

        return nil
    }

Fungsi :func:`FormatAndSave` melakukan 2 tugas sekaligus yaitu melakukan *text formatting*
dalam bentuk CSV (Comma Separated Value) lalu meyimpannya ke dalam file CSV. Namun
ciri yang paling terlihat ketika tidak memenuhi aturan SRP adalah detail implementasi
algoritma dilakukan di dalam fungsi itu sendiri (baris ke-9 dan baris 12-21).

Misalkan saja kita perlu melakukan modifikasi yaitu perlu menyimpannya ke dalam
database, alih-alih sebagai CSV file. Kita tidak bisa langsung mengganti fungsi
tanpa merubah format penyimpanan.

.. warning::
    Kode di bawah menjadi tidak praktis karena selain perlu melakukan perubahan
    fungsi penyimpanan ke database, kita juga perlu melakukan perubahan input format.

.. code-block:: go

    type Report struct {
        Title   string
        Content string
    }

    // Fungsi untuk mengubah format dan menyimpan laporan
    func (r *Report) FormatAndSave() error {
        // Format CSV
        csvFormat := fmt.Sprintf("%s,%s", r.Title, r.Content)

        // bentuk CSV mungkin tidak kompatibel dengan format peyimpanan database
        saveErr := saveToMySQL(csvFormat)
        return saveErr
    }

Untuk memenuhi prinsip SRP, kita memecah fungsi ke dalam 2 fungsi sebagai bentuk
pemisahan tanggung jawab (*separation concern*).

.. note::
    Fungsi :func:`FormatAndSave` dipecah menjadi 2 fungsi dengan tugas yang independen
    satu dengan lainnya.

.. code-block:: go

    type Report struct {
        ...
    }

    func (r *Report) Format() string {
        return fmt.Sprintf("%s,%s", r.Title, r.Content)
    }

    func SaveToFile(csv string) error {
        // Simpan ke dalam file (implementasi tidak diberikan)
        // ...

        return nil
    }


Menggunakan dan Menggabungkan Fungsi yang Memenuhi SRP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Lalu, bagaimana kita memakai dan menggabungkan fungsi dan modul yang sudah dipecah menjadi
unit yang memenuhi SRP? Bukankah pada akhirnya kita akan tetap menggunakan fungsi
seperti :func:`FormatAndSave` seperti potongan kode di bawah? Seperti tidak terlihat
perbedaan sebelum melakukan pemisahan fungsi.

Dalam praktiknya di dunia nyata kita perlu menggabungkan kedua fungsi
:func:`Format` dan :func:`SaveToFile` karena kebutuhan alur proses.
Kunci dari proses ini disebut sebagai **composition**.

Tidak masalah jika pada akhirnya kita memiliki fungsi dengan nama :func:`FormatAndSave`,
**selama fungsi ini hanya bertanggung jawab untuk menggabungkan beberapa fungsionalitas
ke dalam sebuah workflow dan hanya untuk mengorkestrasi sub-step tadi, bukan untuk
melibatkan dan memproses logic yang kompleks**. Jika ditinjau secara prinsip SRP, maka
fungsi :func:`FormatAndSave` kini hanya memiliki 1 tugas: *mengkoordinasikan*
proses-proses yang berkaitan.

.. note::
    Fungsi :func:`FormatAndSave` kini memenuhi prinsip SRP:

    - Tidak mencampuradukkan tugas dan tanggung jawab dalam sebuah fungsi
    - Bergantung pada 2 fungsi independen: :func:`Format` dan :func:`SaveToFile` 
    - Tugasnya menjadi lebih jelas: mengorkestrasi proses memformat lalu menyimpan data.

.. code-block:: go

    type Report struct {
        ...
    }

    func (r *Report) Format() string {
        return fmt.Sprintf("%s,%s", r.Title, r.Content)
    }

    func SaveToFile(csv string) error {
        // Simpan ke dalam file (implementasi tidak diberikan)
        // ...

        return nil
    }
 
    func FormatAndSave(r *Report) error {
        formatted := r.Format()
        return SaveToFile(formatted)
    }

Dengan memecah logic ke dalam unit fungsi yang lebih kecil, kita mendapatkan
manfaat:

- Setiap fungsi menjadi lebih kecil, mudah diuji (*testable*), dan mudah digunakan
  kembali (*reusable*)
- Perubahan yang dilakukan pada fungsi formatting :func:`Format` dan fungsi peyimpanan
  :func:`SaveToFile` di kemudian hari bisa dilakukan tanpa perlu menyentuh fungsi
  orkestrasi :func:`FormatAndSave`

Adaptasi Perubahan dan Solusi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Jika di kemudian hari misalkan kita ingin mengganti tempat penyimpanan menjadi ke
sebuah database, kita perlu menukar fungsi yang lama ke baru,
misalkan :func:`SaveToFile` menjadi :func:`SaveToDB` di dalam
:func:`FormatAndSave` dan logikanya akan tetap sama.

.. note::
    Untuk perubahan sederhana (tidak dinamis) pada :func:`FormatAndSave`,
    bisa dengan mengganti fungsi yang
    lama menjadi fungsi yang baru dan menghilangkan kode yang tidak perlu, misalnya
    menghapus fungsi formatting karena tidak dibutuhkan.

.. code-block:: go

    type Report struct {
        Title   string
        Content string
    }

    ...

    func SaveToDB(r *Report) error {
        // Simpan ke dalam database (implementasi hanya sebagian)
        // ...

        result, err := db.ExecContext(ctx,
            "INSERT INTO reports (title, content) VALUES ($1, $2)",
            r.Title,
            r.Content,
        )

        ...

        return nil
    }

    func FormatAndSave(r *Report) error {
        return SaveToDB(r)
    }

Namun seringkali perubahan ini tidak berjalan dengan mulus terutama jika mode penyimpanan bersifat dinamis atau
tidak bisa ditentukan sejak awal, apakah akan menyimpan ke file atau ke database.
Hal ini menjadi sulit karena masalah kompatibilitas tipe data (*type compatibility*).
Misalkan fungsi :func:`SaveToFile` menerima input
string, sedangkan misalnya :func:`SaveToDB` tidak memerlukan string sebagai input,
melainkan membutuhkan *raw struct* dari *Report*.

Ada beberapa pendekatan solusi dari masalah ini sambil tetap mempertahankan prinsip
SRP. Tugas inti :func:`FormatAndSave` yaitu *formatting* dan *saving* tetap terpisah.
Namun logika orkestrasi menjadi lebih kompleks karena kebutuhan yang bercabang.

**1. Menggunakan Logika Orkestrasi yang Lebih Ketat**

.. note::
    Fungsi :func:`FormatAndSave` dengan tambahan logic terbatas sesuai kebutuhan,
    namun tetap memenuhi prinsip SRP.

.. code-block:: go

    ...
 
    func FormatAndSave(r *Report, toDB bool) error {
        if toDB {
            return SaveToDB(r)
        } else {
            formatted := r.Format()
            return SaveToFile(formatted)
        }
    }

Pendekatan ini menyaratkan *programmer* yang memanggil fungsi ini harus hati-hati
terhadap kebutuhannya. Penambahan flag ``toDB`` dan penambahan logic pada fungsi
:func:`FormatAndSave` untuk memilih fungsi mana yang dipakai masih bisa diterima
dan masih mematuhi prinsip SRP karena tugasnya masih tetap hanya untuk mengkoordinasikan
dan mengorkestrasi sub-step di dalamnya.

**2. Menggunakan Interface atau Adapter Pattern**

Cara lain yang umum dipakai adalah membuat abstraksi menggunakan *interface*. Hal ini
sangat berguna jika ada banyak implementasi yang bisa digunakan, misalnya saja
selain bisa menyimpan ke file CSV dan database, dibutuhkan juga penyimpanan ke *cloud storage*
(misal S3), penyimpanan ke format JSON, dan penyimpanan ke format *binary file*.

.. note::
    ``interface`` adalah sebuah tipe yang memuat kesepakatan fungsi-fungsi yang harus
    diimplementasikan untuk semua ``struct`` yang hendak comply dengan interface itu.

    Pada ``interface`` di bawah ini, :func:`Save` adalah nama fungsi yang sebelumnya
    merupakan representasi dari :func:`SaveToFile` dan :func:`SaveToDB`. Dengan
    ini, maka fungsi sebelumnya :func:`FormatAndSave` bisa digantikan oleh
    :func:`SaveReport` yang hanya perlu memanggil fungsi :func:`Save`
    tanpa perlu menggunakan percabangan logika seperti pada solusi 1 di atas.

    Namun, karena sebelumnya :func:`SaveToFile` menerima string sebagai input, sedangkan
    dengan menggunakan ``interface``, maka kita perlu memodifikasi implementasinya
    karena :func:`Save` hanya menerima *raw struct* sebagai input.

.. code-block:: go

    // Report data type
    type Report struct {
        Title   string
        Content string
    }
    
    // interface Saver
    type Saver interface {
        Save(r *Report) error
    }

    // Semua struct type yang perlu mengimplementasikan interface Saver
    type CSVFileSaver struct{}
    type DBSaver struct{}
    type S3Saver struct{}

    // Implementasi Save() untuk masing-masing struct di atas pada interface Saver
    func (fs CSVFileSaver) Save(r *Report) error {
        return SaveToFile(r.Format())
    }
    func (ds DBSaver) Save(r *Report) error {
        return SaveToDB(r)
    }
    func (ds S3Saver) Save(r *Report) error {
        return SaveToS3(r)
    }

    func (r *Report) Format() string {
        return fmt.Sprintf("%s,%s", r.Title, r.Content)
    }

    func SaveToFile(csv string) error {
        // Simpan ke dalam file (implementasi tidak diberikan)
        // ...
        return nil
    }

    func SaveToDB(r *Report) error {
        // Simpan ke dalam database (implementasi hanya sebagian)
        // ...
        result, err := db.ExecContext(ctx,
            "INSERT INTO reports (title, content) VALUES ($1, $2)",
            r.Title,
            r.Content,
        )
        ...
        return nil
    }
    // ... fungsi lainnya SaveToS3()

    func SaveReport(s Saver, r *Report) error {
        return s.Save(r)
    }

    // Contoh penggunaan fungsi SaveReport()
    func main(){
        outputTarget := S3Saver{}
        report := &Report{
            Title   : "Prinsip Desain Aplikasi dengan Go (Golang)"
            Content : "..."
        }
        SaveReport(outputTarget, report)
    }

Dengan menggunakan ``interface``, fungsi :func:`SaveReport` kini hanya perlu memanggil
fungsi :func:`Save` sebagai logika orkestrasinya tanpa membutuhkan logika percabangan.
Koordinasi kode menjadi lebih sederhana dan bisa mengadaptasi berbagai kebutuhan
implementasi lainnya di kemudian hari.

Pola desain di bagian ini dengan menggunakan ``interface`` dikenal juga sebagai
**Adapter Pattern**, di mana ``interface`` digunakan untuk mengikat 2 atau lebih
implementasi suatu fungsi pada ``struct``. Di dalam bahasa Go, ikatan ini bersifat
implisit tanpa perlu kode tambahan dan *compiler* akan mendeteksinya secara otomatis.
Penggunaan ``interface`` untuk menyelesaikan isu yang ada, juga akan dibahas
pada Open-Closed Principle.

Catatan tambahan untuk potongan kode di atas, ``outputTarget`` diset secara statis, namun
dalam implementasinya, ``outputTarget`` bisa diset secara dinamis sesuai kebutuhan.

**3. Injeksi Fungsi (Function Injection)**

Cara lain yang bisa digunakan adalah injeksi fungsi sebagai parameter. Misalnya dalam
contoh di bawah, fungsi :func:`SaveReport` menerima sebuah ``function`` sebagai parameter
kedua yang kemudian dieksekusi di dalamnya. Implementasi fungsi ditulis di luar sesuai
kebutuhan logika dan algoritma yang digunakan.

.. note::
    ``saveFunc`` merupakan nama variabel bertipe fungsi.
    Ada 2 contoh penggunaan yang diberikan: pertama untuk menyimpan ke database
    dan kedua untuk menyimpan ke file.
    Namun untuk contoh ke-2, karena SaveToFile berbeda tipe dengan parameter
    yang dibutuhkan, maka implementasinya dituliskan langsung sebagai parameter

.. code-block:: go

    func SaveReport(r *Report, saveFunc func(*Report) error) error {
        return saveFunc(r)
    }

    func SaveToDB(r *Report) error {
        // Simpan ke dalam database (implementasi hanya sebagian)
        // ...
        result, err := db.ExecContext(ctx,
            "INSERT INTO reports (title, content) VALUES ($1, $2)",
            r.Title,
            r.Content,
        )
        ...
        return nil
    }
    func SaveToFile(csv string) error {
        // Simpan ke dalam file (implementasi tidak diberikan)
        // ...
        return nil
    }

    // Contoh penggunaan jika hendak menyimpan file ke dalam database,
    // maka fungsi SaveToDB diberikan sebagai parameter kedua
    SaveReport(r, SaveToDB)

    // Contoh penggunaan jika hendak menyimpan ke file.
    SaveReport(r, func(r *Report) error {
        return SaveToFile(r.Format())
    })

Ketiga pendekatan di atas tidak menyalahi prinsip SRP karena setiap fungsi yang
disebutkan di atas tetap mempertahankan *single responsibility* dan fungsi
orkestrasi tetap hanya bertugas sebagai fungsi koordinasi.
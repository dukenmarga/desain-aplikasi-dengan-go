Liskov Substitution Principle (LSP)
===================================

**Liskov Substitution Principle (LSP)** diterapkan untuk mendesain kode yang kuat
(*robust*) dan mudah dipelihara (*maintanable*).

Prinsip ini diformulasikan oleh
Barbara Liskov (1987) dengan menyatakan **bahwa sebuah objek dari sebuah tipe induk
(supertype) harus bisa digantikan oleh objek dari turunannya (subtype) tanpa
memengaruhi keakuratan dan kebenaran dari program tersebut** [1]_.

Sebagai catatan, banyak penjelasan dan contoh dari berbagai sumber menyajikan bahwa
prinsip ini terlihat mudah, namun banyak hal inti yang belum digali.
Bahkan Robert Martin sendiri sempat menyalahartikan arti dari definisi di atas [2]_.
Konsep ini akan sedikit kompleks dibandingkan ekspektasi awal kita. Oleh karena itu,
bagian ini bisa dibaca dan diulang kembali secara rutin untuk lebih memahaminya dengan
lebih baik.

Kekeliruan pertama yang seringkali disalahartikan oleh banyak *programmer* adalah
*supertype* merujuk pada sebuah kelas induk (*superclass*), sedangkan *subtype* merujuk
pada *subclass* akibat *inheritance*. Jika merujuk pada pemrograman berbasis objek
(*object oriented programming*), sebagian pernyataan ini benar, namun seringkali tidak
memenuhi definisi aslinya.

Go tidak mendukung konsep *class* dan *inheritance* karena bukan merupakan bahasa
pemrograman berbasis objek (*object oriented programming*) seperti Java atau Python.
Namun, konsep serupa bisa digantikan dengan ``struct`` dan ``interface``.

Kekeliruan kedua adalah menyalahartikan bahwa menggunakan ``interface`` berarti sudah
memenuhi prinsip Liskov. Padahal, ``interface`` adalah salah satu bagian untuk memenuhi
prinsip ini, masih ada aspek kebenaran dari program yang juga harus dipenuhi.

Mari kita ambil contoh sederhana kode yang memenuhi prinsip Liskov. Penjelasan lebih
detail tentang apa yang harus dilakukan untuk memenuhi prinsip akan diberikan pada sub-bab
selanjutnya.

.. note::
    Contoh di bawah mengimplementasikan Liskov Substitution Principle dengan
    benar dengan menggunakan ``interface`` dan ``struct``. Interface ``Shape``
    memiliki 1 fungsi :func:`Area` sebagai kontrak. Sedangkan implementasinya
    dimiliki oleh *struct* ``Rectangle`` dan ``Square``.
    
    Misalkan saja kita ingin menghitung harga total luasan area dengan menggunakan
    fungsi :func:`TotalPrice`. Fungsi :func:`TotalPrice` menggunakan ``Shape``
    sebagai parameter input, alih-alih menggunakan ``struct``, yang berarti
    :func:`TotalPrice` bisa menerima *struct* apa saja, misalkan ``Rectangle`` dan
    ``Square`` selama *struct* tersebut memiliki *method* yang didefinisikan
    pada *interface*.

.. code-block:: go

    type Shape interface {
        Area() float64
    }

    type Rectangle struct {
        Width, Height float64
    }

    func (r Rectangle) Area() float64 {
        return r.Width * r.Height
    }

    type Square struct {
        Side float64
    }

    func (s Square) Area() float64 {
        return s.Side * s.Side
    }

    // Contoh penggunaan
    func pricePerUnit() int {
        return 100
    }
    func TotalPrice(shape Shape) {
        return pricePerUnit() * shape.Area()
    }

    rect := Rectangle{
        Width: 10,
        Height: 15,
    }
    fmt.Printf("Total price rectangle: %v", TotalPrice(rect))

    sq := Square{
        Side: 8,
    }
    fmt.Printf("Total price rectangle: %v", TotalPrice(sq))

Beberapa poin yang bisa disimpulkan:

- **Interface adalah supertype** atau tipe dasar atau kontrak,
  sedangkan **struct yang mengimplementasikan interface (implementor)
  adalah subtype**.
- *Interface* adalah abstraksi, sedangkan implementor adalah tipe konkrit.
- Tidak ada konsep *inheritance* seperti pada OOP, namun Go tetap bisa
  mengaplikasikan konsep LSP.

Penjelasan lebih detail mengenai apakah sebuah *subtype* telah memenuhi syarat dan
*comply* dengan *supertype*-nya akan dijelaskan pada sub-bab di bawah.

Desain dengan Kontrak (Design By Contract)
------------------------------------------

**Design by contract** adalah salah satu cara untuk menuliskan software yang bisa
diandalkan (*reliable*) dengan menggunakan kontrak eksplisit. Namun, Go tidak
mendukung konsep ini dan hanya beberapa bahasa pemrograman yang secara *native*
menggunakannya. Meskipun demikian, konsep *design by contract* bisa diterapkan dengan
bahasa pemrograman apapun termasuk Go, dengan menggunakan *logical contract* dan bukan
pada level *syntax*.

Kita akan membuat sebuah contoh berupa interface ``Saver``
yang secara umum bertugas untuk menyimpan data. Contoh ini akan kita gunakan
untuk menjelaskan apa itu *design by contract*, yang kemudian pada sub-bab selanjutnya
akan digunakan untuk menjelaskan relasi *subtype*.

.. code-block:: go

    // Interface (supertype)
    type Saver interface {
        Save(data []byte) error
    }

Fungsi abstrak :func:`Save` menerima parameter ``data`` bertipe ``[]byte`` dan
mengembalikan ``error`` jika ada. Ada 2 hal yang menjadi konsep utama dalam
prinsip desain menggunakan kontrak:

- **Precondition**: kondisi atau *requirement* yang harus dipenuhi atau bernilai benar
  **sebelum** suatu metode/operasi inti dijalankan sehingga metode tersebut bisa berjalan
  dengan benar. Pada contoh di atas, maka ``data []byte`` adalah data pada kondisi awal
  yang harus disiapkan sebelum melakukan pemanggilan algoritma inti fungsi :func:`Save`.
  Ini artinya ``data`` tidak boleh bernilai ``nil``.
  Setiap baris kode yang akan memanggil fungsi :func:`Save` harus memastikan bahwa
  ``data`` sudah benar terpenuhi, agar fungsi berjalan dengan benar.

  Contoh lainnya, misalnya *method* ``Divide(a,b)``, maka *precondition*-nya adalah
  memastikan bahwa ``b != 0`` (pembagian dengan nol tidak memungkinkan).
  
  Fungsi ``Withdraw(amount)`` pada sebuah ``BankAccount``, maka *precondition*-nya misalnya
  adalah memastikan ``amount <= balance`` (tidak boleh melebihi saldo yang ada).

  Secara formal, proses *precondition* merupakan tanggung jawab dari kode yang memanggil
  fungsi/metode tersebut atau terjadi sebelum fungsi dipanggil. Namun, karena Go tidak
  mendukung *design by contract* secara *native*, make validasi data, misalnya apakah
  ``b != 0`` bisa dilakukan di dalam fungsi itu sendiri (di dalam fungsi :func:`Divide`)
  dan mengembalikan misalnya ``error`` jika kondisi tidak terpenuhi. Cara seperti ini
  juga meminimalkan *bug* sekaligus mengurangi repetisi kode validasi di luar fungsi.
- **Postcondition**: adalah jaminan terhadap *state* atau hasil akhir dari suatu
  sistem **setelah** suatu metode/operasi dieksekusi, dengan asumsi bahwa *precondition* sudah
  benar. Variabel ``error`` dipakai sebagai penanda *state* apakah benar data sudah tersimpan
  atau sebaliknya. Dalam contoh di atas, *postcondition* adalah setelah melakukan pemanggilan fungsi
  :func:`Save`. Jika ``error`` bernilai ``nil``, maka artinya data sudah tersimpan secara
  permanen.

  Contoh lainnya, misalnya ``Divide(a,b)``, maka *postcondition* adalah nilai hasil dari
  operasi ``a/b``.

  Sedangkan fungsi ``Withdraw(amount)``, maka *postcondition* adalah saldo akhir
  setelah dilakukan pengurangan sejumlah ``amount``.

  *Postcondition* berlangsung di dalam metode/operasi itu sendiri.

Contoh sederhana di atas membawa kita pada kesepakatan bahwa ``interface`` digunakan sebagai
kontrak desain. Setiap ``struct`` yang mau mengimplementasikan kontrak tersebut dengan
*method*-nya masing-masing, harus sepakat dan menghargai kontrak tersebut, baik itu
nama fungsi, tipe masukan, dan tipe luaran.

Karena Go secara *native* tidak mendukung *design by contract*,
proses *precondition* dan *postcondition* bisa juga diekspresikan sebagai:

- *Unit testing*: sebagai kumpulan operasi yang akan mengecek apakah suatu fungsi gagal
  atau sukses untuk menyepakati kontrak, tentunya selain menguji logika algoritmanya
- Dokumentasi, baik sebagai baris komentar atau baris dokumentasi yang lebih komprehensif.
  Dokumentasi ini diharapkan bisa menjadi panduan *programmer* untuk menyepakati
  *logical contract* yang dibuat. Misalnya adalah mendokumentasikan apa yang diharapkan dari
  sebuah fungsi.

Cara-cara di atas untuk mendefinisikan kontrak, termasuk dengan fitur ``interface`` yang
disediakan oleh Go, kita sebut sebagai *logical contract*,
yaitu kontrak implisit yang berdasar pada kesepakatan di antara programmer.
Misalnya ``Sqrt(a float) float`` **tidak hanya** berjanji untuk mengembalikan nilai
berupa ``float``, namun juga berjanji bahwa: akan mengecek bahwa bahwa nilai masukan
haruslah non-negatif, yang dikembalikan juga harus bernilai positif, dan sebagainya.
Semua kontrak ini harus dituangkan ke dalam dokumentasi.


Definisi dan Relasi Subtyping
-----------------------------

Selanjutnya, kita akan melihat apakah sebuah *subtype* memiliki perilaku (*behaviour*)
yang sejalan dari *subtype* lainnya, agar memenuhi kriteria bahwa 
"setiap subtype bisa digantikan oleh subtype lainnya tanpa
memengaruhi keakuratan dan kebenaran dari program tersebut".

Liskov menjelaskan ada 2 definisi atau pendekatan fundamental *subtyping*:

1. **Constraint Rule**
2. **Extension Map**

Kita akan membahas lebih lanjut di bawah ini, kenapa 2 pendekatan ini menjadi penting
dalam pendefinisian prinsip Liskov secara keseluruhan.

Constraint Rule
^^^^^^^^^^^^^^^

**Constraint Rule** menyatakan bahwa spesifikasi dari *subtype* (S) harus merupakan
subset dari spesifikasi *supertype* (T). Sederhananya, *subtype* tidak boleh melakukan
sesuatu yang tidak dilakukan *supertype*.

Dalam konteks bahasa Go, *supertype* hanyalah berupa abstraksi ``interface``
yang tidak mengandung implementasi konkrit atau perilaku dasar.
Sehingga, definisi *constraint rule* di atas bisa kita *extend* sehingga
*subtype* harus menghormati kontrak dan perjanjian perilaku yang didefinisikan
oleh *supertype*.

Meskipun demikian, *subtype* tetap bisa mengimplementasikan
kontrak dengan beberapa penyesuaian, tanpa perlu melanggar
ekspektasi perilaku dari *supertype*, seperti:

- Pelonggaran prasyarat (**weaker precondition**) dengan menerima lebih
  banyak range nilai dibandingkan dengan yang disyaratkan, dan
- Hasil akhir yang lebih ketat (**stronger postcondition**) atau membatasi
  lebih banyak range dibandingkan yang disyaratkan

Kedua implementasi di atas **dianggap tetap memenuhi prinsip Liskov.**

Agar lebih memahami kedua kondisi ini, kita berikan beberapa contoh seperti di bawah.

.. note::
    Contoh 1
    
    Tidak ada perubahan perilaku yang signifikan.
    ``LoggingFileWriter`` meminjam *method* dari ``FileWriter``
    sehingga perilaku kedua *subtype* sama. Hanya ada penambahan
    fungsi *logging* pada ``LoggingFileWriter`` yang tidak menyalahi
    kontrak *behaviour* yang didefinisikan.

.. code-block:: go

    // Kontrak didefinisikan sebagai:
    // - interface (supertype)
    // - dokumentasi behaviour
    type Writer interface {
        // Write harus menuliskan data ke sebuah text file
        // dengan mode append | create dan mode write only
        Write(text string) error
    }

    // Subtype FileWriter yang mengimplementasikan supertype
    type FileWriter struct {
        Filename string
    }
    func (fw *FileWriter) Write(text string) error {
        f, err := os.OpenFile(fw.Filename, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
        if err != nil {
            return err
        }
        defer f.Close()
        _, err = f.WriteString(text)
        return err
    }

    // Subtype LoggingFileWriter
    type LoggingFileWriter struct {
        Base Writer // wraps any Writer, satisfies LSP!
    }
    func (lfw *LoggingFileWriter) Write(text string) error {
        // Selain meminjam FileWriter untuk memenuhi prinsip Liskov,
        // fungsi ini juga menampilkan text sebagai log,
        // namun penambahan ini tidak menyalahi kontrak
        fmt.Println("LOG: Writing to file:", text)
        return lfw.Base.Write(text)
    }

.. note::
    Contoh 2
    
    Sebuah *supertype* (kontrak) didefinisikan untuk memroses sebuah pembayaran,
    lalu ada 2 *subtype* (``EWallet`` dan ``Paypal``) yang mengimplementasikan kontrak.
    *Subtype* pertama, ``EWallet``, mengimplementasikan secara normal
    sesuai kontrak. Namun, *subtype* kedua, ``Paypal`` melakukan
    beberapa perubahan yaitu:

    - Weaker precondition (diperbolehkan dalam LSP)
    - Stronger postcondition (diperbolehkan dalam LSP)

.. code-block:: go

    // Kontrak didefinisikan sebagai:
    // - interface (supertype)
    // - dokumentasi behaviour: setiap fungsi abstrak
    //   mempunyai syarat precondition & postcondition
    type PaymentProcessor interface {
        // Syarat Precondition:
        // - Nilai amount harus positif dan <= 1000
        // Syarat Postcondition:
        // - Kembalikan pesan konfirmasi dengan detail transaksi
        // - Kembalikan error menandakan bahwa transaksi berhasil atau tidak
        ProcessPayment(amount float) (string, error)
    }

    // Subtype EWallet dengan implementasi normal precondition
    // dan normal postcondition
    type EWallet struct {
        Balance float
    }
    func (eWallet *EWallet) ProcessPayment(amount float) (string, error) {
        // Normal Precondition
        // - Nilai amount harus positif dan <= 1000
        // Jika tidak, maka akan mengambalikan error (postcondition)
        if (amount < 0) || (amount > 1000) {
            return "", fmt.Errorf("amount should be positive and <= 1000")
        }

        eWallet.Balance = eWallet.Balance - amount

        // Normal Postcondition
        // - Kembalikan pesan konfirmasi dengan detail transaksi
        // - Kembalikan error menandakan bahwa transaksi berhasil atau tidak
    	txId := "tx_" + strconv.FormatInt(time.Now().UnixNano(), 10)
        txMsg := fmt.Sprintf("Transaksi %s berhasil", txId)
        return txMsg, nil
    }

    // Subtype Paypal dengan implementasi weaker precondition
    // dan stronger postcondition
    type Paypal struct {
        Balance float
    }
    func (paypal *Paypal) ProcessPayment(amount float) (string, error) {
        // Weaker Precondition
        // - Nilai amount harus positif dan <= 5000 (nilai maximum lebih besar)
        // Jika tidak, maka akan mengambalikan error (postcondition)
        if (amount < 0) || (amount > 5000) {
            return "", fmt.Errorf("amount should be positive and <= 5000")
        }

        // Stronger Postcondition
        // - Kembalikan error jika nilai Balance tidak cukup sebelum
        //   melakukan transaksi
        if paypal.Balance < amount {
            return "", fmt.Errorf("Balance is insufficient")
        }

        eWallet.Balance = eWallet.Balance - amount

        // Stronger Postcondition
        // - Kembalikan pesan konfirmasi dengan detail transaksi dan sisa saldo
    	txId := "tx_" + strconv.FormatInt(time.Now().UnixNano(), 10)
        txMsg := fmt.Sprintf("Transaksi %s berhasil, saldo akhir %v",
            txId, paypal.Balance)
        return txMsg, nil
    }

Pada contoh 2 di atas, ada 2 *stronger postcondition*.
Namun, secara umum akan dianggap sebagai
*stronger postcondition* dengan syarat setidaknya ada 1 *stronger postcondition*.

Sebaliknya, modifikasi bertolak berlakang dengan kedua prinsip di atas,
**dianggap tidak memenuhi prinsip Liskov** yaitu:

- Pengetatan prasyarat (**stronger precondition**) dengan membatasi 
  range nilai awal dibandingkan dengan yang disyaratkan, dan
- Hasil akhir yang lebih longgar (**weaker postcondition**) atau 
  lebih sedikit dibandingkan yang disyaratkan

.. note::
    Contoh 3

    Kontrak dan ``interface`` pada contoh 3 ini diambil sama dengan contoh 2.

    Ada 1 buah *subtype* yang mengimplementasikan kontrak yang didefinisikan
    pada ``interface``, namun menyalahi aturan Liskov:

    - Stronger precondition (tidak diperbolehkan dalam LSP)
    - Weaker postcondition (tidak diperbolehkan dalam LSP)

.. code-block:: go

    // Kontrak didefinisikan sebagai:
    // - interface (supertype)
    // - dokumentasi behaviour: setiap fungsi abstrak
    //   mempunyai syarat precondition & postcondition
    type PaymentProcessor interface {
        // Syarat Precondition:
        // - Nilai amount harus positif dan <= 1000
        // Syarat Postcondition:
        // - Kembalikan pesan konfirmasi dengan detail transaksi
        // - Kembalikan error menandakan bahwa transaksi berhasil atau tidak
        ProcessPayment(amount float) (string, error)
    }

    // Subtype ApplePay dengan implementasi stronger precondition
    // dan weaker postcondition
    type ApplePay struct {
        Balance float
    }
    func (applePay *ApplePay) ProcessPayment(amount float) (string, error) {
        // Stronger Precondition
        // - Nilai amount harus positif dan <= 400 (dibatasi lebih kecil dari kontrak)
        if (amount < 0) || (amount > 400) {
            return "", fmt.Errorf("amount should be positive and <= 400")
        }

        applePay.Balance = applePay.Balance - amount

        // Weaker Postcondition
        // - Kembalikan pesan konfirmasi namun tanpa detail transaksi
        // Normal postcondition
        // - Kembalikan error menandakan bahwa transaksi berhasil atau tidak
        txMsg := fmt.Sprint("Transaksi berhasil")
        return txMsg, nil
    }

Contoh 3 di atas menjawab kekeliruan yang disinggung di awal topik ini,
yaitu bahwa menggunakan *interface* berarti sudah memenuhi prinsip Liskov,
padahal belum tentu.

Ada syarat tambahan yaitu *subtype*/*implementor* tidak boleh
melanggar kontrak yang didefinisikan oleh *supertype*; atau penyesuaian
kontrak tetap memungkinkan selama dalam batasan yang diijinkan yaitu bisa
berupa **weaker precondition** atau **stronger postcondition**.

Dalam prinsip subtitusi Liskov, ini menjadi penting,
karena dengan mengacu pada *supertype* dan kontrak yang
didefinisikan, maka apapun *subtype*-nya, tetap akan bisa digantikan oleh
*subtype* lainnya meskipun berbeda implementasinya.

Extension Map
^^^^^^^^^^^^^

Definisi dan pendekatan *Extension Map* lebih formal karena disusun secara
presisi dengan formula matematika.
Kita tidak akan membahasnya secara matematis, namun akan kita
ulas dari sisi praktis dan implementasinya dalam *programming*. 

*Extension map* menyatakan bahwa setiap objek *subtype* ``S`` bisa dilihat
seolah-olah bahwa ini adalah merupakan *supertype* ``T``:
untuk setiap nilai (*state*) dari *subtype* ``S``, akan ada caranya sehingga
kita bisa "melihat bahwa *subtype* ini adalah *supertype*" dan apapun yang
didefinisikan oleh supertype, maka subtype akan berperilaku secara eksak,
meskipun *subtype* mempunyai ekstra *method*.

Poin penting yang perlu dicatat adalah:

- Semua perilaku dari metode (*method*) yang didefinisikan oleh *supertype*,
  harus cocok dengan apa yang akan terjadi pada *subtype* ``S``, bahkan sekalipun
  jika *subtype* ``S`` memiliki *method* baru yang tidak didefinisikan di kontrak.
- Definisi ini lebih ketat (*stricter*) dibandingkan cuma sekedar nama
  metode (*method*) atau *signature*. Ini adalah mengenai apakah
  *subtype* ``S`` tidak terlihat berbeda dari "kacamata" *supertype* ``T``.

Mari kita lihat contoh sederhana untuk memahami konsep ini.

.. note::
    Sebuah kontrak didefinisikan melalui *supertype* (*interface*) ``Switch``
    dan mempunyai 3 *methods*.
    Ketika menganalisis kontrak ini, secara naluriah, misalnya kita membutuhkan
    field ``state`` yang umumnya akan diimplementasikan oleh *subtype*.

    Implementasi dari *subtype* ``RegularSwitch`` hanya mempunyai field dan
    memodifikasinya sesuai kebutuhan kontrak saja.
    Namun, *subtype* ``TimedSwitch`` mempunyai tambahan field ``timerStart``
    yang nantinya digunakan untuk menghitung durasi.

.. code-block:: go

    // Kontrak dalam bentuk interface
    // dan dokumentasi behaviour
    type Switch interface {
        TurnOn()      // menghidupkan switch
        TurnOff()     // mematikan switch
        IsOn() bool   // mengembalikan apakah state switch On
    }

    // Subtype RegularSwitch mempunyai field state
    // guna menyesuaikan dengan kebutuhan kontrak
    type RegularSwitch struct {
        state bool
    }

    // Implementasi kontrak dengan memodifikasi field state
    // dan mengembalikan nilainya
    func (s *RegularSwitch) TurnOn()  { s.state = true }
    func (s *RegularSwitch) TurnOff() { s.state = false }
    func (s *RegularSwitch) IsOn() bool { return s.state }

    // Subtype TimedSwitch selain mempunyai field state,
    // juga mempunyai tambahan field timerStart  
    type TimedSwitch struct {
        state      bool
        timerStart time.Time
    }

    // Implementasi kontrak TurnOn, TurnOff, dan IsOn
    // dengan memodifikasi state dan juga memodifikasi
    // tambahan field timerStart
    func (s *TimedSwitch) TurnOn()  {
        s.state = true
        s.timerStart = time.Now()
    }
    func (s *TimedSwitch) TurnOff() {
        s.state = false
    }
    func (s *TimedSwitch) IsOn() bool { return s.state }

    // Tambahan method yang akan mengembalikan durasi waktu,
    // namun tidak ada di kontrak Switch
    func (s *TimedSwitch) TimeSinceOn() time.Duration {
        if s.state {
            return time.Since(s.timerStart)
        }
        return 0
    }

Dari kacamata *supertype* ``Switch``, maka ``RegularSwitch`` dan ``TimedSwitch``
sudah memenuhi kontraknya dan bisa dilihat bahwa perilakunya sesuai dengan ekspektasi
yang diharapkan oleh *supertype*, bahkan meskipun ketika ``TimedSwitch`` memiliki
*method* yang tidak didefinisikan di kontrak.
Di sini *supertype* bisa mengabaikan tambahan fitur (*method*) selama semua *method*
kontraknya terpenuhi.

Hal ini bisa dikategorikan ke dalam **extension map** dan tetap memenuhi prinsip Liskov.
Semua *client* yang menggunakan *interface* ``Switch`` misalnya kode di bawah ini, tidak
akan bisa membedakannya karena perilakunya sama persis, yaitu misal setelah memanggil
fungsi :func:`TurnOn`, maka :func:`IsOn` akan selalu bernilai ``true``.

.. code-block:: go

    func TurnOnSwitch(switch Switch) {
        switch.TurnOn() // akan selalu membuat IsOn() menjadi true
    }

    rs := RegularSwitch{}
    ts := TimedSwitch{}

    TurnOnSwitch(rs)
    fmt.Println(rs.IsOn()) // true

    TurnOnSwitch(ts)
    fmt.Println(ts.IsOn()) // true

Subtitusi Liskov artinya parameter ``switch`` pada fungsi
:func:`TurnOnSwitch` bisa diganti oleh beragam objek yang mengimplementasikan kontrak
``Switch``.

Sebagai tambahan informasi, istilah **extension** mengacu pada subtype yang memiliki
kemungkinan untuk menambah *state* dan *method* dari *supertype* —
*more fields, more methods, more stuff*. Sedangkan **mapping** mengacu pada pemahaman
konseptual bahwa *method* pada *subtype* adalah benar representasi dari *supertype*.

Dengan dasar ini, *extension map* kemudian digunakan untuk mengecek: jika kita
mengabaikan semua fungsi tambahan pada *subtype* dan hanya melihat kontrak original,
apakah *subtype* ini akan selalu berperilaku seperti yang diharapkan oleh *supertype*?

Jika jawabannya ya, maka tambahan fitur/*method* bukanlah masalah.
Subtitusi (*subtype*) aman dilakukan.

Kesimpulan
^^^^^^^^^^

Kedua definisi yang menjadi dasar *Liskov Substitution Principle*, yaitu *Constraint Rule*
dan *Extension Map* merupakan cara untuk mengekspresikan nilai dan inti yang sama.
Yang satu bukanlah alternatif yang lain, namun merupakan 2 definisi yang ekivalen sama.

Secara matematis, Liskov membuktikan bahwa 2 definisi ini adalah sama dan ekivalen,
yang berarti jika sebuah *subtype* memenuhi satu definisi, maka ini juga memenuhi definisi
yang lainnya (berlaku prinsip jika dan hanya jika).
Kita sebagai *programmer* hanya perlu memilih pendekatan mana yang ingin digunakan.

*Constraint Rule* lebih intuitif dan sederhana dan dilihat dari kacamata client
atau objek *subtype*. Pendekatan ini lebih mudah digunakan oleh *programmer* untuk pekerjaan
sehari-hari.

Sedangkan pendekatan *Extension Map* lebih matematis dan formal,
sangat berguna untuk pembuktian dan tipe dengan *state* kompleks, karena memiliki pertanyaan:
apakah perilaku *subtype* merupakan perilaku yang memenuhi kontrak *supertype*?
Pendekatan ini cocok digunakan misalnya ketika *subtype* banyak menambahkan ekstra *state*
atau ekstra *logic* yang tidak ada di kontrak.

Prinsip Liskov terlihat lebih kompleks dari yang dibayangkan.
Pada Go, prinsip ini jauh lebih dalam dibandingkan hanya sekedar mengimplementasikan sebuah
*interface*. Kepercayaan umum dan banyak artikel menyatakan bahwa mengimplementasikan
*interface* adalah berarti memenuhi LSP. Namun ``interface`` hanyalah bantuan subtitusi
**syntactic** yang disediakan Go pada level *compiling*.

Prinsip Liskov secara fundamental terletak pada subtitusi perilaku, bukan hanya menyamakan
dan mecocokkan *method signature*, tapi mencocokkan ekspektasi dan kontrak yang
telah didefinisikan.

.. rubric:: References

.. [1] H. Liskov, B., & M. Wing, J. (1994). A behavioral notion of subtyping. ACM Transactions on Programming Languages and Systems, 16(6), 1811–1841. https://www.cs.cmu.edu/~wing/publications/LiskovWing94.pdf
.. [2] Solid relevance. (2020, October 18). Clean Coder Blog. https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html

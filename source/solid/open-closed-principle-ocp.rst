Open-Closed Principle (OCP)
===========================

Definisi
--------

**Open-Closed Principle (OCP)** adalah prinsip kedua yaitu O dalam SOLID yang menjadi
dasar dalam mendesain kode yang kuat (*robust*) dan mudah diadaptasi (*adaptable*).
Dengan memahami OCP, kita dengan mudah melakukan modifikasi, terutama menambah fitur,
namun tetap meminimalkan resiko untuk merusak kode yang sudah ada (*breaking change*).

OCP pertama kali dicetuskan oleh Bertrand Meyer dengan mengatakan bahwa **entity software
(seperti modul/module, kelas/class, dan fungsi) sebaiknya terbuka untuk penambahan,
tapi harus tertutup untuk modifikasi** -- *open for extension, but closed for modification*.

Penjelasan detailnya:

- **Open for extension**: Programmer tetap bisa menambahkan fitur atau fungsi ke dalam modul
- **Closed for modification**: Programmer tidak perlu untuk melakukan perubahan pada kode
  eksisting, yang mungkin sudah teruji (tested), jika semisalnya perlu menambahkan fitur baru

Implementasi
------------


Interface
^^^^^^^^^

Implementasi OCP paling *idiomatic* pada Go bisa dilakukan dengan menggunakan ``interface``.
Sebagai contoh, misalkan kita sudah memiliki kode untuk menghitung luasan berbagai jenis
bentuk, misal ada lingkaran dan persegi. Dengan menerapkan OCP, setiap kali kita
menambahkan bentuk baru, misalkan segitiga, kita tidak perlu bersinggungan dengan bagian
kode yang digunakan untuk menghitung luasan bentuk lain.


.. warning::
    Kode di bawah tidak memenuhi OCP karena algoritma dan logika untuk perhitungan
    berbagai luasan area dicampur ke dalam satu blok kode. Setiap kali ingin menambahkan
    logika untuk luasan bentuk yang baru, programmer harus mengedit fungsi :func:`Area`.

.. code-block:: go

    type ShapeType int
    const (
        CircleType ShapeType = iota
        SquareType
    )

    type Shape struct {
        Type   ShapeType
        Radius float64
        Side   float64
    }

    func Area(s Shape) float64 {
        switch s.Type {
        case CircleType:
            return math.Pi * s.Radius * s.Radius
        case SquareType:
            return s.Side * s.Side
        // Bagaimana jika ingin menambahkan bentuk segitiga?
        // Programmer harus mengedit fungsi ini!
        }
        return 0
    }

Dengan menggunakan ``interface`` untuk membuat kontrak implementasi, programmer
bisa menambahkan fungsi baru sesuai dengan kebutuhan tanpa perlu menyentuh kembali
kode yang sudah ada.

.. note::
    Alih-alih menggunakan ``switch-case`` seperti contoh di atas, kita memecah
    masing-masing algoritma luasan menjadi beberapa ``struct`` sesuai bentuknya
    dan memiliki method :func:`Area` yang mengimplementasikan ``interface Shape``.

.. code-block:: go

    type Shape interface {
        Area() float64
    }

    type Circle struct {
        Radius float64
    }
    func (c Circle) Area() float64 {
        return math.Pi * c.Radius * c.Radius
    }

    type Square struct {
        Side float64
    }
    func (s Square) Area() float64 {
        return s.Side * s.Side
    }

    // Jika ingin menambahkan bentuk baru, programmer tidak perlu memodifikasi
    // algoritma dan fungsi yang sudah ada.
    // Programmer cukup menambahkan struct baru dan fungsi Area() - open for extension,
    // namun tidak perlu memodifikasi apapun dalam fungsi TotalArea() - closed for modification

    func TotalArea(shapes []Shape) float64 {
        total := 0.0
        for _, s := range shapes {
            total += s.Area() // Polymorphic call
        }
        return total
    }

.. note::

    Penggunan ``switch-case`` dalam contoh awal yang tidak memenuhi OCP, bukan berarti
    mendorong pembaca untuk tidak menggunakan ``switch-case``.
    
    Kapan harus menggunakan ``switch-case``?

    - Jika tipe yang ingin dicek bersifat tetap (*fixed*) dan terbatas: misalnya untuk
      meng-*handle* state dari suatu protokol, instruksi, state dari *file parsing*, atau
      token, di mana domainnya memang terbatas dan tidak sering berubah. Konsep ini
      cukup sering misalnya dalam aplikasi *tokenizer*.
      
      .. code-block:: go

        switch ch {
            case '+':
                // Handle plus token
            case '-':
                // Handle minus token
            ...
        }

    - Performa atau kejelasan kode (*clarity*) menjadi prioritas utama: seringkali kode
      akan lebih jelas dibaca, lebih cepat performanya, dan mudah dipelihara jika
      menggunakan ``switch``, selama tidak ada perubahan yang signifikan dalam waktu
      yang lama. Dalam kasus ini, ``interface`` sebagai abstraksi akan menambah *overhead*
      dibanding tanpa menggunakannya.
    - Secara umum, menambahkan ``case`` baru tidak sekedar menambahkan fitur, tapi juga
      mengubah *requirement*.

    Sebaliknya, kapan harus menghindari ``switch-case``?

    - Akan ada penambahan tipe atau ``case`` baru secara reguler (misal bentuk baru pada
      aplikasi menggambar).
    - Perilaku (*behaviour*) dari masing-masing ``case`` tidak berhubungan sama sekali,
      kompleks, dan mudah berubah secara individu (independen)
    - Kode dan logika di dalamnya akan digunakan kembali (*reuse*) atau di-*extend* kembali
      di tempat lain tanpa mengubah kode original

    *Rule of thumb*: jika konteksnya *extensibility* adalah hal yang diharapkan,
    ``switch-case`` tidak direkomendasikan.
    Aturan ini juga berlaku untuk penggunaan ``if-else``.

Jika diperhatikan lebih lanjut, penggunaan ``interface`` dalam memenuhi prinsip OCP
sangat mirip dengan penggunaan ``interface`` sebagai solusi untuk mengatasi perubahan
yang sifatnya dinamis pada *Single Responsibility Principle (SRP)*.
Namun terdapat beberapa perbedaan terutama ditinjau dari tujuan kenapa ``interface``
digunakan.

Pada SRP, tujuan utama ``interface`` adalah sebagai **pembatas untuk mencipatakan kode yang
bersih dan pemisahan tugas/tanggung jawab**.

Sedangkan pada OCP, penggunaan ``interface`` adalah **sebagai alat untuk membuat
kode kita mudah di-extend (penambahan fungsi baru) tanpa merubah kode lama yang sudah
stabil**.

Meskipun pada akhirnya, kita akan sering menggunakan ``interface`` dan memenuhi kedua
prinsip, namun dengan tujuan yang berbeda. 

Tipe Fungsi/Fungsi Orde Tinggi (Function Types/Higher-Order Functions)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::
    Penggunaan kata **Function Types** dan **Higher-Order Functions** akan lebih sering
    digunakan dibandingkan Tipe Fungsi dan Fungsi Orde Tinggi karena minimnya referensi
    dalam Bahasa Indonesia yang menggunakan kata ini.

**Higher-order function** atau fungsi orde tinggi adalah istilah yang merujuk pada fungsi
yang bisa menerima fungsi sebagai parameter (bukan hanya variabel saja) dan juga
bisa mengembalikan fungsi (*return function(s)*).
Istilah yang terkait adalah *function as data*, di mana fungsi memiliki sifat seperti
tipe data primitif seperti number atau string, yaitu bisa disimpan ke alam variabel,
dilewatkan sebagai parameter fungsi, dan dikembalikan dari fungsi.
Tidak semua bahasa pemrograman memiliki fitur ini, namun Go mendukung fitur ini.

Sedangkan **functions type** adalah tipe data yang berupa fungsi, bukan variabel.

Sebagai contoh di bawah, ``Comparator`` adalah *function type* yang merupakan definisi
dari ``func(a, b int) bool``.

.. code-block:: go

    type Comparator func(a, b int) bool

    func Sort(nums []int, cmp Comparator) {
        // gunakan cmp untuk membandingkan dan mengurutkan nums
    }

Bagaimana *higher-order function* bisa membantu programmer menerapkan OCP? Dengan
cara menyerahkan mekanisme implementasi fungsi kepada pengguna (programmer) modul
dan memasukkan fungsi ini sebagai parameter.

.. note::
    Pengguna akhir (programmer) yang menggunakan fungsi :func:`Sort` bebas memasukkan
    fungsi yang diinginkan sesuai kebutuhannya. Fungsi :func:`Sort` merupakan
    ``higher-order function``, sedangkan ``Comparator`` adalah ``function type``
    dengan definisi ``func(a, b int) bool``

.. code-block:: go

    type Comparator func(a, b int) bool

    // Algoritma pengurutan menggunakan Bubble Sort
    func Sort(nums []int, cmp Comparator) {
        // cmp adalah fungsi untuk membandingkan 2 integer
        // yang diberikan sesuai kebutuhan
        for i := 0; i < len(nums); i++ {
            for j := 0; j < len(nums)-1; j++ {
                if !cmp(nums[j], nums[j+1]) {
                    nums[j], nums[j+1] = nums[j+1], nums[j]
                }
            }
        }
    }

    func main() {
        numbers := []int{3, 9, 10, 1, 6}
        // Programmer harus memasukkan fungsi pembanding sebagai parameter
        // Dalam kasus ascending, pembandingnya adalah a < b
        Sort(numbers, func(a, b int) bool { return a < b }) // ascending
        // akan menampilkan [1 3 6 9 10]
        fmt.Println(numbers)

        numbers = []int{3, 9, 10, 1, 6}
        // Sementara untuk kasus descending, pembandingnya adalah a > b
        Sort(numbers, func(a, b int) bool { return a > b }) // descending
        // akan menampilkan [10 9 6 3 1]
        fmt.Println(numbers)
    }

Contoh lain akan disajikan untuk memberikan gambaran bagaimana kita bisa mendapatkan
fungis dengan *extensibility* dan memenuhi OCP.


.. note::
    Fungsi :func:`Select` adalah ``higher-order function`` dan ``Filter``
    adalah ``function type`` dengan definisi ``func(a int) bool``.
    Inti dari fungsi :func:`Select` adalah mengumpulkan semua nilai yang lolos
    dari algoritma fungsi seleksi (``filter``). Algoritma fungsi seleksi ``filter``
    diberikan oleh pengguna akhir (programmer) sesuai kebutuhannya.

.. code-block:: go

    type Filter func(a int) bool

    func Select(nums []int, filter Filter) []int {
        filtered := []int{}
        for _, num := range nums {
            if filter(num) {
                filtered = append(filtered, num)
            }
        }
        return filtered
    }

    // DividedByThree, DividedByFive, Odd adalah contoh fungsi yang memenuhi
    // function type Filter dan bisa ditambah di kemudian hari tanpa perlu
    // menyentuh algoritma inti
    func DividedByThree(val int) bool {
        return val%3 == 0
    }
    func DividedByFive(val int) bool {
        return val%5 == 0
    }
    func Odd(val int) bool {
        return val%2 != 0
    }

    func main() {
        numbers := []int{10, 3, 8, 7, 9, 10, 1, 6, 20, 12, 8, 16}

        filter1 := Select(numbers, DividedByThree) // memilih semua angka kelipatan 3
        // akan menampilkan [3 9 6 12]
        fmt.Println(filter1)

        filter2 := Select(numbers, DividedByFive) // memilih semua angka kelipatan 5
        // akan menampilkan [10 10 20]
        fmt.Println(filter2)

        filter3 := Select(numbers, Odd) // memilih semua bilangan ganjil
        // akan menampilkan [3 7 9 1]
        fmt.Println(filter3)
    }

Plugin/Registry Patterns
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Plugin** atau disebut juga **Registry Patterns** digunakan dalam pengembangan kode,
di mana pengguna (programmer) bisa **mendaftarkan (register)
satu atau lebih perilaku atau tipe untuk meng-extend fungsionalitas tanpa perlu
merubah kode inti, dengan memanfaatkan objek bersama (shared objects) yang dikelola
secara terpusat** [1]_.

Contoh *registry pattern* sering dipakai sebagai mekanisme HTTP handler seperti contoh
di bawah.

.. note::
    Fungsi http.HandleFunc menerima 2 parameter: routing path dan fungsi untuk
    menghandlenya.

.. code-block:: go

    http.HandleFunc("/foo", fooHandler)
    http.HandleFunc("/bar", barHandler)
    http.HandleFunc("/user", getUserHandler)

Contoh di atas jika diperhatikan lebih detail, ternyata menggunakan teknik
*higher-order function* di belakangnya. Apa yang membedakan?
*Plugin* atau *registry pattern* tidak langsung mengeksekusi semua fungsi, namun
hanya mendaftarkan semua kemungkinan *behaviour* lewat fungsi.
Pada contoh, ada 3 fungsi yang didaftarkan, namun bukan berarti ketiga fungsi akan
dijalankan secara prosedural baris per baris. Jika ada request dari user untuk
mengakses fungsi /bar, maka hanya fungsi :func:`barHandler` saja yang dijalankan.
Demikian juga dengan fungsi lainnya, dijalankan saat *routing path*-nya sesuai.

Ini sedikit berbeda dengan contoh pada bagian `Higher-Order Function` di mana
fungsi akan dijalankan secara spontan dan sifatnya tunggal (*single operation*)
di mana jika ingin mengulang operasi yang sama, kita perlu menduplikasi
baris kode yang akan diulang.
Hal ini tidak terjadi pada *registry pattern* di mana cukup mendefinisikannya satu
kali dan saat dibutuhkan **registry** akan menghandlenya meskipun
*routing path*-nya dipanggil berulang kali.

Selain digunakan untuk meng-*handling* HTTP, *plugin* juga sering digunakan pada
konfigurasi, *framework* untuk *logging*, dan *middleware*, biasanya untuk merespon
*event*, *input*, atau *routing*. **Registry**, selain menyimpan fungsi, tentu saja
bisa digunakan untuk menyimnpan *struct*, *object*, atau tipe data lainnya.

Di balik layar, *registry* umumnya menggunakan *slice* atau *map*. Secara sederhana,
fungsi-fungsi yang diregister disimpan ke dalam sebuah *slice/map*.

.. note::    
    Fungsi :func:`RegisterPreprocessor` digunakan untuk menambahkan fungsi-fungsi
    ke dalam sebuah slice yang bisa di-extend (ditambahkan) kapan saja.

.. code-block:: go  

    // Jika menggunakan slice
    type Middleware func(next http.Handler) http.Handler
    var middleware []Middleware
    func RegisterMiddleware(mw Middleware) {
        middleware = append(middleware, mw)
    }

    finalHandler := YourHandler() 
    // Apply all middleware in reverse order (common pattern)
    for i := len(middleware) - 1; i >= 0; i-- {
        finalHandler = middleware[i](finalHandler)
    }


    // Jika menggunakan map
    type Player struct {...}
    var build = map[string]func() Player{}
    func RegisterGamePlayer(pType string, p func() Player) {
        build[pType] = p
    }
    // meregister Player dilakukan satu kali
    RegisterGamePlayer("knight", func() Player{
        return Player{...}
    })
    // menjalankan dan memanggil fungsi on-the-fly sesuai kebutuhan pada saat permainan
    // berlangsung.
    knight   := build["knight"]()
    magician := build["magician"]()

Fungsi ``http.HandleFunc`` pada contoh sebelumnya juga menerapkan hal yang serupa,
namun lebih kompleks karena menggunakan sebuah *tree*, alih-alih *slice/map*.
Jika menelusuri fungsi ini lebih lanjut, kita akan melihat potongan kode seperti di bawah.

.. note::    
    Snippet kode diambil dari package ``net/http`` file ``server.go`` pada fungsi
    :func:`(mux *ServeMux) registerErr`

.. code-block:: go

    // menambahkan routing path dan handler ke sebuah decision tree
    mux.tree.addPattern(pat, handler)

    // Menambahkan index sebagai mekanisme conflict detection.
    // Di balik layar, index menggunakan map untuk menyimpan index, seperti:
    // - segments map[routingIndexKey][]*pattern
    // - multis []*pattern
    mux.index.addPattern(pat)

Maps atau Tabel Pencarian (Lookup Tables) dengan Tipe Fungsi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cara ini mirip dengan *Higher-Order Function* dan *Registry Pattern*,
bahkan bisa dikatakan beririsan.
Alih-alih menggunakan sebuah *registry object*, cara ini justru memanfaatkan *maps*
secara langsung.

.. note::
    *Maps* lebih sederhana karena bisa digunakan secara langsung tanpa perlu membutuhkan
    tipe data yang lebih kompleks seperti *struct* untuk membuat sebuah *registry*.

.. code-block:: go

    var mathOps = map[string]func(int, int) int{
        "add": func(a, b int) int { return a + b },
        "sub": func(a, b int) int { return a - b },
    }

    func Operate(op string, a, b int) int {
        return mathOps[op](a, b)
    }

    // Tambahkan operasi baru tanpa perlu mengubah fungsi Operate
    mathOps["mul"] = func(a, b int) int { return a * b }

    // Contoh implementasi
    result := mathOps["mul"](8, 10) // result = 80

    // Kita bahkan bisa menambahkan fungsi baru di tengah-tengah operasi
    mathOps["modulo"] = func(a, b int) int { return a % b }

Untuk fungsi sederhana, *map* lebih mudah diimplementasikan dibandingkan *plugin* dan
*registry pattern*. *Maps* juga cocok digunakan untuk sesuatu yang bersifat dinamis
(*dynamic dispatch by key*), seperti contoh di atas.
Sedangkan *registry patttern* lebih sering diimplementasikan jika kode
digunakan sebagai paket (*package*) atau kerangka kerja (*framework*) yang umumnya
digunakan pada plugin, *middleware*, atau *hook*, di mana fungsi yang digunakan cenderung
statik atau tidak berubah.

Configuration/Metadata-Driven Logic
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Selain dengan merekayasa kode program, konsep OCP juga bisa diaplikasikan
pada level konfigurasi, seperti pada konfigurasi file, *environment variable*, atau
*metadata* yang nantinya akan diolah oleh program inti.

Misalkan saja, aplikasi finansial yang sedang kita bangun hanya bisa digunakan oleh
pengguna berusia 18 tahun ke atas sesuai peraturan undang-undang. Kita mendefinisikan *rule*
ke dalam sebuah konfigurasi, yang disimpan ke dalam server terpusat. Jika suatu saat
peraturan perundangan berganti dengan membolehkan bahwa usia yang boleh adalah 17 tahun
ke atas, maka kita hanya perlu mengganti konfigurasi ini dan aplikasi akan menyesuaikan
dengan *rule* yang terbaru.

.. note::
    Contoh konfigurasi yang bisa disimpan sebagai file, disimpan di database, atau di
    media lainnya. Format data (*rule*) dan tipenya dibuat sesuai kebutuhan.

.. code-block:: go

    {
        "rules": [
            {
                "field": "age",
                "operation": "gt",
                "value": 18
            }
        ]
    }

.. note::
    Contoh kode untuk memroses konfigurasi di atas jika dibaca sebagai file.
    *Rule* diubah dengan memodifikasi file konfigurasi di atas, tidak perlu mengganti
    kode inti di bawah. Catatan: implementasi detail penggunaan rule tidak dijelaskan
    lebih lanjut.

.. code-block:: go

    type Rule struct {
        Field     string `json:"field"`
        Operation string `json:"operation"`
        Value     int    `json:"value"`
    }

    type Config struct {
        Rules []Rule `json:"rules"`
    }

    func main() {
        // Buka config file
        file, err := os.Open("config.json")
        if err != nil {
            ...
        }
        defer file.Close()

        // Decode the JSON
        var config Config
        decoder := json.NewDecoder(file)
        err = decoder.Decode(&config)
        if err != nil {
            ...
        }

        // Print rules untuk memverifikasi
        for _, rule := range config.Rules {
            fmt.Printf("Field: %s, Operation: %s, Value: %d\n",
                rule.Field, rule.Operation, rule.Value)
        }

        // Gunakan rule sesuai algoritma, implementasi tidak dijabarkan
        ...
    }

Composition
^^^^^^^^^^^

Go tidak mendukung pemrograman berbasis objek (*object oriented programming*).
Namun, bukan berarti Go tidak *powerful*. Alih-alih menggunakan OOP, Go mengandalkan
*composition*. Konsep *composition* dilakukan dengan cara melekatkan
(*embedding*) *struct* ke dalam *struct* atau menggabungkan beberapa *interface*.

Misalkan saja kita mempunya sebuah *struct* untuk mengelola file. Untuk melacak
setiap operasi yang terjadi, misalnya pada saat membuat suatu file,
diperlukan mekanisme *logging*. Fungsi *logging* dipisahkan
ke dalam sebuah *struct* dengan semua fungsionalitasnya (*methods*).
Kita melekatkan ``Logger`` ke dalam ``File`` sehingga ``File`` kini memiliki kemampuan
untuk melakukan *logging*.

.. note::
    Pelekatan *struct* ``Logger`` ke *struct* ``File`` menambah fungsionalitasnya
    tanpa perlu memodifikasi methods pada ``File``.

.. code-block:: go

    type Logger struct{ /* ... */ }
    func (l Logger) Info(msg string) { /* ... */ }
    func (l Logger) Warning(msg string) { /* ... */ }
    func (l Logger) Error(msg string) { /* ... */ }

    type File struct {
        Logger
        filename string
    }
    func (f File) Create() {
        ...
        f.Logger.Info("file created")
    }
    func (f File) Read() {
        f.Logger.Info("reading file")
        ...
        if err != nil {
            f.Logger.Warning("file not exist")
        }
    }

    // Implementasi
    logger := Logger{
        ...
    }
    f := File{
        Logger: logger,
        ...
    }
    f.Create(...)

Contoh fungsionalitas lainnya yang bisa ditambah misalnya ``Metrics`` untuk
mengumpulkan statistik dan performa, ``FileConfig`` untuk perilaku yang menyimpan
konfigurasi file seperti *encoding*, *buffering*, atau *permission*. *Custom package*
seperti ``ErrorHandler`` juga bisa dilekatkan yang berfungsi untuk meng-*handle* dan
membungkus (*wrap*) error dengan menambahankan konteks yang lebih banyak. 
Go sendiri menyediakan *standard package* ``sync.Mutex`` untuk meng-*handling*
*concurrent process* untuk mencegah *race condition*.

Kesimpulan
^^^^^^^^^^

Prinsip OCP **diciptakan untuk mendesain masa depan**: kita mendesain struktur kode kita
sehingga perubahan requirement dan fitur baru tetap bisa beradaptasi (*extendable*),
alih-alih menulis ulang logika inti kode yang sudah ada.


Bagaimana menerapkan OCP dengan pada proyek sebenarnya?

1. Definisikan Interface: tulis apa yang akan menjadi perilaku (*behaviour*) yang kemungkinan
   besar bisa bervariasi atau bisa di-*extend*. Contoh kasus di atas adalah :func:`Area` 
2. Percayakan pada kontrak, bukan tipe implementasi konkrit: buatlah fungsi yang menggunakan
   ``interface``, bukan pada detail implementasi yang berdiri sendiri
3. Gunakan komposisi dan delegasi (*composition and delegation*): biarkan tipe baru
   yang baru di-*extend* (misalnya penambahan segitiga pada contoh di atas) masuk ke
   dalam alur kerja yang sudah ada, alih-alih menulis ulang semua inti logic yang sudah ada
4. Pilih plugin atau hook: ketika memungkinkan, biarkan programmer/user yang menggunakan
   modul kode kita untuk menginjeksi algoritma mereka sendiri melalui ``interface`` atau
   melalui *function type*.

Referensi
---------

.. rubric:: References

.. [1] GeeksforGeeks. (2024, July 1). Registry pattern. GeeksforGeeks. https://www.geeksforgeeks.org/registry-pattern/
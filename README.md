# LARAVEL VALIDATION

## POINT UTAMA

### 1. Instalasi

-   Minimal PHP versi 8 atau lebih,

-   Composer versi 2 atau lebih,

-   Lalu pada cmd ketikan `composer create-project laravel/laravel=v10.2.4 belajar-laravel-validation`.

---

### 2. Membuat Validasi

-   Untuk membuat validator, bisa menggunakan _static_ method di Facade `validator::make()`,

-   Dan saat membuat validator kita harus menentukan data validasi dan rulesnya.

-   kode membuat validator

    ```PHP
    public function testValidator()
    {
        $data = [
            "username" => "admin",
            "password" => "123445"
        ];

        $rules = [
            "username" => "required",
            "password" => "required"
        ];

        $validator = Validator::make($data, $rules);
        self::assertNotNull($validator);
    }
    ```

-   Setelah membuat validator, kita perlu mengecek apalah validasinya sukses atau gagal,

-   Kita bisa menggunakan dua method yang mengembalikan nilai _boolean_,

-   `fails()` akan mengembalikan _true_ jika gagal, _false_ jika sukses,

-   `passed()` akan mengembalikan _true_ jika sukses, _false_ jika gagal.

-   Kode menjalankan validasi

    ```PHP
        self::assertTrue($validator->passes());
        self::assertFalse($validator->fails());
    ```

-   Hasil dari setiap validasi yang sudah dibuat bisa di lihat di folder `storage/logs/laravel/log`.

---

### 3. Error Message

-   Kita bisa mendapatkan detail _error_ dengan menggunakan _function_ `message()`, `errors()` atau `getMessageBag()`.

-   Semuanya akan mengembalikan object yang sama yaitu _class_ `MessageBag`.

-   Kode Error Message

    ```PHP
     $message = $validator->getMessageBag();

        Log::info($message->toJson(JSON_PRETTY_PRINT));
    ```

-   Hasil dari message error bisa di lihat di `storage/logs/laravel.log`.

---

### 4. Validation Exception

-   Validator menyediakan fitur _exception_, dengan menggunakan method `validate()`,

-   Saat memanggil method `validate()`, jika data tidak valid maka akan _throwValidationException_.

-   Kode Validation Exception

    ```PHP
    public function testValidatorValidationException()
    {
        $data = [
            "username" => "",
            "password" => ""
        ];

        $rules = [
            "username" => "required",
            "password" => "required"
        ];

        $validator = Validator::make($data, $rules);
        self::assertNotNull($validator);

        // validation exception
        try {
            $validator->validate();
            self::fail("ValidationException not thrown");
        }catch (ValidationException $exception){
            self::assertNotNull($exception->validator);
            $message = $exception->validator->errors();
            Log::error($message->toJson(JSON_PRETTY_PRINT));
        }
    }
    ```

-   Saat menggunakan validator Laravel juga sudah disediakan rules yang bia digunakan untuk membuat validasi.

-   Link: https://laravel.com/docs/11.x/validation#main-content

---

### 5. Valid Data

-   Laravel validator bisa mengembalikan data yang berisikan hanya _attribute_ yang divalidasi, ini bisa digunakan ketika tidak ingin menggunakan _attribute_ yang divalidasi,

-   Untuk mendapatkandata tersebut, bisa menggunakan return value `validate()`.

-   Kode Valid Data

    ```PHP
     public function testValidatorValidData()
    {
        $data = [
            "username" => "admin@hofi.com",
            "password" => "rahasia",
            "admin" => true,
            "others" => "xxx"
        ];

        $rules = [
            "username" => "required|email|max:100",
            "password" => "required|min:6|max:20"
        ];

        $validator = Validator::make($data, $rules);
        self::assertNotNull($validator);

        try {
            $valid = $validator->validate();
            Log::info(json_encode($valid, JSON_PRETTY_PRINT));
        }catch (ValidationException $exception){
            self::assertNotNull($exception->validator);
            $message = $exception->validator->errors();
            Log::error($message->toJson(JSON_PRETTY_PRINT));
        }
    }
    ```

---

### 6. Validation Message

-   Secara defult, message nya menggunakan bahasa inggris, namun bisa diubah jika perlu,

-   Semua message di Laravel akan disimpan di dala folder `lang/{locate}`,

-   Gunakan perintah `php artisan lang:publish` untuk membuat default message.

-   Kita juga bisa meng-custom message, untuk menambahkan Custom Message untuk _attribute_ di file `validation.php`.

-   Kode Custom Message for Attribute

    ```PHP
     'custom' => [
        'attribute-name' => [
            'rule-name' => 'custom-message',
        ],
        'username' => [
            'email' => 'We only except address for username'
        ]
    ],
    ```

-   Kita bisa menambahkan _inline message_ pada parameter ketiga saat membuat validator menggunakan `Validator::make(data, rules, message)`

-   Kode Inline Message

    ```PHP
    public function testValidatorInlineMessage()
    {
        $data = [
            "username" => "gusti",
            "password" => "gusti"
        ];

        $rules = [
            "username" => "required|email|max:100",
            "password" => ["required", "min:6", "max:20"]
        ];

        // inline message
        $messages = [
            "required" => ":attribute harus diisi",
            "email" => ":attribute harus berupa email",
            "min" => ":attribute minimal :min karakter",
            "max" => ":attribute maksimal :max karakter",
        ];

        $validator = Validator::make($data, $rules, $messages);
        self::assertNotNull($validator);

        self::assertFalse($validator->passes());
        self::assertTrue($validator->fails());

        $message = $validator->getMessageBag();

        Log::info($message->toJson(JSON_PRETTY_PRINT));
    }
    ```

---

### 7. Additional Validation

-   Saat ingin menambahkan validasi tambahan, bisa menggunakan method `after(callback)`, dimana kita bisa menambahkan _function_ `callback` sebagai _parameter_.

-   kode Additional Validation

    ```PHP
     public function testValidatorAdditionalValidation()
    {
        $data = [
            "username" => "gusti@hofi.com",
            "password" => "gusti@hofi.com"
        ];

        $rules = [
            "username" => "required|email|max:100",
            "password" => ["required", "min:6", "max:20"]
        ];

        $validator = Validator::make($data, $rules);
        $validator->after(function (\Illuminate\Validation\Validator $validator){
            $data = $validator->getData();
            if($data['username'] == $data['password']){
                $validator->errors()->add("password", "Password tidak boleh sama dengan username");
            }
        });
        self::assertNotNull($validator);

        self::assertFalse($validator->passes());
        self::assertTrue($validator->fails());

        $message = $validator->getMessageBag();

        Log::info($message->toJson(JSON_PRETTY_PRINT));
    }
    ```

---

## PERTANYAAN & CATATAN TAMBAHAN

-   Catatan Tambahan:

    -   Penggunaan Validator class untuk membuat aturan validasi.
        Cara menjalankan validasi dan menangani hasil validasi.

    -   Validasi data yang bersarang (nested arrays), sering digunakan dalam form yang lebih kompleks.

    -   Validasi langsung pada HTTP request menggunakan Form Request Validation.

    -   Menampilkan halaman kesalahan dan menggunakan direktif Blade untuk menampilkan pesan kesalahan.

---

### KESIMPULAN

-   Laravel Validation memberikan panduan yang jelas dan terstruktur mengenai bagaimana melakukan validasi data dalam framework Laravel. Menjelaskan berbagai aspek dari validasi di Laravel mulai dari pengenalan dasar hingga penggunaan aturan kustom.

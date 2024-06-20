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

-   Link: [Validator](https://laravel.com/docs/11.x/validation#main-content)

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

### 8. Custom Rule

-   Kita bisa membuat custom rule jika memang diperlukan,

-   Gunakan perintah `php artisan make:rule NamaRule`.

-   Nanti akan dibuatkan folder di `app/Rules`.

-   Kode Membuat Rule

    ```PHP
     public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if ($value !== strtoupper($value)) {
            $fail("The $attribute must be UPPERCASE");
        }
    }
    ```

-   Kode Menggunakan Custom Rule

    ```PHP
    public function testValidatorCustomRule()
    {
        $data = [
            "username" => "gusti@hofi.com",
            "password" => "gusti@hofi.com"
        ];

        $rules = [
            "username" => ["required", "email", "max:100",
            "password" => ["required", "min:6", "max:20"()]
        ];

        $validator = Validator::make($data, $rules);
        self::assertNotNull($validator);

        self::assertFalse($validator->passes());
        self::assertTrue($validator->fails());

        $message = $validator->getMessageBag();

        Log::info($message->toJson(JSON_PRETTY_PRINT));
    }
    ```

---

### 9. Translation

-   Saat membuat custom rule, _function_ validate terdapat _parameter_ ke-3 berupa _Closure_,

-   _Closure_ tersebut jika dipanggil, maka mengembalikan object `PotentialTranslatedString`.

-   Link: [Translation](https://laravel.com/docs/11.x/localization#defining-translation-strings)

-   Tambahkan validasi di folder `lang/en/validation.php`.

    ```PHP
        'custom.uppercase' => 'The :attribute field with value :value must be UPPERCASE',
    ```

-   Kode Uppercase Rule

    ```PHP
     public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if ($value !== strtoupper($value)) {
            $fail("validation.custom.uppercase")->translate([
                "attribute" => $attribute,
                "value" => $value
            ]);
        }
    }
    ```

---

### 10. Data Aware & Validation Aware

-   Saat membutuhkan custom rule yang membutuhkan bisa melihat seluruh data yang di validasi, bisa implementasi interface `DataAwareRule`,

-   Dan jika kita butuh object Validator, kita bisa implemantasi interface `ValidatorAwareRule`.

-   Perintah Membuat Rule Registration `php artisan make:rule RegistrationRule`.

-   Kode Registration Rule 1

    ```PHP
    class RegistrationRule implements ValidationRule, DataAwareRule, ValidatorAwareRule
    {
        private array $data;
        private Validator $validator;

        public function setData(array $data): RegistrationRule
        {
            $this->data = $data;
            return $this;
        }

        public function setValidator(Validator $validator): RegistrationRule
        {
            $this->validator = $validator;
            return $this;
        }

        public function validate(string $attribute, mixed $value, Closure $fail): void
        {
            $password = $value;
            $username = $this->data['username'];

            if($password == $username){
                $fail("$attribute must be different with username");
            }
        }
    }
    ```

-   Kode Menggunakan Registration Rule

    ```PHP
    public function testValidatorCustomRule()
    {
        $data = [
            "username" => "gusti@hofi.com",
            "password" => "gusti@hofi.com"
        ];

        $rules = [
            "username" => ["required", "email", "max:100", new Uppercase()], // new Uppercase()
            "password" => ["required", "min:6", "max:20", new RegistrationRule()] // new RegistrationRule()
        ];

        $validator = Validator::make($data, $rules);
        self::assertNotNull($validator);

        self::assertFalse($validator->passes());
        self::assertTrue($validator->fails());

        $message = $validator->getMessageBag();

        Log::info($message->toJson(JSON_PRETTY_PRINT));
    }
    ```

---

### 11. Custom Function Rule

-   Pada kasus kita perlu membuat custom rule, namun jika membuat _class rule_ terlalu berlebihan kita bisa menggunakan _custom function rule_ ketika membuat rule,

-   Gunakan _function_ dimana terdapat 3 _parameter_ `$attribute`, `$value` dan `$fail`.

-   Kode Custom Function Rule

    ```PHP
    public function testValidatorCustomFunctionRule()
    {
        $data = [
            "username" => "gusti@hofi.com",
            "password" => "gusti@hofi.com"
        ];

        $rules = [
            "username" => ["required", "email", "max:100", function(string $attribute, string $value, \Closure $fail){
                if(strtoupper($value) != $value){
                    $fail("The field $attribute must be UPPERCASE");
                }
            }],
            "password" => ["required", "min:6", "max:20", new RegistrationRule()]
        ];

        $validator = Validator::make($data, $rules);
        self::assertNotNull($validator);

        self::assertFalse($validator->passes());
        self::assertTrue($validator->fails());

        $message = $validator->getMessageBag();

        Log::info($message->toJson(JSON_PRETTY_PRINT));
    }
    ```

---

### 12. Nested Array Validation

-   Saat membuat validasi data yang di validasi tidak hanya berformat _key-value_, kadang terdapat _nested array_, seperti _key address_ dimana didalamnya dapat berisi _array_ lagi.

-   Pada kasus data jenis _nested array_, kita bisa membuat Rule menggunakan `.` (titik), misalnya `address.street`, `address.city` dll.

-   Kode Nested Array Validation

    ```PHP
    public function testNestedArray()
    {
        $data = [
            "name" => [
                "first" => "Gusti",
                "last" => "Akbar"
            ],
            "address" => [
                "street" => "Jalan. Mangga",
                "city" => "Jakarta",
                "country" => "Indonesia"
            ]
        ];

        $rules = [
            "name.first" => ["required", "max:100"],
            "name.last" => ["max:100"],
            "address.street" => ["max:200"],
            "address.city" => ["required", "max:100"],
            "address.country" => ["required", "max:100"],
        ];

        $validator = Validator::make($data, $rules);
        self::assertTrue($validator->passes());
    }
    ```

---

### 13. Index Array Validation

-   Saat nested array nya adalah _indexed_, artinya bisa lebih dari satu,

-   Pada kasus ini, tidak lagi menggunakan `.` (titik), melaikan menggunakan `*` (bintang).

-   Kode Index Array Validation

    ```PHP
    public function testNestedIndexedArray()
    {
        $data = [
            "name" => [
                "first" => "Gusti",
                "last" => "Akbar"
            ],
            "address" => [
                [
                    "street" => "Jalan. Mangga",
                    "city" => "Jakarta",
                    "country" => "Indonesia"
                ],
                [
                    "street" => "Jalan. Manggis",
                    "city" => "Jakarta",
                    "country" => "Indonesia"
                ]
            ]
        ];
    }
    ```

---

### 14. HTTP Request Validation

-   Laravel Validator sudah terintegrasi baik dengan HTTP Request di Laravel,

-   Class request memiliki method `validate()` untuk melakukan validasi data request yang dikirim oleh _User_, misal dari From atau Query Parameter.

-   Sebelum membuat http request, kita harus membuat _controller_ nya terlebih dahulu. `php artisan make:controller FormController`.

-   Link: [HTTP](https://laravel.com/docs/11.x/requests#retrieving-an-input-value)

-   Kode Form Controller

    ```PHP
    public function login(Request $request): Response
    {
        try {
            $rules = [
                "username" => "required",
                "password" => "required"
            ];

            $data = $request->validate($rules);
            // data
            return response("OK", Response::HTTP_OK);
        }catch (ValidationException $validationException){
            return response($validationException->errors(), Response::HTTP_BAD_REQUEST);
        }
    }
    ```

-   Kode Route

    ```PHP
    Route::post('/form/login', [\App\Http\Controllers\FormController::class, 'login']);
    ```

-   Unit Test HTTP Request Validation

    ```PHP
    public function testLoginFailed() // login gagal
    {
        $response = $this->post('/form/login', [
            'username' => '',
            'password' => ''
        ]);
        $response->assertStatus(400);
    }

    public function testLoginSuccess() // login berhasil
    {
        $response = $this->post('/form/login', [
            'username' => 'admin',
            'password' => 'rahasia'
        ]);
        $response->assertStatus(200);
    }
    ```

---

### 15. Error Page

-   Kita bisa menampilkan _error_ dari `MessageBag` di `Laravel Blade Template`,

-   Cukup gunakan _variable_ `$errors` di `Blade Template`.

-   Kode Form Blade Template

    ```PHP
    @if ($errors->any())
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    @endif

    <form action="/form" method="post">
        @csrf
        <label>Username : @error('username')
                {{ $message }}
            @enderror
            <input type="text" name="username" value="{{ old('username') }}"></label> <br>
        <label>Password : @error('password')
                {{ $message }}
            @enderror
            <input type="password" name="password" value="{{ old('password') }}"></label><br>
        <input type="submit" value="Login">
    </form>
    ```

-   Kode Login Controller

    ```PHP
    public function form(): Response {
        return response()->view("form");
    }

    public function submitForm(Request $request): Response
    {
        $data = $request->validate([
            "username" => "required",
            "password" => "required"
        ]);
        return response("OK", response::HTTP_OK);
    }
    ```

-   Unit Test Error Page

    ```PHP
    public function testFormFailed() // tampil form gagal
    {
        $response = $this->post('/form', [
            'username' => '',
            'password' => ''
        ]);
        $response->assertStatus(302);
    }

    public function testFormSuccess() // tampil form berhasil
    {
        $response = $this->post('/form', [
            'username' => 'admin',
            'password' => 'rahasia'
        ]);
        $response->assertStatus(200);
    }
    ```

---

### 16. Membuat Form Request

-   Untuk membuat Form Request sendiri bisa gunakan perintah `php artisan make:request NamaFormRequest`.

-   Nanti akan dibuat kan folder baru di `app/Http/Request`

-   Kode Login Request 1

    ```PHP
    public function rules(): array
    {
        return [
            "username" => ["required", "email", "max:100"],
            "password" => ["required", Password::min(6)->letters()->numbers()->symbols()]
        ];
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

---
layout: page
title: Penambahan Pages
permalink: /adding-pages
---

Tugasan kita di dalam panduan ini ialah untuk menambah dua page kepada aplikasi Phoenix kita.  Satu adalah page statik dan lagi satu akan mengambil sebahagian dari URL sebagai input dan menghantarnya kepada satu templat untuk paparan.  Sepanjang jalan, kita akan dapat membiasakan diri dengan komponen-komponen asas aplikasi Phoenix: router, controller, view, dan template. 

Apabila Phoenix menjana satu aplikasi baru, ia akan membina satu direktori aras-atas seperti berikut:

```text
├── _build
├── config
├── deps
├── lib
├── priv
├── test
├── web
```

Kebanyakan kerja kita di dalam panduan ini akan berada di dalam direktori `web`, yang nampak seperti berikut apabila dikembangkan:

```text
├── channels
├── controllers
│   └── page_controller.ex
├── models
├── static
├── router.ex
├── templates
│   ├── layout
│   │   └── app.html.eex
│   └── page
│       └── index.html.eex
└── views
|   ├── error_view.ex
|   ├── layout_view.ex
|   └── page_view.ex
└── web.ex
```

Semua fail yang sekarang ini di dalam direktori-direktori `controllers`, `templates`, dan `views` adalah untuk membina page "Welcome to Phoenix!" yang kita lihat di dalam panduan yang lepas.  Kita akan melihat bagaimana kita akan mengguna semula beberapa kod tersebut sebentar lagi.  Secara convention, di dalam persekitaran pembangunan, apa-apa di dalam direktori `web` akan secara otomatik dikompil apabila ada satu permohonan web yang baru.

Semua aset statik aplikasi kita berada di dalam direktori `priv/static`, masing-masing di dalam direktori yang berkenaan dengan jenis fail masing-masing - css, imej atau js.  Kita letakkan asset yang perlukan fasa 'build' di dalam `web/static`, dan fail-fail sumber akan dibina ke dalam bundle `app.js`/`app.css` masing-masing di dalam `priv/static`.  Kita tidak akan membuat sebarang perubahan di sini buat masa ini, tetapi ianya bagus untuk tahu di mana untuk melihat sebagai rujukan masa hadapan.

```text
priv
└── static
    └── images
        └── phoenix.png
```

```text
web
└── static
    ├── css
    |   └── app.css
    ├── js
    │   └── app.js
    └── vendor
        └── phoenix.js
```

Direktori `lib` juga mengandungi fail-fail yang perlu kita ketahui.  Endpoint aplikasi kita berada di `lib/hello_phoenix/endpoint.ex`, dan fail aplikasi kita (yang memulakan aplikasi dan 'supervision tree' aplikasi tersebut) berada di `lib/hello_phoenix.ex`.

```text
lib
├── hello_phoenix
|   ├── endpoint.ex
│   └── repo.ex
└── hello_phoenix.ex
```

Berlainan dari direktori `web`, Phoenix tidak akan mengkompil fail-fail di dalam `lib` setiap kali terdapat permohonan web baru.  Ini disengajakan!  Perbezaan antara `web` dan `lib` memberikan convention untuk cara-cara berbeza kita menguruskan 'state' di dalam aplikasi kita.  Direktori `web` mengandungi apa-apa yang 'state'-nya hanya untuk durasi satu permohonan web.  Direktori `lib` mengandungi kedua-dua modul kongsian dan apa-apa yang diperlukan untuk mengurus 'state' di luar durasi satu permohonan web.

Cukup dengan persediaan, kita mula dengan page pertama Phoenix!

### Satu Route Baru

Routes memetakan pasangan HTTP verb/path yang unik kepada pasangan controller/action yang akan menguruskan mereka.  Phoenix menjana satu fail router secara lalai di dalam `web/router.ex`.  Inilah tempat kita akan bekerja di dalam bahagian ini.

Route untuk page "Welcome to Phoenix!" kita dari panduan lepas adalah seperti berikut.

```elixir
get "/", PageController, :index
```

Cuba kita hadamkan apa yang diberitahu oleh route ini.  Capaian kepada [http://localhost:4000/](http://localhost:4000/) akan menjana satu permohonan HTTP `GET` kepada root path.  Semua permohonan seperti ini akan diuruskan oleh fungsi `index` di dalam modul `HelloPhoenix.PageController` dari `web/controllers/page_controller.ex`.

Page yang akan kita bina cuma akan mengatakan "Hello World, from Phoenix!" apabila kita mencapai [http://localhost:4000/hello](http://localhost:4000/hello).

Perkara pertama yang perlu kita lakukan untuk membina page tersebut ialah menetapkan satu route untuknya.  Buka fail `web/router.ex`.  Ia sepatutnya nampak seperti ini:

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloPhoenix do
  #   pipe_through :api
  # end
end

```
Buat masa ini, kita tidak akan hiraukan 'pipeline' dan penggunaan `scope` dan cuma fokus kepada menambah satu route.  (Kita akan meliputi topik-topik ini di dalam [Routing Guide](http://www.phoenixframework.org/docs/routing).) 

Mari kita tambah satu route kepada router yang memetakan satu permohonan `GET` untuk `/hello` kepada action `index` `HelloPhoenix.HelloController`:

```elixir
get "/hello", HelloController, :index
```

Blok `scope "/"` di dalam fail `router.ex` kita sepatutnya seperti berikut:

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
  get "/hello", HelloController, :index
end
```

### Satu Controller Baru

Controller adalah modul-modul Elixir, dan action adalah fungsi-fungsi Elixir yang ditetapkan di dalam mereka.  Tujuan utama action ialah untuk mengumpulkan apa-apa data dan melaksanakan apa-apa tugasan yang diperlukan untuk pemaparan.  Route kita menetapkan kita memerlukan modul `HelloPhoenix.HelloController` dan action `index/2`.

Untuk itu, buat satu fail `web/controllers/hello_controller.ex`, dan ubahnya seperti berikut:

```elixir
defmodule HelloPhoenix.HelloController do
  use HelloPhoenix.Web, :controller

  def index(conn, _params) do
    render conn, "index.html"
  end
end
```
Kita akan tangguhkan perbincangan mengenai `use HelloPhoenix.Web, :controller` untuk [Controllers Guide](http://www.phoenixframework.org/docs/controllers).  Buat masa ini, kita fokus kepada action `index/2`.

Semua controller action mengambil dua argumen.  Pertama ialah `conn`, satu struct yang menyimpan banyak data mengenai permohonan web tersebut.  Keduanya ialah `params`, iaitu parameter-parameter permohonan tersebut.  Di sini, kita tidak menggunakan `params`, dan kita elakkan amaran-amaran dari pengkompil dengan menambah `_`.

Teras kepada action ini ialah `render conn, "index.html"`.  Ia memberitahu Phoenix untuk mencari satu template bernama `index.html.eex` dan memaparkannya.  Phoenix akan mencari template tersebut di dalam direktori yang dinamakan sama dengan nama controller, jadi untuk kes ini, ianya `web/templates/hello`.

>Nota: Penggunaan atom sebagai nama template juga dibolehkan, `render conn, :index`, tetapi template akan dipilih berdasarkan kepada Accept headers, e.g. `"index.html"` atau `"index.json"`.

Modul-modul yang bertanggungjawab membuat pemaparan adalah views, dan kita akan membuat satu yang baru seterusnya.

### Satu View Baru

View di dalam Phoenix mempunyai beberapa tugasan penting.  Mereka memaparkan template.  Mereka juga bertindak sebagai lapisan persembahan untuk data mentah dari controller, menyediakannya untuk digunakan di dalam template.  Fungsi-fungsi yang menjalankan transformasi data sebegini sepatutnya berada di dalam view.

Sebagai contoh, katakan kita mempunyai satu struktur data yang mewakili satu pengguna yang mempunyai satu medan `first_name` dan satu medan `last_name`, dan di dalam template, kita mahu memaparkan nama penuh pengguna berkenaan.  Kita boleh menulis kod di dalam template untuk menggabungkan kedua-dua medan tersebut kepada nama penuh, tetapi cara yang lebih baik adalah untuk menulis satu fungsi di dalam view untuk tugasan ini, kemudian memanggil fungsi tersebut dari dalam template.  Hasilnya adalah template yang lebih bersih dan lebih mudah dibaca. 

Untuk memaparkan apa-apa template untuk `HelloController`, kita perlukan satu `HelloView`.  Nama-nama yang digunakan adalah amat penting - bahagian pertama nama controller dan view mesti padan.  Kita akan bina satu fail view asas buat masa ini dan tinggalkan keterangan lebih terperinci untuk kemudian.  Buat fail `web/views/hello_view.ex` dan masukkan kod berikut:

```elixir
defmodule HelloPhoenix.HelloView do
  use HelloPhoenix.Web, :view
end
```

### Satu Template Baru

Template-template Phoenix adalah tempat dimana data boleh dipaparkan.  Templating engine yang Phoenix gunakan ialah EEx, untuk [Embedded Elixir](http://elixir-lang.org/docs/stable/eex/).  Semua fail template akan mempunyai extension `.eex`.

Template-template diskopkan kepada satu view, yang diskopkan kepada controller.  Ini bermakna, kita akan membuat satu direktori di yang mempunyai nama sama dengan controller di dalam direktori `web/templates`.  Untuk page hello kita, ia bermakna kita akan membuat satu direktori `hello` di dalam `web/templates` dan kemudian buat satu fail `index.html.eex` di dalamnya.

Kita akan lakukannya sekarang.  Buat fail `web/templates/hello/index.html.eex` dan isikannya dengan berikut:

```html
<div class="jumbotron">
  <h2>Hello World, from Phoenix!</h2>
</div>
```

Sekarang kita telah mempunyai route, controller, view dan template, jadi sepatutnya bila kita mencapai [http://localhost:4000/hello](http://localhost:4000/hello) kita akan dapat melihat paparan alu-aluan dari Phoenix! (Jika anda telah mematikan pelayan Phoenix, hidupkan semula dengan `mix phoenix.server`.)

![Phoenix Greets Us](/images/hello-from-phoenix.png)

Terdapat beberapa perkara menarik untuk diperhatikan berkenaan dengan apa yang baru kita lakukan.  Kita tidak perlu mematikan dan menghidupkan semula pelayan Phoenix apabila kita membuat apa-apa perubahan.  Ya, Phoenix mempunyai pemasangan kod panas!  Juga, walaupun fail `index.html.eex` hanya mengandungi satu tag `div`, page yang kita dapat ialah satu fail HTML yang lengkap.  Template index kita dimasukkan ke dalam rekabentuk aplikasi - `web/templates/layout/app.html.eex`.  Jika dibuka, anda akan melihat satu baris kod seperti berikut:

    <%= render @view_module, @view_template, assigns %>

iaitu apa yang memasukkan template kita ke dalam rekabentuk tersebut sebelum HTML dihantar kepada browser.

## Satu Lagi Page Baru

Sekarang kita akan menambah sedikit kerumitan ke dalam aplikasi kita.  Kita akan menambah satu page yang akan mengenali satu bahagian dari URL, melabelnya sebagai "messenger" dan menghantarnya melalui controller ke dalam template supaya "messenger" tersebut boleh berkata hello.

Seperti sebelum ini, perkara pertama kita akan lakukan ialah membuat satu route baru.

### Satu Route Baru

Di dalam latihan ini, kita akan menggunakan semula `HelloController` yang kita bina sebelum ini dan cuma menambah satu action `show`.  Kita akan menambah satu baris di bawah route terakhir kita, seperti ini:

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser # Use the default browser stack.

  get "/", PageController, :index
  get "/hello", HelloController, :index
  get "/hello/:messenger", HelloController, :show
end
```
Perhatikan kita memasukkan atom `:messenger` di dalam path.  Phoenix akan mengambil apa sahaja nilai yang wujud di posisi tersebut di dalam URL dan menghantar satu [Map](http://elixir-lang.org/docs/stable/elixir/Map.html) dengan kunci `messenger` menunjukkan nilai tersebut kepada controller.

Sebagai contoh, jika kita menunjukkan browser kepada: [http://localhost:4000/hello/Frank](http://localhost:4000/hello/Frank), nilai ":messenger" ialah "Frank".

### Satu Action Baru

Permohonan kepada route baru kita akan diuruskan oleh action `HelloPhoenix.HelloController` `show`.  Controller tersebut telah sedia ada di `web/controllers/hello_controller.ex`, jadi apa yang kita perlu lakukan ialah mengemaskini fail tersebut dan menambahkan action `show` kepadanya.  Kali ini, kita akan perlu untuk menyimpan salah satu item di dalam map parameter yang dihantar kepada action tersebut, supaya kita akan dapat menghantarkannya(messenger) kepada template.  Untuk itu, kita tambah fungsi show ini kepada controller:

```elixir
def show(conn, %{"messenger" => messenger}) do
  render conn, "show.html", messenger: messenger
end
```
Terdapat beberapa perkara yang perlu diperhatikan di sini.  Kita padan-corakkan params yang dihantar ke dalam fungsi show supaya pembolehubah `messenger` akan diikatkan kepada nilai yang kita masukkan di posisi `:messenger` di dalam URL.  Sebagai contoh, jika URL kita ialah [http://localhost:4000/hello/Frank](http://localhost:4000/hello/Frank), pembolehubah messenger akan diikatkan kepada  `Frank`.

Di dalam badan action `show`, kita juga menghantar argumen ketiga ke dalam fungsi render, satu pasangan key/value di mana `:messenger` sebagai key dan pembolehubah `messenger` sebagai value.

> Nota: Jika badan action tersebut perlu mengakses map penuh parameter yang diikat kepada pembolehubah params sebagai tambahan kepada pembolehubah messenger, kita boleh tetapkan `show/2` seperti ini:

```elixir
def show(conn, %{"messenger" => messenger} = params) do
  ...
end
```

Bagus untuk diingat bahawa kunci-kunci(key) kepada map `params` akan sentiasa dari jenis string, dan simbol persamaan(=) tidak melambangkan assignment tetapi satu tetapan [pemadanan corak(pattern match)](http://elixir-lang.org/getting-started/pattern-matching.html).

### Satu Template Baru

Sebagai bahagian terakhir di dalam teka-teki ini, kita akan memerlukan satu template baru.  Oleh sebab ianya untuk action `show` dari `HelloController`, ia akan masuk ke dalam direktori `web/templates/hello` dan dinamakan sebagai `show.html.eex`.  Ia akan nampak seperti template `index.html.eex` kita, kecuali kita akan perlu memaparkan nama messenger.

Untuk itu, kita akan menggunakan tag-tag khas EEx untuk melaksanakan ekspresi-ekspresi Elixir - `<%= %>`.  Perhatikan bahagian awal tag tersebut mengandungi satu simbol persamaan, seperti ini: `<%=`.  Ini bermakna mana-mana kod Elixir yang masuk di antara tag-tag tersebut akan dilaksanakan, dan nilai yang dihasilkan akan menggantikan tag-tag tersebut.  Jika tag-tag tersebut digunakan tanpa simbol persamaan, kod tersebut masih dilaksanakan cuma nilai yang dihasilkan tidak akan dipaparkan.

Dan inilah rupa sepatutnya template tersebut: 

```html
<div class="jumbotron">
  <h2>Hello World, from <%= @messenger %>!</h2>
</div>
```

Messenger kita wujud sebagai `@messenger`.  Di dalam kes ini, ia bukan satu 'module attribute'.  Ia adalah secebis sintaks khas yang bermaksud `Dict.get(assigns, :messenger)`.  Hasilnya nampak lebih cantik dan lebih mudah digunakan di dalam template.

Kita telah siap. Jika anda tunjukka browser anda ke sini: [http://localhost:4000/hello/Frank](http://localhost:4000/hello/Frank), anda akan depat melihat satu page seperti berikut:

![Frank Greets Us from Phoenix](/images/hello-world-from-frank.png)

Cuba bermain dengannya sedikit.  Apa-apa nilai yang anda masukkan selepas `/hello/` akan dipaparkan sebagai messenger anda.
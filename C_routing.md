---
layout: page
title: Penggunaan Route
permalink: /routing
---

Router adalah titik temu utama aplikasi Phoenix.  Mereka memadankan permohonan HTTP kepada action di dalam controller, memasang pengurusan channel masa-nyata, dan menetapkan satu siri transformasi 'pipepline' untuk membuat skop middleware kepada satu kumpulan route.

Fail router yang dijanakan Phoenix, `web/router.ex`, akan kelihatan seperti berikut:

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

Nama yang anda berikan kepada aplikasi anda akan dipaparkan dan bukannya `HelloPhoenix` untuk kedua-dua modul router dan nama controller.

Baris pertama module ini, `use HelloPhoenix.Web, :router`, adalah untuk menyebabkan fungsi-fungsi router Phoenix boleh digunakan di dalam router kita.

'Scope' mempunyai bahagian mereka sendiri di dalam panduan ini, jadi kita tidak akan menghabiskan masa menerangkan blok `scope "/", HelloPhoenix do` di sini.  Baris `pipe_through :browser` akan diperincikan di dalam bahagian Pipeline di dalam panduan ini.  Untuk masa sekarag, anda cuma perlu tahu bahawa 'pipeline' membenarkan satu set trasformasi middleware dilaksanakan kepada beberapa kumpulan route.

Di dalam blok `scope`, kita ada route pertama kita:

```elixir
  get "/", PageController, :index
```

`get` ialah satu makro Phoenix yang dikembangkan untuk menetapkan satu klausa fungsi `match/3`.  Ianya bersamaan dengan HTTP verb GET.  Makro-makro sebanding dengannya wujud untuk HTTP verb yang lain, seperti POST, PUT, PATCH, DELETE, OPTIONS, CONNECT, TRACE dan HEAD.

Argumen pertama makro-makro ini ialah 'path'.  Di sini, ianya root kepada aplikasi tersebut, `/`.  Dua argumen seterusnya adalah controller dan action yang kita mahu gunakan untuk menguruskan permohonan web tersebut.  Makro-makro ini juga boleh menerima pilihan-pilihan lain, yang akan dapat kita lihat sepanjang panduan ini.

Jika hanya inilah route di dalam modul router kita, klausa fungsi `match/3` akan nampak seperti berikut selepas makro tersebut dikembangkan:

```elixir
  def match(conn, "GET", ["/"])
```

Badan fungsi `match/3` menyediakan hubungan dan membangkitkan controller action yang berpadanan.

Apabila kita menambah lebih banyak route, lebih banyak klausa akan ditambah kepada fungsi `match/3` di dalam modul router kita.  Tabiat mereka adalah sama dengan mana-mana fungsi pelbagai klausa(satu fungsi banyak kepala) dalam Elixir. Mereka akan cuba dipadankan dari atas ke bawah, dan klausa pertama yang berjaya dipadankan dengan parameter yang diberikan(path dan verb) akan dilaksanakan.  Setelah padanan dijumpai, proses pemadanan corak itu akan dihentikan dan klausa-klausa lain tidak akan diuji.

Ini bermakna ianya berkemungkinan untuk membuat satu route yang tidak akan dipadankan bergantung kepada HTTP verb dan path.

Jika kita membuat satu route yang tidak jelas, modul router tersebut masih dikompil, tetapi kita akan diberkan amaran. Sebagaimana dalam contoh di bawah.

Tetapkan route berikut dibahagian bawah blok `scope "/", HelloPhoenix do` di dalam router.

```elixir
get "/", RootController, :index
```

Kemudian jalankan `$ mix compile` di root projek.  Anda dapat melihat amaran berikut dari pengkompil:

```text
web/router.ex:1: warning: this clause cannot match because a previous clause at line 1 always matches
Compiled web/router.ex
```

### Memeriksa Routes

Phoenix menyediakan satu alat yang bagus untuk memeriksa routes di dalam satu aplikasi, iaitu mix task `phoenix.routes`.

Mari lihat bagaimana ia berfungsi.  Pergi ke root aplikasi Phoenix dan jalankan `$ mix phoenix.routes`. (Jika masih belum dilakukan, anda perlu jalankan `$ mix do deps.get, compile` sebelum menjalankan tugasan `routes` tersebut.)  Anda sepatutnya dapat melihat sesuatu seperti berikut, dijanakan dari satu-satunya route yang kita ada:

```console
$ mix phoenix.routes
page_path  GET  /  HelloPhoenix.PageController :index
```
Output itu memberitahu kita bahawa mana-mana permohonan HTTP GET kepada root aplikasi ini akan diuruskan oleh action `index` dari `HelloPhoenix.PageController`.

`page_path` adalah satu contoh apa yang Phoenix panggil sebagai path helper, dan kita akan bincangkan mereka sekejap lagi.

### Resources

Modul router menyokong makro-makro lain selain dari HTTP verbs seperti `get`, `post`, dan `put`.  Makro yang paling penting di antara mereka adalah `resources`, yang akan berkembang kepada lapan klausa fungsi `match/3`. 

Mari tambahkan satu resource kepada fail `web/router.ex` seperti berikut:

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
  resources "/users", UserController
end
```
Untuk tujuan ini, ianya tidak penting yang kita sebenarnya tidak mempunyai satu `HelloPhoenix.UserController`.

Kemudian pergi ke projek itu, dan jalankan `$ mix phoenix.routes`.

Anda sepatutnya dapat melihat sesuatu seperti berikut:

```elixir
user_path  GET     /users           HelloPhoenix.UserController :index
user_path  GET     /users/:id/edit  HelloPhoenix.UserController :edit
user_path  GET     /users/new       HelloPhoenix.UserController :new
user_path  GET     /users/:id       HelloPhoenix.UserController :show
user_path  POST    /users           HelloPhoenix.UserController :create
user_path  PATCH   /users/:id       HelloPhoenix.UserController :update
           PUT     /users/:id       HelloPhoenix.UserController :update
user_path  DELETE  /users/:id       HelloPhoenix.UserController :delete
```

`HelloPhoenix` akan digantikan dengan nama projek anda.

Ini adalah matriks biasa untuk HTTP verb, path dan controller action.  Kita akan melihat mereka satu persatu, di dalam susunan yang agak berbeza.

- Permohonan GET kepada `/users` akan menjalankan action `index` untuk menyenaraikan semua 'users'.
- Permohonan GET kepada `/users/:id` akan menjalankan action `show` bersama satu id untuk memaparkan maklumat satu 'user' yang diidentifikasikan dengan ID tersebut.
- Permohonan GET kepada `/users/new` akan menjalankan action `new` untuk memaparkan satu borang untuk mencipta satu 'user' baru.
- A POST request to `/users` will invoke the `create` action to save a new user to the data store.
- Permohonan POST kepada `/users` akan menjalankan action `create` untuk  menyimpan maklumat 'user' baru ke pangkalan data.
- Permohonan GET kepada `/users/:id/edit` akan menjalankan action `edit` bersama ID untuk memaparkan 'user' yang mempunyai ID tersebut di dalam borang untuk tujuan kemaskini.
- Permohonan PATCH kepada `/users/:id` akan menjalankan action `update` bersama ID untuk mengemaskini maklumat 'user' ke pangkalan data.
- Permohonan PUT kepada `/users/:id` juga akan menjalankan action `update` bersama ID untuk mengemaskini maklumat 'user' ke pangkalan data.
- Permohonan DELETE kepada `/users/:id` akan menjalankan action `delete` bersama ID untuk memadamkan maklumat 'user' tersebut dari pangkalan data.

Jika kita rasa tidak perlukan semua route di atas, kita boleh memilih mana-mana route dengan menggunakan pilihan `:only` dan `:except`.


Katalah kita ada satu resource baca-sahaja post.  Kita boleh menetapkannya seperti ini:

```elixir
resources "/posts", PostController, only: [:index, :show]
```

Menjalankan `$ mix phoenix.routes` menunjukkan kita cuma mempunyai routes yang ditetapkan untuk action index dan show sahaja.

```elixir
post_path  GET     /posts HelloPhoenix.PostController :index
post_path  GET     /posts/:id HelloPhoenix.PostController :show
```

Sama juga, jika kita mempunyai resource 'comments', dan kita tidak mahu membekalkan route untuk memadamkannya, kita boleh tetapkan satu route seperti ini.

```elixir
resources "/comments", CommentController, except: [:delete]
```

Menjalankan `$ mix phoenix.routes` menunjukkan kita mempunyai kesemua route kecuali untuk permohonan DELETE kepada action delete.


```elixir
comment_path  GET     /comments HelloPhoenix.CommentController :index
comment_path  GET     /comments/:id/edit HelloPhoenix.CommentController :edit
comment_path  GET     /comments/new HelloPhoenix.CommentController :new
comment_path  GET     /comments/:id HelloPhoenix.CommentController :show
comment_path  POST    /comments HelloPhoenix.CommentController :create
comment_path  PATCH   /comments/:id HelloPhoenix.CommentController :update
              PUT     /comments/:id HelloPhoenix.CommentController :update
```
### Penyokong Path(Path Helpers)

Penyokong path adalah fungsi-fungsi yang ditetapkan secara dinamik oleh modul `Router.Helpers` untuk satu aplikasi.  Untuk kita, ianya `HelloPhoenix.Router.Helpers`.  Nama-nama mereka dibina dari nama controller yang digunakan dalam penetapan route.  Controller kita `HelloPhoenix.PageController`, dan `page_path` adalah fungsi yang akan memulangkan path kepada root aplikasi.

Mari lihat bagaimana ia berfungsi.  Jalankan `$ iex -S mix` di root projek tersebut.  Apabila kita memanggil fungsi `page_path` dengan argumen `Endpoint` atau hubungan dan action, ia akan memulangkan path kepada kita.

```elixir
iex> HelloPhoenix.Router.Helpers.page_path(HelloPhoenix.Endpoint, :index)
"/"
```

Ini penting kerana kita boleh gunakan fungsi `page_path` di dalam templat untuk membuat hubungan kepada root aplikasi kita.  Nota:  Jika pemanggilan fungsi itu nampak terlalu panjang, ada penyelesaiannya, termasuk `import HelloPhoenix.Router.Helpers` di dalam view utama aplikasi.

```html
<a href="<%= page_path(@conn, :index) %>">To the Welcome Page!</a>
```
Mohon lihat [View Guide](http://www.phoenixframework.org/docs/views) untuk maklumat lanjut.

Ini amat berfaedah jika kita perlu mengubah path kepada route di dalam router kita.  Oleh sebab penyokong path dibina secara dinamik dari route, apa-apa panggilan kepada `page_path` di dalam templat kita masih boleh berfungsi.

### Lagi Mengenai Penyokong Path (Path Helpers)

Apabila kita jalankan tugasan `phoenix.routes` untuk resource user, ia menyenaraikan `user_path` sebagai fungsi penyokong path di dalam setiap baris output tersebut.  Berikut adalah apa yang setiap action tersebut diterjemahkan:

```elixir
iex> import HelloPhoenix.Router.Helpers
iex> alias HelloPhoenix.Endpoint
iex> user_path(Endpoint, :index)
"/users"

iex> user_path(Endpoint, :show, 17)
"/users/17"

iex> user_path(Endpoint, :new)
"/users/new"

iex> user_path(Endpoint, :create)
"/users"

iex> user_path(Endpoint, :edit, 37)
"/users/37/edit"

iex> user_path(Endpoint, :update, 37)
"/users/37"

iex> user_path(Endpoint, :delete, 17)
"/users/17"
```

Bagaimana pula dengan path yang mempunyai query string?  Dengan menambah argumen pasangan key/value keempat, penyokong path akan memulangkan pasangan tersebut di dalam bentuk query string.

```elixir
iex> user_path(Endpoint, :show, 17, admin: true, active: false)
"/users/17?admin=true&active=false"
```

Perlukan url penuh dan bukannya path? Tukarkan `_path` dengan `_url`:

```elixir
iex(3)> user_url(Endpoint, :index)
"http://localhost:4000/users"
```
Endpoint aplikasi akan mempunyai panduan mereka sendiri dalam masa terdekat.  Untuk sekarang, gambarkan mereka sebagai entiti yang menguruskan permohonan hanya sehingga di mana permohonan itu diambil alih oleh router.

Fungsi `_url` akan mencapai host, port, port proksi dan maklumat SSL yang diperlukan untuk membina URL penuh tersebut daripada parameter konfigurasi yang ditetapkan untuk setiap persekitaran.  Kita akan bincangkan mengenai konfigurasi dengan lebih terperinci di dalam panduan mereka sendiri.  Buat masa sekarang, anda boleh melihat konfigurasi tersebut di dalam fail `/config/dev.exs`.

### Nested Resources

Ianya juga dibolehkan untuk menetapkan nested resources di dalam router Phoenix.  Katalah kita juga mempunyai satu resource `posts` yang mempunyai satu hubungan satu ke banyak dengan `users`.  Dengan kata lain, satu user boleh membuat banyak post, dan setiap satu post individu akan dimilki oleh satu user. Kita boleh membinanya dengan menambah satu nested route di dalam `web/router` seperti ini: 

```elixir
resources "/users", UserController do
  resources "/posts", PostController
end
```
Apabila kita jalankan `$ mix phoenix.routes`, kita akan mendapati satu set route berikut ditambah kepada route yang kita dapati sebelum ini:

```elixir
. . .
user_post_path  GET     users/:user_id/posts HelloPhoenix.PostController :index
user_post_path  GET     users/:user_id/posts/:id/edit HelloPhoenix.PostController :edit
user_post_path  GET     users/:user_id/posts/new HelloPhoenix.PostController :new
user_post_path  GET     users/:user_id/posts/:id HelloPhoenix.PostController :show
user_post_path  POST    users/:user_id/posts HelloPhoenix.PostController :create
user_post_path  PATCH   users/:user_id/posts/:id HelloPhoenix.PostController :update
                PUT     users/:user_id/posts/:id HelloPhoenix.PostController :update
user_post_path  DELETE  users/:user_id/posts/:id HelloPhoenix.PostController :delete
```

Kita dapat melihat setiap route di atas mengaitkan setiap post kepada satu ID user.  Untuk yang pertama, kita akan memanggil action `index` `PostController`, dan kita juga hantarkan satu `user_id`.  Ini menunjukkan yang kita akan memaparkan semua post untuk user tersebut sahaja.  'Scoping' yang sama ditetapka untuk semua route tersebut.

Apabila memanggil fungsi-fungsi penyokong path untuk 'nested routes', kita perlu menghantar ID dalam susunan sama mereka ditetapkan di dalam route.  Untuk route `show` berikut, `42` ialah untuk `user_id`, dan `17` ialah untuk `post_id`.  Kita juga perlu memasukkan baris `alias HelloPhoenix.Endpoint` sebelum bermula.

```elixir
iex> alias HelloPhoenix.Endpoint
iex> HelloPhoenix.Router.Helpers.user_post_path(Endpoint, :show, 42, 17)
"/users/42/posts/17"
```

Lagi sekali, jika kita tambah satu pasangan key/value kepada penghujung satu-satu panggilan fungsi, ianya akan ditambah kepada query string.

```elixir
iex> HelloPhoenix.Router.Helpers.user_post_path(Endpoint, :index, 42, active: true)
"/users/42/posts?active=true"
```

### Route Berskop

Skop adalah cara untuk menggabungkan route di bawah satu prefiks path dan satu set plug middleware.  Kita mungkin mahu gunakan ini untuk fungsi-fungsi admin, API dan lebih-lebih lagi API berversi.  Katakan kita mempunyai ulasan yang dijanakan pengguna di dalam satu laman, dan semua ulasan-ulasan itu memerlukan pengesahan oleh admin.  Semantik untuk resource-resource itu agak berbeza, dan mereka mungkin tidak berkongsi satu controller yang sama.  Skop memberikan keupayaan untuk mengasingkan route-route tersebut.

Path untuk ulasan yang menghadap pengguna mungkin nampak seperti resource biasa. 

```text
/reviews
/reviews/1234
/reviews/1234/edit
. . .
```

Path untuk ulasan kegunaan admin mungkin diprefikskan dengan `/admin`.

```text
/admin/reviews
/admin/reviews/1234
/admin/reviews/1234/edit
. . .
```

Kita melaksanakan ini dengan satu route berskop yang menetapkan satu pilihan path kepada `/admin` seperti ini.  Untuk masa ini, kita tidak akan membuat nest kepada skop di dalam skop-skop yang lain (seperti `scope "/", HelloPhoenix do` yang dibekalkan kepada kita di dalam satu aplikasi baru).

```elixir
scope "/admin" do
  pipe_through :browser

  resources "/reviews", HelloPhoenix.Admin.ReviewController
end
```

Perhatikan bahawa Phoenix akan menganggap path yang ditetapkan akan bermula dengan satu '/', jadi `scope "/admin" do` dan `scope "admin" do` akan mendapat hasil yang sama.

Perhatikan juga, dengan tetapan sekarang, kita perlu menggunakan nama penuh controller, `HelloPhoenix.Admin.ReviewController`.  Kita akan membaikinya sekejap lagi.

Menjalankan `$ mix phoenix.routes` semula, kita akan mendapat yang berikut sebagai tambahan kepada route yang sedia ada:

```elixir
. . .
review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
             PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
```

Ini nampak bagus, tetapi ada masalah di sini.  Ingat bahawa kita mahu kedua-dua route ulasan menghadap pengguna `/reviews`, dan juga untuk admin `/admin/reviews`.  Jika kita tambahkan route untuk ulasan mengahadap pengguna seperti ini:  

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser
  . . .
  resources "/reviews", ReviewController
  . . .
end

scope "/admin" do
  resources "/reviews", HelloPhoenix.Admin.ReviewController
end
```

dan jalankan `$ mix phoenix.routes`, kita akan mendapat output ini:

```elixir
. . .
review_path  GET     /reviews HelloPhoenix.ReviewController :index
review_path  GET     /reviews/:id/edit HelloPhoenix.ReviewController :edit
review_path  GET     /reviews/new HelloPhoenix.ReviewController :new
review_path  GET     /reviews/:id HelloPhoenix.ReviewController :show
review_path  POST    /reviews HelloPhoenix.ReviewController :create
review_path  PATCH   /reviews/:id HelloPhoenix.ReviewController :update
             PUT     /reviews/:id HelloPhoenix.ReviewController :update
review_path  DELETE  /reviews/:id HelloPhoenix.ReviewController :delete
. . .
review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
             PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
```

Route sebenar yang dijanakan nampak betul, kecuali untuk bahagian `review_path` di permulaan setiap baris.  Kita mendapat peyokong path yang sama untuk route ulasan menghadap pengguna dan juga route ulasan menghadap admin, bukan apa yang kita mahu.  Kita boleh mengatasi masalah ini dengan menambah `as: :admin` kepada skop admin.

```elixir
scope "/", HelloPhoenix do
  pipe_through :browser
  . . .
  resources "/reviews", ReviewController
  . . .
end

scope "/admin", as: :admin do
  resources "/reviews", HelloPhoenix.Admin.ReviewController
end
```

Sekarang `$ mix phoenix.routes` akan memaparkan apa yang kita mahu.

```elixir
. . .
      review_path  GET     /reviews HelloPhoenix.ReviewController :index
      review_path  GET     /reviews/:id/edit HelloPhoenix.ReviewController :edit
      review_path  GET     /reviews/new HelloPhoenix.ReviewController :new
      review_path  GET     /reviews/:id HelloPhoenix.ReviewController :show
      review_path  POST    /reviews HelloPhoenix.ReviewController :create
      review_path  PATCH   /reviews/:id HelloPhoenix.ReviewController :update
                   PUT     /reviews/:id HelloPhoenix.ReviewController :update
      review_path  DELETE  /reviews/:id HelloPhoenix.ReviewController :delete
. . .
admin_review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
admin_review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
admin_review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
admin_review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
admin_review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
admin_review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
                   PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
admin_review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
```

Penyokong path juga memulangkan apa yang kita mahu dari mereka.  Jalankan `$ iex -S mix` dan cuba sendiri.

```elixir
iex(1)> HelloPhoenix.Router.Helpers.review_path(Endpoint, :index)
"/reviews"

iex(2)> HelloPhoenix.Router.Helpers.admin_review_path(Endpoint, :show, 1234)
"/admin/reviews/1234"
```

Bagaimana pula jika kita mempunyai beberap resource yang semuanya diuruskan oleh admin?  Kita boleh memasukkan mereka ke dalam skop yang sama seperi ini:

```elixir
scope "/admin", as: :admin do
  pipe_through :browser

  resources "/images", HelloPhoenix.Admin.ImageController
  resources "/reviews", HelloPhoenix.Admin.ReviewController
  resources "/users", HelloPhoenix.Admin.UserController
end
```

Ini apa yang dipaparkan oleh `$ mix phoenix.routes`:

```elixir
. . .
 admin_image_path  GET     /admin/images HelloPhoenix.Admin.ImageController :index
 admin_image_path  GET     /admin/images/:id/edit HelloPhoenix.Admin.ImageController :edit
 admin_image_path  GET     /admin/images/new HelloPhoenix.Admin.ImageController :new
 admin_image_path  GET     /admin/images/:id HelloPhoenix.Admin.ImageController :show
 admin_image_path  POST    /admin/images HelloPhoenix.Admin.ImageController :create
 admin_image_path  PATCH   /admin/images/:id HelloPhoenix.Admin.ImageController :update
                   PUT     /admin/images/:id HelloPhoenix.Admin.ImageController :update
 admin_image_path  DELETE  /admin/images/:id HelloPhoenix.Admin.ImageController :delete
admin_review_path  GET     /admin/reviews HelloPhoenix.Admin.ReviewController :index
admin_review_path  GET     /admin/reviews/:id/edit HelloPhoenix.Admin.ReviewController :edit
admin_review_path  GET     /admin/reviews/new HelloPhoenix.Admin.ReviewController :new
admin_review_path  GET     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :show
admin_review_path  POST    /admin/reviews HelloPhoenix.Admin.ReviewController :create
admin_review_path  PATCH   /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
                   PUT     /admin/reviews/:id HelloPhoenix.Admin.ReviewController :update
admin_review_path  DELETE  /admin/reviews/:id HelloPhoenix.Admin.ReviewController :delete
  admin_user_path  GET     /admin/users HelloPhoenix.Admin.UserController :index
  admin_user_path  GET     /admin/users/:id/edit HelloPhoenix.Admin.UserController :edit
  admin_user_path  GET     /admin/users/new HelloPhoenix.Admin.UserController :new
  admin_user_path  GET     /admin/users/:id HelloPhoenix.Admin.UserController :show
  admin_user_path  POST    /admin/users HelloPhoenix.Admin.UserController :create
  admin_user_path  PATCH   /admin/users/:id HelloPhoenix.Admin.UserController :update
                   PUT     /admin/users/:id HelloPhoenix.Admin.UserController :update
  admin_user_path  DELETE  /admin/users/:id HelloPhoenix.Admin.UserController :delete
```

Ini bagus, betul-betul apa yang kita mahu, tetapi kita mahu membuat penambahbaikan.  Perhatikan setiap resource, kita perlu memanggil nama penuh controller dengan prefiks `HelloPhoenix.Admin`.  Ianya susah dan terdedah kepada kesilapan.  Anggapkan nama setiap controller bermula dengan `HelloPhoenix.Admin`, jadi kita boleh mena,bah `HelloPhoenix.Admin` kepada tetapan skop selepas tetapan path skop, dan semua route kita akan memiliki nama penuh controller yang tepat.

```elixir
scope "/admin", HelloPhoenix.Admin, as: :admin do
  pipe_through :browser

  resources "/images",  ImageController
  resources "/reviews", ReviewController
  resources "/users",   UserController
end
```

Sekarang jalankan `$ mix phoenix.routes` sekali lagi dan anda akan dapatmelihat hasil seperti di mana kita meletakkan nama penuh setiap controller setiap satu.

Sebagai bonus tambaha, kita boleh nest semua route untuk aplikasi kita di dalam satu skop yang hanya mempunyai satu alias kepada nama aplikasi Phoenix kita dan mengelakkan dari duplikasi di dalam nama controller kita.

Phoenix telah melakukan ini untuk kita di dalam router yang dijanakan untuk setiap aplikasi baru (lihat permulaan bahagian ini).  Perhatikan di sini penggunaan `HelloPhoenix.Router` di dalam tetapan `defmodule`:

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  scope "/", HelloPhoenix do
    pipe_through :browser

    get "/images", ImageController, :index
    resources "/reviews", ReviewController
    resources "/users",   UserController
  end
end
```

Sekali lagi `$ mix phoenix.routes` memberitahu kita semua controller kita mempunyai nama penuh.

```elixir
image_path   GET     /images            HelloPhoenix.ImageController :index
review_path  GET     /reviews           HelloPhoenix.ReviewController :index
review_path  GET     /reviews/:id/edit  HelloPhoenix.ReviewController :edit
review_path  GET     /reviews/new       HelloPhoenix.ReviewController :new
review_path  GET     /reviews/:id       HelloPhoenix.ReviewController :show
review_path  POST    /reviews           HelloPhoenix.ReviewController :create
review_path  PATCH   /reviews/:id       HelloPhoenix.ReviewController :update
             PUT     /reviews/:id       HelloPhoenix.ReviewController :update
review_path  DELETE  /reviews/:id       HelloPhoenix.ReviewController :delete
  user_path  GET     /users             HelloPhoenix.UserController :index
  user_path  GET     /users/:id/edit    HelloPhoenix.UserController :edit
  user_path  GET     /users/new         HelloPhoenix.UserController :new
  user_path  GET     /users/:id         HelloPhoenix.UserController :show
  user_path  POST    /users             HelloPhoenix.UserController :create
  user_path  PATCH   /users/:id         HelloPhoenix.UserController :update
             PUT     /users/:id         HelloPhoenix.UserController :update
  user_path  DELETE  /users/:id         HelloPhoenix.UserController :delete
```

Walaupun secara teknikal skop-skop boleh di-nest-kan (seperti resource), penggunaan skop ber-nest selalunya tidak digalakkan sebab ia kadang-kadang membuat kod kita kurang jelas.  Walaupunbegitu, katakan kita ada API berversi dengan tetapan resource untuk imej, review dan user.  Secara teknikalnya kita boleh menetapkan route untuk API berversi itu seperti ini:

```elixir
scope "/api", HelloPhoenix.Api, as: :api do
  pipe_through :api

  scope "/v1", V1, as: :v1 do
    resources "/images",  ImageController
    resources "/reviews", ReviewController
    resources "/users",   UserController
  end
end
```

`$ mix phoenix.routes` memberitahu kita memiliki route yang kita cari.

```elixir
 api_v1_image_path  GET     /api/v1/images HelloPhoenix.Api.V1.ImageController :index
 api_v1_image_path  GET     /api/v1/images/:id/edit HelloPhoenix.Api.V1.ImageController :edit
 api_v1_image_path  GET     /api/v1/images/new HelloPhoenix.Api.V1.ImageController :new
 api_v1_image_path  GET     /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :show
 api_v1_image_path  POST    /api/v1/images HelloPhoenix.Api.V1.ImageController :create
 api_v1_image_path  PATCH   /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :update
                    PUT     /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :update
 api_v1_image_path  DELETE  /api/v1/images/:id HelloPhoenix.Api.V1.ImageController :delete
api_v1_review_path  GET     /api/v1/reviews HelloPhoenix.Api.V1.ReviewController :index
api_v1_review_path  GET     /api/v1/reviews/:id/edit HelloPhoenix.Api.V1.ReviewController :edit
api_v1_review_path  GET     /api/v1/reviews/new HelloPhoenix.Api.V1.ReviewController :new
api_v1_review_path  GET     /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :show
api_v1_review_path  POST    /api/v1/reviews HelloPhoenix.Api.V1.ReviewController :create
api_v1_review_path  PATCH   /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :update
                    PUT     /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :update
api_v1_review_path  DELETE  /api/v1/reviews/:id HelloPhoenix.Api.V1.ReviewController :delete
  api_v1_user_path  GET     /api/v1/users HelloPhoenix.Api.V1.UserController :index
  api_v1_user_path  GET     /api/v1/users/:id/edit HelloPhoenix.Api.V1.UserController :edit
  api_v1_user_path  GET     /api/v1/users/new HelloPhoenix.Api.V1.UserController :new
  api_v1_user_path  GET     /api/v1/users/:id HelloPhoenix.Api.V1.UserController :show
  api_v1_user_path  POST    /api/v1/users HelloPhoenix.Api.V1.UserController :create
  api_v1_user_path  PATCH   /api/v1/users/:id HelloPhoenix.Api.V1.UserController :update
                    PUT     /api/v1/users/:id HelloPhoenix.Api.V1.UserController :update
  api_v1_user_path  DELETE  /api/v1/users/:id HelloPhoenix.Api.V1.UserController :delete
```
Yang menraiknya, kita boleh menggunakan beberap skop dengan path yang sama jika kita berhati-hati untuk tidak membuat route bertindih.  Jika kita ada route bertindih, kita akan mendapat amaran.

```console
warning: this clause cannot match because a previous clause at line 16 always matches
```
Router ini boleh berfungsi dengan tetapan dua skop untuk path yang sama.

```elixir
defmodule HelloPhoenix.Router do
  use Phoenix.Router
  . . .
  scope "/", HelloPhoenix do
    pipe_through :browser

    resources "/users", UserController
  end

  scope "/", AnotherApp do
    pipe_through :browser

    resources "/posts", PostController
  end
  . . .
end
```
Dan apabila kita menjalankan `$ mix phoenix.routes`, kita akan melihat output berikut:

```elixir
user_path  GET     /users           HelloPhoenix.UserController :index
user_path  GET     /users/:id/edit  HelloPhoenix.UserController :edit
user_path  GET     /users/new       HelloPhoenix.UserController :new
user_path  GET     /users/:id       HelloPhoenix.UserController :show
user_path  POST    /users           HelloPhoenix.UserController :create
user_path  PATCH   /users/:id       HelloPhoenix.UserController :update
           PUT     /users/:id       HelloPhoenix.UserController :update
user_path  DELETE  /users/:id       HelloPhoenix.UserController :delete
post_path  GET     /posts           AnotherApp.PostController :index
post_path  GET     /posts/:id/edit  AnotherApp.PostController :edit
post_path  GET     /posts/new       AnotherApp.PostController :new
post_path  GET     /posts/:id       AnotherApp.PostController :show
post_path  POST    /posts           AnotherApp.PostController :create
post_path  PATCH   /posts/:id       AnotherApp.PostController :update
           PUT     /posts/:id       AnotherApp.PostController :update
post_path  DELETE  /posts/:id       AnotherApp.PostController :delete
```

### Pipelines

Kita telah jauh meneroka panduan ini tanpa membincangkan salah satu baris yang kita lihat di dalam router - `pipe_through :browser`.  Sekarang masa untuk bincangkannya.

Di dalam panduan [Overview Guide](http://www.phoenixframework.org/docs/overview) kita menerangkan bahawa plug adalah tersusun dan dilaksanakan mengikut urutan yang telah ditetapkan, seperti satu saluran paip? Sekarang kita aka mengambil pandangan lebih dekat bagaimana susunan plug berfungsi di dalam router.

Pipeline hanyalah satu susunan plug disusun di dalam satu urutan dan diberikan satu nama.  Mereka membenarkan kita mengubahsuai carakerja dan transformasi berkaitan dengan pengurusan permohonan web.  Phoenix membekalkan kita beberapa pipeline lalai untuk beberapa tugasan biasa.  Kita pula boleh mengubahsuai mereka dan juga membina pipeline baru sesuai dengan kegunaan kita.

Satu aplikasi yang baru dijanakan menetapkan dua pipeline dipanggil `:browser` dan `:api`.  Kita akan bincangkan mereka sebentar lagi, tetapi terlebih dahulu kita perlu bincangkan susunan plug di dalam plug Endpoint.

##### Plug Endpoint Plug

Endpoint menguruskan semua plug yang lazim kepada setiap permohonan, dan melasanakan mereka sebelum diagihkan kepada router bersama dengan `:browser`, `:api` dan pipeline lain.  Plug Endpoint lalai melakukan banyak tugasan.  Ini senarai mereka, dalam susunan.

- [Plug.Static](http://hexdocs.pm/plug/Plug.Static.html) - melayan aset statik.  Oleh kerana plug ini dilaksanakan sebelum logger, layanan aset statik tidak disimpan di dalam log.

- [Plug.Logger](http://hexdocs.pm/plug/Plug.Logger.html) - membuat log setiap permohonan masuk

- [Phoenix.CodeReloader](http://hexdocs.pm/phoenix/Phoenix.CodeReloader.html) - satu plug yang membenarkan pemasangan kod untuk semua isi kandungan direktori `web`. 

- [Plug.Parsers](http://hexdocs.pm/plug/Plug.Parsers.html) - menghurai badan kandungan permohonan apabila ada pustaka penghurai. Pustaka penghurai yang dipasang secara lalai ialah urlencoded, multipart dan json (bersama dengan pustaka poison).  Badan kandungan permohonan tidak akan disentuh jika content-type permohonan itu tidak boleh dihurai. 

- [Plug.MethodOverride](http://hexdocs.pm/plug/Plug.MethodOverride.html) - menukarkan fungsi permohonan kepada 
  PUT, PATCH atau DELETE untuk permohonan POST dengan parameter `_method` yang sah.

- [Plug.Head](http://hexdocs.pm/plug/Plug.Head.html) - meukarkan permohonan HEAD kepada permohonan GET dan mempelaskan kandungan badan sambutan.

- [Plug.Session](http://hexdocs.pm/plug/Plug.Session.html) - plus yang menyediakan pengurusan session.
  Fungsi `fetch_session/2` mesti dipanggil secara eksplisit sebelum menggunakan session tersebut kerana plug ini cuma menyediakan bagaimana session itu dicapai.

- [Plug.Router](http://hexdocs.pm/plug/Plug.Router.html) - plug satu router kepada kitar permohonan.

##### Saluran Paip `:browser` and `:api` 

Phoenix menakrifkan dua lagi saluran paip secara lalai, `:browser` dan `:api`.  Router itu akan membangkitkan saluran paip ini setelah ia memadankan satu route, dengan sangkaan kita telah memanggil `pipe_through/1` dengan mereka di dalam skop.

As their names suggest, the `:browser` pipeline prepares for routes which render requests for a browser. The `:api` pipeline prepares for routes which produce data for an api.
Seperti nama mereka, saluran paip `:browser` menyediakan route yang menghantar permohonan kepada satu browser.  Saluran paio `:api` menyediakan route yang menghasilkan data untuk satu API.

Saluran paip `:browser` mememiliki lima plug: ``plug :accepts, ["html"]` menakrifkan format-format permohoman yang akan diterima.  `:fetch_session`, mencapai data session dan membuatnya sebagai boleh sedia di dalam hubungan, `:fetch_flash` yang mencapai mesej-mesej flash yang telah ditetapkan, begitu juga `:protect_from_forgery` and `:put_secure_browser_headers`, yang melindungi hantaran borang dari cross site forgery. 

Buat masa ini, saluran paip `:api` hanya menakrifkan `plug :accepts, ["json"]`.

Router tersebut akan membangkitkan satu saluran paip yang berada di dalam skop di mana route itu berada.  Jika tidak ada skop ditakrifkan, router itu akan membangkitkan saluran paip kesemua route di dalam router.  Walaupun penggunaan skop bersarang tidak digalakkan, jika kita memanggil `pipe_through` dari dalam satu skop bersarang, router itu akan membangkitkan semua `pipe_through` dari skop induk, di ikuti oleh yang bersarang.

Itu semua banyak perkataan bercampur-campur.  Mari lihat beberapa contoh untuk menjelaskan mereka.

Berikut adalah pandang semula satu router dari satu janaan baru aplikasi Phoenix, masa ini dengan skop api dinyahkomen dan satu route ditambah.

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
    pipe_through :browser

    get "/", PageController, :index
  end

  # Other scopes may use custom stacks.
  scope "/api", HelloPhoenix do
    pipe_through :api

    resources "/reviews", ReviewController
  end
end
```

Apabila pelayan menerima satu permohonan, permohonan itu akan sentiasa melalui plug-plug di dalam Endpoint kita, yang mana selepas itu akan cuba dipadankan kepada path dan HTTP verb.

Katalah permohonan tersebut padan dengan route pertama kita: satu GET kepada `/`.  Router tersebut pertama sekali akan melalukan permohonan tersebut kepada saluran paip `:browser` - yang akan mencapai data sessi, mencapai mesej-mesej flash, dan melaksanankan perlindungan pemalsuan - mengagihkan permohonan tersebut  kepada action `index` `PageController`.

Sebaliknya, jika permohonan itu padan dengan mana-mana route ditakrifkan oleh makro `resources/2`, router itu akan melalukannya melalui saluran paip `:api` - yang semasa tidak melakukan apa-apa - sebelumn ia diagihkan kepada action yang betul dari `HelloPhoenix.ReviewController`.

Jika kita tahu aplikasi kita cuma memproses view untuk browser, kita boleh meringkaskan router kita sedikit dengan mengeluarkan apa-apa berkaitan dengan `api` dan juga mengeluarkan penggunaan skop:

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

  pipe_through :browser

  get "/", HelloPhoenix.PageController, :index

  resources "/reviews", HelloPhoenix.ReviewController
end
```
Dengan mengeluarkan semua skop, router akan membangkitkan saluran paip `:browser` ke atas semua route.

Bagaimana pula jika kita perlu melalukan permohonan melalui kedua-dua saluran paip `:browser` dan satu atau lebih saluran paip custom?  Kita cuma perlu `pipe_through` beberapa saluran paip, dan Phoenix akan membangkitkan mereka mengikut susunan.

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
  ...

  scope "/reviews" do
    # Use the default browser stack.
    pipe_through [:browser, :review_checks, :other_great_stuff]

    resources "/reviews", HelloPhoenix.ReviewController
  end
end
```

Berikut adalah contoh di mana dua skop mempunyai saluran paip berlainan:

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
  ...

  scope "/", HelloPhoenix do
    pipe_through :browser

    resources "/posts", PostController
  end

  scope "/reviews", HelloPhoenix do
    pipe_through [:browser, :review_checks]

    resources "/reviews", ReviewController
  end
end
```

Secara am, aturan skop untuk saluran paip bertindak sebagaimana yang anda jangkakan.  Di dalam contoh ini, semua route akan dilalukan melalui saluran paip `:browser`.  Cuma, hanya route resource `reviews` akan memlalui saluran paip `:review_checks`.  Oleh sebab kita menetapkan kedua-dua paip `pipe_through [:browser, :review_checks]` di dalam senarai saluran paip, Phoenix akan `pipe_through` setiap satu semasa ia membangkitkan mereka mengikut susunan.

##### Membina Saluran Paip(Pipeline) Baru

Phoenix membenarkan kita membina saluran paip ubahsuai kita sendiri di mana-mana bahagian di dalam router.  Untuk melakukannya, kita panggil makro `pipeline/2` dengan argumen berikut: satu atom sebagai nama saluran paip baru kita dan  satu blok mengandungi semua plug yang kita perlukan.

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

  pipeline :review_checks do
    plug :ensure_authenticated_user
    plug :ensure_user_owns_review
  end

  scope "/reviews", HelloPhoenix do
    pipe_through :review_checks

    resources "/reviews", ReviewController
  end
end
```

### Route Untuk Channel

Channels are a very exciting, real-time component of the Phoenix framework. Channels handle incoming and outgoing messages broadcast over a socket for a given topic. Channel routes, then, need to match requests by socket and topic in order to dispatch to the correct channel. (For a more detailed description of channels and their behavior, please see the [Channel Guide](http://www.phoenixframework.org/docs/channels).)
Channel adalah komponen masa-nyata untuk Phoenix.  Channel menguruskan proses keluar masuk mesej-mesej siaran melalui satu soket untuk sesatu topik.  Route Channel, perlu memadankan permohonan melalui soket dan topik untuk pengagihan kepada channel yang betul.  (Mohon lihat [Channel Guide](http://www.phoenixframework.org/docs/channels) untuk maklumat lanjut.)

Kita pasangkan pengurus soket di dalam endpoint di `lib/hello_phoenix/endpoint.ex`.  Pengurus soket menguruskan authentication callback dan route-route channel.

```elixir
defmodule HelloPhoenix.Endpoint do
  use Phoenix.Endpoint

  socket "/socket", HelloPhoenix.UserSocket
  ...
end
```

Seterusnya, kita perlu membuka fail `web/channels/user_socket.ex` dan gunakan makro `channel/3` untuk menakrifkan route channel kita.  Route tersebut akan memadankan satu corak topik dengan satu channel untuk menguruskan event.  Jka kita mempunyai satu modul channel bernama `RoomChannel` dan satu topik bernama `"rooms:*"`, kod untuknya amat jelas.

```elixir
defmodule HelloPhoenix.UserSocket do
  use Phoenix.Socket

  channel "rooms:*", HelloPhoenix.RoomChannel
  ...
end
```

Topik-topik hanyalah pendanda-penanda string.  Bentuk yang kita gunakan di sini ialah satu resam yang membenarkan kita menakrifkan topik-topik dan subtopik di dalam string yang sama - "topic:subtopic".  Simbol `*` ialah aksara kad liar yang membenarkan kita untuk memadankan mana-mana subtopik, jadi `"rooms:lobby"` dan `"rooms:kitchen"` akan boleh memadankan route ini.


Phoenix meringkaskan socket transfer layer dan memasukkan dua mekanisma transport secara lalai - WebSockets dan Long-Polling.  Jika kita mahu memastikan channel kita diuruskan oleh hanya satu jenis transport, kita boleh membuat takrifan menggunakan pilihan `via`, seperti ini.

```elixir
channel "rooms:*", HelloPhoenix.RoomChannel, via: [Phoenix.Transports.WebSocket]
```

Setiap soket boleh menguruskan permohonan untk pelbagai channel.

```elixir
channel "rooms:*", HelloPhoenix.RoomChannel, via: [Phoenix.Transports.WebSocket]
channel "foods:*", HelloPhoenix.FoodChannel
```

Kita boleh memasang pelbagai pengurus soket di dalam endpoint kita:

```elixir
socket "/socket", HelloPhoenix.UserSocket
socket "/admin-socket", HelloPhoenix.AdminSocket
```


### Ringkasan

Routing ialah satu topik yang besar, dan kita telah meneroka banyak kawasan di sini.  Perkara penting untuk diambil dari panduan ini adalah:
- Route yang bermula dengan satu nama HTTP verb akan dikembangkan kepada satu sahaja klausa fungsi `match`.
- Route yang bermula dengan 'resources' akan dikembangkan kepada 8 klausa fungsi `match`.
- 'Resources' boleh menyekat jumlah klausa fungsi `match` menggunakan pilihan `only:` atau `except:`. 
- Mana-mana route boleh disarangkan.
- Mana-mana route boleh diskopkan kepada satu path.
- Menggunakan pilihan `as:` akan mengurangkan pertindihan.
- Using the helper option for scoped routes eliminates unreachable paths.
- Menggunaka pilihan penyokong path untuk route berskop akan menyingkirkan path yang tidak dapat dicapai.

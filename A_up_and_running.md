---
layout: page
title: Siap Sedia Mula!
permalink: /up-and-running
---

Tujuan utama panduan pertama ini adalah untuk mendapatkan aplikasi Phoenix berfungsi secepat mungkin.

Sebelum kita bermula, mohon ambil sedikit masa untuk membaca [Panduan Pemasangan](http://www.phoenixframework.org/docs/installation).  Dengan memasang apa-apa dependency yang diperlukan terlebih dahulu, kita akan dapat melicinkan proses menjalankan aplikasi kita.

Pada masa ini, kita sepatutnya sudah ada Elixir, Erlang, Hex dan arkib Phoenix terpasang.  Kita juga sepatutnya mempunyai PostgreSQL dan node.js siap terpasang untuk membina satu aplikasi lalai.

Ok, kita sedia untuk bermula!

Kita boleh jalankan `mix phoenix.new` dari mana-mana direktori untuk mula membina aplikasi Phoenix.  Phoenix akan menerima samada jejak absolut atau relatif kepada direktori projek baru kita.  Anggapkan nama aplikasi kita ialah `hello_phoenix`, kedua-dua arahan di bawah boleh digunakan:

```console
$ mix phoenix.new /Users/me/work/elixir-stuff/hello_phoenix
```

```console
$ mix phoenix.new hello_phoenix
```

>Nota mengenai [Brunch.io](http://brunch.io) sebelum kita bermula: Phoenix akan menggunakan Brunch.io untuk pengurusan aset secara lalai.  Dependency Brunch.io dipasang melalui `node package manager(npm)`, bukan melalui `mix`.  Phoenix akan prompt kita untuk memasang dependency tersebut di akhir tugasan `mix phoenix.new`.  Jika kita memilih "no" pada masa itu, dan kita tidak membuat pemasangan melalui `npm install`, aplikasi kita akan menimbulkan ralat apabila mula dijalankan, dan aset kita tidak dipasang dengan sempurna.  Jika kita langsung tidak mahu menggunakan Brunch.io, kita cuma perlu hantarkan penanda `--no-brunch` kepada `mix phoenix.new`.

Setelah bersedia, kita boleh jalankan `mix phoenix.new` dengan satu jejak relatif 

```console
mix phoenix.new hello_phoenix
* creating hello_phoenix/config/config.exs
* creating hello_phoenix/config/dev.exs
* creating hello_phoenix/config/prod.exs
...
* creating hello_phoenix/web/views/layout_view.ex
* creating hello_phoenix/web/views/page_view.ex

Fetch and install dependencies? [Yn]
```

Phoenix menjana struktur direktori tersebut dan semua fail-fail yang akan kita perlukan untuk aplikasi kita.  Apabila siap, ia akan bertanya jika kita mahukan ia memasang dependency untuk kita.  Kita akan katakan 'yes' untuk itu.

```console
Fetch and install dependencies? [Yn] y
* running npm install && node node_modules/brunch/bin/brunch build
* running mix deps.get

We are all set! Run your Phoenix application:

    $ cd hello_phoenix
    $ mix ecto.create
    $ mix phoenix.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phoenix.server
```

Setelah dependency kita dipasang, tugasan itu akan prompt supaya kita berpindah ke dalam direktori projek dan jalankan aplikasi kita.

Phoenix menganggap bahawa pangkalan data PostgreSQL kita mempunyai satu akaun pengguna `postgres` dengan pelepasan yang betul dan kata laluan "postgres".  Jika itu bukan keadaannya, mohon lihat arahan-arahan untuk tugasan mix [ecto.create](http://www.phoenixframework.org/docs/mix-tasks#section--ecto-create).


Ok, mari kita mencuba.  Mula-mula kita akan `cd` ke dalam direktori `hello_phoenix/` yang baru kita buat:

    $ cd hello_phoenix

Sekarang kita akan membina pangkalan data:

```
$ mix ecto.create
The database for HelloPhoenix.Repo has been created.
```

>Nota: jika ini kali pertama anda menjalankan arahan ini, Phoenix mungkin akan memohon untuk memasang Rebar.  Teruskan dengan pemasangan kerana Rebar digunakan untuk membina pakej-pakej Erlang. 

Dan akhirnya, kita akan mulakan pelayan Phoenix:

```console
$ mix phoenix.server
[info] Running HelloPhoenix.Endpoint with Cowboy on http://localhost:4000
23 Nov 05:25:14 - info: compiled 5 files into 2 files, copied 3 in 1724ms
```

Jika kita memilih untuk tidak membenarkan Phoenix memasang dependency ketika menjana aplikasi baru, tugasan `phoenix.new` akan menggesa kita utuk menlakukan langkah-langkah yang diperlukan bila kita mahu memasang mereka.

```console
Fetch and install dependencies? [Yn] n

Phoenix uses an optional assets build tool called brunch.io
that requires node.js and npm. Installation instructions for
node.js, which includes npm, can be found at http://nodejs.org.

After npm is installed, install your brunch dependencies by
running inside your app:

    $ npm install

If you don't want brunch.io, you can re-run this generator
with the --no-brunch option.


We are all set! Run your Phoenix application:

    $ cd hello_phoenix
    $ mix deps.get
    $ mix ecto.create
    $ mix phoenix.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phoenix.server
```

Phoenix menerima permohonan di port 4000 secara lalai.  Jika kita tunjukkan browser kita kepada [http://localhost:4000](http://localhost:4000), kita sepatutnya dapat melihat paparan aluan Framework Phoenix.

![Phoenix Welcome Page](/images/welcome-to-phoenix.png)

Jika paparan anda adalah seperti di atas, tahniah! Anda mempunyai satu aplikasi Phoenix yang berfungsi.  Jika anda tidak mendapat paparan sebagaimana di atas, cuba mengakses melalui [http://127.0.0.1:4000](http://127.0.0.1:4000) dan kemudian pastikan sistem pengendalian anda menetapkan "localhost" sebagai "127.0.0.1".

Secara lokal, aplikasi kita berjalan di dalam satu sesi iex.  Untuk menghentikannya, tekan ctrl-c dua kali, sama jika kita mahu hentikan sesi iex secara normal.

Langkah seterusnya ialah mengubahsuai aplikasi kita sedikit untuk memberikan kita rasa bagaimana satu aplikasi Phoenix dibina.

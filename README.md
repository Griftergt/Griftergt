# Домашнее задание. 5. Сборка RPM-пакета и создание репозитория

## Цель задания
Научиться собирать RPM-пакеты.
Создавать собственный RPM-репозиторий.
## Задание
- создать свой RPM (можно взять свое приложение, либо собрать к примеру Apache с определенными опциями);
- cоздать свой репозиторий и разместить там ранее собранный RPM;
- реализовать это все либо в Vagrant, либо развернуть у себя через Nginx и дать ссылку на репозиторий.

## Выполнение
Буду собирать deb-пакет и создавать локальный APT-репозиторий 

### 1. Подготовка 
Установка инструментов сборки
```bash
root@ubuntu-server:~# sudo apt install -y build-essential dpkg-dev devscripts fakeroot nginx wget git cmake gcc g++ libpcre3-dev zlib1g-dev libssl-dev
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Уже установлен пакет wget самой новой версии (1.21.4-1ubuntu4.5).
wget помечен как установленный вручную.
Уже установлен пакет git самой новой версии (1:2.43.0-1ubuntu7.4).
git помечен как установленный вручную.
Будут установлены следующие дополнительные пакеты:
  binutils binutils-common binutils-x86-64-linux-gnu bzip2 cmake-data cpp cpp-13 cpp-13-x86-64-linux-gnu cpp-x86-64-linux-gnu dctrl-tools diffstat dput g++-13
  g++-13-x86-64-linux-gnu g++-x86-64-linux-gnu gcc-13 gcc-13-base gcc-13-x86-64-linux-gnu gcc-x86-64-linux-gnu gettext intltool-debian libalgorithm-diff-perl
  libalgorithm-diff-xs-perl libalgorithm-merge-perl libaliased-perl libapt-pkg-perl libarchive-zip-perl libarray-intspan-perl libasan8 libatomic1 libauthen-sasl-perl
  libb-hooks-endofscope-perl libb-hooks-op-check-perl libberkeleydb-perl libbinutils libcapture-tiny-perl libcc1-0 libcgi-fast-perl libcgi-pm-perl libclass-data-inheritable-perl
  
  ...

Настраивается пакет libxml-sax-expat-perl (0.51-2) …
update-perl-sax-parsers: Registering Perl SAX parser XML::SAX::Expat with priority 50...
update-perl-sax-parsers: Updating overall Perl SAX parser modules info file...
Replacing config file /etc/perl/XML/SAX/ParserDetails.ini with new version
Обрабатываются триггеры для install-info (7.1-3build2) …
Обрабатываются триггеры для libc-bin (2.39-0ubuntu8.9) …
Обрабатываются триггеры для ufw (0.36.2-6) …
Обрабатываются триггеры для man-db (2.12.0-4build2) …
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

```
 

Загрузка и распаковка исходников
```bash
root@ubuntu-server:~# cd /root
root@ubuntu-server:~# ls -l
total 0
root@ubuntu-server:~# git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
Cloning into 'ngx_brotli'...
remote: Enumerating objects: 240, done.
remote: Counting objects: 100% (73/73), done.
remote: Compressing objects: 100% (23/23), done.
remote: Total 240 (delta 56), reused 50 (delta 50), pack-reused 167 (from 2)
Receiving objects: 100% (240/240), 78.96 KiB | 15.79 MiB/s, done.
Resolving deltas: 100% (117/117), done.
Submodule 'deps/brotli' (https://github.com/google/brotli.git) registered for path 'deps/brotli'
Cloning into '/root/ngx_brotli/deps/brotli'...
remote: Enumerating objects: 9891, done.
remote: Counting objects: 100% (458/458), done.
remote: Compressing objects: 100% (216/216), done.
remote: Total 9891 (delta 325), reused 251 (delta 241), pack-reused 9433 (from 2)
Receiving objects: 100% (9891/9891), 41.99 MiB | 7.14 MiB/s, done.
Resolving deltas: 100% (6390/6390), done.
Submodule path 'deps/brotli': checked out 'db24a98aed38a16de78cc05ca15cdd8dc2a0d92d'
root@ubuntu-server:~# ls -l
total 4
drwxr-xr-x 7 root root 4096 окт 28 07:14 ngx_brotli
root@ubuntu-server:~# cd ngx_brotli/deps/brotli
root@ubuntu-server:~/ngx_brotli/deps/brotli# ls -l
total 128
-rw-r--r-- 1 root root  2682 окт 28 07:14 BUILD.bazel
drwxr-xr-x 8 root root  4096 окт 28 07:14 c
-rw-r--r-- 1 root root  6855 окт 28 07:14 CHANGELOG.md
-rw-r--r-- 1 root root 14731 окт 28 07:14 CMakeLists.txt
-rw-r--r-- 1 root root  1633 окт 28 07:14 CONTRIBUTING.md
drwxr-xr-x 3 root root  4096 окт 28 07:14 csharp
drwxr-xr-x 2 root root  4096 окт 28 07:14 docs
drwxr-xr-x 2 root root  4096 окт 28 07:14 fetch-spec
drwxr-xr-x 4 root root  4096 окт 28 07:14 go
drwxr-xr-x 3 root root  4096 окт 28 07:14 java
drwxr-xr-x 2 root root  4096 окт 28 07:14 js
-rw-r--r-- 1 root root  1084 окт 28 07:14 LICENSE
-rw-r--r-- 1 root root   409 окт 28 07:14 MANIFEST.in
-rw-r--r-- 1 root root   294 окт 28 07:14 MODULE.bazel
-rw-r--r-- 1 root root    94 окт 28 07:14 pyproject.toml
drwxr-xr-x 3 root root  4096 окт 28 07:14 python
-rw-r--r-- 1 root root   650 окт 28 07:14 README
-rw-r--r-- 1 root root  4586 окт 28 07:14 README.md
drwxr-xr-x 3 root root  4096 окт 28 07:14 research
-rw-r--r-- 1 root root   929 окт 28 07:14 sbom.cdx.json
drwxr-xr-x 3 root root  4096 окт 28 07:14 scripts
-rw-r--r-- 1 root root   314 окт 28 07:14 SECURITY.md
-rw-r--r-- 1 root root    53 окт 28 07:14 setup.cfg
-rw-r--r-- 1 root root 11754 окт 28 07:14 setup.py
drwxr-xr-x 3 root root  4096 окт 28 07:14 tests
root@ubuntu-server:~/ngx_brotli/deps/brotli# mkdir out && cd out
root@ubuntu-server:~/ngx_brotli/deps/brotli/out# cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF ..
-- The C compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type is 'Release'
-- Performing Test BROTLI_EMSCRIPTEN
-- Performing Test BROTLI_EMSCRIPTEN - Failed
-- Compiler is not EMSCRIPTEN
-- Looking for log2 in m
-- Looking for log2 in m - found
-- Configuring done (0.5s)
-- Generating done (0.0s)
-- Build files have been written to: /root/ngx_brotli/deps/brotli/out
root@ubuntu-server:~/ngx_brotli/deps/brotli/out# cmake --build . --config Release -j 2 --target brotlienc
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/context.c.o
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/constants.c.o
[  9%] Building C object CMakeFiles/brotlicommon.dir/c/common/dictionary.c.o
[ 12%] Building C object CMakeFiles/brotlicommon.dir/c/common/platform.c.o
[ 16%] Building C object CMakeFiles/brotlicommon.dir/c/common/shared_dictionary.c.o
[ 19%] Building C object CMakeFiles/brotlicommon.dir/c/common/transform.c.o
[ 22%] Linking C static library libbrotlicommon.a
[ 22%] Built target brotlicommon
[ 25%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references_hq.c.o
[ 29%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references.c.o
[ 32%] Building C object CMakeFiles/brotlienc.dir/c/enc/bit_cost.c.o
[ 35%] Building C object CMakeFiles/brotlienc.dir/c/enc/block_splitter.c.o
[ 38%] Building C object CMakeFiles/brotlienc.dir/c/enc/brotli_bit_stream.c.o
[ 41%] Building C object CMakeFiles/brotlienc.dir/c/enc/cluster.c.o
[ 45%] Building C object CMakeFiles/brotlienc.dir/c/enc/command.c.o
[ 48%] Building C object CMakeFiles/brotlienc.dir/c/enc/compound_dictionary.c.o
[ 51%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment.c.o
[ 54%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment_two_pass.c.o
[ 58%] Building C object CMakeFiles/brotlienc.dir/c/enc/dictionary_hash.c.o
[ 61%] Building C object CMakeFiles/brotlienc.dir/c/enc/encode.c.o
[ 64%] Building C object CMakeFiles/brotlienc.dir/c/enc/encoder_dict.c.o
[ 67%] Building C object CMakeFiles/brotlienc.dir/c/enc/entropy_encode.c.o
[ 70%] Building C object CMakeFiles/brotlienc.dir/c/enc/fast_log.c.o
[ 74%] Building C object CMakeFiles/brotlienc.dir/c/enc/histogram.c.o
[ 77%] Building C object CMakeFiles/brotlienc.dir/c/enc/literal_cost.c.o
[ 80%] Building C object CMakeFiles/brotlienc.dir/c/enc/memory.c.o
[ 83%] Building C object CMakeFiles/brotlienc.dir/c/enc/metablock.c.o
[ 87%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_dict.c.o
[ 90%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_dict_lut.c.o
[ 93%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_init.c.o
[ 96%] Building C object CMakeFiles/brotlienc.dir/c/enc/utf8_util.c.o
[100%] Linking C static library libbrotlienc.a
[100%] Built target brotlienc
root@ubuntu-server:~/ngx_brotli/deps/brotli/out# cd ../../../..
root@ubuntu-server:~# pwd
/root

```


Включение источника исходного кода
```bash
root@ubuntu-server:~# cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb
URIs: http://ru.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
root@ubuntu-server:~# ^C
root@ubuntu-server:~# nano /etc/apt/sources.list.d/ubuntu.sources
root@ubuntu-server:~# nano /etc/apt/sources.list.d/ubuntu.sources
root@ubuntu-server:~# cat /etc/apt/sources.list.d/ubuntu.sources
Types: deb deb-src
URIs: http://ru.archive.ubuntu.com/ubuntu/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb deb-src
URIs: http://security.ubuntu.com/ubuntu/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

root@ubuntu-server:~# apt update

```

Получение исходного кода nginx 

```bash
root@ubuntu-server:~# cd /usr/local/src
root@ubuntu-server:/usr/local/src# ls -l
total 0
root@ubuntu-server:/usr/local/src# sudo apt-get source nginx
Чтение списков пакетов… Готово
ВНИМАНИЕ: работа над пакетом «nginx» ведётся в системе управления версиями «Git»:
https://salsa.debian.org/nginx-team/nginx.git
Используйте:
git clone https://salsa.debian.org/nginx-team/nginx.git
для получения последних (возможно, невыпущенных) обновлений пакета.
Необходимо скачать 1 219 kB архивов исходного кода.
Пол:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main nginx 1.24.0-2ubuntu7.19 (tar) [1 112 kB]
Пол:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main nginx 1.24.0-2ubuntu7.19 (diff) [103 kB]
Пол:3 http://ru.archive.ubuntu.com/ubuntu noble-updates/main nginx 1.24.0-2ubuntu7.19 (dsc) [3 624 B]
Получено 1 219 kB за 1с (1 302 kB/s)
dpkg-source: info: extracting nginx in nginx-1.24.0
dpkg-source: info: unpacking nginx_1.24.0.orig.tar.gz
dpkg-source: info: unpacking nginx_1.24.0-2ubuntu7.19.debian.tar.xz
dpkg-source: info: using patch list from debian/patches/series
dpkg-source: info: applying 0003-define_gnu_source-on-other-glibc-based-platforms.patch
dpkg-source: info: applying nginx-fix-pidfile.patch
dpkg-source: info: applying nginx-ssl_cert_cb_yield.patch
dpkg-source: info: applying CVE-2023-44487.patch
dpkg-source: info: applying ubuntu-branding.patch
dpkg-source: info: applying CVE-2024-7347-1.patch
dpkg-source: info: applying CVE-2024-7347-2.patch
dpkg-source: info: applying CVE-2025-23419.patch
dpkg-source: info: applying CVE-2025-53859.patch
dpkg-source: info: applying CVE-2026-1642.patch
dpkg-source: info: applying CVE-2026-27651.patch
dpkg-source: info: applying CVE-2026-27654.patch
dpkg-source: info: applying CVE-2026-27784.patch
dpkg-source: info: applying CVE-2026-28753.patch
dpkg-source: info: applying CVE-2026-28755.patch
dpkg-source: info: applying CVE-2026-32647.patch
dpkg-source: info: applying cve-2026-42945.patch
dpkg-source: info: applying CVE-2026-40701.patch
dpkg-source: info: applying CVE-2026-42934.patch
dpkg-source: info: applying CVE-2026-42946-1.patch
dpkg-source: info: applying CVE-2026-42946-2.patch
dpkg-source: info: applying CVE-2026-9256.patch
dpkg-source: info: applying CVE-2026-49975.patch
dpkg-source: info: applying CVE-2026-42055.patch
dpkg-source: info: applying CVE-2026-48142.patch
dpkg-source: info: applying CVE-2026-56434.patch
dpkg-source: info: applying CVE-2026-60005.patch
dpkg-source: info: applying CVE-2026-42533.patch
dpkg-source: info: applying CVE-2026-15176.patch
dpkg-source: info: applying CVE-2026-93479.patch
dpkg-source: info: applying CVE-2026-97205.patch
dpkg-source: info: applying CVE-2026-97629.patch
W: Загрузка выполняется от лица суперпользователя без ограничений песочницы, так как файл «nginx_1.24.0.orig.tar.gz» недоступен для пользователя «_apt». - pkgAcquire::Run (13: Permission denied)
root@ubuntu-server:/usr/local/src# ls -l
total 1200
drwxr-xr-x 10 root root    4096 окт 28 08:19 nginx-1.24.0
-rw-r--r--  1 root root  102960 окт 26 16:51 nginx_1.24.0-2ubuntu7.19.debian.tar.xz
-rw-r--r--  1 root root    3624 окт 26 16:51 nginx_1.24.0-2ubuntu7.19.dsc
-rw-r--r--  1 root root 1112471 июн 28  2023 nginx_1.24.0.orig.tar.gz

```

### 2. Просмотр и правка debian/rules
Было
```bash

# configure flags
basic_configure_flags := \
                        --prefix=/usr/share/nginx \
                        --conf-path=/etc/nginx/nginx.conf \
                        --http-log-path=/var/log/nginx/access.log \
                        --error-log-path=stderr \
                        --lock-path=/var/lock/nginx.lock \
                        --pid-path=/run/nginx.pid \
                        --modules-path=/usr/lib/nginx/modules \
                        --http-client-body-temp-path=/var/lib/nginx/body \
                        --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
                        --http-proxy-temp-path=/var/lib/nginx/proxy \
                        --http-scgi-temp-path=/var/lib/nginx/scgi \
                        --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
                        --with-compat \
                        --with-debug \
                        --with-pcre-jit \
                        --with-http_ssl_module \
                        --with-http_stub_status_module \
                        --with-http_realip_module \
                        --with-http_auth_request_module \
                        --with-http_v2_module \
                        --with-http_dav_module \
                        --with-http_slice_module \
                        --with-threads
```

Стало
```bash

# configure flags
basic_configure_flags := \
                        --prefix=/usr/share/nginx \
                        --conf-path=/etc/nginx/nginx.conf \
                        --http-log-path=/var/log/nginx/access.log \
                        --error-log-path=stderr \
                        --lock-path=/var/lock/nginx.lock \
                        --pid-path=/run/nginx.pid \
                        --modules-path=/usr/lib/nginx/modules \
                        --http-client-body-temp-path=/var/lib/nginx/body \
                        --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
                        --http-proxy-temp-path=/var/lib/nginx/proxy \
                        --http-scgi-temp-path=/var/lib/nginx/scgi \
                        --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
                        --with-compat \
                        --with-debug \
                        --with-pcre-jit \
                        --with-http_ssl_module \
                        --with-http_stub_status_module \
                        --with-http_realip_module \
                        --with-http_auth_request_module \
                        --with-http_v2_module \
                        --with-http_dav_module \
                        --with-http_slice_module \
                        --with-threads \
                        --add-module=/root/ngx_brotli
```



 
### 3. Установка зависимостей сборки


```bash
root@ubuntu-server:/usr/local/src/nginx-1.24.0/debian# sudo apt-get build-dep nginx -y
Чтение списков пакетов… Готово
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Следующие НОВЫЕ пакеты будут установлены:
  autoconf automake autopoint autotools-dev debhelper debugedit dh-autoreconf dh-strip-nondeterminism dwz geoip-bin icu-devtools libaom-dev libbrotli-dev libbz2-dev libdav1d-dev libdav1d7
  libde265-0 libde265-dev libdebhelper-perl libdeflate-dev libexpat1-dev libfile-stripnondeterminism-perl libfontconfig-dev libfreetype-dev libgd-dev libgeoip-dev libgeoip1t64 libheif-dev
  libicu-dev libjbig-dev libjpeg-dev libjpeg-turbo8-dev libjpeg8-dev liblerc-dev liblzma-dev libpcre2-16-0 libpcre2-32-0 libpcre2-dev libpcre2-posix3 libperl-dev libpkgconf3 libpng-dev
  libpthread-stubs0-dev libsharpyuv-dev libsub-override-perl libtiff-dev libtiffxx6 libtool libvpx-dev libvpx9 libwebp-dev libwebpdecoder3 libwebpdemux2 libwebpmux3 libx11-dev libx265-199
  libx265-dev libxau-dev libxcb1-dev libxdmcp-dev libxml2-dev libxpm-dev libxslt1-dev libzstd-dev m4 pkgconf pkgconf-bin po-debconf uuid-dev x11proto-dev xorg-sgml-doctools xtrans-dev
Обновлено 0 пакетов, установлено 72 новых пакетов, для удаления отмечено 0 пакетов, и 0 пакетов не обновлено.
Необходимо скачать 32,3 MB архивов.
После данной операции объём занятого дискового пространства возрастёт на 151 MB.
Пол:1 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 m4 amd64 1.4.19-4build1 [244 kB]
Пол:2 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 autoconf all 2.71-3 [339 kB]
Пол:3 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 autotools-dev all 20220109.1 [44,9 kB]

...

Настраивается пакет libxcb1-dev:amd64 (1.15-1ubuntu2) …
Настраивается пакет libx11-dev:amd64 (2:1.8.7-1ubuntu0.1) …
Настраивается пакет libxpm-dev:amd64 (1:3.5.17-1ubuntu0.24.04.2) …
Настраивается пакет libgd-dev:amd64 (2.3.3-9ubuntu5) …
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

```

### 4 Сборка
Переходим в каталог исходников и запускаем сборку
```bash
root@ubuntu-server:/usr/local/src/nginx-1.24.0/debian# cd /usr/local/src/nginx-1.24.0
root@ubuntu-server:/usr/local/src/nginx-1.24.0# pwd
/usr/local/src/nginx-1.24.0
root@ubuntu-server:/usr/local/src/nginx-1.24.0# sudo DEB_BUILD_OPTIONS=nocheck dpkg-buildpackage -b -uc -us 2>&1 | tee /root/nginx-build.log
dpkg-buildpackage: info: source package nginx
dpkg-buildpackage: info: source version 1.24.0-2ubuntu7.19
dpkg-buildpackage: info: source distribution noble-security
dpkg-buildpackage: info: source changed by Marc Deslauriers <marc.deslauriers@ubuntu.com>
 dpkg-source --before-build .
dpkg-buildpackage: info: host architecture amd64
 debian/rules clean
dh clean --without autoreconf
 ...
 dpkg-deb: сборка пакета «nginx-doc» в «../nginx-doc_1.24.0-2ubuntu7.19_all.deb».
 dpkg-genbuildinfo --build=binary -O../nginx_1.24.0-2ubuntu7.19_amd64.buildinfo
 dpkg-genchanges --build=binary -O../nginx_1.24.0-2ubuntu7.19_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)

```
Проверяем, что пакеты созданы
```bash

root@ubuntu-server:/usr/local/src# ls -lh *.deb *.ddeb 2>/dev/null | head -40
-rw-r--r-- 1 root root  24K окт 28 08:42 libnginx-mod-http-geoip_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root  36K окт 28 08:42 libnginx-mod-http-geoip-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root  28K окт 28 08:42 libnginx-mod-http-image-filter_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root  43K окт 28 08:42 libnginx-mod-http-image-filter-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root  36K окт 28 08:42 libnginx-mod-http-perl_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root 106K окт 28 08:42 libnginx-mod-http-perl-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root  27K окт 28 08:42 libnginx-mod-http-xslt-filter_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root  53K окт 28 08:42 libnginx-mod-http-xslt-filter-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root  59K окт 28 08:42 libnginx-mod-mail_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root 104K окт 28 08:42 libnginx-mod-mail-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root  85K окт 28 08:42 libnginx-mod-stream_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root 171K окт 28 08:42 libnginx-mod-stream-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root  23K окт 28 08:42 libnginx-mod-stream-geoip_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root  22K окт 28 08:42 libnginx-mod-stream-geoip-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root 842K окт 28 08:42 nginx_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root  45K окт 28 08:42 nginx-common_1.24.0-2ubuntu7.19_all.deb
-rw-r--r-- 1 root root  19K окт 28 08:42 nginx-core_1.24.0-2ubuntu7.19_all.deb
-rw-r--r-- 1 root root 1,3M окт 28 08:42 nginx-dbgsym_1.24.0-2ubuntu7.19_amd64.ddeb
-rw-r--r-- 1 root root 122K окт 28 08:42 nginx-dev_1.24.0-2ubuntu7.19_all.deb
-rw-r--r-- 1 root root  27K окт 28 08:42 nginx-doc_1.24.0-2ubuntu7.19_all.deb
-rw-r--r-- 1 root root  19K окт 28 08:42 nginx-extras_1.24.0-2ubuntu7.19_amd64.deb
-rw-r--r-- 1 root root  19K окт 28 08:42 nginx-full_1.24.0-2ubuntu7.19_all.deb
-rw-r--r-- 1 root root  19K окт 28 08:42 nginx-light_1.24.0-2ubuntu7.19_all.deb
root@ubuntu-server:/usr/local/src# date
Ср 28 окт 2026 08:52:36 UTC

```
Проверяем, что brotli вкомпилирован

```bash
root@ubuntu-server:/usr/local/src# grep -i brotli /root/nginx-build.log | head -20
cd /usr/local/src/nginx-1.24.0/debian/build-bin && ./configure --with-cc-opt="-g -O2 -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -ffile-prefix-map=/usr/local/src/nginx-1.24.0=. -flto=auto -ffat-lto-objects -fstack-protector-strong -fstack-clash-protection -Wformat -Werror=format-security -fcf-protection -fdebug-prefix-map=/usr/local/src/nginx-1.24.0=/usr/src/nginx-1.24.0-2ubuntu7.19 -fPIC -Wdate-time -D_FORTIFY_SOURCE=3" --with-ld-opt="-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -Wl,-z,relro -Wl,-z,now -fPIC" --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=stderr --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --add-module=/root/ngx_brotli --with-http_addition_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_secure_link_module --with-http_sub_module --with-mail_ssl_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-stream_realip_module --with-http_geoip_module=dynamic --with-http_image_filter_module=dynamic --with-http_perl_module=dynamic --with-http_xslt_module=dynamic --with-mail=dynamic --with-stream=dynamic --with-stream_geoip_module=dynamic
....

```
Проверяем наличие brotli в бинарнике

```bash
root@ubuntu-server:/usr/local/src# rm -rf /tmp/nginx-check
dpkg-deb -x /usr/local/src/nginx_1.24.0-2ubuntu7.19_amd64.deb /tmp/nginx-check
/tmp/nginx-check/usr/sbin/nginx -V 2>&1 | tr ' ' '\n' | grep -i brotli
--add-module=/root/ngx_brotli


```

### 5 Размещаем в репозитории

```bash


```

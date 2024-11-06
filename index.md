---
layout: default
---

[Link to another page](./another-page.html).
# Latar Belakang
    Di Era digitalisasi sekarang yang semakin pesat, banyak perusahaan yang memerlukan solusi yang efektif dan handal dalam mengelola aplikasi yang bersifat fleksibel, otomatis serta aman dari kejahatan digital. Serta dengan adanya Cloud Computing menjadikan banyak perusahaan membutuhkan tools yang dapat di jalankan pada environment tersebut. Kubernetes menjadi salah satu tools yang paling banyak diminati oleh beberapa perusahaan, karena kemampuannya dalam mengelola container dalam skala besar.
    Karena Hal tersebut juga menyebabkan adanya kebutuhan untuk penyimpanan Image private atau biasa disebut sebagai Private Registry agar menjaga keamanan dari perusahaannya, dan juga karena banyak nya kasus terkait pencurian data, menjadikan kebutuhan akan Scanning Vulnerability atau kerentanan menjadi cukup penting. Apalagi di era Cloud dan Containerisasi, Trivy dan Harbor merupakan salah satu solusi terkait dua hal tersebut. Yang dimana harbor sebagai tempat registry dan Trivy untuk scanning otomatis yang di trigger oleh Harbor saat ada event tertentu seperti Ada images yang baru di Push ke Harbor Registry.
    Serta dibutuhkan juga Tools untuk mengirim notifikasi secara cepat dan otomatis, sehingga dapat secara cepat juga untuk mengatasi Vulnerability atau kerentanan terhadap Images yang akan di simpan di Harbor, malasah tersebut juga bisa diatasi dengan menggunakan Slack.

# Tools
1. Kubernetes – v1.28.15
2. Kubeadm
3. Kubectl
4. Kubelet
5. Harbor – 2.10.0
6. Trivy – 0.47.0
7. Docker – 27.3.1
8. OpenSSL – 3.0.2
9. Slack

# Topologi
![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

# Teori
## Container

Container adalah sebuah unit yang mengemas code dan semua dependensinya. Sehingga dapat berjalan atau berpindah environment dengan lebih cepat dan efisien. Container tersebut sangat ringan tidak seperti Virtual Machine (VM) yang memerlukan OS untuk setiap VM nya, karena dalam container hanya berisikan source code dan dependensinya saja, jadi memungkinkan menginstal apa yang di perlukan saja.

## Docker

Docker adalah salah satu platform software yang digunakan untuk membuat, mengelola aplikasi yang nantinya dikemas dalam sebuah wadah yang terisolasi yaitu container. Docker nantinya akan mengemas aplikasi berserta dependensi yang diperlukan dalam satu paket yang ringan. Sehingga dapat dijalankan secara konsisten tanpa mengubah konfigurasi.

## Kubernetes (K8s)

Kubernetes merupakan platform container orchestration yang berbasis open source, yang digunakan untuk management workload suatu aplikasi yang dikontainerisasi. Kubernetes juga menyediakan konfigurasi dan automation untuk mengelola aplikasi berbasis container, serta dapat mengelola Workload / beban kerja dari sebuah container apps secara Efisien dengan menggunakan fitur Horizontal Pod auto Scaling (HPA).

## Harbor

Harbor adalah registry open source yang digunakan untuk menyimpan dan mengelola Images yang akan digunakan untuk membuat Container. Harbor sendiri cukup diminati karena mudahnya integrasi dengan tools scanning Vulnerability seperti Trivy atau Clair, yang memungkinkan untuk melakukan pengecekan kerentanan (Vulnerability) terhadap Images yang disimpan menjadi lebih mudah.

## Trivy

Trivy adalah salah satu tools scanning yang berbasis open source yang digunakan untuk mendeteksi kerentanan yang berfokus pada images container. Trivy memindai mulai dari Library atau Apps Dependencies (seperti Composer, npm, yarn) sampai kedalam Operating System (OS) yang digunakan, apakah aman atau tidak.

## SSL (Secure Sockets Layer)

SSL merupakan Protocol keamanan yang digunakan untuk mengenkrip si data seperti informasi pribadi, password, rekening, dan data lain yang bersifat sensitif, saat data dikirim kan ke server, data tersebut akan di enkripsi untuk menjaga keamanan dari data tersebut. SSL sertifikat yaitu sertifikat digital digunakan untuk autentikasi indentitas dari situs web yang memungkinkan koneksi enkripsi yang aman. Dan sering digunakan untuk menjaga keamanan data pengguna yang perlu memverifikasi kepemilikan situs website.

## Slack
Slack adalah tools komunikasi yang digunakan di tempat kerja, yang memungkinkan pengguna untuk mengirim pesan, file dan tools. Slack memiliki 2 metode, yaitu Direct Message (pesan langsung dari satu user ke user lain) dan Channel Group. Dan juga Slack dapat di integrasikan dengan aplikasi atau tools lain juga yang memungkinkan melakukan otomatisasi untuk notifikasi dari suatu aplikasi.

# Langkah Implementasi
# 1. Konfigurasi SSH ke semua Node
  * Ubah Hostname pada Node agar memudahkan dalam mengidentifikasi Node yang ada.
    ```
    node-master   :~$ sudo hostnamectl set-hostname pod-master
    node-worker01 :~$ sudo hostnamectl set-hostname pod-worker01
    node-worker02 :~$ sudo hostnamectl set-hostname pod-worker02
    node-harbor   :~$ sudo hostnamectl set-hostname pod-harbor
    ```
  * Menambahkan IP dari Node dan Hostname semua Node ke “/etc/hosts” agar dapat di akses dengan menggunakan Hostname yang sudah di konfigurasi.
    ```
    ~$ sudo nano /etc/hosts

    10.18.18.10 pod-master
    10.18.18.20 pod-worker01
    10.18.18.30 pod-worker02
    10.18.18.40 pod-harbor
    ```
  * Buat Public dan Private key, lalu copy public key dari setiap Node ke dalam Node yang lain. Agar dapat melakukan SSH tanpa password.
    ```
    ~$ ssh-keygen

    ~$ ssh-copy-id user@pod-master
    ~$ ssh-copy-id user@pod-worker01
    ~$ ssh-copy-id user@pod-worker02
    ~$ ssh-copy-id user@pod-harbor
    ```
# 2. Install Docker pada Node-Harbor Untuk Harbor Registry
  * Menambahkan repository dari Docker.
    ```
    node-harbor:~$ sudo apt-get update
    node-harbor:~$ sudo apt-get install ca-certificates curl -y
    node-harbor:~$ sudo install -m 0755 -d /etc/apt/keyrings
    node-harbor:~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    node-harbor:~$ sudo chmod a+r /etc/apt/keyrings/docker.asc
    node-harbor:~$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
  * Install Docker.
    ```
    node-harbor:~$ sudo apt-get update
    node-harbor:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    ```
  * Mengatur agar user “student” atau user biasa (Bukan Root) dapat menggunakan perintah docker
    ```
    node-harbor:~$ sudo usermod -aG docker $USER
    node-harbor:~$ sudo chmod 666 /var/run/docker.sock
    node-harbor:~$ docker version
    ```
# 3. Konfigurasi SSH ke semua Node
# 4. Konfigurasi SSH ke semua Node
# 5. Konfigurasi SSH ke semua Node
# 6. Konfigurasi SSH ke semua Node
# 7. Konfigurasi SSH ke semua Node
# 8. Konfigurasi SSH ke semua Node
# 9. Konfigurasi SSH ke semua Node
# 10. Konfigurasi SSH ke semua Node


## Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```

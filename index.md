# Task
Perusahaan memiliki cluster kubernetes sebagai teknologi untuk menjalankan aplikasi yang dimiliki. Manajer tim operasional ingin membuat private registry menggunakan harbor untuk menyimpan image yang dimiliki agar tidak bisa diakses publik dan menggunakan tools trivy untuk scanning vulnerability image yang disimpan di private registry.

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
![Branching](./assets/images/topologi.png)

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
## 1. Konfigurasi SSH.
  > **_Note: jalankan pada Semua Node_**

  * Ubah Hostname pada Node agar memudahkan dalam mengidentifikasi Node yang ada.
    ```
    node-master   :~$ sudo hostnamectl set-hostname pod-master
    node-worker01 :~$ sudo hostnamectl set-hostname pod-worker01
    node-worker02 :~$ sudo hostnamectl set-hostname pod-worker02
    node-harbor   :~$ sudo hostnamectl set-hostname pod-harbor
    ```
  * Menambahkan IP dari Node dan Hostname semua Node ke **_“/etc/hosts”_** agar dapat di akses dengan menggunakan Hostname yang sudah di konfigurasi.
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
## 2. Install Docker Untuk Harbor Registry
  > **_Note: jalankan pada Node Harbor_**

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
  * Mengatur agar user **_“student”_** atau user biasa (Bukan Root) dapat menggunakan perintah docker
    ```
    node-harbor:~$ sudo usermod -aG docker $USER
    node-harbor:~$ sudo chmod 666 /var/run/docker.sock
    node-harbor:~$ docker version
    ```
## 3. Create SSL Certificate untuk Harbor
  > **_Note: jalankan pada Node Harbor_**
  * Membuat file Config IP SAN, agar SSL Certificate dapat membaca akses jika melalui IP, dan di simpan di directory **_+“/etc/ssl/harbor/”_**.
    ```
    ~$ echo "subjectAltName=IP:<IP Address Node Harbor>" > harbor.txt
    ```
  * Membuat Certificate dan Key untuk koneksi https pada Harbor, yang nantinya di simpan di directory **_“/etc/ssl/harbor/”_**.
    ```
    ~$ sudo openssl genrsa -out harbor.key 4096

    ~$ sudo openssl req -sha512 -new \
         -subj "/C=IN/ST=jateng/L=kendal/O=<Company Name>/OU=Personal/CN=<Domain>" \
         -key harbor.key \
         -out harbor.csr

    ~$ sudo openssl x509 -req -sha512 -days 3650 \
         -extfile harbor.ext \
         -in harbor.csr \
         -out harbor.crt

    ```
## 4. Install Harbor
  > **_Note: jalankan pada Node Harbor_**
  * Download Source Code untuk harbor, lalu unzip file source code harbor yang sudah di download.
    ```
    ~$ wget https://github.com/goharbor/harbor/releases/download/v2.10.0/harbor-offline-installer-v2.10.0.tgz
    ~$ tar -xvzf harbor-offline-installer-v2.10.0.tgz
    ```
  * Lalu edit pada file ”harbor.yaml”. sesuaikan seperti di contoh berikut.
    ```
    ~$ cp harbor.yml.tmpl harbor.yml
    ~$ nano harbor.yml
    ```
    ```
    hostname: <IP or Domain> #10.18.18.40
    http:
        port: 80
    https:
        port: 443
        certificate: /path/to/ca_harbor.crt
        private_key: /path/to/ca_habror.key

    harbor_admin_password: <password admin harbor>
    ```
  * Lalu jalankan script **_“install.sh”_** dengan parameter **_“--with-trivy”_** untuk secara otomatis integrasi Harbor dengan Trivy. Lalu verifikasi container komponen dari harbor nya berjalan dengan baik tanpa error.
    ```
    ~$ sudo ~/harbor/install.sh --with-trivy
    ~$ docker ps -a
    ```
## 5. Konfigurasi Harbor untuk Trigger Scan Trivy saat ada Image yang baru di Push dan mengatur agar user dapat Pull Image dengan tingkat kerentanan yang rendah atau aman.
  * Login ke Harbor dengan user **_“admin”_**, lalu masuk ke project yang ingin di konfigurasi.
    ![Branching](./assets/images/9.19.png)
    
  * Lalu pilih **_“Configuration”_**, kemudian centang pada bagian :
      - **_“Prevent vulnerable images from running.”_** Untuk mengatur agar user tidak dapat Pull Images dengan kerentanan yang tinggi. Contohnya di level **_“Critical”_** (yang berbahaya sekali) atau yang Levelnya diatasnya lagi.
      - b)	**_“Vulnerability Scanning”_**. Untuk mengatur agar saat ada Push images akan secara otomatis di scanning.
    ![Branching](./assets/images/harbor_config.png)
    
  
## 6. Install Kubernetes Cluster
  > **_Note: jalankan pada Node Master dan Worker_**
  * Update dan Upgrade packages.
    ```
    ~$ sudo apt update && \
        sudo apt upgrade -y && \
        sudo apt autoremove -y
    ```
  * Install dependensi dan menambah repository untuk install containerd.
    ```
    ~$ sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

    ~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
    ~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```
  * Install Containerd lalu Config Containerd.
    ```
    ~$ sudo apt update
    ~$ sudo apt install -y containerd.io

    ~$ containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
    ~$ sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

    ~$ sudo systemctl restart containerd
    ~$ sudo systemctl enable containerd
    ```
  * Add kernel setting overlay.
    ```
    ~$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
        overlay
        br_netfilter
        EOF

    ~$ sudo modprobe overlay
    ~$ sudo modprobe br_netfilter
    ```
  * Konfigurasi iptables.
    ```
    ~$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-iptables  = 1
        net.bridge.bridge-nf-call-ip6tables = 1
        net.ipv4.ip_forward = 1
        EOF

    ~$ sudo sysctl --system
    ```
  * Menambahkan repository untuk kubectl, kubelet, dan kubeadm dan Install tools tersebut.
    ```
    ~$ sudo apt-get install -y apt-transport-https ca-certificates
    ~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    ~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    ~$ sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
    ~$ sudo apt-mark hold kubelet kubeadm kubectl
    ```
  * Initialze untuk membuat Cluster Kubernetes di Master Node.
  > **_Note: jalankan pada Node Master_**
    ```
    ~$ sudo kubeadm init --pod-network-cidr=10.244.XX.0/16
    ~$ mkdir -p $HOME/.kube
    ~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    ~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
  * Add Flannel.
    ```
    ~$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    ~$ kubectl apply -f kube-flannel.yml
    ~$ kubectl get pods --all-namespaces
    ```
  * Menampilkan Token dan CA Certificate.
    ```
    ~$ sudo kubeadm token list
    ~$ sudo openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
    ```
  * Initialze Node Worker
  > **_Note: jalankan pada Node Worker_**
    ```
    ~$ sudo kubeadm join --token [TOKEN] [NODE-MASTER]:6443 --discovery-token-ca-cert-hash sha256:[TOKEN-CA-CERT-HASH]
    ```
## 7. Configuration SSL Certificate ke Cluster Kubernetes
  > **_Note: jalankan pada Node Master dan Worker_**
  * Mengatur letak SSL Certificate untuk containerd pada semua Node yang ada di Cluster Kubenetes.
    ```
    ~$ sudo mkdir -p /etc/containerd/certs.d/
    ~$ sudo mkdir -p /etc/containerd/certs.d/<IP or Domain_registry>/

    # Example
    ~$ sudo mkdir -p /etc/containerd/certs.d/10.18.18.40:8443/    
    ```
  * Menambahkan / Copy SSL Certificate yang ada di Node Harbor atau yang digunakan oleh Harbor ke semua Node yang ada di Cluster Kubernetes.
    ```
    ~$ sudo nano /etc/containerd/certs.d/10.18.18.40:8443/са.crt
        -----BEGIN CERTIFICATE-----
        MIID6jCCAtKgAwIBAgIUJ@ipQt1@mbC+oFh7HqornSJ2UxAwDQYJKoZIhvcNAQEL
        ...
        eE6/aLPRXcF/72YD3eoER35h/@tnlPuuZTK7iCfYPOFTEsfa@cXGzRtXb2vV4A=
        -----END CERTIFICATE-----
    ~$ sudo nano /etc/containerd/certs.d/10.18.18.40:8443/ca.key
        -----BEGIN PRIVATE KEY-----
        MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDHj+SCxIwcgBlM
        ...
        Vkauk44NJ+0iyBPIzizD6qmY
        -----END PRIVATE KEY-----
    ```
## 8. Konfigurasi kubernetes untuk Pull images ke Harbor
  > **_Note: jalankan pada Node Master_**
  * Atur untuk Credentials Harbor registry dengan secret. Yang nantinya akan digunakan saat membuat Pod atau saat Pull Images ke Registry tertentu.
    ```
    ~$ kubectl create secret docker-registry <name Secret> \
        --docker-server=<IP or Domain Registry> \
        --docker-username=<User> \
        --docker-password=<Password user> \
        --docker-email=<email for user>
    ```
## 9. Konfigurasi Notifikasi Otomatis dari Hasil Scan Harbor ke Slack
  * Login ke Website [Slack](https://slack.com/), Buat akun baru jika belum ada.
    ![Branching](./assets/images/9.1.png)
    ![Branching](./assets/images/9.2.png)
    
  * Lalu isi nama untuk Company atau Team yang sesuai, contoh **_“Harbor Vuln Scan”_**.
    ![Branching](./assets/images/9.3.png)

  * Nama untuk user sendiri, contoh **_“Najwan”_**.
    ![Branching](./assets/images/9.4.png)

  * Dibagian ini pilih yang **_“skip this step”_** jika tidak ingin invite user lain ke Slack.
    ![Branching](./assets/images/9.5.png)

  * Lalu isi nama untuk Channel nya, contoh **_“Harbor Vuln”_**.
    ![Branching](./assets/images/9.6.png)

  * Lalu pilih yang **_“Start with the limit free version”_** kalau ingin menggunakan yang gratis.
    ![Branching](./assets/images/9.7.png)

  * Lalu klik pada titik tiga di kanan atas, lalu klik yang **_“Edit Settings”_**.
    ![Branching](./assets/images/9.8.png)

  * Lalu pilih yang **_“Integrations”_**, lalu klik **_“Add an App”_**.
    ![Branching](./assets/images/9.9.png)

  * Lalu klik **_“Manage Apps…”_** di bagian kiri atas.
    ![Branching](./assets/images/9.10.png)

  * Lalu pilih **_“Build”_** di kanan atas.
    ![Branching](./assets/images/9.11.png)

  * Lalu klik **_“Create an App”_**.
    ![Branching](./assets/images/9.12.png)

  * Lalu pilih yang **_“From scratch”_** untuk lebih mudah, karena tidak perlu membuat template. Kalau **_“From a manifest”_** kita perlu membuat template dengan format **_“json”_** atau **_“YAML”_**.
    ![Branching](./assets/images/9.13.png)

  * Lalu isi untuk nama App nya dan pilih Workspace yang sesuai. Lalu klik **_“Create App”_**.
    ![Branching](./assets/images/9.14.png)

  * Lalu setalah selesai membuat app, kita perlu membuat untuk endpoint webhook agar Harbor dapat mengirim ke Slack, dengan pilih **_“Incoming WebHooks”_**, lalu aktifkan **_“Activate Incoming WebHooks”_**.
    ![Branching](./assets/images/9.15.png)

  * Lalu scroll kebawah, lalu klik **_“Add New WebHook to Workspace”_**.
    ![Branching](./assets/images/9.16.png)

  * Lalu pilih tempat untuk mengirim pesan dari WebHook nya, bisa ke Channel dari Slack, atau Direct Message ke user tertentu. Contoh misalkan ke channel **_“harbor-vuln”_**.
    ![Branching](./assets/images/9.17.png)

  * Lalu copy link yang sudah di berikan.
    ![Branching](./assets/images/9.18.png)

  * Lalu login ke Harbor dengan user **_“admin”_** dengan password yang sesuai. Pilih project yang ingin di konfigurasi. Contoh **_“testing”_**.
    ![Branching](./assets/images/9.19.png)

  * Lalu pilih **_“Webhooks”_**, lalu klik **_“+ New WebHook”_**.
    ![Branching](./assets/images/9.20.png)

  * Lalu isikan data berikut :
    - Nama WebHook.
    - Deskripsi apabila ada.
    - Pada bagian **_“Notify Type”_** pilih yang **_“Slack”_**.
    - Pada bagian **_“Event Type”_** untuk memilih saat apa Harbor mengirim notif ke Slack (Trigger), centang pada bagian:
      -	**_Scanning failed_**
      -	**_Scanning stopped_**
      - **_Scanning finished_**
    - Isikan Endpoint URL sesuai dari slack pada step sebelumnya.
    ![Branching](./assets/images/9.21.png)
    
## 10. Konfigurasi SSH ke semua Node
  * Login ke Slack, lalu pergi ke pengaturan App, atau lewat [link ini] (https://api.slack.com/apps/). Lalu pilih App yang sesuai, contohnya **_“python-harbor”_**.
    ![Branching](./assets/images/10.1.png)

  * lalu pilih yang **_“OAuth & Permissions”_**.
    ![Branching](./assets/images/10.2.png)

  * Lalu Scroll kebawah sampai ke bagian **_“Scopes”_**, lalu pilih **_“Add an QAuth Scope”_**, lalu pilih yang **_“chat:write”_** untuk mengatur role agar dapat menulis pesan atau chat.
    ![Branching](./assets/images/10.3.png)

  * Lalu Scroll ke atas sampai di bagian **_"OAuth Tokens”_**, lalu **_“Install to Harbor Vuln Scan”_**.
    ![Branching](./assets/images/10.4.png)
    
  * Lalu pada bagian ini klik **_“Allow”_**.
    ![Branching](./assets/images/10.5.png)

  * Lalu Copy **_“Bot User OAuth Token”_**.
    ![Branching](./assets/images/10.6.png)

  * Ambil Sample aplikasi sederhana untuk filter data hasil Scanning Trivy dari Harbor dan nanti akan dikirim ke Slack di [github ini](https://github.com/vianAja/python-slack-harbor.git). Lalu sesuaikan untuk data berikut ini.
    ```
    token_slack = "TOKEN_OAUTH"
    channel_id = "CHANNEL_ID_SLACK"
    name_bot = "NAME_BOT"
    ```
  * Lalu jalankan perintah ini :
      a)	Install Library yang dibutuhkan Python. 
      b)	Copy file service ke **_“/etc/system/system/”_**.
      c)	Copy file **_“main.py”_** ke **_“/usr/local/bin/”_**, menyesuaikan dari konfigurasi yang ada di file service nya.
    ```
    ~$ sudo pip3 install -r requirement.txt
    ~$ sudo cp python-slack.service /etc/systemd/system/
    ~$ sudo cp main.py /usr/local/bin/
    ```
  * Reload system daemon untuk update service. Lalu start dan lihat status service yang sudah di Copy tadi.
    ```
    ~$ sudo systemctl daemon-reload
    
    ~$ sudo systemctl start python-slack.service
    ~$ sudo systemctl status python-slack.service
    ```
# Hasil
## Tanpa menggunakan Program Pyhon.
![Branching](./assets/images/hasil_slack.1.png)
## Menggunakan Program Python.
![Branching](./assets/images/hasil_slack.2.png)
## Tidak Bisa Pull Images jika level Vulnerability Critical atau yang lebih tingggi.
![Branching](./assets/images/failed_pull.png)

# Referensi
[KubeAdm Tools](https://kubernetes.io/id/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
[Install Harbor](https://medium.com/@tanmaybhandge/how-to-deploy-the-harbor-on-vm-using-self-signed-certificate-ebfe29c4803a)
[Trivy Scanner](https://www.jit.io/resources/appsec-tools/when-and-how-to-use-trivy-to-scan-containers-for-vulnerabilities)
[Trivy Scanner](https://medium.com/@maheshwar.ramkrushna/scanning-docker-images-for-vulnerabilities-using-trivy-for-effective-security-analysis-fa3e2844db22)
[WebHooks Slack](https://api.slack.com/messaging/webhooks)
[Kubernetes Pull Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
[Container](https://www.docker.com/resources/what-container/)
[Docker](https://www.hostinger.co.id/tutorial/apa-itu-docker)
[harbor dokumentasi](https://goharbor.io/)
[Slack](https://www.nusa.id/apa-itu-slack-yuk-cari-tahu-lebih-jauh-di-sini#:~:text=Apa%20Itu%20Slack%3F,in%20untuk%20tools%20kerja%20lainnya.)
['SSL (Secure Sockets Layer)'](https://www.jagoanhosting.com/blog/pengertian-ssl)
[Apps Manifest Slack](https://api.slack.com/reference/manifests)
[Integrasi Python ke Slack](https://www.datacamp.com/tutorial/how-to-send-slack-messages-with-python)
[Send Messages Python ke Slack](https://medium.com/@sid2631/automating-slack-notifications-sending-messages-as-a-bot-with-python-2beb6c16cd8)

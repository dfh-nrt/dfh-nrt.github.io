## Terraform X Proxmox : Simple Main TF dengan Struktur Directory

#### Terraform

Terraform merupakan aplikasi Infrastructure as Code (IaC) yang dibuat oleh Hashicorp dengan terraform memungkinkan untuk mengelola infrastructure (mengkonfigurasi, memodifikasi server, jaringan dan storage) melalui code. 

Terraform bersifat deklaratif, artinya cukup dengan menyatakan kondisi / state infrastructure yang diinginkan dan terraform yang akan mengurus proses penerapannya.

Terraform digunakan dengan sebelumnya melakukan init terhadap folder kerja terraform menggunakan command 

```
	terraform init : menyiapkan direktori / folder kerja terraform
```

Selanjutnya setelah menambahkan file – file tf yang diinginkan gunakan perintah

```
	terraform plan : menampilkan rencana perubahan yang akan diterapkan pada infrastruktur
```

Setelah itu, perintah terakhir adalah 

```
	terraform apply : menerapkan perubahan yang telah disetujui / mendapat OK / cocok
```

Saat mengeksekusi terraform plan akan muncul tampilan perubahan apa saja yang akan terjadi pada infra. Pastikan dibaca dan dimengerti sebelum melakukan apply.

### Core konsep dalam Terraform

`Variabel` : mirip seperti variabel pada bahasa pemograman, digunakan untuk menampung nilai tertentu. Pada terraform ada variable input dan output, variable input digunakan untuk parameter runtime sedangkan variable output untuk digunakan di konfigurasi lain ataupun ditampilkan sebagai output.

`Provider` : merupakan plugin yang digunakan untuk berinteraksi dengan penyedia layanan ataupun hypervisor seperti AWS, GCP, proxmox, virtualbox, dll. Provider ini yang memungkinkan user untuk melakukan create , delete resources dll.

`Modul` : merupakan set konfigurasi / code dalam sebuah folder. Set konfigurasi utama disebut sebagai root dan directory – directory / folder – folder di dalamnya disebut modul. Mirip seperti function dalam pemograman.

`State` : merupakan informasi tentang infrastruktur yang dimanage oleh terraform berupa file dan jika tidak ada file state maka terraform tidak akan dapat melakukan pekerjaannya.

`Resources` : dalam terraform, resources bisa berarti layanan seperti EC2, database, storage, VM dsb.

`Data Source` : data source memungkinkan untuk mengambil data dari resources yang tidak dikelola langsung oleh terraform. Hanya read saja.

`Plan` : tahap dimana terraform membaca apa saja yang akan dibuat, diubah atau dihapus untuk mencapai kondisi yang diinginkan sesuai code.

`Apply` : tahap dimana terraform menerapkan perubahan yang disetujui setelah membaca plan yang diberikan.

Berikut merupakan contoh struktur directory dari terraform.

#### Struktur Directory :

```
terraform-proxmox(root)/

main.tf
providers.tf
versions.tf
variables.tf
cred.auto.tfvars
outputs.tf

/module/
  |----	vm/
	      main.tf
	      versions.tf
	      variables.tf
	      outputs.tf
```

Pada struktur directory di atas terdapat root directory, disini dengan nama terraform-proxmox yang di dalamnya terdapat file – file .tf (main, providers, versions, variables, outputs) , file credentials dan module direktori.

Sebelumnya sudah diberitahukan kalau module merupakan salah satu core konsep dari terraform yang pada prinsipnya mirip dengan function di pemograman. Pada struktur direktori terraform di atas module berisikan direktori vm yang akan difungsikan sejenis template untuk menyederhakan parameter pembuatan VM di main.tf di root direktori. Berikut merupakan sedikit isi main.tf di vm direktori dalam module :

```
# module main.tf create vm
resource "proxmox_vm_qemu" "servers" {
    name = var.vm_name
    vmid = var.vm_id
    memory = var.vm_memory
    cpu {
        cores = var.vm_core
    }

    target_node = "proxmox"
    agent = 1
    clone = "ubuntu-template"
    scsihw = "virtio-scsi-pci"
    bootdisk = "scsi0"
    ......
```
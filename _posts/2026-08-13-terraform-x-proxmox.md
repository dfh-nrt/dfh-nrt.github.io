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
Gambar 1
{:style="text-align: center;"}

Pada gambar 1 diperlihatkan bahwa parameter atau konfigurasi yang akan disesuaikan setiap pembuatan VM baru adalah parameter yang valuenya diisi dengan nama variable yang mana nilainya akan didapatkan dari root main.tf nantinya, sedangkan untuk parameter yang valuenya akan hampir selalu tetap seperti agent dll, untuk memudahkan penulisan dan tidak selalu mengulang menulis code yang sama dibuat menjadi module. 

Module memudahkan untuk menggunakan ulang / reuse code yang sering digunakan. Pada infrastruktur dengan kompleksitas yang tinggi dan skala besar, sangat dibutuhkan penggunaan module atau pengelompokkan komponen – komponen agar lebih mudah untuk digunakan lagi.

Selanjutnya, dari mana nama variable yang digunakan pada main.tf pada gambar 1 berasal atau didefine. Jawabannya adalah dari variables.tf, berikut contoh isi dari variables.tf :

```
# module variables.tf
variable "vm_name"{
    type = string
}

variable "vm_id"{
    type = number
}

variable "vm_memory"{
    type = number
}
.....
```
Gambar 2
{:style="text-align: center;"}

Pada gambar 2 ditunjukkan define variable dilakukan dengan memberi nama variable pada bagian setelah variable dan type variable yang sesuai dengan ketentuan parameter yang akan diisi dengan variable tersebut. Contohnya jika ingin mengisi memory yang parameternya secara default diisi angka maka akan memunculkan error unexpected jika diisi oleh string dsb.

Sekarang ada permasalah jika menggunakan module yaitu root akan menganggap modul sebagai black box, yang artinya root direktori tidak dapat secara langsung mengambil informasi dari VM yang parameternya berada di module. Disinilah digunakan outputs.tf untuk mengoper informasi yang dibutuhkan tersebut, berikut isi file outputs.tf :

```
# output tf module
output "vm_id" {
    value = proxmox_vm_qemu.servers.vmid
}
output "vm_ip" {
    value = proxmox_vm_qemu.servers.default_ipv4_address
}
output "vm_hostname" {
    value = proxmox_vm_qemu.servers.name
....

# output tf root
output "vm_id" {
    value = module.vm.vm_id
}

output "ip_address" {
    value = module.vm.vm_ip
}

output "hostname" {
    value = module.vm.vm_hostname
....
```
Gambar 3
{:style="text-align: center;"}

Pada gambar 3 terlihat di outputs.tf module value yang digunakan adalah nama resources yang dijalankan di main.tf module direktori sedangkan di outputs.tf root yang digunakan adalah value yang dioper dari outputs.tf module dengan format `module.<nama_module>.<variable_output>`.

Pada struktur directory di atas, selain dari 3 file yang telah dibahas (main.tf, variables.tf, dan outputs.tf) ada juga versions.tf yang digunakan untuk define providers dan versinya yang digunakan. Version ini akan terasa penting jika digunakan di real production dari pada di simulasi atau devel karena akan membatasi agar tidak melakukan update / upgrade sehingga mengurangi resiko bug / error tiba – tiba.

Selanjutnya, jika diperhatikan pada struktur direktori antara root direktori dengan modul direktori file – file tf memiliki nama yang sama satu sama lain. Hal tersebut karena memang fungsinya sama hanya scopenya yang berbeda dan agar memudahkan. Maka dari itu, perbedaan yang terlihat dari struktur direktori di atas adalah pada file providers.tf dan cred.auto.tfvars.

File providers.tf untuk struktur direktori di atas yang menggunakan proxmox berisikan informasi untuk berkomunikasi dengan proxmox di sini menggunakan token dan secret yang dikonfigurasi atau didapat dari proxmox sedangkan cred.auto.tfvars merupakan file yang digunakan untuk menyimpan credential yang digunakan di providers.tf. Berikut contoh providers.tf.

```
provider "proxmox" {
  pm_api_url = var.proxmox_url
  # Pro Tip : Never store your creds on Terraform Configuration files 
  # that can or may be commited to VCS 
  # Pro Tip : Use either Env Variables or git-ignored *.auto.tfvars files
  pm_api_token_id     = var.proxmox_token_id
  pm_api_token_secret = var.proxmox_token_secret
  # Disable TLS (not recommanded for Production envirement)
  pm_tls_insecure = true
  # Define number of parrallel tasks that proxmox can handle.
  pm_parallel     = 10
  pm_minimum_permission_check = false
}
```
Gambar 4
{:style="text-align: center;"}

Pada gambar 4 ditunjukkan parameter yang dikonfigurasi untuk dapat berkomunikasi dengan proxmox dan variable yang didefine di variables.tf root dengan value dari cred.auto.tfvars file. Penggunaan file cred.auto.tfvars penting untuk menghindari file dengan harcode credential agar tidak terekspos ke public dan agar mudah meng-ignore file tersebut untuk version controlling.


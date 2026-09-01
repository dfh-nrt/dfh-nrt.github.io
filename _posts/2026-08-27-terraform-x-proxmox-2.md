---
title: "Terraform X Proxmox : VM GW, Devel and Production"
date: 2026-08-27
tags:
  - Terraform
  - Proxmox
---

## Terraform X Proxmox : VM GW, Devel and Production

#### Terraform

### for_each

Merupakan cara untuk melakukan perulangan terhadap kelompok data (bertipe set atau map) untuk membuat resources, data source, dll dalam terraform.

Berbeda dengan count yang menggunakan penomoran, for_each menggunakan metode key dan value memberikan fleksibilitas lebih.

### Dynamic Block

Merupakan salah satu cara untuk merubah data bertipe list ke map. Selain menggunakan dynamic block untuk merubah list ke map bisa juga dengan menggunakan for atau tosset untuk merubah list ke set.

Pada dasarnya dynamic block digunakan untuk menyederhanakan perulangan nested block agar code menjadi lebih clean dan sederhana.

![Topologi I](/assets/topologi_i.png)

Gambar di atas merupakan skema yang akan saya gunakan pada percobaan kali ini. Pada gambar terlihat sebuah infrastruktur sederhana yang mungkin cukup umum untuk digunakan yaitu infrastruktur dengan sebuah VM bertindak sebagai gateway untuk dilewati oleh jaringan internal jika ingin mengakses internet. Secara konsep mungkin mirip dengan service NAT gateway di AWS maupun provider cloud lainnya. 

Pada gambar diperlihatkan bahwa VM GW harus terhubung ke tiga network yaitu 192.168.18.0/24, 10.10.10.0/24, dan  10.10.20.0/24. Pada implementasinya di sini saya menggunakan dua interface yang mana satu interface akan dikonfigurasi vlan interface sehingga bisa memiliki dua IP address.

Hal yang dikonfigurasi di VM GW seperti enable ip forwarding untuk meneruskan paket ke tujuan lain dan firewall rule untuk masquerading IP dari jaringan internal ke luar. 

`Kenapa perlu ip forwarding?`
Karena pada umumnya OS hanya akan bertindak sebagai host dan hanya akan menerima paket yang ditujukan untuk dirinya sendiri. Dengan dikonfigurasi IP forward OS dapat bertindak sebagai router dan meneruskan paket ke tujuan lain.

`Kenapa perlu masquerading IP?`
Karena paket yang datang dari internal akan memiliki source address dari jaringan internal contohnya 10.10.10.x, di sini alamat tersebut tidak akan dikenali oleh router yang hanya mengenali trafik dari source 192.168.18.x sehingga digunakan masquerade agar trafik dari 10.10.10.x terlihat sebagai trafik dari 192.168.18.x.

![Topologi I](/assets/struktur_directory_i.png)

Gambar di atas merupakan struktur direktori yang digunakan pada percobaan kali ini. Pada percobaan kali ini ditambahkan modules baru yaitu network. Hal ini dikarenakan akan digunakan untuk pembuatan internal network. Code yang digunakan seperti di bawah :

```
#/modules/network/main.tf

resource "proxmox_network_linux_bridge" "int-net" {
    node_name = "proxmox"
    name = var.net_name

    vlan_aware = true
    vids = "10 20"
}
```

Pada code di atas resource yang digunakan adalah proxmox_network_linux_bridge dari BPG dan dikonfigurasi dengan nama resources int-net. 

	node_name = merupakan nama node / hypervisor yang diakses
	name = nama interface yang akan dibuat
	vlan_aware = agar interface mengenali vlan
	vids = vlan id yang diizinkan / dikenali

Adapun main.tf di environments net yang digunakan untuk memanggil modules tersebut adalah seperti code di bawah ini :

```
#/environments/net/main.tf

module "int-net" {
  source = "../../modules/network"
  net_name = "internal"
}
```

Karena pada modul int-net untuk parameter lain sudah di set nilainya sedangkan hanya parameter name yang belum untuk disesuaikan sesuai kebutuhan. Oleh karena itu, pada pemanggilan module tetap diperlukan mengisi parameter name.

Dengan adanya modul int-net, kebutuhan interface bisa disesuaikan secara code dari terraform. Namun, perlu menjadi perhatian kalau interface yang dibuat akan tersedia juga untuk vm – vm lainnya di dalam proxmox artinya tidak perlu membuat interface baru setiap pembuatan vm.

Selanjutnya, karena pada vm gw ini menggunakan dua network interface sedangkan umumnya vm biasanya hanya akan menggunakan satu network interface perlu dilakukan penyesuaian di modul vm untuk mengakomodir kebutuhan tersebut. Caranya adalah dengan menggunakan dynamic block dan for_each. 

Pertama variable network akan dibuat dalam tipe list. Setelah itu, dibuat dynamic block untuk menggunakan list tersebut supaya compatible dengan for_each. Berikut contoh code yang digunakan :

```
#/modules/vm/variables.tf

variable "vm_network" {
    type = list (object({
        bridge = string
        vlan_id = optional(number)
        trunks = optional(string)
    }))
}
```

Setiap item di list harus sebuah objek dengan defined attributes adalah bridge dengan string, vlan_id dengan number tapi optional dan trunks dengan string tapi optional.

```
#/modules/vm/main.tf

    dynamic "network_device" {
        for_each = var.vm_network

        content {
            bridge = network_device.value.bridge
            vlan_id = network_device.value.vlan_id
            trunks = network_device.value.trunks
        }
    }
```

Pada dynamic block, ada content block yang berisikan original content yang akan diulangi / dilakukan perulangan. Pada code di atas nilai dari komponen bridge, vlan_id dan trunks akan diisi oleh nilai dari values dari attribute variable vm_network yang pada pengaplikasiannya akan seperti code berikut.

```
#/environments/net/

 vm_network = [
    {
      bridge = "vmbr0"
    },
    {
      bridge = "internal"
      trunks = "10;20"
    }
  ]
```

Code di atas digunakan untuk VM GW yang menggunakan dua interface dengan interface internal yang mengarah ke jaringan lokal dan hanya melewatkan vlan 10 dan 20.
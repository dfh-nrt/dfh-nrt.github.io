## Terraform X Proxmox : VM GW, Devel and Production

#### Terraform

![Topologi I](/assets/topologi_i.png)

### for_each

Merupakan cara untuk melakukan perulangan terhadap kelompok data (bertipe set atau map) untuk membuat resources, data source, dll dalam terraform.

Berbeda dengan count yang menggunakan penomoran, for_each menggunakan metode key dan value memberikan fleksibilitas lebih.

### Dynamic Block

Merupakan salah cara untuk merubah data bertipe list ke map. Selain menggunakan dynamic block untuk merubah list ke map bisa juga dengan menggunakan for atau tosset untuk merubah list ke set.
Pada dasarnya dynamic block digunakan untuk menyederhanakan perulangan nested block agar code menjadi lebih clean dan sederhana.
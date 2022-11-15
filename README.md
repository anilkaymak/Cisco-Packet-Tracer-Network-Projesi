# Cisco Packet Tracer Network Projesi

## Cisco Packet Tracer

Packet Tracer, Cisco Systems tarafından tasarlanmış, kullanıcıların ağ topolojileri oluşturmasına ve modern bilgisayar ağlarını taklit etmesine olanak tanıyan, platformlar arası bir görsel simülasyon aracıdır.

Cisco tarafından tasarlanan Packet Tracer programı ile, 3 lokasyonlu hem IPV4 hem de IPV6 omurgalı dual stack çalışabilen birbirleri arasında haberleşebildiği, hem IPV4 hem de IPV6 omurganın Access Control List ile sadece hizmet portlarından erişilip diğer tüm portların kapalı olduğu bir network tapolojisi tasarlanmıştır.

![image](https://user-images.githubusercontent.com/115885531/201934233-4bd0177a-fdab-46fb-b9ea-7230c11ca057.png)

### IPV4 VE IPV6 KONFİGÜRASYONU
Ağa bağlı kaynak ya da hedef cihazlar olarak tanımlanan end device'lar olan Laptop 0, PC 0 ve serverların "*Desktop*" arayüzünde bulunan "*IP Configuration*" sekmesine IPV4 ve IPV6 adresleri tanımlandı ve DNS ayarları yapıldı.


![IPV6 KONF 1](https://user-images.githubusercontent.com/115885531/201941025-bbf0ad6b-d0ed-4341-b495-81a066092a28.png)
![DNS ayarı İPV6 ](https://user-images.githubusercontent.com/115885531/201941167-f014daf0-cffd-4941-a798-e16999b9d376.png)

Routerların sırasıyla cli sekmesine girilerek fast ethernet ve serial arayüzlerine girilerek gerekli IPV4 ve IPV6 adresleri aşağıdaki kodlar sayesinde girildi. Gerekli düzenlemeler yapıldıktan sonra her networkun kendi içinde haberleştiği ama farklı lokasyondaki networklerin haberleşmediği ping testi yapılarak görüldü.

Router 0 Konfigürasyonu:
```
enable (privileged mod açılır)
configure terminal (konfigürasyon modu kontrol edilir)
interface s0/0/0
clock rate 2000000(donanım bağlantılarının saat hızı 2000000 bps)
ip add 11.0.0.1 255.255.255.252 (ipv4 adresi ve subnet maskı)
no sh (arayüzü açmak)
ipv6 add 1ef0:abc:bc:c::1/126 (ipv6 adresi ve prefixi)
interface s0/0/1
clock rate 2000000
ip add 13.0.0.1 255.255.255.252
no sh 
ipv6 add 1ef0:def:ef:f::1/126
int fa0/0
ip add 192.168.1.1 255.255.255.0
no sh
ipv6 add 1ef0:111:11:1::1/64
```

![IPV4 KONF 9](https://user-images.githubusercontent.com/115885531/201946447-fdcfdf59-ce59-44ec-aaa0-d407d58bd3ea.png)

Router 1'den Laptop 0'a yapılan ping testi:
![image](https://user-images.githubusercontent.com/115885531/201949158-576ce64c-9a70-46e5-91b2-67febd648d47.png)


### IPV4 ve IPV6 Dynamic Routing OSPF
OSPF, bir TCP/IP ağındaki router'ların birbirini otomatik olarak tanımasında kullanılan bir protokoldür. Bu protokol sayesinde farklı lokasyonlardaki networklerin biribirleriyle haberleşmesi sağlanır.

Router 0 IPV4 OSPF konfigürasyonu:
```
en
conf t
router ospf 1 ("1" sayısı bir işlem kimliğidir ve istediğimiz herhangi bir sayıyı seçebiliriz.)
network 192.168.1.0 0.0.0.255 area 0 (network çağrımı whitecard mask ile yapılır. Tek alanlı OSPF'yi yapılandırdık. Tüm yönlendiriciler 0 alanına aittir.)
network 11.0.0.0 0.0.0.3 area 0
network 13.0.0.0 0.0.0.3 area 0
```
Router 0 IPV6 OSPF konfigürasyonu:
```
en
conf t
ipv6 unicast-routing (Bir IPv6 yönlendirme tablosu oluşturur, arayüzlerde IPv6 adreslerini yapılandırmanıza izin verir )
ipv6 ospf 1
router-id 1.1.1.1
int fa0/0
ipv6 ospf 1 area 0
int s0/0/0
ipv6 ospf 1 area 0
int s0/0/1
ipv6 ospf 1 area 0
```
Router 1'den PC 0'a yapılan ping testi:
![image](https://user-images.githubusercontent.com/115885531/201952808-552a98b6-4d92-4b72-a54e-44c940d6b5b1.png)

IPV4 ve IPV6 Route Table:

![OSPF 4](https://user-images.githubusercontent.com/115885531/201953282-5a2af0bf-b0c0-4324-b49b-bd9e7b102b05.png)
![IPV6 ROUTE TABLE](https://user-images.githubusercontent.com/115885531/201953318-7313a5cc-0e0e-4b31-aedd-92d946d15fbb.png)

### IPV4 ve IPV6 Access Control List
ACL ağ trafiği için kullanılan kurallar kümesidir. Görevi ise kendi oluşturduğumuz kurallar ile hangi ağ trafiğinin izin verileceğini veya reddedeceğinin kontrolünü sağlar.
Projede web server'ına sadece servis portu olan tcp 80 portundan, dns server'ına sadece servis portu olan udp 53 portundan hizmet verilmesi istenmektedir. OSPF protokolü ile haberleştirdiğimiz networkler birbilerine ping atabilirken, ACL ile internete erişim sağlanabilirken serverlarımıza herhangi bir ping atılamamaktadır.

IPV4 ACL(Router 0):
```
en
conf t
ip access-list extended anil
permit udp any host 192.168.1.2 eq 53
permit tcp any host 192.168.1.3 eq 80
int fa0/0
ip access-group anil out
```
IPV6 ACL(Router 0):
```
en
conf t
ipv6 access-list anil6
permit udp any host 1ef0:111:11:1::2 eq 53
permit tcp any host 1ef0:111:11:1::3 eq 80
int fa0/0
ipv6 traffic-filter anil6 out
```
![image](https://user-images.githubusercontent.com/115885531/201959737-51a1647b-9612-4c97-ad53-8bdbd81e0ceb.png)
![image](https://user-images.githubusercontent.com/115885531/201959911-ae40686e-d32c-4cce-acd3-9c6f027e5bd7.png)
![image](https://user-images.githubusercontent.com/115885531/201959804-dbc99d8b-6508-49b7-bc32-ff7ea5964a1f.png)
![image](https://user-images.githubusercontent.com/115885531/201959992-c17b29d4-ffde-4b93-b477-06d4705628b1.png)




### ネットワークトポロジー

                 192.168.1.0/24         
          .10                          .11
   [Linux]---------[Cloud0] --------------[R1]





### R1 (Cisco IOL) の設定

```plaintext
hostname R1
ip domain name flex-one.com
crypto key generate rsa modulus 2048

username cisco password 0 cisco

interface Ethernet0/0
 ip address 192.168.1.11 255.255.255.0
 no shut

line vty 0 4
 login local
 transport input all

end
wr
```

### Linux サーバーの設定 (/etc/network/interfaces)
```#iface ens3 inet dhcp

iface ens3 inet static
 address 192.168.1.10
 network 192.168.1.0
 netmask 255.255.255.0
 broadcast 192.168.1.255
 gateway 192.168.1.1
 dns-nameserver 8.8.8.8
```


```apt-get install -y software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install -y ansible
```


# 1. Maski:

* Każdy router potrzebuje 5 adresów, maska ``255.255.255.248`` /29
* Każde laboratorium posiada ``35 stanowisk``, więc maska ``255.255.255.192`` /26
* Wi-Fi: ``maks 800``, więc maska ``255.255.252.0`` /22

# 2. Sieci:

  Adres bazowy: ``188.156.220.160``

  
  Router Główny:
  * piętro 0: ``188.156.220.0/29``
  * piętro 1: ``188.156.221.0/29``
  * piętro 2: ``188.156.222.0/29``
  
  Router 1:
  * sala 009: ``10.0.9.0/26``
  * sala 013: ``10.0.13.0/26``
  * sala 014: ``10.0.14.0/26``
  * sala 017: ``10.0.17.0/26``
  
  
  Router 2:
  * sala 115: ``10.0.115.0/26``
  * sala 116: ``10.0.116.0/26``
  * sala 117: ``10.0.117.0/26``
  * sala 122: ``10.0.122.0/26``
  
  
  Router 3:
  * sala 201: ``10.0.201.0/26``
  * sala 202: ``10.0.202.0/26``
  * sala 203: ``10.0.203.0/26``
  * sala 204: ``10.0.204.0/26``
  
# 3. Konfiguracja komputerów

## Router główny:
```   enp0s3: (bez zmian)
    enp0s8 (piętro 0)
      address 188.156.220.1
      netmask 255.255.255.248
    enp0s9 (piętro 1)
      address 188.156.221.1
      netmask 255.255.255.248
    enp0s10 (piętro 2)
      address 188.156.222.1
      netmask 255.255.255.248
    enp0s11 (wifi)
      address 188.156.224.1
      netmask 255.255.252.0
```
      
## Router 1:

```
    enp0s3
      address 188.156.220.2
      netmask 255.255.255.248
    enp0s8
      address 10.0.9.62
      netmask 255.255.255.192
    enp0s9
      address 10.0.13.62
      netmask 255.255.255.192
    enp0s10
      address 10.0.14.62
      netmask 255.255.255.192
    enp0s11
      address 10.0.17.62
      netmask 255.255.255.192
```

## Router 2:

```
    enp0s3
      address 188.156.221.2
      netmask 255.255.255.248
    enp0s8
      address 10.0.115.62
      netmask 255.255.255.192
    enp0s9
      address 10.0.116.62
      netmask 255.255.255.192
    enp0s10
      address 10.0.117.62
      netmask 255.255.255.192
    enp0s11
      address 10.0.122.62
      netmask 255.255.255.192
```

## Router 3:

```
    enp0s3
      address 188.156.222.2
      netmask 255.255.255.248
    enp0s8
      address 10.0.201.62
      netmask 255.255.255.192
    enp0s9
      address 10.0.202.62
      netmask 255.255.255.192
    enp0s10
      address 10.0.203.62
      netmask 255.255.255.192
    enp0s11
      address 10.0.204.62
      netmask 255.255.255.192
```

# 4. DHCP, DNS i routing:

## Router główny (WiFi):
#### 1.
```
      nano /etc/default/isc-dhcp-server 
        (odkomentować ścieżkę do pliku config DHCPDv4_CONF
        dopisać interfejs INTERFACESv4="enp0s10")
```
#### 2.
```
      nano /etc/dhcp/dhcpd.conf 
      (dopisać konfigurację sieci)
        subnet 188.156.224.0 netmask 255.255.252.0 {
          range 188.156.224.2 188.156.227.254;
          option routers 188.156.224.1;
          option domain-name-servers 1.1.1.1, 1.0.0.1;
        }
      systemctl restart isc-dhcp-server
```
#### 3. Routing:

WiFi: ``up ip rotue add default via 188.156.224.1``

## Router 1, 2, 3, dla każdych sal analogicznie:
#### 1. Jak wyżej, interfejs INTERFACESv4="enp0s8, enp0s9, enp0s10, enp0s11"
#### 2. sala = [9, 13, 14, 17, 115, 116, 117, 122, 201, 202, 203, 204], sala 9-17 - router 1, sala 115-122 - router 2, sala 201-204 - router 3
```
nano /etc/dhcp/dhcpd.conf
        subnet 10.0.sala.0 netmask 255.255.255.192 {
          range 10.0.sala.1 10.0.sala.61;
          option routers 10.0.sala.62;
          option domain-name-servers 1.1.1.1, 1.0.0.1;
        }
      systemctl restart isc-dhcp-server
```
#### 3. Routing:

R1: ``up ip rotue add default via 188.156.220.1``

R2: ``up ip rotue add default via 188.156.221.1``

R3: ``up ip rotue add default via 188.156.222.1``

Oraz każdy komputer w sali: ``up ip route add default via 10.0.sala.62``

# 5. Forwardowanie (routery):

``nano /etc/sysctl.d/99-sysctl.conf``

Odkomentować ``net.ipv4.ip_forward=1``

# 6. IP Tables:

Router Główny
```
iptables -t nat -A POSTROUTING -s 188.156.220.0/29 -o enp0s3 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 188.156.221.0/29 -o enp0s3 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 188.156.222.0/29 -o enp0s3 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 188.156.224.0/22 -o enp0s3 -j MASQUERADE
  
```

Routery 1, 2, 3, jak w punkcie 4:
```
iptables -t nat -A POSTROUTING -s 10.0.sala.0/26 -o enp0s3 -j MASQUERADE
```

Zapisanie reguł:
``ipatables-save > /etc/iptables.up.rules``


/etc/network/interfaces:
``post-up iptables-restore < /etc/iptables.up.rules``

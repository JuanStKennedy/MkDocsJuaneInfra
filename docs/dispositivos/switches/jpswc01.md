# Configuración: JPSWC01 (Cisco Core)

Este switch actúa como el **núcleo (core)** de la red. Su única función es agregar los enlaces troncales (trunks) y conectar el switch de acceso (`JPSWA01`) con el router principal (`JPROO2`).

```
Current configuration : 4139 bytes
!
! Last configuration change at 19:41:33 UTC Fri Dec 19 2025
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
service compress-config
!
hostname JPSWC01
!
boot-start-marker
boot-end-marker
!
!
enable secret 5 $1$FA8d$jtbhQt1ukmDHFHUj2uvlM/
!
username JPSWC01-admin secret 5 $1$UTU0$BGdoUkycN.8eeCuP46y2W.
no aaa new-model
!
!
!
!
!
!
!
!
ip domain-name juane.prod
ip cef
no ipv6 cef
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan internal allocation policy ascending
!
!
!
!
!
!
!
!
!
!
!
!
!
!
interface GigabitEthernet0/0
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/2
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/3
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/0
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/1
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/2
 media-type rj45
 negotiation auto
!
interface GigabitEthernet1/3
 media-type rj45
 negotiation auto
!
interface GigabitEthernet2/0
 media-type rj45
 negotiation auto
!
interface GigabitEthernet2/1
 media-type rj45
 negotiation auto
!
interface GigabitEthernet2/2
 media-type rj45
 negotiation auto
!
interface GigabitEthernet2/3
 media-type rj45
 negotiation auto
!
interface GigabitEthernet3/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 media-type rj45
 negotiation auto
!
interface GigabitEthernet3/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 media-type rj45
 negotiation auto
!
interface GigabitEthernet3/2
 media-type rj45
 negotiation auto
!
interface GigabitEthernet3/3
 media-type rj45
 negotiation auto
!
interface Vlan1
 description "Vlan administrativa"
 ip address 172.16.1.3 255.255.255.0
!
interface Vlan99
 ip address 172.16.99.10 255.255.255.0
!
ip forward-protocol nd
!
no ip http server
no ip http secure-server
!
ip route 0.0.0.0 0.0.0.0 172.16.1.1
!
ip access-list standard ADMIN_ONLY
 permit 172.16.10.10
!
!
!
!
snmp-server community public RO
snmp-server location "Topologia red interna"
!
control-plane
!
banner exec ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner incoming ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
banner login ^C
**************************************************************************
* IOSv is strictly limited to use for evaluation, demonstration and IOS  *
* education. IOSv is provided as-is and is not supported by Cisco's      *
* Technical Advisory Center. Any use or disclosure, in whole or in part, *
* of the IOSv Software or Documentation to any third party for any       *
* purposes is expressly prohibited except as otherwise authorized by     *
* Cisco in writing.                                                      *
**************************************************************************^C
!
line con 0
line aux 0
line vty 0 4
 access-class ADMIN_ONLY in
 login local
 transport input ssh
!
!
end
```
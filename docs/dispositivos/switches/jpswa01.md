# Configuración: JPSWA01 (Cisco Acceso)

Este es el **switch de acceso** que conecta los dispositivos finales (servidores) a la red. Su principal responsabilidad es asignar cada puerto a su VLAN correspondiente (VLAN 5, 10, 200) y conectarse al switch core (`JPSWC01`) mediante un enlace troncal.

```
Current configuration : 4304 bytes
!
! Last configuration change at 19:40:57 UTC Fri Dec 19 2025
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
service compress-config
!
hostname JPSWA01
!
boot-start-marker
boot-end-marker
!
!
enable secret 5 $1$Mfh6$9rwqs4tPMH7DIWqd0qNk/0
!
username JPSWA01-admin secret 5 $1$ZNYt$AmaYwOhigv2QcslFx46tC.
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
 switchport access vlan 5
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet0/1
 switchport access vlan 5
 switchport mode access
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
 switchport access vlan 200
 switchport mode access
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
 switchport access vlan 10
 switchport mode access
 media-type rj45
 negotiation auto
!
interface GigabitEthernet2/1
 switchport access vlan 10
 switchport mode access
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
 ip address 172.16.1.4 255.255.255.0
!
interface Vlan99
 ip address 172.16.99.40 255.255.255.0
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
snmp-server location "switch de acceso red interna"
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
# Configuración: JPROO2 (Cisco)

Este es el router principal que gestiona el enrutamiento Inter-VLAN (Router-on-a-Stick) y se conecta al core de la red.

```
Current configuration : 4616 bytes
!
! Last configuration change at 00:10:33 UTC Fri Jan 2 2026
!
version 15.9
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
!
hostname JPRO02
!
boot-start-marker
boot-end-marker
!
!
enable secret 9 $9$0rnYaxWinayPJv$l0LB5IwbNze0I17fatX605.ZTk5jeUf/wL8MXqX08.Y
!
no aaa new-model
!
!
!
mmi polling-interval 60
no mmi auto-configure
no mmi pvc
mmi snmp-timeout 180
!
!
!
!
!
no ip icmp rate-limit unreachable
!
!
!
!
!
!
no ip domain lookup
ip domain name juane.prod
ip cef
no ipv6 cef
!
multilink bundle-name authenticated
!
!
!
!
username JPRO02-admin secret 9 $9$iyExofUW1N/EVP$R0wLVa4qnUVBwatPkLVrryzTb.MU7wmomuLHu2HzdLk
!
redundancy
!
no cdp log mismatch duplex
!
ip tcp synwait-time 5
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
interface Loopback0
 ip address 10.255.255.2 255.255.255.255
!
interface GigabitEthernet0/0
 description "Hacia router vyos"
 ip address 10.10.1.2 255.255.255.252
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 no ip address
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 no ip address
 shutdown
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3
 no ip address
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/3.5
 description Gateway de Vlan 5
 encapsulation dot1Q 5
 ip address 172.16.5.1 255.255.255.0
!
interface GigabitEthernet0/3.10
 description Gateway de Vlan 10
 encapsulation dot1Q 10
 ip address 172.16.10.1 255.255.255.0
!
interface GigabitEthernet0/3.99
 description Gateway de Vlan 99 Administrativa
 encapsulation dot1Q 99
 ip address 172.16.99.1 255.255.255.0
!
interface GigabitEthernet0/3.200
 description Gateway de Vlan 200
 encapsulation dot1Q 200
 ip address 172.16.200.1 255.255.255.0
!
interface GigabitEthernet0/3.500
 description VLAN_NATIVA
 encapsulation dot1Q 500 native
!
router ospf 1
 network 10.10.1.0 0.0.0.3 area 10
 network 10.255.255.2 0.0.0.0 area 10
 network 172.16.5.0 0.0.0.255 area 10
 network 172.16.10.0 0.0.0.255 area 10
 network 172.16.99.0 0.0.0.255 area 10
 network 172.16.200.0 0.0.0.255 area 10
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip route 0.0.0.0 0.0.0.0 10.10.1.1
!
ip access-list standard ADMIN_ONLY
 permit 172.16.10.10
!
ipv6 ioam timestamp
!
snmp-server community public RO
snmp-server location "GNS3 Lab - Network Core"
snmp-server chassis-id
snmp-server enable traps snmp authentication linkdown linkup coldstart warmstart
!
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
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 access-class ADMIN_ONLY in
 login local
 transport input ssh
!
no scheduler allocate
!
end
```
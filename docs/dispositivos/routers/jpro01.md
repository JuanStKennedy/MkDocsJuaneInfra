# Configuración: JPROO1 (VyOS)

Este es el router de núcleo que gestiona el enrutamiento entre el PfSense y el router cisco.
```
  interfaces {
     ethernet eth0 {
         address 10.10.0.2/30
         description "hacia firewall PfSense"
         hw-id 0c:08:d8:d9:00:00
     }
     ethernet eth1 {
         address 10.10.1.1/30
         description "Hacia router cisco"
         hw-id 0c:08:d8:d9:00:01
     }
     ethernet eth2 {
         hw-id 0c:08:d8:d9:00:02
     }
     ethernet eth3 {
         hw-id 0c:08:d8:d9:00:03
     }
     ethernet eth4 {
         hw-id 0c:08:d8:d9:00:04
     }
     loopback lo {
         address 10.255.255.1/32
     }
 }
 nat {
     source {
         rule 5 {
             outbound-interface {
                 name eth0
             }
             source {
                 address 172.16.5.0/24
             }
             translation {
                 address masquerade
             }
         }
         rule 200 {
             outbound-interface {
                 name eth0
             }
             source {
                 address 172.16.200.0/24
             }
             translation {
                 address masquerade
             }
         }
     }
 }
 protocols {
     ospf {
         area 10 {
             network 10.10.1.0/30
             network 10.10.0.0/30
             network 10.255.255.1/32
         }
     }
     static {
         route 0.0.0.0/0 {
             next-hop 10.10.0.1 {
             }
         }
         route 172.16.1.0/24 {
             next-hop 10.10.1.2 {
             }
         }
     }
 }
 service {
     ntp {
         allow-client {
             address 127.0.0.0/8
             address 169.254.0.0/16
             address 10.0.0.0/8
             address 172.16.0.0/12
             address 192.168.0.0/16
             address ::1/128
             address fe80::/10
             address fc00::/7
         }
         server time1.vyos.net {
         }
         server time2.vyos.net {
         }
         server time3.vyos.net {
         }
     }
     snmp {
         community public {
             authorization ro
         }
         listen-address 10.255.255.1 {
         }
     }
     ssh {
         port 22
     }
 }
 system {
     config-management {
         commit-revisions 100
     }
     console {
         device ttyS0 {
             speed 115200
         }
     }
     host-name JPRO01
     name-server 8.8.8.8
     name-server 1.1.1.1
     option {
         reboot-on-upgrade-failure 5
     }
     syslog {
         local {
             facility all {
                 level info
             }
             facility local7 {
                 level debug
             }
         }
     }
 }
```
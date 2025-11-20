## CCNPCOREコマンド集

## No1(rpvst+LACP)
show run

conf t
spanning-tree mode rapid-pvst

interface e0/0
switchport trunk encapsulation dot1q 
switchport mode trunk
switchport trunk allowed vlan 500

ping 10.10.100.10

int Port-channel 10
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 500

show etherchannel summary (Po10がSU)
ping 10.10.100.20

copy run start

## No2(VRF)
show ip interface brief(show runとの差別化、構成図見ればよくね、など)

R10>
interface Tunnel 0
tunnel source Ethernet0/1 //初期設定なしの場合
tunnel destination 10.10.2.1
vrf forwarding CORP

※% Interface TunnelO IPv4 disabled and address (es) removed due to disabling VRF CORPが表示された場合※
ip address 10.100.100.1 255.255.255.0

R20>
interface Tunnel 0
tunnel source Ethernet0/2
tunnel destination 10.10.1.1
vrf forwarding CORP
ip address 10.100.100.2 255.255.255.0

R10>
ping vrf CORP 10.100.100.2

R10>
show run

interface Tunnel 0
tunnel protection ipsec profile MyProfile

R20>
interface Tunnel 0
tunnel protection ipsec profile MyProfile

R10>
ip route vrf CORP 10.101.2.0 255.255.255.0 Tunnel 0
R20>
ip route vrf CORP 10.100.1.0 255.255.255.0 Tunnel 0

R10>
ping vrf CORP 10.101.2.1 source e0/0.100
copy run start

R20>
ping vrf CORP 10.100.1.1 source e0/0.101
copy run start

## No3(OSPF DR BDR)
R3>
interface e0/1
ip ospf priority 255
R20>
clear ip ospf process
Reset all OSPF processes? :yes

R3>
show ip ospf neighbor(10.2.203.2:FULL/BDR)

R10>
int e0/0
ip ospf priority 0

show ip ospf neighbors(10.1.102.10:FULL/DROTHER)

R3,R10>
copy run start

## No4(eBGP Neighbor)
show ip interface brief

router bgp 130
bgp router-id 3.3.3.3
neighbor 209.165.202.130 remote-as 110
neighbor 209.165.200.230 remote-as 120

router bgp 130
network 209.165.203.1 mask 255.255.255.255
network 200.200.203.2 mask 255.255.255.255

show ip route bgp

## No5(OSPF/Prefix-list)
show run
show ip interface brief

ip prefix-list deny_R1_Lo0 seq 1 deny 1.1.1.1/32

router ospf 1
area 10 filter-list prefix deny_R1_Lo0 in

ping 1.1.1.1(失敗)

show run
show ip interface brief

ip prefix-list deny_R20_Lo0 seq 1 deny 20.20.20.20/32

router ospf 1
area 20 filter-list prefix deny_R20_Lo0 out

ping 20.20.20.20 (失敗)

copy run start

## No6(OSPF DR/Summarization)
int e0/1
ip ospf priority 255

clear ip ospf process

show ip ospf neighbor

R2>show run(ospfプロセスIDの確認)
R10>show run(R10のループバックアドレスの確認)

router ospf 10
area 10 range 10.1.0.0 255.255.0.0

show ip route

copy run start

## No7(Trunk UDLD&LACP)
SW10>
show run

conf t
interface e0/0
switchport trunk encapsulation dot1q
switchport mode trunk
end

PC2>
ping 10.10.100.10

SW10>
conf t
int e0/0
udld port aggressive

interface range e0/2-3
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 1,300
channel-group 10 mode active
exit

PC2>
ping 10.10.100.30

SW10>
interface e0/1
spanning-tree portfast
end

copy run start

## No8(OSPF DR BDR)
R3>
conf t
int e0/1
ip ospf network point-to-point
exit
clear ip ospf process y

R20>
conf t
int e0/0
ip ospf network point-to-point
exit
clear ip ospf process y

R3>
show ip ospf neighbor

R10>
conf t
int e0/0
ip ospf priority 255

R2,R10>
clear ip ospf process y

R2>
show ip ospf neighbor 

R3,R10>
copy run start

## No9(eBGP neighbor)

R1>
show run

conf t
router bgp 100
bgp router-id 10.1.1.100
neighbor 209.165.200.226 remote-as 200
neighbor 209.165.202.130 remote-as 300

address-family ipv4
neighbor 209.165.200.226 active
neighbor 209.165.202.130 active

network 10.1.1.100 mask 255.255.255.255
network 209.165.201.0 mask 255.255.255.248
network 209.165.201.8 mask 255.255.255.248

end

show ip bgp summary

R1>
copy run start

## No10()
R1>
show run
conf t
flow exporter MyFlowExporter
destination 10.14.1.2
transport udp 2055
export protocol netflow v9
exit

SW1>
show run
conf t

monitor session 11 source interface e0/1 both
monitor session 11 destination interface e1/1 
exit

show monitor session 11

RT1>
ip sla schedule 5 life forever start-time now
exit
show ip sla summary

R1,SW1>
copy run start

## No11()
R1>
show run
conf t

int e0/0
ip flow monitor Monitor-R1Flow input
ip flow monitor Monitor-R1Flow output
exit

SW1>
conf t
monitor session 12 source vlan 12 both
monitor session 12 destination interface e1/3
show monitor session 12


R1>
ip sla 1
icmp-echo 10.12.1.2 source-interface e0/0
frequency 300
exit

ip sla schedule 1 life forever start-time now
exit
show ip sla summary

R1,SW1>
copy run start

## No12()
R1>
show run 
conf t

router ospf 1
exit

int lo0
ip ospf 1 area 0
exit

int e0/0
ip ospf 1 area 0
exit

int e0/1
ip ospf 1 area 0
end

show ip ospf interface brief
show ip route ospf

R2,R3>
show run

R2>
conf t

router ospf 1
area 10 range 10.1.0.0 255.255.0.0
end

show ip route

R3>
conf t
router ospf 1
area 10 range 10.2.0.0 255.255.0.0
end

show ip route

R1,R2,R3>
copy run start

## No13()

R10>

show ip eigrp neighbors
show ip access-list 150

conf t
ip access-list extended 150
5 permit eigrp any any 
end

show ip eigrp neighbors

R20>
conf t

access-list 100 permit icmp 192.168.24.0 0.0.0.255 any

class-map CoPP_ICMP
match access-group 100

policy-map CHECK_ICMP
class CoPP_ICMP
police 8000
conform-action transit
exceed-action drop 

control-plante
service-policy input CHECK_ICMP

show policy-map control-plane

R10,R20>
copy run start

## No14()

SW10>
show run

conf t
no int po10
int po10
switchport trunk encapsulation dot1q
switchport mode trunk 
exit

int range e0/0-2
no switchport mode access
switchport trunk encapsulation dot1q
switchport mode trunk 
switchport trunk alloed vlan 20
channel-group 11 mode active
exit

int po11
shut
no shut
exit

show interfaces trunk

spanning-tree vlan 10,30 root primary

exit

show spanning-tree vlan 10,30
copy run start

## No15()

R30>
show run

conf t
router ospf 100
router-id 10.30.30.30
network 10.30.30.30 0.0.0.0 area 0
network 10.20.30.30 0.0.0.255 area 0
network 10.0.30.30 0.0.0.255 area 0
network 10.50.40.30 0.0.0.255 area 0
end

show ip ospf neighbors

router ospf 100
area 50 range 10.50.0.0 255.255.192.0
end

show ip route
copy run start

R10>
show run

## No16()

conf t
router ospf 20
router-id 20.20.20.20
exit

int loopback 0
ip ospf 20 area 0

int e0/0
ip ospf 20 area 0
exit 

int e0/1
ip ospf 20 area 0
exit

int e0/2
ip ospf 20 area 40
end

show ip ospf neighbors

conf t

router ospf 20
area 40 10.40.0.0 255.255.192.0 
end

show ip route 
copy run start

## No17()
R22>
show run

conf t

ip vrf Finance
exit

int Tunnel 10
vrf forwarding Finance
ip address 10.10.10.2 255.255.255.252
tunnel source e0/0
tunnel destination 209.165.200.230
no shut

end

ping vrf Finance 10.10.10.1

conf t

int e0/1
vrf forwarding Finance
ip address 10.22.22.1 255.255.255.252
exit

ip route vrf Finance 10.10.111.0 255.255.255.0 tunnel 10
ip route vrf Finance 10.10.222.0 255.255.255.0 10.22.22.2
exit

ping 10.10.111.1 source vlan 222

copy run start

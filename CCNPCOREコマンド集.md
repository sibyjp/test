## CCNPCOREコマンド集

## No1(rpvst+LACP)
1.SW20でRapid PVST+を設定
2.SW20と30間のトランクは機能していない(e0/0がaccessポート)。修正ののち、ping(10.10.100.10)を通す。
3.SW10と20の間のLACPポートチャネルは動作していない。修正ののちping(10.10.100
20)を通す。

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
1.Tunnel 0を利用して、R10、R20間のCORPVRFを拡張
2.事前構成済みのプロファイルを使用してVRFを保護
3.R10とR20でスタティックルーティングを構成し、VLAN100とVLAN101が互いに通信できるようにする。

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
1.R20がBDRになるようにR3を設定
2.DR/BDR選択に参加しないようにR10を設定

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
1.ルーターIDにループバック0を使用してeBGPを設定
2.R3のループバック100及びループバック200ネットワークをAS110及びAS120にアドバタイズ

router bgp 130
bgp router-id 3.3.3.3
neighbor 209.165.202.130 remote-as 110
neighbor 209.165.200.230 remote-as 120

router bgp 130
network 209.165.203.1 mask 255.255.255.255
network 200.200.203.2 mask 255.255.255.255

show ip route bgp

## No5(OSPF/Prefix-list)
1.R1のループバック0のルートはエリア10にアドバタイズされるべきではない。部分的に設定されたプレフィックスリストを利用し、タスクを完了する。
2.R20ループバック0のルートは、エリア20の外にアドバタイズされるべきではない。部分的に設定されたプレフィックスリストを利用し、タスクを完了する。

show run
show ip interface brief

ip prefix-list deny_R1_Lo0 seq 1 deny 1.1.1.1/32

router ospf 1
area 10 filter-list prefix deny_R1_Lo0 in

ping 1.1.1.1

show run
show ip interface brief

ip prefix-list deny_R20_Lo0 seq 1 deny 20.20.20.20/32

router ospf 1
area 20 filter-list prefix deny_R20_Lo0 out

ping 20.20.20.20 (失敗)

copy run start

## No6(OSPF DR/Summarization)
1.R2が常にエリア10のDRになるよう設定
2.R2で一つのコマンドを設定して、エリア10のルートを一つのルートに集約

int e0/1
ip ospf priority 255

clear ip ospf process

show ip ospf neighbor

router ospf 10
area 10 range 10.1.0.0 255.255.0.0

show ip route

copy run start

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


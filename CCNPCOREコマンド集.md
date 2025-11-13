## CCNPCOREコマンド集

## ひな形
### タスク
### ソリューション
### コマンド
(ソリューションの注釈は任意)

## No1(rpvst+LACP)
### タスク
1.SW20でRapid PVST+を設定
2.SW20と30間のトランクは機能していない(e0/0がaccessポート)。修正ののち、ping(10.10.100.10)を通す。
3.SW10と20の間のLACPポートチャネルは動作していない。修正ののちping(10.10.100
20)を通す。

### ソリューション
1.割愛
2.show run(R2)でSW20,30間の修正点を確認し修正。(今回はvlan設定がaccessなのでtrunkに再設定)
3.Po10に対して2と同様の対処。

### コマンド
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
### タスク
1.Tunnel 0を利用して、R10、R20間のCORPVRFを拡張
2.事前構成済みのプロファイルを使用してVRFを保護
3.R10とR20でスタティックルーティングを構成し、VLAN100とVLAN101が互いに通信できるようにする。
事前構成済みのプロファイルはMyProfile

### ソリューション
1.R10,R20でそれぞれshow ip interface briefによってvrf設定を確認。destinationが設定されておらず、tunnel0が疎通できていないので、設定ののちpingで疎通確認
2.1項でプロファイル名を確認し、vrfを保護
3.R10,R20でそれぞれtunnel0を利用したスタティックルートを構成。設定ののちpingで疎通確認

### コマンド
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
### タスク
1.R20がBDRになるようにR3を設定
2.R10がDR/BDR選択に参加しないようにR2を設定

### ソリューション
1.構成図を見てR20のネクストホップとなるR3のインターフェースで、適切なpriorityを設定。
2.構成図を見てR10のネクストホップとなるR2のインターフェースで、適切なpriorityを設定。

### コマンド
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
### タスク
1.ルーターIDにループバック0を使用してeBGPを設定
2.R3のループバック100及びループバック200ネットワークをAS110及びAS120にアドバタイズ

### ソリューション
1.ループバックアドレスなどを確認するため、R3でshow ip interface briefを行う。のち、eBGPを設定。ルーターidのほか、neighborなども設定する。
2.BGPにループバック100,200のネットワークを登録する。configを見れば問題なし。show ip route bgpで確認。

### コマンド
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
### タスク
1.R1のループバック0のルートはエリア10にアドバタイズされるべきではない。部分的に設定されたプレフィックスリストを利用し、タスクを完了する。
2.R20ループバック0のルートは、エリア20の外にアドバタイズされるべきではない。部分的に設定されたプレフィックスリストを利用し、タスクを完了する。
R1のループバックアドレスは1.1.1.1です。
R20のループバックアドレスは20.20.20.20です。
R2の初期configで、プレフィックスリスト:deny_R1_Lo0 が部分的に構成されています。
R3の初期configで、プレフィックスリスト:deny_R20_Lo0 が部分的に構成されています。

### ソリューション
1.R1とエリア10の境界であるR2において、プレフィックスリストを設定する。R1でのshow runでループバックアドレスの確認、R2でのshow runで事前構成されたプレフィックスリストを確認。R1のLo0を除外するプレフィックスリストを追加したのち、ospfプロセスに内側で適用する。
2.R20とエリア0の境界であるR3において、プレフィックスリストを設定する。R20でのshow runでループバックアドレスの確認、R3でのshow runで事前構成されたプレフィックスリストを確認。R20のLo0を除外するプレフィックスリストを追加したのち、ospfプロセスに外側で適用する。

### コマンド
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
### タスク
1.R2が常にエリア10のDRになるよう設定
2.R2で一つのコマンドを設定して、エリア10のルートを一つのルートに集約
R10の全てのループバックアドレスは、10.1.x.xの範囲内にあるとします。

### ソリューション
1.構成図でR2→R10のインターフェースを確認し、該当ifで最高のpriorityを設定。R10ではclear ip ospf processで設定をクリアしたのち、show ip ospf neighborで検証。
2.R2のshow runでospfプロセスIDを確認、R10のshow runでR10のループバックアドレスの確認。のち、R10のループバックアドレスがどのように集約できるか検討し、設定。show ip route で検証。

### コマンド
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
### タスク
### ソリューション
### コマンド
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
### タスク
### ソリューション
### コマンド
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
### タスク
### ソリューション
### コマンド

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

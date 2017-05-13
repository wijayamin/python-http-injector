# HTTP-INJECTOR OPENWRT

### packages yang wajib di install
* python 2.7
    * python-codecs 
    * python-logging 
    * python-base 
    * python-light
    ````
    opkg update
    opkg install python-base python-light python-logging python-codecs 
    ````
* openvpn
    * openvpn-openssl
    ````
    opkg update
    opkg install openvpn-openssl
    ````

1. install semua package yang dibutuhkan
2. buat interface openvpn
    ````
    uci set network.openvpn=interface
    uci set network.openvpn.proto='none'
    uci set network.openvpn.ifname='tun0'

    uci commit
    ````
3. atur firewall untuk interface openvpn yang baru saja dibuat  
    *firewall zone rule*
    ````
    uci add firewall.zone
    uci set firewall.@zone[-1].name='openvpn'
    uci set firewall.@zone[-1].output='ACCEPT'
    uci set firewall.@zone[-1].forward='REJECT'
    uci set firewall.@zone[-1].input='ACCEPT'
    uci set firewall.@zone[-1].masq='1'
    uci set firewall.@zone[-1].mtu_fix='1'
    uci set firewall.@zone[-1].network='openvpn'
    ````
    *firewall forwading rule*
    ````
    uci add firewall forwarding
    uci set firewall.@forwarding[-1].src='lan'
    uci set firewall.@forwarding[-1].dest='openvpn'

    uci commit
    ````
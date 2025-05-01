# wifi-hotspot-in-wlp1s0

## 1.  our Wi-Fi adapter must support AP mode (check with iw list).

## Make sure hostapd, dnsmasq, and iptables are installed.
List packages
```bash
sudo apt update
sudo apt install hostapd dnsmasq iptables
```
You may disable hostapd and dnsmasq auto-start to use manual config:
```
sudo systemctl disable hostapd
sudo systemctl disable dnsmasq

```
### Step 1: Verify Wi-Fi AP Capability
```
iw list | grep -A 10 'Supported interface modes'

```


### Step 2: Stop Network Manager for wlp1s0
```
sudo nmcli radio wifi off
sudo rfkill unblock wifi
```


### Step 3: Create hostapd Configuration
```
sudo nano /etc/hostapd/hostapd.conf
```
interface=wlp1s0
driver=nl80211
ssid=MyUbuntuHotspot
hw_mode=g
channel=6
auth_algs=1
wmm_enabled=1
wpa=2
wpa_passphrase=yourpassword123
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP


### Step 4: Configure Static IP for Hotspot Interface
```
sudo ip addr add 192.168.12.1/24 dev wlp1s0
sudo ip link set wlp1s0 up


```


### Step 5: Configure dnsmasq for DHCP/DNS
```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.backup

sudo nano /etc/dnsmasq.conf

interface=wlp1s0
dhcp-range=192.168.12.10,192.168.12.50,255.255.255.0,24h


sudo dnsmasq


```


### Step 6: Enable IP Forwarding and Setup NAT
```
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

for making paramanet

sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo sysctl -p


# Check internet route interface (may be same if STA+AP)
ip route

# Assuming `wlp1s0` is connected:
sudo iptables -t nat -A POSTROUTING -o wlp1s0 -j MASQUERADE
sudo iptables -A FORWARD -i wlp1s0 -o wlp1s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlp1s0 -o wlp1s0 -j ACCEPT

```


### Step 7: Start the Hotspot
```
sudo hostapd /etc/hostapd/hostapd.conf


```
### Step 8: To stop 
```
sudo pkill hostapd
sudo pkill dnsmasq
sudo iptables -F
sudo iptables -t nat -F
sudo ip addr flush dev wlp1s0
```


## 1.  if wifi adapter is not working Force Unload and Reload iwlwifi Driver

## Check for Loaded Modules
```bash
lsmod | grep iwl

```
Stop Services Using Wi-Fi
```
sudo systemctl stop NetworkManager
sudo systemctl stop wpa_supplicant

```
Try Removing Modules

```
sudo modprobe -r iwlmvm
sudo modprobe -r iwlwifi


```
If iwlmvm is removed successfully, proceed with iwlwifi
```
sudo rmmod iwlwifi
sudo rmmod iwlmvm

```
Reload the iwlwifi Module

```
sudo modprobe iwlwifi

```
Restart Network Services

```
sudo systemctl start wpa_supplicant
sudo systemctl start NetworkManager

```
Bring Up Wi-Fi Interface
```
sudo rfkill unblock wifi
sudo ip link set wlp1s0 up


```
Check Wi-Fi Networks
```
nmcli device wifi list


```


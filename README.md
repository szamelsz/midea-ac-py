
English Version | [中文版](./中文.md#)

This is a custom component for Home Assistant to integrate the Midea Air Conditioners via the Local area network.

Tested with hass version 0.110.2

## WiFi module version 3
If you have *103 WiFi module (ex. SK-103 or OSK-103), your AC use new protocol 8370. For this protocol, you need give your `WiFi AP access data`, `AC WIFI MAC` and install and run `fake cloud script`. Without this script, your AC connect to `module.appsmb.com:443` and this script cannot connect to your AC over LAN! If you connect your AC to WiFi without internet access, your AC WiFi will has boot loop, and restart over 15 seconds (7s for DHCP request, and 8 seconds for 3 try to connect to cloud).
But, if you run some fake-cloud server on your LAN, and change DNS record on your local DNS server, AC WiFi will connect to your fake-cloud, and can working in LAN mode to 600s! After this, AC WiFi do reboot and after 8s for DHCP request, again working in LAN mode to 600s. To install fake-cloud, read # Install fake-cloud.

## Installation

### Install fake-cloud (only if you have protocol version 3)
#### Requirements
* Router with local DNS server with possible to add DNS entry (ex. Openwrt)
* DHCP confiugration which send local DNS server

#### Install fake-cloud
Is both fake-cloud scripts. My, very simple script - [`fake-cloud-wmp.py`](fake-cloud-wmp.py) which make echo for requests and give you 600s of LAN mode, or `fake-midea-cloud.py` from Colin Kubeler (dont release yet).

1. Download to /usr/local/sbin/ and run fake-cloud script on port 443. If you need you can change FAKE_CLOUD_IP.

2. If you want, you can use systemd module [`fake-cloud.service`](fake-cloud.service). You mmust save this file in `/etc/systemd/system/` then:
```
systemctl enable fake-cloud
systemctl start fake-cloud
```

3. In your local DNS server, set DNS record `module.appsmb.com` to resolve on your FAKE_CLOUD_IP. 

On openwrt add to `/etc/config/dhcp`:
```
config dnsmasq
    list addnhosts '/etc/hosts_my'
```
In `/etc/hosts_my` :
```
FAKE_CLOUD_IP module.appsmb.com
```
Then `/etc/init.d/dnsmasq restart`



### Install manually
1. Clone this repo
2. Place the `custom_components/midea_ac` folder into your `custom_components` folder
3. Remove msmart version and install msmart from [branch support-8370 from kubelc](https://github.com/kueblc/midea-msmart/tree/support-8370). If you have HA in docker, you must execute this commands in docker:
```
pip3.7 uninstall msmart
pip3.7 install git+https://github.com/kueblc/midea-msmart.git@support-8370
```



## Configuration

**Configuration variables:**  
key | description | example 
:--- | :--- | :---
**platform (Required)** | The platform name. | midea_ac
**host (Required)** | Midea AC Device's IP Address. | 192.168.1.100
**id (Required)** | Midea AC Device's applianceId. | 123456789012345
**8370_only_ac_mac (Optional)** | Midea MAC Device, required if you have protocol 3 version | 12B4567C8901
**8270_only_wifi_ssid (Optional)** | WiFi Access Point to which air conditioning is connected, required if you have protocol 3 version |  WiFi-AccessPoint-Name
**8370_only_wifi_pw (Optional)** | WiFi Access Point Password to which air conditioning is connected, required if you have protocol 3 version |  MyPassw0rd
**use_fan_only_workaround (Optional)** | Set this to true if you need to turn off device updates because they turn device on and to fan_only | true

**How to Get applianceId:**

- you can use command ```midea-discover``` to discover midea devices on the host in the same Local area network. Note: This component only supports devices with model 0xac (air conditioner) and words ```supported``` in the output.
```shell
pip3 install msmart
midea-discover
```

- if you use Midea Air app outside China, there is a easy way to get your deviceid.

1. open Midea Air app, and share the device, you will get a QR Code.
2. save the QR Code 
3. upload QR Code Sreenshort to https://zxing.org/w/decode.jspx or decode QR code use other tool.
4. you will get the data like MADEVICESHARE:<base64_string>
5. decode base64 string online https://www.base64decode.org/ or use other tool
6. you will get the device id

- if you use android, you can use ```adb```，filter from log:
```shell
adb logcat | grep -i deviceid
```

- if you use iPhone，iPhone connects to macOS with a data cable and filters the applianceId from the console log

- If you do not have the above environment and conditions, you need to capture the air conditioner and save the files, after can be used [pcap-decrypt.py](./pcap-decrypt.py#) to Get. Remember to use the number, not hex string.

**Example configuration.yaml:**
* Single device
```yaml
climate:
  - platform: midea_ac
    host: 192.168.1.100
    id: 123456789012345
    8370_only_ac_mac: 12B4567C8901
    8270_only_wifi_ssid:  WiFi-AccessPoint-Name
    8370_only_wifi_pw: MyPassw0rd
```
* Multiple device
```yaml
climate:
  - platform: midea_ac
    host: 192.168.1.100
    id: 123456789012345
    8370_only_ac_mac: 12B4567C8901
    8270_only_wifi_ssid:  WiFi-AccessPoint-Name
    8370_only_wifi_pw: MyPassw0rd
  - platform: midea_ac
    host: 192.168.1.200
    id: 543210987654321
    8370_only_ac_mac: 12B4567C8902
    8270_only_wifi_ssid:  WiFi-AccessPoint-Name
    8370_only_wifi_pw: MyPassw0rd
```

## Buy me a cup of coffee to help maintain this project further?

- [via Paypal](https://www.paypal.me/himaczhou)
- [via Bitcoin](bitcoin:3GAvud4ZcppF5xeTPEqF9FcX2buvTsi2Hy) (**3GAvud4ZcppF5xeTPEqF9FcX2buvTsi2Hy**)
- [via AliPay(支付宝)](https://i.loli.net/2020/05/08/nNSTAPUGDgX2sBe.png)
- [via WeChatPay(微信)](https://i.loli.net/2020/05/08/ouj6SdnVirDzRw9.jpg)

Your donation will make me work better for this project.

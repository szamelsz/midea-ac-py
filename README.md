English Version | [中文版](./中文.md#)

This is a custom component for Home Assistant to integrate the Midea Air Conditioners via the Local area network.

Tested with `hass` version 0.110.2

## WiFi module version 3
If you have \*103 WiFi module (ex. SK-103 or OSK-103), your AC is using the new protocol called “8370”. For this protocol, you need provide your **WiFi AP access data**, **MAC address of your AC WIFI module**, install and run the **fake cloud script**. Without this script, your AC will connect to `module.appsmb.com:443` (may vary depending on its manufacturer) and this script would not be able to connect to your AC over LAN! If your AC is connected to a WiFi with no internet access, the WiFi module of your AC will be in a boot loop, and will restart in 15 seconds (DHCP request for 7 seconds, and 3 attempts to connect to the cloud in 8 seconds).

However, if you run the fake-cloud server on your LAN, and change the corresponding DNS record on your local DNS server, the WiFi module of your AC will connect to your fake server, and thus will be able to working in LAN mode to 600 seconds. After this, it will reboot in 8 seconds for DHCP request, and then working again in LAN mode for 600 seconds. To install fake-cloud, read [Install fake-cloud](#Install-fake-cloud).

## Installation

### Install fake-cloud (only if you have protocol version 3)
#### Requirements
* A router with a local DNS server that is possible to add custom DNS entries (ex. OpenWRT)
* Configurable DHCP configuration which send local DNS server

#### Install fake-cloud
There are 2 types of fake cloud scripts. One, by Marcin Janowski, is a very simple script ([`fake-cloud-wmp.py`](fake-cloud-wmp.py)) which echoes the payload for each requests and give you 600 seconds in LAN mode. The other is `fake-midea-cloud.py` from Colin Kuebler (not released yet).

1. Save the script to `/usr/local/sbin/` and run it on port 443. Change FAKE_CLOUD_IP to your own IP accordingly.

2. If you want, you can use `systemd` module [`fake-cloud.service`](fake-cloud.service). You must save this file in `/etc/systemd/system/`, and then run:
```
systemctl enable fake-cloud
systemctl start fake-cloud
```

3. In your local DNS server, set the DNS record for `module.appsmb.com` to your `FAKE_CLOUD_IP`. 

If you are using OpenWRT, add the following lines to `/etc/config/dhcp`:
```
config dnsmasq
    list addnhosts '/etc/hosts_my'
```

Then the following line to `/etc/hosts_my`:
```
FAKE_CLOUD_IP module.appsmb.com
```
Finally run `/etc/init.d/dnsmasq restart`.



### Install manually
1. Clone this repo
2. Place the `custom_components/midea_ac` folder into your `custom_components` folder
3. Remove exisitng `msmart` versions and install it from [branch `support-8370` from `kubelc`](https://github.com/kueblc/midea-msmart/tree/support-8370). If you are running HA from a Docker container, you should run the following commands in your container:

```
pip3.7 uninstall msmart
pip3.7 install git+https://github.com/kueblc/midea-msmart.git@support-8370
```



## Configuration

**Configuration variables:**  
key | description | example 
:--- | :--- | :---
**platform (Required)** | The platform name. | midea_ac
**host (Required)** | IP address of your Midea AC Device. | 192.168.1.100
**id (Required)** | `applianceId` of your Midea AC Device. | 123456789012345
**8370_only_ac_mac (Optional)** | MAC address of your Media AC Device, required if it is using protocol version 3 | 12B4567C8901
**8370_only_wifi_ssid (Optional)** | SSID of the WiFi AP where the AC is connected to, required if it is using protocol version 3 |  WiFi-AccessPoint-Name
**8370_only_wifi_pw (Optional)** | Password of the WiFi AP where the AC is coonnected to, required if it is using protocol version 3 |  MyPassw0rd
**use_fan_only_workaround (Optional)** | Set this to true if you need to turn off device updates because they turn device on and to `fan_only` | true

**How to Get `applianceId`:**

- you can use command `midea-discover` to discover Midea devices on the host in the same Local area network. Note: This component only supports devices with model 0xac (air conditioner) and words `supported` in the output.
```shell
pip3 install msmart
midea-discover
```

- if you use Midea Air app outside China, there is an easier way to get your `deviceid`.

1. Open Midea Air app, and share the device, you will get a QR Code.
2. Save the QR Code
3. Upload QR Code Screenshot to https://zxing.org/w/decode.jspx or decode it with other tools.
4. You will get a string like `MADEVICESHARE:<base64_string>`
5. Decode the `<base64_string>` part online using https://www.base64decode.org/ or other tools you prefer
6. You will get the device ID from it

- If you are using the Android app, you can use `adb`, and get the ID from the log:

```shell
adb logcat | grep -i deviceid
```

- if you are using the iOS app, connect your iPhone to a macOS computer over cable and search for `applianceId` from the console log.

- If you do not have either of these, you need to capture traffics from your AC, then run through the captured PCAP file with [pcap-decrypt.py](./pcap-decrypt.py#) to get your Device ID. Remember to use the decimal number, not hex string.

**Example configuration.yaml:**
* Single device
```yaml
climate:
  - platform: midea_ac
    host: 192.168.1.100
    id: 123456789012345
    8370_only_ac_mac: 12B4567C8901
    8370_only_wifi_ssid:  WiFi-AccessPoint-Name
    8370_only_wifi_pw: MyPassw0rd
```
* Multiple device
```yaml
climate:
  - platform: midea_ac
    host: 192.168.1.100
    id: 123456789012345
    8370_only_ac_mac: 12B4567C8901
    8370_only_wifi_ssid:  WiFi-AccessPoint-Name
    8370_only_wifi_pw: MyPassw0rd
  - platform: midea_ac
    host: 192.168.1.200
    id: 543210987654321
    8370_only_ac_mac: 12B4567C8902
    8370_only_wifi_ssid:  WiFi-AccessPoint-Name
    8370_only_wifi_pw: MyPassw0rd
```

## Buy me a cup of coffee to help maintain this project further?

- [via Paypal](https://www.paypal.me/himaczhou)
- [via Bitcoin](bitcoin:3GAvud4ZcppF5xeTPEqF9FcX2buvTsi2Hy) (**3GAvud4ZcppF5xeTPEqF9FcX2buvTsi2Hy**)
- [via AliPay(支付宝)](https://i.loli.net/2020/05/08/nNSTAPUGDgX2sBe.png)
- [via WeChatPay(微信)](https://i.loli.net/2020/05/08/ouj6SdnVirDzRw9.jpg)

Your donation will make me work better for this project.

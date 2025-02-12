
- Команды для вывода Статистики 
```
wfb-cli gs
wfb-cli drone

sudo journalctl -xu wifibroadcast@gs
sudo journalctl -xu wifibroadcast@drone
```

### Установка Драйверов(RTL8812AU)
```
git clone -b v5.2.20 https://github.com/svpcom/rtl8812au.git
cd rtl8812au
make
sudo ./dkms-install.sh
sudo depmod -a
sudo modprobe 88XXau_wfb
sudo reboot

Установка net-tools

Sudo apt-get install net-tools -y

ethtool -i wlx* Проверка драйвера
```

- Как должен выглядеть вывод команды
```
driver: rtl88xxau_wfb  ---- Нужный драйвер
version: 6.1.0-1025-rockchip
firmware-version: 
expansion-rom-version: 
bus-info: 1-1:1.0
supports-statistics: no
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
```
### Установка Воздуха
- Убедитесь, что у вас есть исходники ядра!
```
ls /usr/lib/$(uname -r)/build
```
- Если их нет, то следует выбрать другой образ или билдить свой. Альтернативный вариант - использовать apt-get install linux-headers, но шанс успеха мал
- Приступать строго после установки драйверов
```
sudo apt install python3-all libpcap-dev libsodium-dev python3-pip python3-pyroute2 \
python3-future python3-twisted python3-serial python3-all-dev iw virtualenv \
debhelper dh-python build-essential -y

git clone -b Stable https://github.com/DroneCustom/wfb-ng.git
cd wfb-ng
sudo make deb
cd deb_dist
sudo dpkg -i (Название .deb файла)
```
- Проверка установки
```
sudo systemctl status wifibroadcast@drone
```
- Пример вывода
```
wifibroadcast@drone.service - WFB profile drone
     Loaded: loaded (/usr/lib/systemd/system/wifibroadcast@.service; disabled; preset: enabled)
     Active: inactive (dead)
```

### Установка GS (1 вариант)

```
См шаги установки драйвера
```
- Узнайте название wifi модуля, ибо каждому присвоено свое имя
```
ifconfig
```
- Пример вывода 
```
wlx1ca770fb0a16: flags=4099<UP,BROADCAST,MULTICAST>  mtu 2312
        ether 1c:a7:70:fb:0a:16  txqueuelen 1000  (Ethernet)
        RX packets 41  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
- Пропишите `$ ethtool -i wlx1ca770fb0a16(название карты)` и убедитесь, чт: `rtl88xxau_wfb` or `rtl8812eu`
- Скачайте [install_gs.sh](https://raw.githubusercontent.com/svpcom/wfb-ng/refs/heads/master/scripts/install_gs.sh):
```
curl -o install_gs.sh https://raw.githubusercontent.com/svpcom/wfb-ng/refs/heads/master/scripts/install_gs.sh
sudo bash ./install_gs.sh
```
- Если у вас выдает ошибку
```
gpg: no valid OpenPGP data found.
--------------------------------------------------------------------------------
WFB-ng setup failed
```
- Скачайте ключ вручную `wget https://apt.wfb-ng.org/public.asc`
- Добавьте его `sudo gpg --dearmor --yes -o /usr/share/keyrings/wfb-ng.gpg public.asc`
- Удалите `sudo rm public.asc`
- Установить `sudo bash ./install_gs.sh`

### Установка GS (2 вариант)

```
См шаги установки драйвера
```
- Узнайте название wifi модуля, ибо каждому присвоено свое имя
```
ifconfig
```
- Пример вывода 
```
wlx1ca770fb0a16: flags=4099<UP,BROADCAST,MULTICAST>  mtu 2312
        ether 1c:a7:70:fb:0a:16  txqueuelen 1000  (Ethernet)
        RX packets 41  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
- Пропишите `$ ethtool -i wlx1ca770fb0a16(название карты)` и убедитесь, что: `rtl88xxau_wfb` or `rtl8812eu`
```
git clone -b v5.2.20 https://github.com/svpcom/rtl8812au.git
cd wfb-ng
sudo ./scripts/install_gs.sh wlx1ca770fb0a16
```
- Проверка установки 
```
sudo systemctl status wifibroadcast@gs
```

- Пример вывода
```
○ wifibroadcast@gs.service - WFB-ng standalone server, profile gs
     Loaded: loaded (/usr/lib/systemd/system/wifibroadcast@.service; enabled; preset: enabled)
     Active: inactive (dead)
```
###Работа с конфигами

1. На обоих юнитах | RTL8812AU
     ```
     sudo nano /etc/modprobe.d/wfb.conf

     # blacklist stock module
     blacklist 88XXau
     blacklist 8812au
     blacklist 8812
     options cfg80211 ieee80211_regdom=RU
     # maximize output power by default
     options 88XXau_wfb rtw_tx_pwr_idx_override=-10
     ```
2. Создайте ключи и переместите их в /etc
     ```
     cd wfb-ng
     wfb_keygen

     sudo mv drone/gs.key /etc

     sudo scp filename.key name@192.168.68.###:/etc
     ```
3. Добавьте net.core.bpf_jit_enable = 1 в /etc/sysctl.conf
     ```
     sudo nano /etc/sysctl.conf

     net.core.bpf_jit_enable = 1

     sudo sysctl -p
     ```
4. Настройка конфига для Земли
     ```
     sudo nano /etc/wifibroadcast.cfg

     [common]
     wifi_channel = 165     # 165 -- radio channel @5825 MHz, range: 5815–5835 MHz, width 20MHz
                       # 1 -- radio channel @2412 Mhz,
                       # see https://en.wikipedia.org/wiki/List_of_WLAN_channels for reference
     wifi_region = 'BO'     # Your country for CRDA (use BO or GY if you want max tx power)

     [gs_mavlink]
     peer = 'connect://127.0.0.1:14550'  # outgoing connection
     # peer = 'listen://0.0.0.0:14550'   # incoming connection

     [gs_video]
     peer = 'connect://127.0.0.1:5600'  # outgoing connection for
                                   # video sink (QGroundControl on GS)

     ```
5. Настройка конфига для Воздуха
     ```
     [drone_mavlink]
     # use autopilot connected to /dev/ttyUSB0 at 115200 baud:
     # peer = 'serial:ttyUSB0:115200'

     # Connect to autopilot via malink-router or mavlink-proxy:
     # peer = 'listen://0.0.0.0:14550'   # incoming connection
     # peer = 'connect://127.0.0.1:14550'  # outgoing connection

     [drone_video]
     peer = 'listen://0.0.0.0:5602'  # listen for video stream (gstreamer on drone)
     ```
6. Добавьте WFB_NICS="wlx*" в /etc/default/wifibroadcast
     ```
     sudo nano /etc/default/wifibroadcast

     WFB_NICS="wlx*" <-- заменить на название карты
     ```
7. Добавьте unmanaged-devices=interface-name:wlx* в /etc/NetworkManager/NetworkManager.conf
     ```
     [keyfile]
     unmanaged-devices=interface-name:wlx*
     ```
8. Добавьте denyinterfaces wlx* в /etc/dhcpcd.conf
     ```
     denyinterfaces wlx*
     ```
9. Отключите Rkfill
     ```
     rfkill list all
     sudo rfkill unblock all

     ```
  
  WFB-ng: http://wfb-ng.org
Setup HOWTO: https://github.com/svpcom/wfb-ng/wiki/Setup-HOWTO
Community chat: (wfb-ng support) https://t.me/wfb_ng

Version: master

Quickstart (x86 laptop):
1. Run "wfb-cli gs" to monitor link state
2. Run QGroundControl

Quickstart (SBC + RTP video):
1. Run "wfb-cli gs" to monitor link state
2. Edit /etc/wifibroadcast.cfg and in section [gs_video] set peer to ip address of your laptop with QGC
3. Edit /etc/wifibroadcast.cfg and in section [gs_mavlink] set peer to ip address of your laptop with QGC
4. Reboot SBC.
5. Run QGroundControl on your laptop

Quickstart (SBC + RTSP video):
1. Run "wfb-cli gs" to monitor link state
2. Run "sudo systemctl enable rtsp@h264" or "sudo systemctl enable rtsp@h265" (according to your video codec)
3. Edit /etc/wifibroadcast.cfg and in section [gs_mavlink] set peer to ip address of your laptop with QGC
4. Reboot SBC.
5. Run QGroundControl on your laptop. Set video QGC source to rtsp://x.x.x.x:8554/wfb , where x.x.x.x is GS IP address.
6. (optional) Run any other RTSP video player(s) for rtsp://x.x.x.x:8554/wfb

To set TX power edit /etc/modprobe.d/wfb.conf and reboot.

In case of any failures check "sudo systemctl status wifibroadcast@gs" service status.
See full logs via: "sudo journalctl -xu wifibroadcast@gs"

--------------------------------------------------------------------------------
GS setup successfully finished

  ```


- Done! To monitor the link use the following command on the ground station:
```
wfb-cli gs
```
```

**Проблемы с подключением?**

1. Проверьте логи `sudo journalctl -xu wifibroadcast@gs`. 
2. Если у вас проблемы с расшифровкой пакетов, то перегенерируйте ключи через wfb_keygen
2. Make sure the WiFi channel and link domain on the ground and on the drone are the same. To check, see `/etc/wifibroadcast.cfg` and ensure that `[common] wifi_channel` and `[drone] link_domain` / `[gs] link domain`  is the same on the ground and on the drone.

---



## Theory
WFB-ng puts the wifi cards into monitor mode. This mode allows to send and receive arbitrary packets without association and waiting for ACK packets.
- [WFB-ng design and protocol description](https://github.com/svpcom/wfb-ng/blob/master/doc/wfb-ng-std-draft.md)
- [Analysis of Injection Capabilities and Media Access of IEEE 802.11 Hardware in Monitor Mode](https://github.com/svpcom/wfb-ng/blob/master/doc/Analysis%20of%20Injection%20Capabilities%20and%20Media%20Access%20of%20IEEE%20802.11%20Hardware%20in%20Monitor%20Mode.pdf)
- [802.11 timings](https://github.com/ewa/802.11-data)

## Пример:
```
Camera --[RTP stream (UDP)]--> wfb_ng --//--[ RADIO ]--//--> wfb_ng --[RTP stream (UDP)]--> gstreamer --> Display
```



- Воздух(была использована камера MJPEG:
 ```
 gst-launch-1.0 v4l2src device=/dev/video0 ! 	image/jpeg,width=1280,height=720,framerate=30/1 ! 	jpegparse ! jpegdec ! 	videoconvert ! 	                        x264enc                 tune=zerolatency speed-preset=ultrafast bitrate=5000 key-int-max=15 ! 	h264parse ! 	rtph264pay mtu=1400 ! 	                udpsink host=127.0.0.1 port=5602                         sync=false
 ```

- Земля декодирование:
 ```
 gst-launch-1.0 udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264'            	      !                 rtph264depay ! avdec_h264 ! clockoverlay valignment=bottom ! autovideosink fps-update-interval=1000 sync=false
 ```


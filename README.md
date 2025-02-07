![WFB-ng](doc/images.jpeg)

## Wiki:
See https://github.com/svpcom/wfb-ng/wiki for additional info

## Community support:
Telegram group: (**wfb-ng support**) https://t.me/wfb_ng
Please note, that it is only one official group.


## Getting Started
/
For detailed instructions on how to get started read through
[PX4-Guide](https://docs.px4.io/main/en/companion_computer/video_streaming_wfb_ng_wifi.html)
and follow the [Setup HowTo](https://github.com/svpcom/wfb-ng/wiki/Setup-HOWTO)




- Команды для вывода Статистики 
```
wfb-cli gs
wfb-cli drone

sudo journalctl -xu wifibroadcast@gs
sudo journalctl -xu wifibroadcast@drone
```

### Установка AU
```
git clone -b v5.2.20 https://github.com/svpcom/rtl8812au.git
cd rtl8812au
make
sudo ./dkms-install.sh
sudo depmod -a
sudo modprobe 88XXau_wfb
sudo reboot

ethtool -i wlx*
```

### Установка GS

- Install patched `RTL8812AU` or `RTL8812EU` driver:
```
sudo apt-get install dkms
# For 8812au:
git clone -b v5.2.20 https://github.com/svpcom/rtl8812au.git
cd rtl8812au/
# For 8812eu:
git clone -b v5.15.0.1 https://github.com/svpcom/rtl8812eu.git
cd rtl8812eu/
# For both:
sudo ./dkms-install.sh
```
- Get the name of the WiFi card by running:
```
ifconfig
```
- You should see output similar to:
```
wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 2312
        ether 0c:91:60:0a:5a:8b  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
- Run `$ ethtool -i wlan0` and ensure that it show right driver: `rtl88xxau_wfb` or `rtl8812eu`
- Download and run [install_gs.sh](https://raw.githubusercontent.com/svpcom/wfb-ng/refs/heads/master/scripts/install_gs.sh):
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


**Failing to get connection?**

1. Check WFB-ng GS logs with `sudo journalctl -xu wifibroadcast@gs`. If there is any errors then try to resolve it.
2. If there are any encryption errors then ensure that `drone.key` and `gs.key` on drone and gs corresponds each other.
2. Make sure the WiFi channel and link domain on the ground and on the drone are the same. To check, see `/etc/wifibroadcast.cfg` and ensure that `[common] wifi_channel` and `[drone] link_domain` / `[gs] link domain`  is the same on the ground and on the drone.

---


## FAQ
**Q: What type of data can be transmitted using WFB-ng?**

**A:** Any UDP with packet size <= 3993. For example x264/265 inside of RTP, Mavlink or generic IPv4 via tunnel.

**Q: What are transmission guarantees?**

**A:** WFB-ng uses FEC (forward error correction) which can recover 4 lost packets from 12 packets block with default settings. You can tune it to fit your needs.

**Q: Is only Raspberry PI supported?**

**A:** WFB-ng is not tied to any GPU - it operates with UDP packets. But to get RTP stream you need a video encoder (which encodes raw data from camera to x264 stream). In my case RPI is only used for video encoding (because RPI Zero is too slow to do anything else) and all other tasks (including WFB-ng) are done by other board (NanoPI NEO2).

**Q: What is a difference from original wifibroadcast?**

**A:** Original version of wifibroadcast uses a byte-stream as input and splits it to packets of fixed size (1024 by default). If radio packets were lost and this is not corrected by FEC you'll get a hole at random (unexpected) place of stream. This is especially bad if data protocol is not resistant to (was not desired for) such random erasures. So i've rewritten it to use UDP as data source and pack one source UDP packet into one radio packet. Radio packets now have variable size depending on payload size. This reduces video latency a lot.

## Theory
WFB-ng puts the wifi cards into monitor mode. This mode allows to send and receive arbitrary packets without association and waiting for ACK packets.
- [WFB-ng design and protocol description](https://github.com/svpcom/wfb-ng/blob/master/doc/wfb-ng-std-draft.md)
- [Analysis of Injection Capabilities and Media Access of IEEE 802.11 Hardware in Monitor Mode](https://github.com/svpcom/wfb-ng/blob/master/doc/Analysis%20of%20Injection%20Capabilities%20and%20Media%20Access%20of%20IEEE%20802.11%20Hardware%20in%20Monitor%20Mode.pdf)
- [802.11 timings](https://github.com/ewa/802.11-data)

## Sample usage chain:
```
Camera --[RTP stream (UDP)]--> wfb_ng --//--[ RADIO ]--//--> wfb_ng --[RTP stream (UDP)]--> gstreamer --> Display
```

- For encoding video from OpenIPC-based security camera connected via ethernet to your PC or SBC you don't need a special pipeline. Just set `outgoing` to `udp://your_ip_address:5602` in [majestic.yaml](https://github.com/OpenIPC/wiki/blob/master/en/majestic-streamer.md#other-outgoing-options)

- For encoding video from a Raspberry Pi Camera (obsolete, latency is high comparing to OpenIPC):
 ```
 raspivid -n  -ex fixedfps -w 960 -h 540 -b 4000000 -fps 30 -vf -hf -t 0 -o - | \
                gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=35 ! udpsink sync=false host=127.0.0.1 port=5602
 ```

- For encoding video from Logitech C920 camera (obsolete, camera is EOL many years ago):
 ```
 gst-launch-1.0 uvch264src device=/dev/video0 initial-bitrate=6000000 average-bitrate=6000000 iframe-period=1000 name=src auto-start=true \
                src.vidsrc ! queue ! video/x-h264,width=1920,height=1080,framerate=30/1 ! h264parse ! rtph264pay ! udpsink host=localhost port=5602
 ```

- To decode:
 ```
 gst-launch-1.0 udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' \
                ! rtph264depay ! avdec_h264 ! clockoverlay valignment=bottom ! autovideosink fps-update-interval=1000 sync=false
 ```

## HOWTO build:

For development (inline build)
```
make
```

For binary distribution RHEL or Fedora
```
make rpm
```

For binary distribution Debian or Ubuntu
```
sudo apt install python3-all libpcap-dev libsodium-dev python3-pip python3-pyroute2 \
            python3-future python3-twisted python3-serial python3-all-dev iw virtualenv \
            debhelper dh-python build-essential -y
sudo make deb
```

For binary distribution (tar.gz)
```
make bdist
```

You need to generate encryption keys for gs(ground station) and drone:
```
wfb_keygen
```
Leave them in place for development build or copy to `/etc` for binary install.
Put `drone.key` to drone and `gs.key` to gs.

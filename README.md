# radioshark-v1-rpi
RadioShark v1 on the Raspberry Pi 3

Following are my notes on getting a v1.0 RadioShark and a 808 Bluetooth speaker working on a Raspberry Pi 3, running 2017-11-19-raspbian-stretch-lite.  Installation of the OS is considered outside of the scope of these notes.  Following steps are performed as root, unless otherwise noted.

## Installing libhid

### Steps

1) Find a copy of the older libhid.  I'd searched Google and found a copy at:
http://sources.openelec.tv/mirror/libhid/

2) Download the libhid code
```c
wget http://sources.openelec.tv/mirror/libhid/libhid-0.2.16.tar.gz
```

3) Untar the libhid tarball and cd into it
```c
tar xvfz libhid0.2.16.tar.gz
cd libhid-0.2.16
```

4) edit test/lshid.c and comment out line 41 so that it looks like:
```c
//len = *((unsigned long*)custom);
```

5) Immediately after the above line, add:
```c
len=len;
custom=custom
```
The above is needed because the compiler see that those two variables are declared but never used.

6) Build and install the binary via:
```c
make
make install
```

7) Cd back to home and get the shark code:
```c
cd
wget http://www.productivity.org/projects/shark/download/shark-1.0.tar
```

8) Untar the shark tarball and cd into it:
```c
tar xvf shark-1.0.tar
cd shark-1.0
```

9) Using your favorite editor, edit the Makefile and cause the line after the "shark:" tag to look like:
```c
shark:
        ${CC} -L/usr/local/lib/ -lusb -lhid -o shark shark.c
```

10) Build and install the shark binary by running:
```c
make
make install
```

11) Plug in your RadioShark and run the following:
```c
/usr/local/bin/shark -red 1
/usr/local/bin/shark -red 0
```
The above should cause the red LEDs in the RadioShark to come on and go off.

## Connecting the 808 Bluetooth speaker to the Raspberry Pi 3

### Steps

1) Install various packages via:
```c
apt-get install pulseaudio pulseaudio-module-bluetooth bluez bluez-firmware mplayer
```

2) Add root to the pulse permissions
```c
adduser root pulse-access
```

3) Create /etc/dbus-1/systemd/pulseaudio-bluetooth.conf so that it contains:
```c
<busconfig>
  <policy user="pulse">
    <allow send_destination="org.bluez"/>
  </policy>
</busconfig>
```

4) Append the following to the end of /etc/pulse/system.pa
```c
#
# Bluetooth support #
.ifexists module-bluetooth-discover.so
  load-module module-bluetooth-discover
.endif
#
```

5) Create /etc/systemd/system/pulseaudio.service so that it contains:
```c
[Unit]
Description=Pulse Audio

[Service]
Type=simple
ExecStart=/usr/bin/pulseaudio --system --disallow-exit --disable-shm --exit-idle-time=-1

[Install]
WantedBy=multi-user.target
```

6) Run the following to cause PulseAudion to start at boot:
```c
systemctl daemon-reload
systemctl enable pulseaudio.service
```

7) Restart Bluetooth via:
```c
systemctl restart bluetooth
```
Note: you can check to see if it's running via:
```c
systemctl status bluetooth
```

8) Pair up the speaker by running:
```c
bluetoothctl
power on
agent on
default-agent
scan on
```

Note: Pause at this point, until the MAC address for "[808]" shows up, then run the following:
```c
pair MAC_ADDRESS
trust MAC_ADDRESS
connect MAC_ADDRESS
scan off
exit
```

9) Reboot your system

10) Run the following
```c
pactl list short|grep bluez
```
The output should look something like what's starting on line 201 at: https://github.com/davidedg/NAS-mod-config/blob/master/bt-sound/bt-sound-Bluez5_PulseAudio5.txt

11) Test the config by running:
```c
mplayer -ao pulse a.mp3
```

12) Run the following:
```c
pactl list |more
```
Make note of the index number for the source and sink that you want to use (for me it was source=0 and sink=1

13) set the proper source and sink
```c
pactl load-module module-loopback source=0 sink=1
```

14) Tune the RadioShark to a local station by running:
```c
shark -fm 98.7
```

15) adjust the volume (on the sink and source) to suit, by running:
```c
pactl -- set-sink-volume 1 50%
pactl -- set-sink-volume 1 -5%
pactl -- set-source-volume 0 80%
```
Get the current volume by running:
```c
pactl list sinks|perl -000ne 'if(/#1/){/(Volume:.*)/;print "$1\n"}'
```

## Troubleshooting

1) You may need to add the following to /etc/pulse/daemon.conf:
```c
resample-method = trivial
```
## Sources

* The libhid repair: https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=21932
* The older libhid code: http://sources.openelec.tv/mirror/libhid/
* The shark code: http://www.productivity.org/projects/shark/
* Richard MacCutchan's hint for fixing the shark.c Makefile: https://www.codeproject.com/Questions/529381/GetplusUSBplusdeviceplusinformationplususingplusli
* hints for setting up Bluetooth: https://github.com/davidedg/NAS-mod-config/blob/master/bt-sound/bt-sound-Bluez5_PulseAudio5.txt
* hints for determine which sink and source: https://www.raspberrypi.org/forums/viewtopic.php?f=91&t=88418

## Other points of interest

* Code for RadioShark v2: http://hoop.euqset.org/archives/2015_11.html
* variant with DEBUG: https://github.com/cmhdave/shark/

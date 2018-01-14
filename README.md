# radioshark-v1-rpi
RadioShark v1 on the Raspberry Pi 3

Following are my notes on getting a v1.0 RadioShark and a 808 Bluetooth speaker working on a Raspberry Pi 3, running 2017-11-19-raspbian-stretch-lite.  Installation of the OS is considered outside of the scope of these notes.  Following steps are performed as root, unless otherwise noted.

## Steps

1) Install various packages via:

    apg-get install pulseaudio pulseaudio-module-bluetooth bluez bluez-firmware mplayer

2) Add root to the pulse permissions

    adduser root pulse-access

3) Create /etc/dbus-1/systemd/pulseaudio-bluetooth.conf so that it contains:

    <busconfig>

      <policy user="pulse">
        <allow send_destination="org.bluez"/>
      </policy>

    </busconfig>

4) Append the following to the end of /etc/pulse/system.pa

    #
    # Bluetooth support #
    .ifexists module-bluetooth-discover.so
    load-module module-bluetooth-discover
    .endif
    #

5) Create /etc/systemd/system/pulseaudio.service so that it contains:

    [Unit]
    Description=Pulse Audio
    
    [Service]
    Type=simple
    ExecStart=/usr/bin/pulseaudio --system --disallow-exit --disable-shm --exit-idle-time=-1

    [Install]
    WantedBy=multi-user.target

6) Run the following to cause PulseAudion to start at boot:

    systemctl daemon-reload
    systemctl enable pulseaudio.service

7) Restart Bluetooth via:

    systemctl restart bluetooth

Note: you can check to see if it's running via:

    systemctl status bluetooth

8) Pair up the speaker by running:

    bluetoothctl
    power on
    agent on
    default-agent
    scan on

Note: Pause at this point, until the MAC address for "[808]" shows up, then run the following:

    pair MAC_ADDRESS
    trust MAC_ADDRESS
    connect MAC_ADDRESS
    scan off
    exit

9) Reboot your system

10) Run the following

    pactl list short|grep bluez

The output should look something like what's starting on line 201 at: https://github.com/davidedg/NAS-mod-config/blob/master/bt-sound/bt-sound-Bluez5_PulseAudio5.txt

11) Test the config by running:

    mplayer -ao pulse a.mp3

12) Run the following:

    pactl list |more

Make note of the index number for the source and sink that you want to use (for me it was source=0 and sink=1

13) set the proper source and sink

    pactl load-module module-loopback source=0 sink=1

14) Tune the RadioShark to a local station by running:

    shark -fm 98.7

15) adjust the volume (on the sink and source) to suit, by running:

    pactl -- set-sink-volume 1 50%
    pactl -- set-sink-volume 1 -5%
    pactl -- set-source-volume 0 80%

Get the current volume by running:

    pactl list sinks|perl -000ne 'if(/#1/){/(Volume:.*)/;print "$1\n"}'

## Troubleshooting

1) You may need to add the following to /etc/pulse/daemon.conf:

    resample-method = trivial


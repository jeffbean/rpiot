# rpiot
Repo of writing a RaspberryPi Security Camera


## Youtube Live streaming service
This you can just use the youtube DVR settings for viewing into the past. This is a basic streaming setup that can be useful to just have monitoring of different cameras.


We can make a FIFO socket to keep the streaming video off disk, the pi can not perform this from disk so we use Linux FIFO to pipe from the raspivid output to the ffmpeg streaming.


### Service files for systemd
We are using socket activation with the FIFO socket from systemd. So anything that writes into /run/youtube will be streamed into youtube based on your configuration.


[/etc/systemd/system/youtube.socket](rpi/youtube-stream-service/etc/systemd/system/youtube.socket)
```
[Unit]
Description=The fifo pipe for streaming youtube

[Socket]
ListenFIFO=/run/youtube

[Install]
WantedBy=sockets.target
```

[/etc/systemd/system/youtube.service](rpi/youtube-stream-service/etc/systemd/system/youtube.service)
```
[Unit]
Description=FFMpeg Service stream to Youtube
Requires=youtube.socket

[Service]
Type=simple
ExecStart=/usr/local/bin/ffmpeg ... -i - ...
StandardInput=socket
StandardOutput=journal
StandardError=journal
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
```

The last part on this is to setup another service that writes to this activation socket.

[/etc/systemd/system/raspivid.service](rpi/youtube-stream-service/etc/systemd/system/raspivid.service)
```
[Unit]
Description=RPI Service to pump video to youtube socket

[Service]
Type=simple
ExecStart=/usr/bin/raspivid -o /run/youtube -t 0 -w 1280 -h 720 -fps 30 -b 5000000
StandardError=journal
TimeoutStopSec=5
Restart=always
RestartSec=120

[Install]
WantedBy=multi-user.target
```

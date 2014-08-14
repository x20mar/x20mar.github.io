---
title: "Broadcasting with a Raspberry Pi"
layout: "post"
---

So one of the challenges I came up with working in the charity-based radio station is that they wanted to broadcast some stuff from another room back to the studio. Currently they worked around this by having a laptop with skype in the room connected to a call with another computer with skype back in the studio, which worked for them but they wanted a more stable way of doing this.

So I piped up and said, "Oh we can get a Raspberry Pi to do this".

In truth I had no idea if a Raspberry Pi could this, I just assumed that it could. After some googling I came across two articles explaining how to do this;

* [t3node.com](http://www.t3node.com/blog/live-streaming-mp3-audio-with-darkice-and-icecast2-on-raspberry-pi/)
* [glyman3home](https://sites.google.com/site/glyman3home/raspi-streaming-to-broadcastify)

The glyman3home article does reference the t3node article but I felt it missed some logic in the steps provided, plus I wanted a stream to be used internally. So here I am, article one, Broadcasting with a Raspberry Pi

### For this you will need the following

* A Raspberry Pi (Model B), with case, a 4GB+ SD card, power supply, etc
* A USB Sound Card (for my work I choose a Creative Soundblaster Play Sound Card, other usb sound cards can work but I wanted one that would just "plug and play" with the Raspberry Pi)
* A microphone/headset

### Preconfigs

Download and install Raspbian image for the Raspberry Pi. I used the NOOBS image to do this, which can be found [here](http://www.raspberrypi.org/downloads/). If you are using a 4GB SD card use NOOBS lite and perform a network install.

Once that is up and running, we want to install an extra repo and do updates based upon it, it might be a good idea to install your favourite text editor too, in my case vim.

```shell
$ sudo sh -c "echo 'deb-src http://mirrordirector.raspbian.org/raspbian/ wheezy main contrib non-free rpi' >> /etc/apt/sources.list"
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install vim
```

If you have not already plugged in the sound card, plug it into the Raspberry Pi and reboot it, so that it is loaded in the config

```shell
$ sudo apt-get reboot
```

### Configure the Sound Card

Okay so now we need to make sure that the sound card and the Raspberry Pi can talk to each other, to do this we can use arecord to tell us

```shell
$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 1: U0x46d0x825 [USB Device 0x46d:0x825], device 1: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

arecord will then provide a list available sound cards each with a device number (we'll need that later). In truth thou there should only be one device listed.

Plugin your microphone into the sound card and connect speakers to the Raspberry Pi audio (not the sound card audio out), we're now going to test if it works. Once you are ready run this command and talk into the microphone, it creates a file called temp.wav

```shell
$ arecord -D plughw:1,0 temp.wav #Replace 1 with your device number
```

Once that is up and running, we want to install an extra repo and do updates based upon it, it might be a good idea to install your favourite text editor too, in my case vim.

```shell
$ sudo sh -c "echo 'deb-src http://mirrordirector.raspbian.org/raspbian/ wheezy main contrib non-free rpi' >> /etc/apt/sources.list"
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install vim
```

If you have not already plugged in the sound card, plug it into the Raspberry Pi and reboot it, so that it is loaded in the config

```shell
$ sudo apt-get reboot
```

Once rebooted, repeat the steps in the section to test if recording works, n.b. your device number may change

If it is too soft or is distorted, try using the following command to access the tool to balance the recording level.

```shell
$ sudo alsamixer
```

Once you are happy with the sound quality run the following command to save the settings

```shell
$ sudo alsactl store
```

### DarkIce

Both articles mentions compiling darkice to be used with mp3 support and all that. Rather than installing lots of packages to do this, I've done this for you, as you get get the deb package from my github account. 

So download the darkice deb package and install

```shell
$ wget https://github.com/x20mar/darkice-with-mp3-for-raspberry-pi/blob/master/darkice_1.0.1-999~mp3+1_armhf.deb?raw=true #note link may change!
$ mv darkice_1.0.1-999~mp3+1_armhf.deb?raw=true darkice_1.0.1-999~mp3+1_armhf.deb
$ sudo apt-get install libmp3lame0 libtwolame0 
$ sudo dpkg -i darkice_1.0.1-999~mp3+1_armhf.deb
```

After all that we now need to configure darkice for our needs. Fortunately darkice comes prepackaged with a template config that we can use

```shell
$ sudo cp /usr/share/doc/darkice/examples/darkice.cfg /etc/
```

Now before we start configuring, darkice will need to connect to an icecast2 server, and you will need to know the configuration details of it. if you don't have one then you can install it on your Raspberry Pi

```shell
$ sudo apt-get install icecast2
```

Now using your favourite text editor, edit the darkice config file to use the right sound card and talk to icecast2. You will need to change device to be plughw:1,0 (change to reflect your device number)

```shell
$ sudo vim sudo vim /etc/darkice.cfg
```

I should point out that this is based upon a sample config and it is not optimised or secure. Look at the man page for more information

```shell
$ man darkice.cfg
```

I wasn't too fussed about security as it was being used internally so in the end my config looked like this

{% highlight bash %}
# this section describes general aspects of the live streaming session
[general]
duration        = 0        # duration of encoding, in seconds. 0 means forever
bufferSecs      = 5         # size of internal slip buffer, in seconds
reconnect       = yes       # reconnect to the server(s) if disconnected

# this section describes the audio input that will be streamed
[input]
# device          = /dev/dsp  # OSS DSP soundcard device for the audio input
device          = plughw:0,0  # OSS DSP soundcard device for the audio input
sampleRate      = 22050     # sample rate in Hz. try 11025, 22050 or 44100
bitsPerSample   = 16        # bits per sample. try 16
channel         = 2         # channels. 1 = mono, 2 = stereo

# this section describes a streaming connection to an IceCast2 server
# there may be up to 8 of these sections, named [icecast2-0] ... [icecast2-7]
# these can be mixed with [icecast-x] and [shoutcast-x] sections
[icecast2-0]
bitrateMode     = abr       # average bit rate
format          = mp3       # format of the stream: ogg vorbis
bitrate         = 96        # bitrate of the stream sent to the server
server          = localhost # host name of the server
port            = 8000      # port of the IceCast2 server, usually 8000
password        = lolcat123 # source password to the IceCast2 server
mountPoint      = mic  # mount point of this stream on the IceCast2 server
name            = Microphone Raspberry Pi # name of the stream
description     = Broadcast from 2nd room # description of the stream
url             = http://example.com/ # URL related to the stream
genre           = my own    # genre of the stream
public          = yes       # advertise this stream?
{% endhighlight %}

Once you have done that it's time to test your configuration file

```shell
$ sudo darkice
```

If you get any errors, google them and fix them and then try again. After that, connect to the icecast2 server via a web browser on another computer and test if the stream works.
Boot on start

Now that we have darkice working and everything we now need to make it automaticly when the Raspberry Pi is booted. First step we an init script for darkice. One can be found back in the github repo. So get the file and make it executable.

```shell
$ wget https://raw.githubusercontent.com/x20mar/darkice-with-mp3-for-raspberry-pi/master/init.d-darkice
$ sudo mv init.d-darkice /etc/init.d/darkice
$ sudo chmod +x /etc/init.d/darkice
```

Now darkice can be started by calling

```shell
$ sudo /etc/init.d/darkice start
```

and stopped

```shell
$ sudo /etc/init.d/darkice stop
```

Finally run this command to add the init to be started at boot-up

```shell
$ sudo update-rc.d darkice defaults
```

Done

And we're finished. A big thanks again to t3node and glyman3home for their articles, I hope you find it useful.

## Enabling 8000, 11025, 16000, 22050, 32000, 88200, 96000, 176400, and 192000 Hz sample rates on _ALL_ C-MEDIA 8770 sound cards ##

An awesome little work-around that I discovered after reverse-engineering the Windows drivers for the C-Media 8770... Currently, the HT Omega Striker 7.1 and all other 8770 chipset based sound cards only support 16-bit/44100 and 16-bit/48000 in Windows 7 and Vista. If you need or want 16/192000 or 16/96000 (or many others) then here's how!

### Developer's notes ###

(1) This is a Windows 7 hack and will not help Linux users achieve better (higher) sample rates.

(2) This currently breaks surround sound playback. True surround sound and 2-channel to 5.1 or 7.1 does not work. I am hoping to fix this issue, or get the manufacturer to do so. It would be great to have this feature added to the official drivers.

### HOWTO ###

First, make sure you completely uninstall the driver for your sound card:

Control Panel > Uninstall a program > C-Media PCI Audio Device or HT Omega Striker 7.1 (Don't reboot)

Control Panel > Hardware and Sound > Device Manager > Right click on sound card > Uninstall > Check box that says "Delete the driver software for this device"

Reboot.

Now download the latest C-Media driver. The latest version is 8.17.42 (Provided exclusively by HT Omega) however there is very little difference (if any, from what I can see) between 8.17.42 and 8.17.40.

After installation, don't reboot. Download my cmudax3.sys file here:

http://neo-technical.googlecode.com/files/cmudax3.zip

Copy the cmudax3.sys file to C:\Windows\System32\drivers

Restart.

To select your sample rates, please do not use the Xear3D control panel as this is now bugged (for now.) Instead, right click on the Speaker icon at the bottom right, select "Playback devices" and Highlight your C-Media 8770 sound card / HTO STRIKER 7.1. Go to "Properties" > "Advanced"  and now select your sample rate. For highest quality, select "16 bit, 192000 Hz (Studio Quality)" or "16 bit, 96000 Hz (Studio Quality.)

Tip: After selecting "Playback devices" you can highlight your device, press configure, click "next" and check the box for "Full-range speakers." This can increase audio quality as well. Currently only stereo audio is supported.

Hit apply, and OK.

Enjoy all the sample rates that C-Media and HT Omega don't want you to have!
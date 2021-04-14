---
layout: default
title: NAO
---

## V6

- SBR – Documentation ([Link](http://doc.aldebaran.com/2-8/news/index.html))
- SBR – NAOqi 2.8.5 image for SPL teams (Please mail the [TC](mailto:rc-spl-tc@lists.robocup.org) for a link)
- SBR – Lola RoboCup Documentation (Please mail the [TC](mailto:rc-spl-tc@lists.robocup.org) for a copy)
- HTWK – LolaConnector ([Link](https://github.com/NaoHTWK/LolaConnector))
- B-Human – Hints ([Link](https://spl.robocup.org/wp-content/uploads/downloads/nao-v6-hints.pdf))
- RoDEO – V6 Meeting Summary 3/3/2019 ([Link](https://spl.robocup.org/wp-content/uploads/downloads/2019-03-03_v6-meeting.pdf))
- Nao Devils – OpenCL Integration ([Link](https://spl.robocup.org/wp-content/uploads/downloads/OpenCL-V6.html.pdf))
- HULKs – How to work with the camera ([Link](https://github.com/HULKs/NaoV6/wiki))

TODO: Integrate the content of the last four links here
TODO: Integrate findings from the V6 Slack

# older revisions (deprecated)

- Somewhere between NaoQi 1.10.37 and 1.14.1 the value of `Device/SubDeviceList/InertialSensor/AccY/Sensor/Value` had the sign flipped (this is the Accelerometer in the y-dimension). Multiply it by -1 to get the old values. (Contributed by RoboEireann)
- Upgrading to 1.14.1 from 1.10.37, the raw gyroscope values `Device/SubDeviceList/InertialSensor/GyrX/Sensor/Value` and `Device/SubDeviceList/InertialSensor/GyrY/Sensor/Value` changed in both range and scale. RoboEireann empirically found scaling these raw values by 0.7 brought them back to the old sort of values they had previously seen.
- The V3 cameras have a few issues with them in 1.14.1. RoboEireann found that the frame buffers were still not marked as cacheable and that auto black level functions were impossible to disable. RoboEireann forked the Aldebaran kernel and patched the camera driver to fix both of these issues. The kernel is hosted at [https://github.com/mp3guy/linux-aldebaran](https://github.com/mp3guy/linux-aldebaran), with versions 1.12 and 1.14. To compile the patched driver, just checkout the source above and then: 
  - Get the kernel config off a robot (/proc/config.gz)
  - Extract it to the root of the kernel source as .config
  - Run `make ARCH=i386`
  - Make coffee
  - If you get an error about arch/x86/vdso and -m elf_i386, open arch/x86/vdso/Makefile and find the line containing VDSO_LDFLAGS and replace -m elf_i386 with -m32
  - Copy drivers/media/video/lxv4l2/lxv4l2.ko to your nao on /lib/modules/2.6.29.6-rt24-aldebaran-rt/kernel/drivers/media/video/lxv4l2/lxv4l2.ko (you'll need root for this)
  - Reboot and enjoy the image being loaded into the Geode's cache and auto black level settings being disabled by default.
- The sonars work differently from what the documentation suggests (NaoQi 1.14.0-2). First of all, in firing modes 0-3 the readings are all returned as measurements of the right sensor. In addition, only 70 ms after firing a sensor, the readings seem to be correct. Before that, they still might correspond to previous measurements. The firing modes 4-7 always fire both transmitters and read with both receivers. In modes 4 and 7, the receivers measure the pulse sent by the transmitters on the same side. In modes 5 and 6, they measure the pulse sent by the transmitters on the opposite side. They do not seem to disturb each other. They either use different frequencies or they are fired at different points in time. (Contributed by B-Human)
- A new NAO V4 kernel can be found at [https://b-human.de/downloads/KernelV4.tar.bz2](https://b-human.de/downloads/KernelV4.tar.bz2). It activates SMP and comes with different drivers for WLAN, LAN, and cameras. (Contributed by B-Human)

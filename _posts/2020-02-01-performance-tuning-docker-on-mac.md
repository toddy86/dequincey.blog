---
layout: post
title: Performance Tuning Docker on Mac
categories: resources
---

Docker on Mac can have some performance problems. The below are a collection of steps which you can take to improve the performance of Docker. 

<!-- more -->

### Disk Image 
Ensure that the disk image being used by docker is the **.raw** version.

To check:

- Click on the docker icon, then Preferences
- Click Resources
- Check the extension of the Disk Image Location is set to .raw
- If not set to .raw, reset the image

### Set Storage Driver on Daemon
Update the Docker engine by adding `"storage-driver": "overlay2"` and click Apply & Restart.

```
{
  "experimental": false,
  "debug": true,
  "storage-driver": "overlay2"
}
```

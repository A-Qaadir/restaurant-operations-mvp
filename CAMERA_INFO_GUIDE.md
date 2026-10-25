# Finding the camera/NVR details for a pilot

You can collect these details without changing the camera configuration. **Do not enable protocols, create accounts, share passwords, or copy a stream URL.** We only need make/model and whether a protocol is listed or enabled.

## Look for the recorder first

If several cameras connect to one recorder, the NVR/DVR model is the best first clue. Also note a camera make/model if it differs from the recorder; the recorder and the camera may have different manufacturers.

1. Look at the label on the recorder (usually the box near the network/router or monitor). Note the manufacturer and model number.
2. If there is a monitor connected, look under a read-only page such as **System**, **About**, or **Device information** for the same details.
3. If there is a phone app, its device list or device information page may show the model. Do not open or copy any page that displays a username, password, QR code, device ID, or stream address.

## Check protocol support without changing it

If you can safely view the existing recorder settings, look for a page named **Network**, **Integration**, **Services**, or **Platform access**. Check whether it lists **RTSP** or **ONVIF**, and whether each is shown as supported/enabled. If an ONVIF profile is named, note **Profile T**, **Profile S**, or the exact other profile shown. Do not turn a setting on or off for this check; enabling a stream protocol can change who or what can access camera video.

## What to send back

You can reply with just:

```text
Recorder brand:
Recorder model:
Recorder firmware (if visible):
Camera brand (if different):
Camera model (if different):
Camera firmware (if different):
RTSP listed/enabled: yes / no / not sure
ONVIF listed/enabled: yes / no / not sure
ONVIF profile (if shown): T / S / other / not sure
Number of cameras covering the dining area (roughly):
```

If there is no label or you cannot safely access the read-only device information, “not sure” is enough. The restaurant installer or camera vendor can identify compatibility later.

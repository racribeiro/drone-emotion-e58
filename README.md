# Emotion Drone E58 Reverse Engineering

The goal is to read the drone's telemetry and video, take photos, start and
stop recording, and issue movement commands: up, down, left, right, rotate left,
and rotate right.

## Observed Wi-Fi Link

A 30-second passive survey using the internal USB Wi-Fi adapter found an access
point whose SSID begins with `WiFi-720P-` on 2.4 GHz channel 1. The saved capture
was analyzed locally; it is not included in this repository.

The capture contained clear UDP traffic associated with that access point:

- A client sent 62 eight-byte datagrams to `192.168.4.153:8090`. They match the
  frame shape `[66, roll, pitch, throttle, yaw, command, checksum, 99]`; all
  observed checksums match the XOR of the four axis bytes. The command byte was
  zero in these frames.
- The client sent `42 76` and `42 77` to UDP port `8080`. The
  `maritaca-e88-controller` reference identifies these as enter/exit Wi-Fi
  control mode commands.
- The survey hopped across channels, so it does not establish sustained packet
  rates or cover every exchange. It did not establish the video stream or
  photo/recording commands.

This is a strong match for the `WIFI_8K_` E88/E58 protocol variant documented by
[`maritaca-e88-controller`](https://github.com/popolony2k/maritaca-e88-controller).
The target's exact firmware and command semantics still need controlled
verification. The `WiFi-720P-*` SSID also appears in the older
[`ESP_E58-Drone`](https://github.com/martin-ger/ESP_E58-Drone) project, but that
variant uses UDP port `50000`; the observed traffic here uses port `8090`.
Similar SSIDs do not guarantee the same protocol variant.

The packet capture may contain nearby network data, so it remains in the
capture service and temporary analysis storage. Do not commit raw captures,
unique device identifiers, or live control values.

## How the DJI Projects Inspected Their Link

The sibling DJI research combines static analysis of the DJI Fly APK with
offline decoding of app-visible packet captures. The capture review uses
sanitized IP/UDP traces; raw captures are kept outside Git because they can
contain Wi-Fi credentials and other private data. Its runtime integration joins
the aircraft's access point, then sends and receives UDP datagrams through a
host-owned socket. The documented Neo endpoint is `192.168.2.1:9003`, specific
to that product.

For this drone, the internal USB adapter's monitor capture exposed the target's
clear UDP payloads directly. The 30-second survey ran through the Network
Inspector API because this execution environment does not expose the radio as a
local network interface. The target SSID suffix, BSSID, and control-axis values
are omitted from this public-facing README.

## Reference Projects

The Git submodules in [`other-projects/`](other-projects/) are:

- [`maritaca-e88-controller`](other-projects/maritaca-e88-controller/): best
  match to the observed IP and UDP ports, mode-switch packets, and eight-byte
  control frame.
- [`ESP_E58-Drone`](other-projects/ESP_E58-Drone/): E58-family reference with
  the matching SSID prefix and a related eight-byte flight packet on port
  `50000`.
- [`e58-drone-reversing`](other-projects/e58-drone-reversing/): E58 video
  startup and JPEG-fragment reconstruction on UDP port `8800`; that video path
  has not been confirmed on this unit.

## Current Status

- The observed flight packet format is a strong protocol match, but no flight
  command has been sent from this project.
- Direction polarity, command flags, arming behavior, and keepalive timing need
  controlled verification before movement control is implemented.
- Telemetry decoding, the target's video transport, snapshot, and recording
  commands remain unknown.
- No flight or camera commands have been transmitted by this project.

Next, capture a longer fixed-channel session while using the stock app's video,
photo, and recording controls. Keep the raw capture private and compare only
sanitized packet summaries with the observed control format.

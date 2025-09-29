## Overview

The Tenda AC15 firmware contains an arbitrary-write flaw in the `/goform/setcfm` POST endpoint.An attacker can craft `nameX`/`valueX` pairs to overwrite any configuration item on the device (e.g., change the Wi-Fi SSid) and potentially trigger memory corruption or other higher-severity consequences.

## Affected version

V15.03.05.18

## Firmware download website 

https://www.tenda.com.cn/download/detail-2710.html

## Vulnerability details

Endpoint: `POST /goform/setcfm`

Input handling: parameters named `name%d` are parsed into variable `a1`; parameters named `value%d` are parsed into variable `a2`.

Sink: `SetValue(a1, a2)` stores the key `a1` and the value `a2` directly into the device’s configuration store.

Impact: Because there is no restriction or validation on `a1`/`a2`, remote attackers can set arbitrary configuration keys and values. This permits:

- Changing critical settings (e.g., SSID, Wi-Fi password, admin credentials).
- Corrupting configuration data in ways that cause crashes or logic errors.
- Potentially exploiting memory-management issues (e.g., buffer overflows) if configuration handling elsewhere assumes bounded input — which could escalate to code execution on vulnerable firmware versions.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_091826.png)

## Exploit

### Changing SSID

1. Find the name of the `ssid` configuration item.Through continuous testing, we found that the name of the ssid configuration item is `wl2g.ssid0.ssid`.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_093301.png)

2. Use the HackBar tool to modify the `SSID` through the `/goform/setcfm` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_094144.png)

3. Successfully modified

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_100955.png)

### Stack-overflow attacks

1. Stack Overflow Endpoint: `POST /goform/getWanParameters`.The GetValue function receives the value of wl2g.public.country and stores it in the s variable, causing a stack overflow.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_101129.png)

2. Use the HackBar tool to modify the `wl2g.public.country` through the `/goform/setcfm` interface.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_101930.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20250929_102621.png)


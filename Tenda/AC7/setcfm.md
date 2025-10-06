## Summary

A remote attacker can use the `/goform/setcfm` POST endpoint on Tenda AC7 firmware **V15.03.06.44** to write arbitrary configuration keys and values. The endpoint maps `name%d` → key and `value%d` → value, then calls `SetValue(key, value)` without validation. This allows changing critical settings (SSID, Wi-Fi password, admin credentials), corrupting configuration, and — when combined with other vulnerable handlers — triggering memory-corruption (e.g., stack overflow) that could escalate to higher-impact outcomes.

## Affected product & version

- Product: Tenda AC7 (firmware published on Tenda support site)
- Affected firmware version: **V15.03.06.44**

## Vendor download page

https://www.tenda.com.cn/product/download/AC7.html

## Technical details

### Endpoint and parameter parsing

- **Endpoint:** `POST /goform/setcfm`
- **Parameter parsing:** Client-supplied POST parameters named `name0`, `value0`, `name1`, `value1`, … are parsed into `Var` (key) and `v8` (value).
- **Sink:** `SetValue(Var, v8)` — the key/value pair is written straight into the device configuration store **without validation or key whitelisting**.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_085520.png)

### Security weakness

- No server-side validation of configuration keys (`Var`) or values (`v8`) allows:
  - Arbitrary modification of configuration entries.
  - Injection of unexpected values that downstream code assumes to be bounded/structured.
  - Overwriting of sensitive entries (Wi-Fi, admin credentials).
- When combined with other endpoints or configuration consumers that copy values into fixed-size buffers (e.g., `GetValue` usage in another handler), an attacker can cause memory corruption such as stack overflow.

## Example attack scenarios

### 1) Change Wi-Fi SSID (proof-of-concept workflow)

1. Discover the internal config key for the SSID (found during testing): `wl2g.ssid0.ssid`.

2. Send a `POST /goform/setcfm` with form fields:

   - `name0=wl2g.ssid0.ssid`
   - `value0=ATTACKER_SSID`
   - `save=1`

   ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_090231.png)

3. The device accepts the key/value and the SSID is changed immediately (no authentication validation of the key).
    *Result:* Wireless network name is modified to attacker-chosen value.

### 2) Trigger stack overflow via configuration poisoning (observed chain)

1. Using `/goform/setcfm` an attacker writes an overly long string into `wl2g.public.country`.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_090455.png)

2. Some handlers (e.g., `POST /goform/getWanParameters`) call `GetValue("wl2g.public.country", buf)` and copy the value into a fixed-size stack buffer.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_090805.png)

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_090620.png)

3. A later request to the vulnerable handler copies the attacker-controlled long string into the stack buffer and causes a stack overflow.
    *Result:* Crash or potential memory-corruption exploitability depending on surrounding code.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_091337.png)

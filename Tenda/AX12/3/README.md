## Vulnerability description ##
Affect device：Tenda-AX12 V22.03.01.21_CN([https://www.tenda.com.cn/download/detail-3237.html](https://www.tenda.com.cn/download/detail-3237.html))

Vulnerability Type: Command Injection

Attack effect: execute any command

## Vulnerability cause ##
This vulnerability occurs in the /goform/setMacFilterCfg interface, and its corresponding processing function is sub_424334. Originally, this is a function with stack overflow, but I accidentally discovered that it may have command injection.

At the bottom of this function, there is this code.
![](D:\learning record\学术报告\路由器\Tenda\AX12\3\image\1.png)

It can be seen that it calls the doSystemCmd function dangerously, just to output a paragraph to the /tmp/macfilter file. After careful observation, v2 is the incoming macFilterType parameter. According to the sentence of printf, &v14[2] is not It just points to the value of the last macFilterType. If this parameter can be controlled, it may cause command injection.

This interface corresponds to the background management interface->advanced function->MAC address filtering, and the macFilterType parameter corresponds to the whitelist and blacklist, that is, write and black. Make a visit first, but I modify the value of macFilterType to test.
![](D:\learning record\学术报告\路由器\Tenda\AX12\3\image\2.png)

Immediately afterwards, continue to visit once normally, and find that the value of &v14[2] is test.
![](D:\learning record\学术报告\路由器\Tenda\AX12\3\image\3.png)

This means, command injection does exist.

## POC ##

According to this, I tried to write a script to attack, mainly restart command, restore factory settings command, and /bin/sh command, the attack effect is as follows.
![](D:\learning record\学术报告\路由器\Tenda\AX12\3\image\4.png)

![](D:\learning record\学术报告\路由器\Tenda\AX12\3\image\5.png)

Reboot command and restore factory settings both worked, but /bin/sh was not successful, but after debugging, it was found that the parameter before entering the dosystemcmd function was /bin/sh, but I don’t know why getshell failed.
![](D:\learning record\学术报告\路由器\Tenda\AX12\3\image\6.png)

In any case, there is indeed command injection in this place. Although authentication is required once, it is still more harmful.


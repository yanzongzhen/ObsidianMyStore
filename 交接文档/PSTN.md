

## 简介

> PSTN 是百家云在电话网关中做出的定制化的项目，主要用于连接SIP协议设备和RTC平台，是一套完善的中间协议转换方案

## 部署指南

部署文件项目地址： [PSTN](https://git.baijiashilian.com/cloud/BRTC/pstn)


### 1. 一键更换 IP
部署模版的公网IP为 103.234.22.174 内网IP为 10.16.30.66
调整changeip中的新的公网IP和内网IP后然后执行批量变更
```
./changeip.sh
```
### 2. drachtio 流程
#### drachtio 配置项
​	配置⽂件：drachtio.conf.xml
​		可能需要修改的配置： （这⾥的配置端⼝，不要与 freeswitch 默认端⼝冲突） 
```xml
<contacts> 
	<contact>sip:*:5062;transport=udp,tcp</contact> 
</contacts>
```
#### 启动规则
​	配置后 docker-compose.yml 文件启动 docker 容器，镜像名:drachtio/drachtio-server
### 3. freeswitch 流程
#### freeswitch 配置项
​	配置⽂件⽬录：freeswitch/conf
​	配置脚本⽬录： freeswitch/scripts
##### 配置⽂件：freeswitch/conf/vars.xml
​	1.修改 freeswitch 自带用户密码
``` xml
<X-PRE-PROCESS cmd="set" data="default_password=43123"/>
```
​	2.修改 freeswitch 默认鉴权和端口
```xml
<!-- Internal SIP Profile --> 
<X-PRE-PROCESS cmd="set" data="internal_auth_calls=false"/> 
<X-PRE-PROCESS cmd="set" data="internal_sip_port=5767"/> 
<X-PRE-PROCESS cmd="set" data="internal_tls_port=5061"/>
<X-PRE-PROCESS cmd="set" data="internal_ssl_enable=false"/>
```
注：需要运营商配置接入 ip 和端口
##### 配置⽂件：freeswitch/conf/dialplan/default.xml
1.新增拨号规则
​	expression: 联通号码正则匹配
```xml
<extension name="shoufeng"> 
	<condition field="destination_number" expression="^01086355057$">
  	<action application="lua" data="sip.lua"/> 
  </condition>
</extension>
```
##### 配置⽂件：freeswitch/conf/dialplan/public.xml
1.新增拨号规则
​	 expression="^(01086355057)|(9999)|(9999[[:punct:]]+[0-9]{6}[[:punct:]]+[0-9]{4,10})$"
```xml
<extension name="shoufeng"> 
  <condition field="destination_number" 
      expression="^(01086355057)|(9999)|(9999[[:punct:]]+[0-9]{6}[[:punct:]]+[0-9]{4,10})$">
    <action application="lua" data="sip.lua"/> 
  </condition> 
</extension>
```
注：此处的正则可匹配一键入会。输入号码和会议号可一键入会。
##### 配置⽂件:	freeswitch/conf/ivr_menus/demo_ivr.xml
1.修改 freeswich 连接 drachtio 地址
```xml
<entry action="menu-exec-app" digits="/([0-9]{6,11})/" 
       param="bridge sofia/$${domain}/$1@123.57.94.221:5062"/>
```
***备注：123.57.94.221:5062 为 drachtio 的地址和端口，按照配置 drachtio 时的配置填写***
#### 配置⽂件： freeswitch/conf/autoload_configs/acl.conf.xml
1.配置运营商⼊⼝权 
```xml
<list name="domains" default="deny">
  <!-- domain= is special it scans the domain from the directory to build the ACL --> 
  <node type="allow" domain="$${domain}"/>
  <!-- use cidr= if you wish to allow ip ranges to this domains acl. --> 
  <node type="allow" cidr="192.168.0.0/24"/> 
  <node type="allow" cidr="112.124.118.98/24"/> 
</list>
```
***备注：如果运营商有多个入口 ip，allow 需要配置多个***
##### 配置外呼 
1. 配置 gateway
freeswitch/conf/sip_profiles/external/my_gate.xml
```xml
<include>
 <gateway name="mygate">
   <param name="realm" value="xxxxxx:5066"/>
   <param name="username" value="xxxxxx"/>
   <param name="register" value="false"/>
 </gateway>
</include>
```
2. 配置拨号规则
- freeswitch/conf/dialplan/default.xml
- freeswitch/conf/dialplan/public.xml
```xml
<include>
 <extension name="mycall">
   <condition field="destination_number" expression="^000(\d+)$">
     <action application="bridge" data="sofia/gateway/mygate/$1"/>
   </condition>
 </extension>
</include>
```
### 4. vloud-sip 流程
#### 配置文件
```js
{
   https: {
    listenIp: '0.0.0.0',
    listenPort: process.env.PROTOO_LISTEN_PORT || 2443,
    tls: {
      cert: process.env.HTTPS_CERT_FULLCHAIN || `${__dirname}/certs/fullchain.pem`,
      key: process.env.HTTPS_CERT_PRIVKEY || `${__dirname}/certs/privkey.pem`
    }
  },  
  http: {
    listenIp: "0.0.0.0",
    listenPort: process.env.PROTOO_NORMAL_LISTEN_PORT || 9880,
  },
  drachtio: {
    host: "localhost", // 同机器上，drachtio的地址
    port: 9022,
    secret: "cymru"
  },
  bmcu: {
    uri: "ws://localhost:8080/?clientId=hw-adapt-test-ss", // bmcu的地址
    test:true, // 保持不变
    businessId:"mcu-business", // 保持不变
    from:"sip:03165714490@103.234.22.174:5060", // 呼出的号码， 需要对应修改
    onjoin:"",
    vt:"http://brtc-apitest.baijiayun.com/vrm/api/auth/token/vt", // Paas 的vt地址
    appid:"YG1O5y61cBcG0DNPRvvCXPBPVy8Gfd8e", // Paas 的appid
    key:"fUdZvJPxnCN40IFDOhj93wA1vZy0mDec",  // Paas 的key
    // proxy can be empty
    proxy: "",
    waitUpdate :50, // 保持不变
    waitWS :500,    // 保持不变
    streamTypes: 2  // 0: audio only, 1: video only, 2: audio and video
  },
  apiConfig: {
    checkPaaSInfo: "http://brtc-apitest.baijiayun.com/vrs/api/pstnInternal/parseShortCode",  // Paas 的短码解析地址
    checkSaaSInfo: "https://bloud-k8s-saasapipd.boom.cn:18143/v2/sip-device/phone/check-room-join", // SaaS 的房间解析地址
    saasToken: "",
    paasToken: "2a638867a2a838d5e68a12d9092bbbf2", // Paas 的token https://ewiki.baijiashilian.com/BRTC/PSTN%E7%94%B5%E8%AF%9D%E5%85%A5%E4%BC%9A/%E7%9F%AD%E7%A0%81%E8%BD%AC%E6%8D%A2.md
    userauth: "",
    // mock 的数据
    checkTable: {
     
    },
    rewriteTable: {
    },
    skipSaas: [
    ]
  }
}
```
### 5. 启动
```
./batch-shell start
```



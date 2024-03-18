## 简介

> Blicense 是BRTC平台在私有化的时候部署的权限认证服务， 主要用于对授权用户进行约束，授权时间和授权的方数进行限制。


![](image/Blicense.excalidraw.png)
## 部署


### 统一签发服务端部署


```shell

./blicense server -p 8080

```

自带的签发页面

![](image/blicense-sign.png)

### 客户授权代理


```shell

./blicense proxy -p 8090

```

客户侧使用证书的页面

![](image/Pasted image 20240314164803.png)

## 集成


```go

type validLicense func(api string, count int64) (int, string)  
  
var validLicenseFunc validLicense  
  
func CheckLicense() (int, string) {  
    if validLicenseFunc == nil {  
       p, err := plugin.Open("sdk.so")  
       if err != nil {  
          return -1, "load plugin license error"  
       }  
       sim, err := p.Lookup("Valid")  
       if err != nil {  
          return -1, "lookup plugin license method error"  
       }  
       validLicenseFunc = sim.(validLicense)  
    }  
    // 获取所有房间  
    userLen, err := GetPlatformUserNum()  
    if err != nil {  
       return -1, "get platform user num error"  
    }  
    return validLicenseFunc(config.GetConfig().Common.LicenseServer, userLen)  
}


```


### 集成编译镜像

```shell
cd msg

./licensebuild.sh -a=x86 -t=image-tag

./licensebuild.sh -a=x86 -t=image-tag -p

```

```shell

./licensebuild.sh -a=arm -t=image-tag

./licensebuild.sh -a=arm -t=image-tag -p

```
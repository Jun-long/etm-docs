# 常见的DApp开发问题  

目录
=================

<!-- TOC -->

- [常见的DApp开发问题](#常见的dapp开发问题)
  - [概述](#概述)
  - [**DApp 无法启动**](#dapp-无法启动)
    - [**config.json文件里边的peers不正确**](#incorrect-peers-in-configjson)
    - [**DApp 的文件目录结构不正确**](#dapp-needs-right-folder-structure)
    - [**DApp 是否已经正确安装并启动**](#is-dapp-correctly-registered-and-installed)
    - [**目录 asch/public/dist/chains 没有创建**](#directory-aschpublicdistchains-must-exist)
    - [**Dapp model 里边的字符串类型的数据没有写长度**](#missing-length-property-in-conjunction-with-string-type-of-new-dapp-model)
  - [**DApp 不出快**](#dapp-is-not-producing-blocks)
    - [** config.json文件里边的代理数量不足**](#not-enough-dapp-delegates-in-configjson)
  - [**交易失败 **](#transaction-failed)
    - [**依赖的交易验证失败**](#depending-transaction-not-verified)
    - [**发送交易到了错误的API地址**](#sending-transaction-to-wrong-api-endpoint)
  - [**无法注册自定义的合约**](#can-not-register-custom-dapp-contracts)
    - [**C合约码必须不小于 1000**](#contract-number-must-be-above-1000)
    - [**合约没有指向正确的文件和函数**](#contract-is-not-pointing-to-the-right-file-and-function)
  - [**请求报 404错误**](#request-failed-with-status-code-404)
    - [**检查请求URL里边的 Dapp-Name是否正确**](#check-correct-dapp-name-in-request-url)
  - [**在ETM目录运行 npm install 失败**](#can-not-run-npm-install-in-asch-directory)
    - [**父目录的命名包含空格**](#parent-directory-has-space-in-its-name)
  - [**请求发送到侧链失败**](#can-not-reach-asch-blockchain)
    - [**检查端口和配置**](#check-port-and-configuration)

<!-- /TOC -->


<br/><br/>

## 概述
在这篇文档里，我们讲讲述常见的 DApp 开发问题和它们的解决方案. 


Dapp 安装在哪个目录？ 
你的DApp 安装在 `ETM_HOME/dapps/your-dapp-name` 目录 your-dapp-name 是你注册Dapp 的时候生成的. 这个目录页是logs 文件存放的地方 (`logs/`).

查看所有注册的DApps: `curl http://localhost:4096/api/dapps`  
所有安装的DApps: `curl http://localhost:4096/api/dapps/installed`  

日志文件 
如果你希望查看dapp 的日志的话，cd 到 `ETM_HOME/dapps/your-dapp-name/logs/debug.log`. 如果希望查看dapp主链的日志，cd 到 `ETM_HOME/logs/`,在里边可以找到日志文件.

__文档通用规定__  
下面所说的文件，是指在`ETM_HOME/dapps/your-dapp-name` 下边的文件 , 确保你修改了正确的文件，并且修改后，重启的dapp侧链

>注:  
`config.json` = `ETM_HOME/dapps/your-dapp-name/config.json`


<br/><br/>

## **DAPP启动失败**  

### **config.json里边的peers配置不正确**

 `config.json` 文件里边的peers 配置，要么不配置，要么至少提供一个peer配置，不要整一个空的数组:

移除 `peers` 属性:  
```diff
- {
-   "peers": [],
-   "secrets": [
-     "flame bottom dragon rely endorse garage supply urge turtle team demand put",
-     "thrive veteran child enforce puzzle buzz valley crew genuine basket start top",
-     "black tool gift useless bring nothing huge vendor asset mix chimney weird",
-   ]
- }
+ {
+   "secrets": [
+     "flame bottom dragon rely endorse garage supply urge turtle team demand put",
+     "thrive veteran child enforce puzzle buzz valley crew genuine basket start top",
+     "black tool gift useless bring nothing huge vendor asset mix chimney weird",
+   ]
+ }
```

或者提供至少一个peer:  
```diff
- {
-   "peers": [],
-   "secrets": [
-     "flame bottom dragon rely endorse garage supply urge turtle team demand put",
-     "thrive veteran child enforce puzzle buzz valley crew genuine basket start top",
-     "black tool gift useless bring nothing huge vendor asset mix chimney weird",
-   ]
- }
+ {
+   "peers": [{"ip":"127.0.0.1","port":4096}],
+   "secrets": [
+     "flame bottom dragon rely endorse garage supply urge turtle team demand put",
+     "thrive veteran child enforce puzzle buzz valley crew genuine basket start top",
+     "black tool gift useless bring nothing huge vendor asset mix chimney weird",
+   ]
+ }
```

<br/><br/>

### **DApp 的目录结构必须正确**

最小的文件配置如下所示:  
1. config.json -> 配置文件<br>
2. dapp.json   -> peer和delegate 配置<br>
3. init.js     -> 初始化配置, 里边主要定义交易费用和合约 <br>
4. model     ->  这个里边定义模型 <br>
5. contract ->  定义合约 <br>
6. interface  -> 定义接口 <br>



### **检查DApp是否已经注册并启动**
确保你的dapp 文件就在 `ETM_HOME/dapps/your-dapp-name`,目录. 打开下面的链接，可以查看到所有已经安装的dapp: `curl http://localhost:4096/api/dapps/installed`


### ** ETM_HOME/public/dist/dapps 这个目录必须存在**

确保 `ETM_HOME/public/dist/dapps` 这个目录必须存在， 如果不存在的话， 需要创建这个目录， 比如在linux 环境下， 运行 mkdir -p public/dist/dapp

### **Dapp model 里边的字符串类型的数据没有写长度**

在 DApp 目录下的 `model/` 目录存放了所有的自定义的表和表的字段的定义. 如果你定义某一列为 `string` 类型的话，你 __必须__ 同时为这个列提供 `length` 属性!

错误的方式:  
```js
module.exports = {
  name: 'articles',
  fields: [
    {
      name: 'tid',
      type: 'String'
    }
  ]
}
```

正确的方式:  
```js
module.exports = {
  name: 'articles',
  fields: [
    {
      name: 'tid',
      type: 'String',
      length: 64
    }
  ]
}
```

<br/><br/>

## **DApp不出快**

### **config.json没有足够的dapp 代理secret**

确保 `config.json`文件里边包含足够的dapp 代理者密码，另外，config.json 里边的dapp 代理者密码要和dapp.json 里边的public key 保持一致。 另外，如果万一需要修改 dapp.json 里边的public key， 则需要重新注册dapp； 修改后需要重启ETM 节点.

__注__  
指定DApp 在注册的时候需要提供的代理者密码


```diff
- {
-   "peers": [{"ip":"127.0.0.1","port":4096}],
-   "secrets": [
-   ]
- }
+ {
+   "peers": [{"ip":"127.0.0.1","port":4096}],
+   "secrets": [
+     "flame bottom dragon rely endorse garage supply urge turtle team demand put",
+     "thrive veteran child enforce puzzle buzz valley crew genuine basket start top",
+     "black tool gift useless bring nothing huge vendor asset mix chimney weird",
+     "ribbon crumble loud chief turn maid neglect move day churn share fabric",
+     "scan prevent agent close human pair aerobic sad forest wave toe dust"
+   ]
+ }
```


<br/><br/>

## **交易失败**

### **以前的交易验证失败**
你的交易没有被执行是因为它依赖于另外一个交易，而另外的交易还没有提交并确认（写入到侧链区块中）.  

> __解决方案:__  
> 等待__10秒__ 直到前面的交易已经被确认，然后把你的交易重新提交一次.区块每10s产生一个. 只要前面的交易还在未确认交易里边 (`http://localhost:4096/api/transactions/unconfirmed`) 你就需要等待.


### **发送交易到了错误的API 地址**

由于dapp会新添加很多新的API地址，你的请求可能方法送到了其他Dapp 的 API 地址去了:

主链:  
- Signed (HTTP __POST__): `http://localhost:4096/peer/transaction`  
- Unsigned (HTTP PUT): `http://localhost:4096/api/transactions`  

DApp:  
- Signed (HTTP PUT): `http://localhost:4096/api/chains/your-dapp-name/transactions/signed`  
- Unsigned (HTTP PUT): `http://localhost:4096/api/chains/your-dapp-name/transactions/unsigned`  



<br/><br/>

## **注册自定义的 DApp 合约失败**

### **Contract 代码必须不小于 1000**

低于1000 的合约代码是系统保留的合约代码，请使用高于1000的合约代码.  

```diff
# init.js file

module.exports = async function () {
  app.logger.info('enter dapp init')

- app.registerContract(800, 'cctime.postArticle')
+ app.registerContract(1002, 'cctime.postArticle')

}
```

### **合约没有指向正确的文件和函数**

字符串 `'news.postArticle'` 表示必须有一个 `news.js` 文件在 `contract/` 目录下并且 `news.js` 文件必须包含一个 `postArticle`的函数.

```js
registerContract(1001, 'news.postArticle')
```

<br/><br/>

## **请求报404错误**
### **检查请求URL里边的Dapp名称 **

再次检查下你是否在访问正确的 DApp. 尤其需要检查你请求里边的 __your-dapp-name__  (`http://localhost:4096/api/dapps/your-dapp-name/endpoint`).  

你也可以参考下 [**发送交易到了错误的 API endpoint**](#sending-transaction-to-wrong-api-endpoint)

查看所有已经安装的 DApps:  

```bash
curl http://localhost:4096/api/dapps/installed

# returns:  
{
  "success":true,
  "chains":[
    {
      "tid":"23f3c877d9a4163d14cb90a10a8132d9b5ae2d25cf568d994720acd85a9272b1",
      "name":"test-rTGrJniQQEys",
      "address":"CNKb1p78kKY9DGT7eNfYQ4Xe2r1B9T91nB",
      "desc":"A hello world demo for asch dapp",
      "link":"https://test-wnNethfKMmTk.zip",
      "icon":"http://o7dyh3w0x.bkt.clouddn.com/hello.png",
      "unlockNumber":3,
      "_version_":1
    }
  ]
}
```

<br/><br/>

## **在ETM 目录运行 npm install 失败**
### **父目录的名称含有空格**

如果 `ETM`的父目录名称包含空格， `npm install` 会无法正常处理:  

Error:  
```
configure: error: The build directory contains whitespaces - This can cause tests/installation to fail due to limitations of some libtool versions
Makefile:61: recipe for target 'libsodium' failed
make: *** [libsodium] Error 1
/home/a1300/test/asch 2/node_modules/sodium/install.js:287
            throw new Error(cmdLine + ' exited with code ' + code);
            ^
```

> __解决方案:__  
> 移除掉父目录名称里边的空格  

<br/><br/>

## **无法访问ETM 侧链**

### **检查 端口和配置**
如果你在本地运行ETM的话，默认的端口是 `4096`.

- 检查下你是否修改了  `config.json` 这个配置文件里边的端口号?
- 你是否在用类似 `node app.js --port 1234` 这样的命令启动dapp，然后调用接口的生活，依旧使用端口 4096?
- 你是否使用 `./etmd start` 作为启动etm 后台服务进程？ 检查`etm.pid` 文件 并且查看当前etm 监听的是哪个端口: `netstat -tulpn | grep -f etm.pid`.


 
<br/><br/>

---------

如果你遇到上面列举的错误以外的问题, 请创建一个 github issue [entanmo/issues](https://github.com/entanmo/etm/issues)

## 操作场景
本文介绍如何启用ENI独立网卡，可以在云服务器上绑定多个弹性网卡，实现高可用网络方案；也可以在弹性网卡上绑定多个内网 IP，实现单主机多 IP 部署。

- [启用ENI独立网卡](#openEniNetwork)
- [关闭ENI独立网卡](#closeEniNetwork)
## 操作步骤
[](id:openEniNetwork)
### 启用ENI独立网卡
1. 登录 [腾讯云容器服务控制台](https://console.cloud.tencent.com/tke2)，选择左侧导航栏中的**边缘集群**。
2. 单击需要启用ENI独立网卡的集群 ID，进入该集群详情页。
3. 选择页面左侧**基本信息**，进入集群基本信息页面，点击**开启独立网卡**开关,开启ENI独立网卡功能，详细操作如下：
      - 3.1 点击**访问API密钥**超链接，跳转至密钥信息页面。
            ![](https://qcloudimg.tencent-cloud.cn/raw/d04413124b004f1144c4f8b1b7db2084.png)
      - 3.2 复制密钥ID和密钥Key,返回确认密钥弹框页并完成输入。
            ![](https://qcloudimg.tencent-cloud.cn/raw/6793d7c93a7c073447c412e56931c819.png)
            ![](https://qcloudimg.tencent-cloud.cn/raw/1a460dc9b1e9e4056577cd18ec9d6f71.png)
      - 3.3 点击**确认**按钮，完成启用ENI网卡。
      ![](https://qcloudimg.tencent-cloud.cn/raw/5bd65c52558866b83afa333f18c18652.png)
4. 选择页面左侧**工作负载 -> Deployment**，进入deployment列表页，若列表中已有deployment，则跳过该步骤;否则，[新建deployment](https://cloud.tencent.com/document/product/457/31705)
5. 选择页面左侧**节点管理 -> 节点**，进入节点列表页，若列表中已有CVM节点，则跳过该步骤;否则，[新建CVM节点](https://cloud.tencent.com/document/product/457/42890#createCVMNode)
6. 远程登录云服务器,验证访问目标边缘集群的pod实例IP，详细操作如下：
   - 6.1 确认目标边缘集群的VPC网络类型
        ![](https://qcloudimg.tencent-cloud.cn/raw/b384c99572024841d953522bcd4a6fc1.png)
   - 6.2 进入[腾讯云云服务器控制台](https://console.cloud.tencent.com/cvm/instance/index?rid=1),选择一个和目标边缘集群<b>相同VPC网络</b>的云服务器，若未找到，则[新建云服务器](https://cloud.tencent.com/document/product/213/2753)。
        ![](https://qcloudimg.tencent-cloud.cn/raw/27e9f07b13f20ff4c004fbda8e356878.png)
   - 6.3 选择相同VPC网络的云服务器，并点击**登录**
        ![](https://qcloudimg.tencent-cloud.cn/raw/bdb22385a5ff3ca231884c8effe548cd.png)
   - 6.4 确认并复制目标集群的pod实例IP
          ![](https://qcloudimg.tencent-cloud.cn/raw/7c94aea8a79ce66d060e7845d36194e1.png)
   - 6.5 执行命令`curl 目标集群的pod实例IP`,验证是否可以访问。（若未在pod中配置eni，默认会访问受限）
        ![](https://qcloudimg.tencent-cloud.cn/raw/e0f0c95f728688c35a13a542b8f07735.png)
7. 在目标边缘集群的pod中配置ENI。
   - 配置截图:
  ![](https://qcloudimg.tencent-cloud.cn/raw/77d0d2111bafa54c86c608624427b5ac.png)
  ![](https://qcloudimg.tencent-cloud.cn/raw/21de8282010886bfaf4ebb4de7456728.png)
   - 实际代码:
  ```
    template:
      metadata:
        annotations:
          tke.cloud.tencent.com/networks: tke-direct-eni,flannel
  ```
  ```
      spec:
        affinity:
          nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: In
                  values:
                  - cvm-2cxgi4ow #访问目标的cvm节点ID
  ```
8. 远程登录云服务器,再次验证访问目标边缘集群的pod实例IP，详细操作如下：
   - 8.1 若已打开云服务器控制台，则跳过该步骤；否则，选择相同VPC网络的云服务器，并点击**登录**
        ![](https://qcloudimg.tencent-cloud.cn/raw/bdb22385a5ff3ca231884c8effe548cd.png)
   - 8.2 确认并复制目标集群的pod实例IP
          ![](https://qcloudimg.tencent-cloud.cn/raw/491a3fb376a2314b60c4640704921da9.png)
   - 8.3 执行命令`curl 目标集群的pod实例IP`,验证是否可以访问。（已在pod实例中配置eni，访问会成功）
        ![](https://qcloudimg.tencent-cloud.cn/raw/575c110ef49353963b46dcd9d583df06.png)
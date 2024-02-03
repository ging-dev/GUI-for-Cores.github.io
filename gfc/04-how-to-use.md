# GUI.for.Clash 使用教程

## 下载内核

转至 `设置` - `内核` 页面，程序会检测本地是否有内核文件，如果没有只需要点击`更新`按钮即可下载到本地，通常这一步需要科学上网环境。

如果下载失败，你也可以使用已有的内核文件，将他重命名为`mihomo-${os}-${arch}.exe`或`mihomo-${os}-${arch}-alpha.exe`，并放置在程序的`data/mihomo`目录下，然后重启软件。

![](/gfc/resources/205_how_to_use.png)

如果一切正常，那么它应该正确显示内核版本，此时你可以使用不同的分支版本，点击一下即可切换。

![](/gfc/resources/206_how_to_use.png)

## 导入节点

来到`概览`，界面上有一个`快速开始`按钮，点击后填入订阅链接，GUI 会自动的下载订阅文件，然后获取其中的节点信息，并创建一个以随机 ID 命名的.yaml 文件来存储它。同时 GUI 会创建一份`配置`文件，并自动的关联上刚刚下载的订阅文件。

![](/gfc/resources/200_how_to_use.png)

如果上一步执行无误，你就可以点击`启动内核`按钮启动程序了。

![](/gfc/resources/207_how_to_use.png)

默认设置下，GUI 不会改变你系统的任何设置，所以你需要手动开启`系统代理`，当然也可以在`设置`里打开`自动配置系统代理`，如果想使用 TUN 模式，则需要打开`以管理员身份运行`。TUN 模式和系统代理应该保证只有一个处于开启状态。

![](/gfc/resources/208_how_to_use.png)

## 进阶玩法

以上的操作仅适合快速上手，如果你想了解程序的运行原理，那么跟着我来一步步的手动的创建订阅以及配置吧。

### 创建订阅

首先我们创建一份订阅，来到`订阅`界面，点击`添加`按钮，依次填写表单，有\*号的为必填项。

若订阅类型选择为`本地`，建议链接填写为`data/local/${filename}.txt`，更新订阅时会从`链接`里读取文本，处理后保存到`保存路径`里，若是`链接`和`保存路径`填写一致，则更新订阅时程序会跳过保存的步骤，仅更新节点数量等元数据。

![](/gfc/resources/201_how_to_use.png)

添加完订阅后，可以点击`更新`按钮，将`链接`处的内容保存到`保存路径`内，注意，GUI 只会保存内容中的`proxies`字段，也就是说 GUI 暂不支持内容文件中的分流规则等其他配置。

### 创建配置

接着我们来到`配置`界面，点击`添加`按钮，依次填写表单，有\*号的为必填项。GUI 支持了大部分的内核参数配置，若是你需要的参数 GUI 上没有，则可以通过插件来解决。下面是一个示例：

创建一个触发器为`生成配置时`的插件。示例源码：

```javascript
const onGenerate = (config) => {
  // 按下Ctrl+Shift+F12来查看config里有哪些内容
  console.log(config);
  // 增加域名嗅探字段
  config.sniffer = {
    enable: false,
    "force-dns-mapping": true,
    "parse-pure-ip": true,
    "override-destination": false,
    sniff: {
      HTTP: {
        ports: [80, "8080-8880"],
        "override-destination": true,
      },
      TLS: {
        ports: [443, 8443],
      },
      QUIC: {
        ports: [443, 8443],
      },
    },
    "force-domain": ["+.v2ex.com"],
    "skip-domain": ["Mijia Cloud"],
  };
  return config;
};
```

此插件在生成配置文件时会执行，并将处理后的配置返回给 GUI。

表单中的`名称设置`、`通用设置`、`TUN设置`、`DNS设置`就不介绍了，对应 mihomo 官方文档理解即可。这里介绍如何配置分流与代理组。

![](/gfc/resources/209_how_to_use.png)

新建的配置是具有默认的代理组的，也就是上图中的 4 个组，每个组至少需要添加一个`proxies`或`use`，否则就会出现左侧的感叹号提醒。点击编辑按钮来到下图。

![](/gfc/resources/210_how_to_use.png)

图中的区域 1 是当前已添加的代理组，包括了内核内置的 `DIRECT`、`REJECT`。点击它的名称，可以将它添加到当前组的`proxies`中，程序限制了`自我引用`，但你要注意不能出现`循环引用`，即组 A 引用了组 B，组 B 又引用了组 A。

图中的区域 2 是程序里添加的订阅列表，点击它的名称，可把它加入到当前组的`use`中。

图中的区域 3 是每一个订阅下的节点列表，展开它，可以勾选部分节点，并添加到当前组的`proxes`中。这通常用于多个订阅联合使用的情况。

![](/gfc/resources/211_how_to_use.png)

规则设置界面对应内核配置文件中的 rules 字段，GUI 没有将它变为更清晰的展示，因为了解内核配置的人，上面的展示是更加清晰易懂的。

![](/gfc/resources/212_how_to_use.png)

点击添加，并选择类型为规则集，下面会出现程序里添加的规则集列表，勾选它即可。如果你的规则集列表是空的，那么在插件中心找到一个叫`一键添加规则集`的插件并运行它吧。

如果类型选择为域名集合(GEOSITE)或国家 IP 代码规则(GEOIP)，则需要参考这个项目来配置：[MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)。
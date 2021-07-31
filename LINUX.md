# つつの订阅转换——爱折腾用户本地搭建教程

> 此文章测试环境为 debian 10，理论 linux 通用（

## 第一步 你需要有一个Github账号

如何注册Github账号？[点这里进入Github注册教程](https://jingyan.baidu.com/article/4e5b3e192200e291911e2468.html)

## 第二步 Fork本项目

进入项目页面 https://github.com/lhl77/subweb-tsutsu

如图所示点击 Fork 按钮

![](https://camo.githubusercontent.com/c71fde237eea0a9ed82fffbf6518b3282d656a6ed5ffcf32dd2005806c18be47/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6c686c37372f7265706f7369746f7279406d61696e2f626c6f672f32303231303733313132353730362e706e67)

接着页面会自动跳转到这个页面（如下图），记住这里的网址！

![](https://camo.githubusercontent.com/5b8404a954d9a5226958d7e03ceeac97485dcd9f5416c6608ff7ba820e5829b5/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f6c686c37372f7265706f7369746f7279406d61696e2f626c6f672f32303231303733313133313131392e706e67)

## 第三步 准备工作

### 安装环境

#### Node.js

本教程使用 [nvs](https://github.com/jasongin/nvs) 项目进行 Node.js 的快速部署。

```bash
export NVS_HOME="$HOME/.nvs"
git clone https://github.com/jasongin/nvs --depth=1 "$NVS_HOME"
. "$NVS_HOME/nvs.sh" install
nvs add 12.12.0
```

至此 Node.js 12.12.0 已安装完毕。

#### yarn

```bash
npm install --global yarn
```

## 第四步 拉取项目

```bash
git clone 第二步获得的链接加上 .git
cd subweb-tsutsu
```

### 安装依赖

```bash
yarn install
```

此步骤的一项依赖需要 `python2` ，请确保 `python2 -V` 或者 `python -V` 可以显示出 python2 的版本号。

## 第五步 构建项目

```bash
yarn build
```

🎉 恭喜你，将新生成的 `dist` 目录设置为网站目录即可体验！

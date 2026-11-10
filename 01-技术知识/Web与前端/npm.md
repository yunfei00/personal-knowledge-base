---
id: tech-web-npm-001
title: npm 是什么：Node.js 包管理、常用命令与网络故障
category: technical
tags: [npm, Node.js, JavaScript, 包管理, package.json, Electron, Windows]
status: draft
created: "2026-11-10"
updated: "2026-11-10"
platform: "Windows / macOS / Linux"
source_project: ""
source_url: "https://docs.npmjs.com/"
sensitivity: public
---

# npm 是什么：Node.js 包管理、常用命令与网络故障

> **一句话：** npm（Node Package Manager）是 Node.js 生态中常用的包管理工具，用来安装、更新、管理 JavaScript 项目依赖，也可以执行项目脚本。

## 1. npm、Node.js 和 Electron 的关系

| 名称 | 是什么 | 主要用途 |
| --- | --- | --- |
| **Node.js** | JavaScript 运行环境 | 在浏览器之外运行 JavaScript |
| **npm CLI** | 命令行包管理工具 | 安装依赖、管理版本、运行脚本 |
| **npm Registry** | 软件包注册表服务 | 存放可下载的软件包与版本信息 |
| **Electron** | 桌面应用框架 | 用 Web 技术开发桌面程序 |

```text
安装 Node.js
    └── 通常同时安装 npm
             ├── npm install electron
             │       └── 下载 Electron 项目依赖
             └── npm run start
                     └── 执行 package.json 中的启动脚本
```

**注意：** npm 不是编程语言，也不是 IDE。npm CLI 与 npm Registry 也不是同一回事：前者是你电脑上的命令，后者是远程包服务。

相关知识：[Electron 是什么](../桌面应用/Electron.md)。

## 2. npm 如何安装？

通常直接从 [Node.js 官网](https://nodejs.org/) 安装 **LTS** 版本，Windows 安装程序会同时安装 npm。安装后打开 PowerShell 检查：

```powershell
node --version
npm --version
```

如果出现版本号，就说明命令可以使用。若提示找不到命令，先重新打开终端，再检查 Node.js 安装和 PATH 环境变量。

## 3. npm 常用命令

| 命令 | 用途 |
| --- | --- |
| `npm init -y` | 创建默认 package.json |
| `npm install` | 安装当前项目声明的依赖 |
| `npm install axios` | 安装运行时依赖 |
| `npm install --save-dev electron` | 安装开发依赖 |
| `npm uninstall axios` | 卸载依赖 |
| `npm update` | 按版本约束更新依赖 |
| `npm run start` | 运行 package.json 的 start 脚本 |
| `npm list --depth=0` | 查看顶层依赖 |
| `npm config get registry` | 查看当前软件包注册表地址 |
| `npm ping` | 检查 Registry 连接 |
| `npm ci` | 根据 lock 文件进行适合 CI 的确定性安装 |
| `npx electron --version` | 执行项目中的 Electron 命令并显示版本 |

`npx` 通常随 npm 一起提供，可运行 npm 包中的命令；如果本地不存在对应包，它可能尝试从网络获取，具体行为受 npm 版本和参数影响。

## 4. npm 项目里的重要文件

```text
my-project/
├── package.json       # 项目元数据、脚本、依赖版本范围
├── package-lock.json  # 锁定依赖解析结果
└── node_modules/      # 已安装的依赖目录
```

- **package.json：** 描述项目名称、版本、脚本，以及 dependencies 和 devDependencies。
- **package-lock.json：** 记录具体依赖解析结果；应用项目通常应提交到 Git，便于复现安装。
- **node_modules：** 本地下载的依赖文件夹；通常加入 .gitignore，不直接提交 Git。

### dependencies 与 devDependencies 的区别

`dependencies` 表示应用运行或生产部署所需的包；`devDependencies` 通常是开发、测试、构建阶段所需的工具。Electron 桌面项目通常把 `electron` 安装为开发依赖，最终发布的应用通过打包工具包含需要的运行时。

## 5. npm install 到底做了什么？

运行 `npm install` 时，npm 会读取项目依赖声明及锁文件，解析依赖版本，从配置的 Registry 下载包并安装到 `node_modules`。部分软件包还会运行安装脚本，下载额外二进制文件或编译原生模块。

因此：**npm Registry 可访问，并不代表所有安装步骤都能访问网络。**

Electron 是一个典型例子：除了 npm 软件包，还可能需要下载 Electron 的二进制运行时。二进制下载源通常与 npm Registry 不同。

## 6. Electron 安装为什么会出现 ETIMEDOUT？

你遇到的错误：

```text
npm ERR! request error
connect ETIMEDOUT
```

表示某个网络连接在规定时间内没有建立或完成。常见原因包括网络受限、代理配置不正确、DNS/防火墙问题、npm Registry 不可达，以及 Electron 二进制下载超时。

**排查时先确定超时发生在哪个地址：**

1. `npm config get registry`：确认 npm Registry。
2. `npm ping`：确认 Registry 是否能连接。
3. 查看 npm 错误日志中的 URL、主机名与失败阶段。
4. 如果 Registry 正常而 Electron 安装失败，检查 Electron 二进制下载地址及代理。
5. 公司网络环境下优先使用经过批准的镜像或代理，不关闭证书校验来规避网络问题。

### Windows PowerShell 常用排查命令

```powershell
node --version
npm --version
npm config get registry
npm ping
npm config get proxy
npm config get https-proxy

# 若确认使用 npm 镜像，才执行以下配置
npm config set registry https://registry.npmmirror.com
npm config set fetch-retries 5
npm config set fetch-timeout 120000

# 仅当前 PowerShell 会话使用 Electron 二进制镜像
$env:ELECTRON_MIRROR = "https://npmmirror.com/mirrors/electron/"

# 在 Electron 项目目录安装
npm install --save-dev electron
npx electron --version
```

上述镜像为第三方服务，可能不可用或不符合公司安全要求。若要恢复官方 npm Registry：

```powershell
npm config set registry https://registry.npmjs.org/
```

**重要：** 不建议通过 `strict-ssl=false`、忽略 TLS 证书或运行未知安装脚本来“解决”超时。

## 7. npm、pnpm、Yarn、uv 和 pip 的区别

| 工具 | 主要生态 | 用途 |
| --- | --- | --- |
| npm | Node.js / JavaScript | 默认常见的 JS 包管理工具 |
| pnpm | Node.js / JavaScript | 强调磁盘复用与依赖管理 |
| Yarn | Node.js / JavaScript | 另一种 JS 包管理方案 |
| pip | Python | 安装 Python 包 |
| uv | Python | Python 环境与依赖管理等 |

**简单理解：** npm 主要服务 JavaScript/Node.js 项目，pip 和 uv 主要服务 Python 项目。Electron 前端常用 npm，而 Python AI 后端可使用 uv；它们可以同时存在于一个桌面应用项目中。

## 8. 在 VS Code 中怎么使用 npm？

VS Code 自带集成终端。打开项目文件夹后选择 **终端 → 新建终端**，即可执行 `npm install`、`npm run start` 等命令。

npm 并不要求必须使用 VS Code；PowerShell、Windows Terminal、WebStorm、Cursor 的终端同样可以运行。

## 9. 常见误区

- **npm 不等于 Node.js：** 一个是包管理工具，一个是 JavaScript 运行环境。
- **npm 不等于 Electron：** npm 可以安装 Electron，也能安装成千上万种其他包。
- **安装 npm 包不等于生成 EXE：** 桌面应用发布仍需要构建和打包。
- **不要随意删除 package-lock.json：** 它有助于团队和 CI 复现依赖版本。
- **npm install 不是纯下载：** 某些依赖会执行安装脚本，因此要注意软件包来源与供应链安全。

## 10. 后续学习路线

1. 熟悉 `package.json`、`package-lock.json`、`node_modules`。
2. 理解 `dependencies` 和 `devDependencies`。
3. 掌握 `npm install`、`npm ci`、`npm run`。
4. 学会排查 Registry、代理、TLS 与二进制下载问题。
5. 了解 npm 与 pnpm、Yarn 的区别。

## 参考资料

- [npm 官方文档](https://docs.npmjs.com/)
- [npm install 命令](https://docs.npmjs.com/cli/commands/npm-install)
- [package.json 说明](https://docs.npmjs.com/cli/configuring-npm/package-json)
- [Node.js 官网](https://nodejs.org/)
- [Electron 官方文档](https://www.electronjs.org/docs/latest/)

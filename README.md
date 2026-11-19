# dsh-go-combined-ios · iOS 适配版静态注册表

给 **iOS 上的 DSH Go（自签 ipa 版）** 用的静态 npm 注册表。按目前的实测，DSH Go 能装的只有 HTTPS 注册表来源的纯 JS 包，这个仓库就是那个来源。

这是为 iOS 重新打包的版本，**不是通用版**。Android、桌面端请直接回上游装原包，不要用这里的东西。

## 这个「自签版」装的是哪个 IPA

来源是 DSH Go 官方仓库，不是第三方改包：

| 项 | 值 |
|---|---|
| 仓库 | [ccch1mneyyy/dsh-Go](https://github.com/ccch1mneyyy/dsh-Go) |
| 版本 | [v1.2.0](https://github.com/ccch1mneyyy/dsh-Go/releases/tag/v1.2.0)（2026-11-18） |
| 文件 | [`dsh-go-1.2.0-ios-unsigned.ipa`](https://github.com/ccch1mneyyy/dsh-Go/releases/download/v1.2.0/dsh-go-1.2.0-ios-unsigned.ipa) |
| 大小 | 123,474,162 字节（约 118 MB） |
| SHA256 | `f8f6d4c6421465f3e2745dd389985dc7a35a4e092411292b3f0c280bf43fa7d1` |

官方对 iOS 只发**未签名**的 ipa，没有公开的 TestFlight 通道，要用 AltStore、爱思助手这类工具自己签名再装。用免费 Apple ID 签名时 Bundle ID 会被改掉，Siri、分享扩展、后台运行这些能力可能用不了。上游自己也写明：iOS 版只在 CI 里编过、跑过模拟器，真机只零星试过，出问题要去开 issue。

同一版本还有安卓安装包 `dsh-go-1.2.0-arm64.apk`，以及 `SHA256SUMS`、`update.json`（自动更新和校验用，不用手动下）。

装上之后，App 不归 App Store 管：你不重新签一个新包，它就不会自己更新，插件和补丁也就不会被更新冲掉。

## 装什么

`dsh-go-combined-ios@1.2.3` —— 破甲合并包（dsh-go-combined）的 iOS 重打包版。

**来源：** 代码取自上游 [jixiaolan7/dsh-go-purge-ios](https://github.com/jixiaolan7/dsh-go-purge-ios) 的 `v1.2.3` 标签，重打包时只改包名，版本号仍为 1.2.3。上游的发布包是 zip，只能在安卓 proot 或桌面上用 `bash install.sh` 装；本仓库把它转成 HTTPS 注册表上的纯 JS 包，iOS 的 `plugin_manager` 才认得。

安装：

```
用 plugin_manager 安装 dsh-go-combined-ios@1.2.3，registry 指定 https://shiyichen413-ops.github.io/dsh-registry/
```

## 装之前、装完、要退回

**装之前先清旧包。** 如果这台机器上还装着 `dsh-go-unrestricted`，先把它移除再装。两者会插入同一组预设 rowId，并存的结果不是"多一套预设"，而是注入打两遍、工具出两套。

**装完怎么确认。** 打开预设菜单，应该多出三个：无限制模式、无限制模式·子任务、红队模式。到这儿就够了，不用每次都跑「破甲自检」——一次几万 token，没有异常别跑。

**要退回。** 用 `plugin_manager` 移除 `dsh-go-combined-ios`；补丁层另外执行 `purge_patch_apply mode=restore` 还原。

## 规格

| 项 | 值 |
|---|---|
| 版本 | 1.2.3（与上游一致） |
| 包体 | 834,515 字节（约 815 KB），零依赖 |
| 要求 | dsh `>=0.2.1-rc.2 <0.3.0-0`，node `>=22.19` |
| 运行时 | 纯 JS，无原生模块，无安装脚本 |

## 为 iOS 做了什么

- **包名改写**：`dsh-go-combined` → `dsh-go-combined-ios`，避免与上游包名撞车，也便于和上游版本区分。
- **保持纯 JS、零原生依赖、无 install 脚本**。按实测，iOS 宿主只收注册表上的纯 JS 包（见下一节），带安装脚本或原生模块的包会在安装阶段被 `OHOS_PLUGIN_UNSUPPORTED` / `ERR_OHOS_PLUGIN_PACKAGE` 直接拒掉，进不来。
- **来源是上游 v1.2.3**，注入文本、补丁表、三套预设内容一律未动，只改包名与作者字段。
- **红队预设的路径问题在 1.2.3 已由上游修掉**：旧版把 `anchor-gate` 写死成安卓 proot 的 `file:///root/.dsh/...`，iOS 上 HOME 不是 `/root`，所以整条预设判定为不可用。

## 为什么有些东西在这台机器上装不上

先把最容易误会的一点说清：**自签 ipa 只解决「App 能不能装进手机」，管不到「插件能不能装进 App」。** 插件安装走的是 App 内部的宿主策略检查，跟签名方式、跟有没有越狱都无关。自签版真正的好处是 App Store 管不到它——不主动重签新包，App 就不会自动更新，已装的插件和补丁不会被一次更新覆盖掉。

我这边实测能走通的插件通道目前只有一条：**HTTPS 注册表上的纯 JS 包**。下面这些方式逐个试过，全部被拒：

| 你试的方式 | 报错 |
|---|---|
| zip 包 / 本地目录 / 本地 tgz | `OHOS_PLUGIN_UNSUPPORTED`：本机仅支持注册表纯 JS 包，不运行安装脚本 |
| `github:` 整包地址 | 同上，不认 git 源 |
| 自建本地注册表（http 或非标准端口） | `ERR_OHOS_PLUGIN_PACKAGE: Package URL must use HTTPS without credentials, query or fragment` |
| 带原生模块（`.node`）的包 | `ERR_OHOS_PLUGIN_PACKAGE: incompatible OHOS arm64 platform`，iOS 跑不了原生模块 |
| 引擎版本要求不满足的包 | 安装阶段直接挡住 |

也就是说，一个包要能在这台机器上装，必须同时满足四条：**托管在 HTTPS 上、纯 JS、无安装脚本、零原生依赖**。这也是本仓库存在的唯一理由——把上游的 zip 重打成符合这四条的形状，挂在 GitHub Pages 上，让 `plugin_manager` 按包名取。

这不是穷举结论：以上只代表在 DSH Go 1.2.0（iOS 自签）上实际试过的方式，不排除还有没被测到的入口，比如 App 以后加了「从文件导入」之类。遇到新的入口，以实测为准，这张表随时可以改。

## iOS 上跑不起来的

装之前先看清这几条，别指望它和桌面端一样全：

- **终端启用不了（仅自签版）。** 这条是**自签名 ipa 版**（从项目 GitHub Release 下载 ipa 自行签名安装的那一套）的表现：终端入口是灰的或起不来。如果以后官方另外发行了签名版或 TestFlight 版，这一点没有实测，不保证一样。后果是「先 `bash install.sh`、再彻底重启」的装法在这里不成立，只能走注册表按包名装；补丁层也只能靠插件自带的 `purge_patch_apply` 来打，没有别的手。
- **面板、工作台、侧栏类界面全部不显示。** App 用的是原生界面，不渲染插件的网页 UI。插件里凡是靠网页呈现的功能——状态页、开关卡片、红队演练台、资产库——装上了也看不见。
- **桌面专属的补丁没有落点。** 补丁表里约 29 条针对桌面端包（`dsh-client-ui-conversation` 之类），iOS 上这些包根本不存在，不计入生效，也不用管。
- **原生二进制插件装不了。** iOS 跑不了 `.node` 原生模块，任何带原生依赖的包都会在平台检查阶段被拒。
- **补丁层每次彻底重启会掉几条。** 实测每次约掉 3 条，用 `purge_patch_apply mode=apply` 补回即可；注入层由宿主进程现场组装，重启不受影响。

## 实测状态

机型：iOS，DSH Go 自签版，引擎 dsh `0.2.2-rc.1`。

- 注入层：开，三套预设（无限制模式 / 无限制·子任务 / 红队模式）均为 active。
- 补丁层：约 54/57 生效，剩余几条为引擎版本措辞不匹配或桌面专属。
- 全局注入对所有会话生效（含酒馆），这是插件本身的设计，不是本仓库改的。

## 上游与作者

| 作者 | 贡献 | 仓库 |
|---|---|---|
| **jixiaolan7** | 合并打包 dsh-go-combined | https://github.com/jixiaolan7/dsh-go-purge-ios |
| **NANA7MI0** | dsh-go-purge 0.4.18：提示词注入层、86 条补丁表 | https://github.com/NANA7MI0/dsh-purge-go |
| **lingsmall-code** | dsh-go-unrestricted：三套 agent 预设、面板、补丁引擎 | https://github.com/lingsmall-code/dsh-go-unrestricted |
| **YuJunZhiXue** | dsh-go-purge 原始作者 | — |

## 本仓库做了什么

只改包名（`dsh-go-combined` → `dsh-go-combined-ios`），并补上作者、贡献者与来源字段。
版本号与上游一致，为 **1.2.3**。注入文本、补丁表、预设内容一律未动，详见 `REPACK.md`。

## 许可

AGPL-3.0。二次分发须保留 `NOTICE` 与上游署名。
本项目完全免费；任何收费行为都是骗局，详见下方免责声明。

---

## 免责声明

- 本仓库仅为**个人学习与技术研究**用途的静态镜像/注册表，不提供任何形式的商业服务、代装或授权。
- 仓库中的插件会修改 DSH 引擎的系统提示与部分行为，可能影响 AI 的回答方式。**是否安装、如何使用、产生什么后果，由使用者自行判断并承担责任。**
- 仓库维护者与上游作者不对因使用本仓库内容导致的任何直接或间接损失负责，包括但不限于数据丢失、设备异常、账号风险、费用支出。
- 请在使用前确认符合你所在地区的法律法规，以及你所使用的服务（模型、平台）的条款。
- 上游项目与本仓库均为免费开源。**任何以"代装""授权""解锁""技术支持""进群""远程协助"等名义向你收钱的人，都是骗子，与仓库维护者和上游作者一概无关，请勿付款。**
- 如你是权利人并认为本仓库内容侵犯你的权益，请提 Issue，我会立即处理。

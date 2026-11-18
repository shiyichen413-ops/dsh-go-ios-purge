# dsh-go-combined-ios · 静态注册表

给 DSH Go（iOS）用的静态 npm 注册表。DSH Go 只接受 HTTPS 注册表来源的纯 JS 包，这个仓库就是那个来源。

## 装什么

`dsh-go-combined-ios` —— 破甲合并包（dsh-go-combined）的重打包版，代码未做功能改动。

安装：

```
用 plugin_manager 安装 dsh-go-combined-ios@1.2.4，registry 指定 https://shiyichen413-ops.github.io/dsh-registry/
```

## 上游与作者

| 作者 | 贡献 | 仓库 |
|---|---|---|
| **jixiaolan7** | 合并打包 dsh-go-combined | https://github.com/jixiaolan7/dsh-go-purge-ios |
| **NANA7MI0** | dsh-go-purge 0.4.18：提示词注入层、86 条补丁表 | https://github.com/NANA7MI0/dsh-purge-go |
| **lingsmall-code** | dsh-go-unrestricted：三套 agent 预设、面板、补丁引擎 | https://github.com/lingsmall-code/dsh-go-unrestricted |
| **YuJunZhiXue** | dsh-go-purge 原始作者 | — |

## 本仓库做了什么

只改包名与版本号（`dsh-go-combined` → `dsh-go-combined-ios`，1.2.3 → 1.2.4），
并补上作者、贡献者与来源字段。注入文本、补丁表、预设内容一律未动。

## 许可

AGPL-3.0。二次分发须保留 `NOTICE` 与上游署名。
本项目完全免费，任何收费代装、收费"授权"都与作者无关。

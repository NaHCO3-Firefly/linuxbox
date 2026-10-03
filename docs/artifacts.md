# 外部产物清单（artifacts）

> **这个仓库不提交大体积二进制。** 下面每一个文件都靠 `URL + SHA-256` 锁定，
> 由获取脚本下载并校验得到。想重建分发包，照这张表取即可。
>
> 为什么不直接提交：ISO 66 MB、QEMU 全套上百 MB、qcow2 母盘几百 MB——
> 加起来是 GB 级，而 GitHub 单文件上限只有 100 MB。仓库应该存「怎么拿到它们」。

---

## 1. Alpine Linux 安装源

| 项目 | 值 |
|---|---|
| 用途 | 首启无人值守安装的安装源（`goal.md` §4.3 方案 B / 兜底） |
| 文件名 | `alpine-virt-3.24.2-x86_64.iso` |
| 放置位置 | `images/alpine-virt-3.24.2-x86_64.iso` |
| 大小 | **69,206,016 字节（66.0 MiB）** |
| SHA-256 | `3ab424762af704b2c2a9e57df1dc37f982af260071504d977f2fb96822e7130b` |
| 下载地址 | `https://mirrors.cernet.edu.cn/alpine/v3.24/releases/x86_64/alpine-virt-3.24.2-x86_64.iso` |
| 官方校验和 | 同目录下 `alpine-virt-3.24.2-x86_64.iso.sha256` |
| 核实日期 | 2026-02 |

**版本为什么锁死在 3.24.2**：不能用 `latest-stable`。它今天指 3.24.2，明天可能就变；
`goal.md` §5 的「可重建」要求任何时候都能装出同一套系统。

**下载注意（已实测踩过的坑）**：`mirrors.cernet.edu.cn` 是 **302 聚合重定向站**，
实测会跳到 `mirrors.hust.edu.cn` 之类的成员镜像。所以：

- 下载器**必须跟随重定向**；不跟随的 HTTP 客户端会把 302 当成失败。
- **不要**把成员镜像域名写进脚本（例如写死 `hust`）。
- 下完**必须校验 SHA-256**：同一个 URL 在不同时刻可能由不同后端提供服务。

细节见 `goal.md` §4.6。

---

## 2. Alpine apk 仓库（guest 内使用，不需要下载存档）

写进 guest 的 `/etc/apk/repositories`：

```
https://mirrors.cernet.edu.cn/alpine/v3.24/main
https://mirrors.cernet.edu.cn/alpine/v3.24/community
```

**两行都必须有。** `xfce4`、`fcitx5`、`font-noto-cjk`、`lightdm`、`spice-vdagent`
**全在 `community`，一个都不在 `main`**——漏掉 community 会让 `apk add xfce4` 直接失败。
（实测依据：直接解析了镜像站上的 `APKINDEX`，见 `goal.md` §4.2。）

---

## 3. QEMU for Windows

**状态：待定，尚未选定具体构建。** 这是 M0 的第 0 步。

| 项目 | 值 |
|---|---|
| 用途 | 产品内置的虚拟机引擎（`goal.md` §4.1） |
| 候选来源 | QEMU for Windows 官方构建 / MSYS2 `mingw-w64-x86_64-qemu` |
| 放置位置 | `vm/qemu/`（整目录，已 gitignore） |
| 许可证 | **GPLv2** —— 分发时必须附许可证文本与源码获取途径 |

选定后必须补齐这张表：精确版本号、下载地址、SHA-256、体积。

选定后还需立即验证两件事（都写进了 `goal.md` §5）：

1. `qemu-system-x86_64.exe -spice help` —— 是否编入了 SPICE **服务端**。
   这决定剪贴板与自适应分辨率能否实现，进而决定整个显示架构（§5.3 的 go/no-go 实验）。
2. `qemu-system-x86_64.exe -accel help` —— 是否支持 `whpx`。

---

## 4. 黄金镜像母盘 `seed.qcow2`

**状态：不存在。** 由 M2 的构建流水线产出，不进 git（体积几百 MB）。

构建流程必须脚本化，否则它会变成一个没人知道怎么重建的「神秘镜像」——
那正是 `goal.md` §4.3 明确指出要避免的情况。

---

## 5. 其他

| 组件 | 状态 | 说明 |
|---|---|---|
| `remote-viewer`（SPICE 客户端） | 待定 | 仅当 §5.3 的 SPICE 实验通过才需要 |
| .NET SDK（开发工具） | 待装 | 不在分发包内，见 `goal.md` §5.8 |
| 各组件许可证文本 | 待收集 | 放 `vm/LICENSES/`，**这个目录要提交** |

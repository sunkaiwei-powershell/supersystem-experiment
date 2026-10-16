# SuperSystem · 超级系统

> ⬇️ **Download / 下载 / ダウンロード / 다운로드 / Herunterladen**: [GitHub Release V1.0.0](https://github.com/sunkaiwei-powershell/supersystem-experiment/releases/tag/V1.0.0)
>
> 🌍 **This README is written in 5 languages. Pick yours and jump straight in.**
> 🌍 **本文档提供 5 种语言版本，请按你的母语直接跳转。**
> 🌍 **この README は 5 言語で書かれています。母国語を選んでそのままお進みください。**
> 🌍 **이 README는 5개 언어로 작성되었습니다. 모국어를 선택해 바로 이동하세요.**
> 🌍 **Diese README gibt es in 5 Sprachen. Wähle einfach deine Sprache und springe direkt hinein.**

## 导引 / Guide · 语言索引 · Language Index

| 国家 / 地区 | 语言 | 直接跳到 |
|---|---|---|
| 🇨🇳 中国 | 简体中文 | [**中文版**](#中文) |
| 🇬🇧🇺🇸 英语国家 | English | [**English version**](#english) |
| 🇯🇵 日本 | 日本語 | [**日本語版**](#日本語) |
| 🇰🇷 韩国 | 한국어 | [**한국어판**](#한국어) |
| 🇩🇪🇦🇹🇨🇭 德语国家 | Deutsch | [**Deutsche Version**](#deutsch) |

> Each version is a complete, self-contained document — the same content, fully translated. / 每个版本都是完整且独立的文档，内容一致、各自翻译。 / 各版は完全に独立したドキュメントで、内容は同一です。 / 각 버전은 완전히 독립된 문서이며 내용은 동일합니다. / Jede Version ist ein vollständiges, eigenständiges Dokument mit demselben Inhalt.

---
---

<!-- ==================== 中文 =================== -->

## 中文

> **一句话简介**：一张把 ReactOS、Darwin/XNU、Linux 三种「内核」塞进同一个启动菜单的可引导 ISO —— 一个**可玩性实验项目**，初心是想验证「macOS 风格化桌面」。
>
> ⚠️ **请先读这句**：这是**玩具 / 探索性项目**，不是能日常使用的系统。除 Linux 两项外，ReactOS 与 macOS 项**目前都跑不起来**（原因见下文）。请把它当作一次「技术可行性折腾」，而不是一个产品。

### 1. 它是什么

启动后先出现一个 GRUB 文本菜单，共 4 个选项：

| # | 菜单项 | 内容 | 现状 |
|---|--------|------|------|
| 1 | **Windows 内核（ReactOS x86）** | ReactOS 0.4.16 LiveCD | ❌ 基本不可用 |
| 2 | **macOS 内核（Darwin/XNU x86-64）** | XNU 内核 + 自写 xnu-loader | ❌ 不可用 |
| 3 | **Linux 内核（命令行版）** | 自编译 Linux 6.12.4 + BusyBox shell | ✅ 可用 |
| 4 | **Linux 内核（图形版，Xfce macOS 风格）** | 自编译内核 + Xfce + WhiteSur 主题 | ✅ 可用（默认项）|

设计初衷是「**三种内核、同一套 macOS 画风的桌面体验**」：不同内核原生跑不同软件生态，但界面风格保持一致。

### 2. 快速开始：把 ISO 装进 U 盘

**准备**：一个 ≥ 2GB 的 U 盘（**数据会被清空，先备份**）、一台 Windows 电脑、管理员权限。

1. 下载发布包中的三个文件：`SuperSystem_USB_Installer.exe`、`stage6.iso`、`README.md`。
2. 双击 `SuperSystem_USB_Installer.exe`（会自动请求管理员权限）。
3. 选择镜像：默认自动找到同目录下的 `stage6.iso`，找不到就点「浏览…」手动选择。
4. 选择目标 U 盘：下拉框显示 `PhysicalDriveN | 容量 | 盘符 | 型号`（容量单位自动换算）；程序会**自动拦截系统盘**。
5. 点击「开始写入 U 盘」：进度区实时显示 **已写/总量、速度、剩余时间**，写完后自动**回读校验首 1MB**。
6. 开机引导：进 BIOS/UEFI 把 U 盘设为第一启动项（UEFI 请**关闭 Secure Boot**），选第 3 或第 4 项即可进入系统。

> 写入后 Windows 若提示 U 盘「需要格式化」——**属正常现象**，这是 hybrid ISO 的特征，说明引导区写好了，别点格式化，直接用它开机。
>
> 也可用 **Ventoy**：把 `stage6.iso` 拷进 Ventoy 盘即可，无需安装器。

### 3. 已知问题（重点）

- **Windows 内核（ReactOS）——❌ 在实体机 / U 盘上起不来**
  - 现象：实体机直接报错、按任意键即重启；U 盘（Ventoy 与 Rufus 都试过）停在 `ReactOS 0.4.16 (isoboot)...` 或弹 `Unable to load second stage loader.`
  - 根因：ReactOS 0.4.16 的 FreeLoader 为两段式加载（`freeldr.sys` → `rosload.exe`），第二段在非标准光盘环境加载失败；官方光盘引导器 `isoboot.bin` **依赖 El Torito 光盘状态**，U 盘不是真实光驱，于是挂死。
  - 目前只在少数「被当作真实光盘」的环境（如 QEMU 以 `-cdrom` 挂载）能完整启动。修复需改走 ReactOS 官方 U 盘方案（FAT32 + `fat32.bin`），**本项目暂停，暂未实现**。
- **macOS 内核（Darwin/XNU）——❌ 实际无法加载**
  - 现象：实体机直接闪退；UEFI 下能起 `xnu-loader.efi`，但内核加载后报 `unknown error`。
  - 根因：`xnu-loader.efi` 是实验性 EFI stub；更本质的是 **XNU 不是通用 PC 内核**，缺少面向普通 x86-64 PC 的整套平台驱动（「黑苹果」靠的是完整 macOS 系统 + 大量补丁，裸 XNU 顶不上）。**工程上不现实**。
- **Linux 两项——✅ 可用**：真机、虚拟机、U 盘（Ventoy / Rufus）均已验证可进系统。命令行版为兼容老显示设备加了 `nomodeset`；图形版用 KMS 驱动接管显示。

### 4. 定位、初心与后续

- **定位**：**可玩性项目**。它证明「三种内核装进同一启动菜单」在引导层可行，但后两者真正跑起来的门槛远超预期，更适合作为一次公开的技术折腾记录。
- **初心**：**实现「macOS 风格化」的桌面体验**——最在意的其实是第 4 项那个 Xfce + WhiteSur 的 macOS 画风桌面。
- **后续**：这个想法会落到**下一个专门的「Linux 美化」项目**里，专心打磨一套好用的 macOS 风格 Linux 桌面。本项目是它的「前传」。

### 5. 技术实现（简述）

- 引导：GRUB2 hybrid ISO（BIOS El Torito + UEFI `efi.img`），`grub-mkrescue` 生成，双模式可引导。
- Linux 内核：自编译 **Linux 6.12.4**，`=y` 驱动从 1631 项扩到 **2003 项**（WiFi / 蓝牙 / AMD·NVIDIA·Intel 显卡 / 全系 HDA 音频 / 触控板 / USB 网卡 / NVMe / exFAT / btrfs / xfs…），力求「不只在一块板子上能跑」。
- 固件：打包精简版 `linux-firmware`（约 382MB），并开启内核 `FW_LOADER_COMPRESS_ZSTD`。
- rootfs：squashfs 只读镜像 + overlay 可写层（tmpfs）；initramfs 支持 `LABEL=` 与 `/dev/sdXN` 两种定位。
- 桌面：Xfce + **WhiteSur**（macOS 风格 GTK 主题 / 图标 / 光标）；输入兼容性用「手工生成 udev 数据库」解决（无 udevd 也能识别键鼠）。

### 6. 交付物

| 文件 | 说明 | 校验 (SHA-256) |
|------|------|------|
| `SuperSystem_Installer.exe` | 全内置安装器（ISO 已嵌在 EXE 内，按 Start 即开始制作） | `69cba00e6cf0d39594913d2a5e92764c5868d841bdf34650bac6677ab96b0735` |
| `SuperSystem_USB_Installer.exe` | 分体安装器（需旁边放 `stage6.iso`） | `9b4ee18d59c4e2d46d740f2170a11ac01559979bbe56e589954ff8c64e262d14` |
| `stage6.iso` | 主镜像（hybrid，BIOS + UEFI，1.19GB） | `fe6ae89dc77a1f02950edba358c3130f06a472b599a03d9fee57d607da49ebf0` |
| `README.md` | 本文档 | — |

### 7. 免责声明

本项目为个人技术实验，仅用于学习与研究，ReactOS / XNU 均使用官方开源代码或社区成果，不包含也不安装任何商业 Windows / macOS 系统。**请勿用于生产环境**；写盘前务必备份数据。

**切换到其它语言**：[English](#english) · [日本語](#日本語) · [한국어](#한국어) · [Deutsch](#deutsch) · [中文](#中文)

---
---

<!-- ==================== ENGLISH =================== -->

## English

> **One-liner**: A bootable ISO that puts three "kernels" — ReactOS, Darwin/XNU and Linux — into a single boot menu. It is a **playable / experimental hobby project**, whose original goal was to explore a "macOS-styled desktop".
>
> ⚠️ **Read this first**: This is a **toy / exploratory project**, not a daily-driver system. Apart from the two Linux entries, ReactOS and macOS **do not actually boot** (see below). Treat it as a technical feasibility experiment, not a product.

### 1. What it is

On boot, a GRUB text menu appears with 4 entries:

| # | Menu entry | Contents | Status |
|---|-----------|----------|--------|
| 1 | **Windows kernel (ReactOS x86)** | ReactOS 0.4.16 LiveCD | ❌ Basically unusable |
| 2 | **macOS kernel (Darwin/XNU x86-64)** | XNU kernel + custom xnu-loader | ❌ Unusable |
| 3 | **Linux kernel (CLI)** | Self-built Linux 6.12.4 + BusyBox shell | ✅ Works |
| 4 | **Linux kernel (GUI, Xfce macOS style)** | Self-built kernel + Xfce + WhiteSur theme | ✅ Works (default) |

The idea was "**three kernels, one consistent macOS-style desktop**": each kernel runs its own native software ecosystem, while the UI style stays the same.

### 2. Quick start: write the ISO to a USB drive

**You need**: a USB drive ≥ 2GB (**all data will be erased — back it up first**), a Windows PC, and admin rights.

1. Download the three files in the release package: `SuperSystem_USB_Installer.exe`, `stage6.iso`, `README.md`.
2. Double-click `SuperSystem_USB_Installer.exe` (it requests admin rights automatically).
3. Pick the image: it auto-finds `stage6.iso` next to itself; otherwise click "Browse…".
4. Pick the target USB drive: the dropdown shows `PhysicalDriveN | size | drive letter | model` (auto-scaled units). The **system disk is blocked automatically**.
5. Click "Start writing to USB": the progress area shows **written/total, speed and ETA**, then automatically **reads back and verifies the first 1MB**.
6. Boot it: set the USB drive as the first boot device in BIOS/UEFI (**disable Secure Boot** for UEFI), then choose entry 3 or 4 to enter the system.

> If Windows says the USB drive "needs to be formatted" after writing — **that is normal**. It is a hallmark of a hybrid ISO; the boot area is written correctly. Don't format it — just boot from it.
>
> You can also use **Ventoy**: simply copy `stage6.iso` onto a Ventoy drive, no installer needed.

### 3. Known issues (important)

- **Windows kernel (ReactOS) — ❌ does not boot on real hardware or USB**
  - Symptom: on real hardware it errors out and reboots after any key; on USB (both Ventoy and Rufus tried) it stops at `ReactOS 0.4.16 (isoboot)...` or shows `Unable to load second stage loader.`
  - Root cause: ReactOS 0.4.16's FreeLoader loads in two stages (`freeldr.sys` → `rosload.exe`), and the second stage fails outside a standard optical-disc environment; the official disc bootloader `isoboot.bin` **relies on El Torito CD state**, and a USB drive is not a real optical drive, so it hangs.
  - It only fully boots in a few environments "presented as a real optical disc" (e.g. QEMU with `-cdrom`). Fixing it requires the official ReactOS USB path (FAT32 + `fat32.bin`). **The project is paused; this is not implemented.**
- **macOS kernel (Darwin/XNU) — ❌ effectively does not load**
  - Symptom: crashes on real hardware; under UEFI the `xnu-loader.efi` starts, but the kernel then reports `unknown error`.
  - Root cause: `xnu-loader.efi` is an experimental EFI stub; more fundamentally, **XNU is not a general-purpose PC kernel** — it lacks the whole platform driver stack for a normal x86-64 PC ("Hackintosh" relies on a full macOS system plus many patches, not a bare XNU kernel). **Not realistic from an engineering standpoint.**
- **The two Linux entries — ✅ work**: verified on real hardware, VMs and USB (Ventoy / Rufus). The CLI entry adds `nomodeset` for old display devices; the GUI entry uses a KMS driver.

### 4. Positioning, original goal and what comes next

- **Positioning**: a **playable hobby project**. It proves that "three kernels in one boot menu" is feasible at the boot level, but making the latter two actually run is far harder than expected — so it works better as a public record of a technical experiment.
- **Original goal**: to **build a "macOS-styled" desktop experience** — what I cared about most is actually entry 4, the Xfce + WhiteSur macOS-look desktop.
- **Next**: that idea will move to a **dedicated "Linux beautification" project**, focused on polishing one great macOS-styled Linux desktop. This project is its "prequel".

### 5. Technical notes (brief)

- Boot: GRUB2 hybrid ISO (BIOS El Torito + UEFI `efi.img`), built with `grub-mkrescue`, bootable in both modes.
- Linux kernel: self-built **Linux 6.12.4**; `=y` drivers expanded from 1631 to **2003** (WiFi / Bluetooth / AMD·NVIDIA·Intel graphics / full HDA audio / touchpads / USB NICs / NVMe / exFAT / btrfs / xfs…), aiming to "run on more than one board".
- Firmware: a trimmed `linux-firmware` bundle (~382MB) plus kernel `FW_LOADER_COMPRESS_ZSTD`.
- rootfs: read-only squashfs image + overlay writable layer (tmpfs); initramfs supports both `LABEL=` and `/dev/sdXN`.
- Desktop: Xfce + **WhiteSur** (macOS-style GTK theme / icons / cursors); input compatibility solved by "hand-generating the udev database" (keyboard/mouse work even without udevd).

### 6. Deliverables

| File | Description | Checksum (SHA-256) |
|------|-------------|----------|
| `SuperSystem_Installer.exe` | All-in-one installer (ISO embedded in the EXE; just press Start) | `69cba00e6cf0d39594913d2a5e92764c5868d841bdf34650bac6677ab96b0735` |
| `SuperSystem_USB_Installer.exe` | Separate installer (needs `stage6.iso` beside it) | `9b4ee18d59c4e2d46d740f2170a11ac01559979bbe56e589954ff8c64e262d14` |
| `stage6.iso` | Main image (hybrid, BIOS + UEFI, 1.19GB) | `fe6ae89dc77a1f02950edba358c3130f06a472b599a03d9fee57d607da49ebf0` |
| `README.md` | This document | — |

### 7. Disclaimer

This is a personal technical experiment for learning and research only. ReactOS / XNU use official open-source code or community work; this project contains and installs no commercial Windows / macOS system. **Do not use it in production**; always back up your data before writing to a drive.

**Switch language**: [中文](#中文) · [日本語](#日本語) · [한국어](#한국어) · [Deutsch](#deutsch) · [English](#english)

---
---

<!-- ==================== 日本語 =================== -->

## 日本語

> **ひと言で**: 3 つの「カーネル」——ReactOS、Darwin/XNU、Linux——を 1 つの起動メニューに入れた起動可能 ISO です。**遊べる系の実験プロジェクト**で、元々の狙いは「macOS 風デスクトップ」の検証でした。
>
> ⚠️ **先にこれを**: これは**おもちゃ / 探索的プロジェクト**であり、日常使いのシステムではありません。Linux の 2 項目を除き、ReactOS と macOS は**現状まったく起動しません**（理由は下記）。「技術的な実現可能性の実験」として扱ってください。

### 1. これは何か

起動するとまず GRUB のテキストメニューが表示され、4 つの項目があります:

| # | メニュー項目 | 内容 | 状態 |
|---|-------------|------|------|
| 1 | **Windows カーネル（ReactOS x86）** | ReactOS 0.4.16 LiveCD | ❌ ほぼ使用不可 |
| 2 | **macOS カーネル（Darwin/XNU x86-64）** | XNU カーネル + 自作 xnu-loader | ❌ 使用不可 |
| 3 | **Linux カーネル（コマンドライン版）** | 自前ビルドの Linux 6.12.4 + BusyBox shell | ✅ 動作 |
| 4 | **Linux カーネル（GUI 版、Xfce macOS 風）** | 自前カーネル + Xfce + WhiteSur テーマ | ✅ 動作（既定）|

狙いは「**3 つのカーネル、1 つの macOS 風デスクトップ体験**」。カーネルごとに動くソフト資産は違っても、見た目のスタイルは揃える、という構想です。

### 2. クイックスタート：ISO を USB に入れる

**用意するもの**: 2GB 以上の USB メモリ（**中身は消去されます。先にバックアップを**）、Windows PC、管理者権限。

1. リリースの 3 ファイルを入手: `SuperSystem_USB_Installer.exe`、`stage6.iso`、`README.md`。
2. `SuperSystem_USB_Installer.exe` をダブルクリック（自動で管理者権限を要求）。
3. ISO を選択: 既定で同梱の `stage6.iso` を自動検出。無ければ「参照…」で選択。
4. 書き込み先 USB を選択: 一覧は `PhysicalDriveN | 容量 | ドライブ文字 | 型番`（容量単位は自動換算）。**システムディスクは自動で拒否**します。
5. 「書き込み開始」をクリック: 進捗は **書込済み/総量・速度・残り時間** を表示し、完了後に**先頭 1MB を読み戻して検証**します。
6. 起動: BIOS/UEFI で当該 USB を第一起動デバイスに設定（UEFI では **Secure Boot を無効化**）。項目 3 か 4 を選ぶとシステムに入れます。

> 書き込み後に Windows が「フォーマットが必要」と表示しても**正常です**。hybrid ISO の特徴で、起動領域は正しく書けています。フォーマットせずそのまま起動してください。
>
> **Ventoy** でも可: `stage6.iso` を Ventoy のドライブにコピーするだけ。インストーラ不要です。

### 3. 既知の問題（重要）

- **Windows カーネル（ReactOS）——❌ 実機 / USB で起動しない**
  - 症状: 実機では即エラー → 任意キーで再起動。USB（Ventoy・Rufus の両方を試）では `ReactOS 0.4.16 (isoboot)...` で停止、または `Unable to load second stage loader.` と表示。
  - 原因: ReactOS 0.4.16 の FreeLoader は 2 段階読み込み（`freeldr.sys` → `rosload.exe`）で、標準的な光ディスク環境以外では 2 段目が失敗。公式のディスク用ブートローダ `isoboot.bin` は **El Torito の CD 状態に依存**するため、USB（実際の光学ドライブではない）ではハングします。
  - 「実光ディスクとして提示される」限られた環境（例: QEMU の `-cdrom`）でのみ完全起動します。修正には ReactOS 公式の USB 方式（FAT32 + `fat32.bin`）が必要で、**本プロジェクトは停止中、未実装**です。
- **macOS カーネル（Darwin/XNU）——❌ 実質読み込み不可**
  - 症状: 実機では即クラッシュ。UEFI では `xnu-loader.efi` が起動するも、カーネル読み込み後に `unknown error`。
  - 原因: `xnu-loader.efi` は実験的な EFI スタブ。さらに本質的に **XNU は汎用 PC カーネルではありません**——通常の x86-64 PC 向けプラットフォームドライバ一式が無いためです（「Hackintosh」は完全な macOS システム + 多数のパッチがあって成立するもので、裸の XNU では無理）。**エンジニアリング的には非現実的**です。
- **Linux 2 項目——✅ 動作**: 実機・仮想マシン・USB（Ventoy / Rufus）で起動確認済み。コマンドライン版は古い表示機器向けに `nomodeset` を付加、GUI 版は KMS ドライバで表示を担当します。

### 4. 位置づけ・原点・その後

- **位置づけ**: **遊べる系プロジェクト**。「3 カーネルを 1 つの起動メニューに」という試みはブート層では可能だと示せましたが、後ろ 2 つを実際に走らせる難易度は想定を大きく超えました。よって**公開された技術的試行の記録**としての価値が中心です。
- **原点**: **「macOS 風デスクトップ体験」の実現**。実は一番こだわったのは項目 4 の Xfce + WhiteSur の macOS 風デスクトップです。
- **その後**: この構想は**次作の「Linux 専用・美化」プロジェクト**へ引き継ぎ、使いやすい macOS 風 Linux デスクトップをじっくり作り込みます。本作はその「前日譚」です。

### 5. 技術メモ（概要）

- ブート: GRUB2 hybrid ISO（BIOS El Torito + UEFI `efi.img`）。`grub-mkrescue` で生成、両モード対応。
- Linux カーネル: 自前ビルドの **Linux 6.12.4**。`=y` ドライバを 1631 → **2003** に拡大（WiFi / Bluetooth / AMD·NVIDIA·Intel GPU / 全 HDA オーディオ / タッチパッド / USB NIC / NVMe / exFAT / btrfs / xfs…）。「1 枚のボードだけで動く」状態を避ける狙い。
- ファームウェア: 絞り込み版 `linux-firmware`（約 382MB）＋ カーネル `FW_LOADER_COMPRESS_ZSTD` を有効化。
- rootfs: 読み取り専用 squashfs + overlay 書込層（tmpfs）。initramfs は `LABEL=` と `/dev/sdXN` の両方式に対応。
- デスクトップ: Xfce + **WhiteSur**（macOS 風 GTK テーマ / アイコン / カーソル）。入力互換は「udev データベースを手動生成」で解決（udevd 無しでもキーボード・マウスを認識）。

### 6. 成果物

| ファイル | 説明 | チェックサム (SHA-256) |
|---------|------|-------------|
| `SuperSystem_Installer.exe` | オールインワン版（ISO を EXE に内蔵、Start を押すだけ） | `69cba00e6cf0d39594913d2a5e92764c5868d841bdf34650bac6677ab96b0735` |
| `SuperSystem_USB_Installer.exe` | 分離版（隣に `stage6.iso` が必要） | `9b4ee18d59c4e2d46d740f2170a11ac01559979bbe56e589954ff8c64e262d14` |
| `stage6.iso` | メインイメージ（hybrid、BIOS + UEFI、1.19GB） | `fe6ae89dc77a1f02950edba358c3130f06a472b599a03d9fee57d607da49ebf0` |
| `README.md` | 本文書 | — |

### 7. 免責

本プロジェクトは個人の技術実験であり、学習・研究目的に限ります。ReactOS / XNU は公式のオープンソースコードまたはコミュニティ成果を利用しており、商用の Windows / macOS システムを同梱もインストールもしません。**本番環境での使用はお控えください**。書き込み前に必ずデータをバックアップしてください。

**言語を切り替え**: [中文](#中文) · [English](#english) · [한국어](#한국어) · [Deutsch](#deutsch) · [日本語](#日本語)

---
---

<!-- ==================== 한국어 =================== -->

## 한국어

> **한 줄 요약**: ReactOS, Darwin/XNU, Linux 세 가지 "커널"을 하나의 부팅 메뉴에 담은 부팅 가능 ISO입니다. **가지고 노는 실험용 프로젝트**이며, 원래 목표는 "macOS 스타일 데스크톱" 검증이었습니다.
>
> ⚠️ **먼저 이 문장을**: 이것은 **장난감 / 탐색적 프로젝트**이며 일상용 시스템이 아닙니다. Linux 두 항목을 제외하면 ReactOS와 macOS는 **현재 전혀 부팅되지 않습니다**(아래 참고). "기술적 실현 가능성 실험"으로 봐 주세요.

### 1. 무엇인가

부팅하면 먼저 GRUB 텍스트 메뉴가 나오고, 항목은 4개입니다:

| # | 메뉴 항목 | 내용 | 상태 |
|---|-----------|------|------|
| 1 | **Windows 커널 (ReactOS x86)** | ReactOS 0.4.16 LiveCD | ❌ 사실상 사용 불가 |
| 2 | **macOS 커널 (Darwin/XNU x86-64)** | XNU 커널 + 자작 xnu-loader | ❌ 사용 불가 |
| 3 | **Linux 커널 (CLI)** | 직접 빌드한 Linux 6.12.4 + BusyBox shell | ✅ 동작 |
| 4 | **Linux 커널 (GUI, Xfce macOS 스타일)** | 직접 빌드한 커널 + Xfce + WhiteSur 테마 | ✅ 동작(기본) |

구상은 "**세 커널, 하나의 macOS 스타일 데스크톱**": 커널마다 네이티브 소프트웨어 생태계는 달라도 UI 스타일은 통일합니다.

### 2. 빠른 시작: ISO를 USB에 넣기

**준비물**: 2GB 이상 USB 메모리(**데이터는 삭제됩니다. 먼저 백업**), Windows PC, 관리자 권한.

1. 릴리스의 세 파일을 받습니다: `SuperSystem_USB_Installer.exe`, `stage6.iso`, `README.md`.
2. `SuperSystem_USB_Installer.exe`를 더블클릭(자동으로 관리자 권한 요청).
3. 이미지 선택: 기본으로 같은 위치의 `stage6.iso`를 자동 탐색, 없으면 "찾아보기…"로 선택.
4. 대상 USB 선택: 목록은 `PhysicalDriveN | 용량 | 드라이브 문자 | 모델`(용량 단위 자동 환산). **시스템 디스크는 자동 차단**됩니다.
5. "USB에 쓰기 시작" 클릭: 진행 영역에 **기록량/총량, 속도, 남은 시간**을 표시하고, 완료 후 **앞 1MB를 되읽어 검증**합니다.
6. 부팅: BIOS/UEFI에서 해당 USB를 첫 부팅 장치로 설정(UEFI는 **Secure Boot 해제**), 항목 3 또는 4를 선택하면 시스템에 들어갑니다.

> 기록 후 Windows가 USB를 "포맷해야 함"으로 표시해도 **정상입니다**. hybrid ISO의 특징이며 부팅 영역은 정상 기록된 것입니다. 포맷하지 말고 그대로 부팅하세요.
>
> **Ventoy**도 가능: `stage6.iso`를 Ventoy 드라이브에 복사하기만 하면 됩니다. 설치 프로그램 불필요.

### 3. 알려진 문제 (중요)

- **Windows 커널 (ReactOS) — ❌ 실기 / USB에서 부팅 안 됨**
  - 증상: 실기에서는 즉시 오류 후 아무 키나 누르면 재부팅. USB(Ventoy·Rufus 모두 시도)에서는 `ReactOS 0.4.16 (isoboot)...`에서 멈추거나 `Unable to load second stage loader.` 표시.
  - 원인: ReactOS 0.4.16의 FreeLoader는 2단계 로딩(`freeldr.sys` → `rosload.exe`)이며 표준 광디스크 환경이 아니면 2단계가 실패합니다. 공식 디스크 부트로더 `isoboot.bin`은 **El Torito CD 상태에 의존**하는데 USB는 실제 광학 드라이브가 아니라서 멈춥니다.
  - "실제 광디스크로 인식되는" 소수 환경(예: QEMU `-cdrom`)에서만 완전 부팅됩니다. 수정에는 ReactOS 공식 USB 방식(FAT32 + `fat32.bin`)이 필요하며, **이 프로젝트는 중단 상태로 미구현**입니다.
- **macOS 커널 (Darwin/XNU) — ❌ 실질적으로 로드 불가**
  - 증상: 실기에서는 즉시 크래시. UEFI에서는 `xnu-loader.efi`가 뜨지만 커널 로드 후 `unknown error`.
  - 원인: `xnu-loader.efi`는 실험적 EFI 스텁이고, 더 근본적으로 **XNU는 범용 PC 커널이 아닙니다** — 일반 x86-64 PC용 플랫폼 드라이버 일습이 없습니다("해킨토시"는 완전한 macOS 시스템 + 다수의 패치로 성립하는 것이고, 맨 XNU로는 불가). **공학적으로 비현실적**입니다.
- **Linux 두 항목 — ✅ 동작**: 실기·가상머신·USB(Ventoy / Rufus)에서 부팅 확인. CLI 버전은 구형 디스플레이용으로 `nomodeset` 추가, GUI 버전은 KMS 드라이버로 화면 처리.

### 4. 위치·초심·다음

- **위치**: **가지고 노는 프로젝트**. "세 커널을 한 부팅 메뉴에"라는 시도가 부팅 계층에서는 가능함을 보였지만, 뒤 두 개를 실제로 돌리는 난이도는 예상을 크게 넘었습니다. 그래서 **공개된 기술 실험 기록**으로서의 가치가 중심입니다.
- **초심**: **"macOS 스타일 데스크톱 경험" 구현**. 사실 가장 신경 쓴 것은 항목 4의 Xfce + WhiteSur macOS 스타일 데스크톱입니다.
- **다음**: 이 구상은 **다음 작품인 "Linux 전용 미화(꾸미기)" 프로젝트**로 이어져, 쓰기 좋은 macOS 스타일 Linux 데스크톱을 제대로 다듬습니다. 본 프로젝트는 그 "프리퀄"입니다.

### 5. 기술 메모 (요약)

- 부팅: GRUB2 hybrid ISO(BIOS El Torito + UEFI `efi.img`), `grub-mkrescue` 생성, 양 모드 부팅.
- Linux 커널: 직접 빌드한 **Linux 6.12.4**. `=y` 드라이버를 1631 → **2003**으로 확대(WiFi / Bluetooth / AMD·NVIDIA·Intel GPU / 전 HDA 오디오 / 터치패드 / USB NIC / NVMe / exFAT / btrfs / xfs…). "한 보드에서만 돌아가는" 상태를 피하려는 목표.
- 펌웨어: 축약판 `linux-firmware`(약 382MB) + 커널 `FW_LOADER_COMPRESS_ZSTD` 활성화.
- rootfs: 읽기 전용 squashfs + overlay 쓰기 레이어(tmpfs). initramfs는 `LABEL=`과 `/dev/sdXN` 둘 다 지원.
- 데스크톱: Xfce + **WhiteSur**(macOS 스타일 GTK 테마 / 아이콘 / 커서). 입력 호환은 "udev 데이터베이스 수동 생성"으로 해결(udevd 없이도 키보드·마우스 인식).

### 6. 산출물

| 파일 | 설명 | 체크섬 (SHA-256) |
|------|------|--------|
| `SuperSystem_Installer.exe` | 올인원 설치 프로그램(ISO가 EXE에 내장, Start만 누르면 됨) | `69cba00e6cf0d39594913d2a5e92764c5868d841bdf34650bac6677ab96b0735` |
| `SuperSystem_USB_Installer.exe` | 분리형 설치 프로그램(옆에 `stage6.iso` 필요) | `9b4ee18d59c4e2d46d740f2170a11ac01559979bbe56e589954ff8c64e262d14` |
| `stage6.iso` | 메인 이미지(hybrid, BIOS + UEFI, 1.19GB) | `fe6ae89dc77a1f02950edba358c3130f06a472b599a03d9fee57d607da49ebf0` |
| `README.md` | 본 문서 | — |

### 7. 면책

본 프로젝트는 개인 기술 실험이며 학습·연구 목적에 한합니다. ReactOS / XNU는 공식 오픈소스 코드 또는 커뮤니티 성과를 사용하며, 상용 Windows / macOS 시스템을 포함하거나 설치하지 않습니다. **운영 환경에서 사용하지 마세요**. 기록 전 반드시 데이터를 백업하세요.

**언어 전환**: [中文](#中文) · [English](#english) · [日本語](#日本語) · [Deutsch](#deutsch) · [한국어](#한국어)

---
---

<!-- ==================== DEUTSCH =================== -->

## Deutsch

> **In einem Satz**: Ein bootfähiges ISO, das drei „Kernel" — ReactOS, Darwin/XNU und Linux — in ein einziges Boot-Menü packt. Es ist ein **spielerisches Experimentierprojekt**, dessen ursprüngliches Ziel ein „macOS-artiger Desktop" war.
>
> ⚠️ **Bitte zuerst lesen**: Das ist ein **Spielzeug- / Erforschungsprojekt**, kein Alltagssystem. Außer den beiden Linux-Einträgen **starten ReactOS und macOS derzeit nicht** (siehe unten). Bitte als technische Machbarkeits-Studie betrachten, nicht als Produkt.

### 1. Was ist das

Beim Start erscheint ein GRUB-Textmenü mit 4 Einträgen:

| # | Menüeintrag | Inhalt | Status |
|---|-------------|--------|--------|
| 1 | **Windows-Kernel (ReactOS x86)** | ReactOS 0.4.16 LiveCD | ❌ Praktisch unbrauchbar |
| 2 | **macOS-Kernel (Darwin/XNU x86-64)** | XNU-Kernel + eigener xnu-loader | ❌ Nicht nutzbar |
| 3 | **Linux-Kernel (Kommandozeile)** | Selbst gebauter Linux 6.12.4 + BusyBox-Shell | ✅ Funktioniert |
| 4 | **Linux-Kernel (GUI, Xfce im macOS-Stil)** | Selbst gebauter Kernel + Xfce + WhiteSur-Theme | ✅ Funktioniert (Standard) |

Die Idee war „**drei Kernel, ein einheitlicher macOS-Desktop-Look**": Jeder Kernel bringt sein eigenes Software-Ökosystem mit, der Stil der Oberfläche bleibt gleich.

### 2. Schnellstart: ISO auf einen USB-Stick schreiben

**Benötigt**: ein USB-Stick ≥ 2 GB (**alle Daten werden gelöscht – vorher sichern**), ein Windows-PC, Administratorrechte.

1. Die drei Dateien aus dem Release herunterladen: `SuperSystem_USB_Installer.exe`, `stage6.iso`, `README.md`.
2. `SuperSystem_USB_Installer.exe` doppelklicken (fordert automatisch Admin-Rechte an).
3. Abbild wählen: `stage6.iso` wird neben der EXE automatisch gefunden; sonst „Durchsuchen…".
4. Ziel-USB-Stick wählen: Die Liste zeigt `PhysicalDriveN | Größe | Laufwerksbuchstabe | Modell` (Einheiten werden automatisch skaliert). **Die Systemplatte wird automatisch blockiert.**
5. „Auf USB schreiben starten" klicken: Der Fortschritt zeigt **geschrieben/gesamt, Geschwindigkeit und Restzeit**, danach wird **das erste 1 MB zurückgelesen und verifiziert**.
6. Booten: im BIOS/UEFI den Stick als erstes Boot-Gerät einstellen (bei UEFI **Secure Boot deaktivieren**), dann Eintrag 3 oder 4 wählen.

> Meldet Windows nach dem Schreiben, der Stick „müsse formatiert werden" – **das ist normal**. Das ist typisch für ein Hybrid-ISO; der Boot-Bereich ist korrekt geschrieben. Nicht formatieren, einfach davon booten.
>
> **Ventoy** geht auch: `stage6.iso` einfach auf einen Ventoy-Stick kopieren, kein Installer nötig.

### 3. Bekannte Probleme (wichtig)

- **Windows-Kernel (ReactOS) — ❌ startet auf echter Hardware / USB nicht**
  - Symptom: Auf echter Hardware Fehler und Neustart nach beliebiger Taste; auf USB (Ventoy und Rufus getestet) bleibt es bei `ReactOS 0.4.16 (isoboot)...` stehen oder zeigt `Unable to load second stage loader.`
  - Ursache: ReactOS 0.4.16 lädt FreeLoader zweistufig (`freeldr.sys` → `rosload.exe`); die zweite Stufe scheitert außerhalb einer Standard-CD-Umgebung. Der offizielle CD-Bootloader `isoboot.bin` **hängt vom El-Torito-CD-Status ab**, und ein USB-Stick ist kein echtes optisches Laufwerk – also hängt es.
  - Vollständig startet es nur in wenigen „als echte CD präsentierten" Umgebungen (z. B. QEMU mit `-cdrom`). Eine Behebung erfordert den offiziellen ReactOS-USB-Weg (FAT32 + `fat32.bin`). **Das Projekt ist pausiert; nicht umgesetzt.**
- **macOS-Kernel (Darwin/XNU) — ❌ lädt faktisch nicht**
  - Symptom: Absturz auf echter Hardware; unter UEFI startet der `xnu-loader.efi`, danach meldet der Kernel `unknown error`.
  - Ursache: `xnu-loader.efi` ist ein experimenteller EFI-Stub; grundlegender ist **XNU kein Allzweck-PC-Kernel** – es fehlt der komplette Plattform-Treiberstapel für einen normalen x86-64-PC (ein „Hackintosh" lebt von einem vollen macOS-System plus vielen Patches, nicht von einem nackten XNU-Kernel). **Technisch unrealistisch.**
- **Die zwei Linux-Einträge — ✅ funktionieren**: Verifiziert auf echter Hardware, VMs und USB (Ventoy / Rufus). Der CLI-Eintrag nutzt `nomodeset` für alte Anzeigegeräte; der GUI-Eintrag nutzt einen KMS-Treiber.

### 4. Einordnung, ursprüngliche Idee und was folgt

- **Einordnung**: ein **spielerisches Hobby-Projekt**. Es zeigt, dass „drei Kernel in einem Boot-Menü" auf Boot-Ebene machbar ist – die beiden letzteren wirklich zum Laufen zu bringen war aber weit schwieriger als erwartet. Daher ist es vor allem ein **öffentliches Protokoll eines technischen Experiments**.
- **Ursprüngliche Idee**: eine **„macOS-artige" Desktop-Erfahrung** zu bauen – am wichtigsten war mir tatsächlich Eintrag 4, der Xfce + WhiteSur-Desktop im macOS-Look.
- **Danach**: Diese Idee wandert in ein **eigenes „Linux-Verschönerungs"-Projekt**, das einen richtig guten Linux-Desktop im macOS-Stil ausarbeitet. Dieses Projekt ist dessen „Vorgeschichte".

### 5. Technische Notizen (kurz)

- Boot: GRUB2 Hybrid-ISO (BIOS El Torito + UEFI `efi.img`), erzeugt mit `grub-mkrescue`, in beiden Modi bootfähig.
- Linux-Kernel: selbst gebauter **Linux 6.12.4**; `=y`-Treiber von 1631 auf **2003** erweitert (WiFi / Bluetooth / AMD·NVIDIA·Intel-Grafik / alle HDA-Audio-Codecs / Touchpads / USB-NICs / NVMe / exFAT / btrfs / xfs…), damit es „auf mehr als einem Board" läuft.
- Firmware: reduziertes `linux-firmware`-Paket (~382 MB) plus Kernel `FW_LOADER_COMPRESS_ZSTD`.
- rootfs: schreibgeschütztes squashfs + overlay-Schreibschicht (tmpfs); initramfs unterstützt `LABEL=` und `/dev/sdXN`.
- Desktop: Xfce + **WhiteSur** (GTK-Theme / Icons / Cursor im macOS-Stil); Eingabe-Kompatibilität über „manuell erzeugte udev-Datenbank" gelöst (Tastatur/Maus funktionieren auch ohne udevd).

### 6. Lieferumfang

| Datei | Beschreibung | Prüfsumme (SHA-256) |
|-------|--------------|-----------|
| `SuperSystem_Installer.exe` | All-in-One-Installer (ISO in der EXE eingebettet; nur Start drücken) | `69cba00e6cf0d39594913d2a5e92764c5868d841bdf34650bac6677ab96b0735` |
| `SuperSystem_USB_Installer.exe` | Separater Installer (benötigt `stage6.iso` daneben) | `9b4ee18d59c4e2d46d740f2170a11ac01559979bbe56e589954ff8c64e262d14` |
| `stage6.iso` | Haupt-Image (hybrid, BIOS + UEFI, 1,19 GB) | `fe6ae89dc77a1f02950edba358c3130f06a472b599a03d9fee57d607da49ebf0` |
| `README.md` | Dieses Dokument | — |

### 7. Haftungsausschluss

Dies ist ein persönliches technisches Experiment, nur für Lernen und Forschung. ReactOS / XNU nutzen offiziellen Open-Source-Code bzw. Community-Arbeit; dieses Projekt enthält und installiert kein kommerzielles Windows-/macOS-System. **Nicht in Produktivumgebungen verwenden**; vor dem Schreiben stets Daten sichern.

**Sprache wechseln**: [中文](#中文) · [English](#english) · [日本語](#日本語) · [한국어](#한국어) · [Deutsch](#deutsch)

---
---

## 许可 / License · 第三方组件 / Third-Party Notices

**本项目自有部分**（构建脚本、安装器、init 脚本、文档、主题整合等）采用
**CC BY-NC-SA 4.0**（署名—非商业性使用—相同方式共享 4.0 国际）许可。

- 你可以自由**查看、学习、复制、修改、再分发**；
- **不得用于商业用途**；
- 分发/修改时须**署名**并以**相同方式共享**。
- 许可全文：https://creativecommons.org/licenses/by-nc-sa/4.0/ · 中文摘要：https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh

> ⚠️ **重要**：ISO 内**打包的第三方开源组件仍适用其各自的许可证**。其中多数（Linux 内核、BusyBox、GRUB2、Xfce、WhiteSur 主题等）为 GPL 系许可，而 **GPL 本身允许商业使用**。因此本「非商业」条款**只能约束本项目自有代码**，无法也不应据此限制这些第三方组件的商业使用。若需商用，请分别遵守各组件的许可证。

### 第三方组件 / Third-Party Components

| 组件 / Component | 许可证 / License | 来源 / Source |
|---|---|---|
| Linux kernel 6.12.4 | GPL-2.0 | https://kernel.org |
| BusyBox | GPL-2.0 | https://busybox.net |
| GRUB 2 | GPL-3.0 | https://www.gnu.org/software/grub |
| Xfce | GPL / LGPL / BSD | https://xfce.org |
| WhiteSur GTK Theme | GPL-3.0 | https://github.com/vinceliuice/WhiteSur-gtk-theme |
| ReactOS 0.4.16 | GPL-2.0 / LGPL-2.1 | https://reactos.org |
| XNU（Darwin） | APSL-2.0 | https://github.com/apple-oss-distributions/xnu |
| linux-firmware | 多种（多为厂商专有，允许再分发）/ various | https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git |
| xnu-loader | 见项目仓库 / see repo LICENSE | https://github.com/PureDarwin |

> GPL 组件对应的源代码可通过上表链接获取；本项目所用 Linux 内核源码一并附于构建工程内（`iso_build/kernels/linux/src/linux-6.12.4`）。

**各语言简版 / In other languages:**

- **English**: My own parts (build scripts, installer, init scripts, docs, theme integration) are licensed under **CC BY-NC-SA 4.0** — free to study, modify and share, but **no commercial use**. Bundled third-party components keep their own licenses (mostly GPL), which **do permit commercial use**.
- **日本語**: 自作部分は **CC BY-NC-SA 4.0**（非営利）。同梱の第三者コンポーネントは各自のライセンス（多くは GPL、商用可）に従います。
- **한국어**: 자체 제작 부분은 **CC BY-NC-SA 4.0**(비상업). 번들된 서드파티 구성 요소는 각자의 라이선스(대부분 GPL, 상업적 사용 허용)를 따릅니다.
- **Deutsch**: Der eigene Code steht unter **CC BY-NC-SA 4.0** (nicht kommerziell). Mitgelieferte Drittanbieter-Komponenten behalten ihre eigenen Lizenzen (überwiegend GPL, kommerzielle Nutzung erlaubt).
- **中文**: 见上方「许可」小节。

> 提示 / Note: 非商业许可证不属于 OSI 认可的开源许可（Open Source），GitHub 会把本项目标记为「非开源」。若你日后想让它成为真正的开源项目，可考虑改用 MIT（允许商用）或 GPL-3.0（允许商用但要求衍生开源）。

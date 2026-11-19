# Big Walk macOS：通过 IP 加入游戏

# Big Walk on macOS: Join a Game by IP

两边都使用 **macOS**，并且 Steam 里的 Big Walk 都是 **1.6.0**。这份仓库是已经配好的 macOS 模组文件，里面包含 BepInEx，以及「通过 IP 加入」。

Both players use **macOS**, and both copies of Big Walk on Steam are **1.6.0**. This repository is a ready-made macOS mod setup. It includes BepInEx and **Join via IP**.

Apple 芯片的 Mac 会通过 Rosetta 以 x86_64 运行。第一次启动 Intel 程序时，如果系统要求安装 Rosetta，先装完再继续。

On Apple silicon, the game runs as x86_64 through Rosetta. If macOS asks to install Rosetta the first time an Intel app starts, install it before continuing.

| 内容 / Included | 版本 / Version | 作用 / What it does |
|---|---|---|
| Direct Connect | 1.0.0 | 在「加入游戏」页面增加「通过 IP 加入」 / Adds **Join via IP** on the Join Game page |
| Mod Settings Menu | 1.1.5 | 主菜单和暂停菜单里的 **Mod Settings** / **Mod Settings** in the main menu and pause menu |
| BigWalkLocalizationAPI | 1.1.0 | Mod Settings Menu 的依赖 / Required by Mod Settings Menu |
| BepInEx IL2CPP | 6.0.0-be.755 | 模组加载器，附带 .NET 运行时 / Mod loader, with the bundled .NET runtime |

这套文件对应的游戏版本是 **1.6.0**。游戏更新之后可能需要新的文件，旧文件会加载失败。

This pack matches game version **1.6.0**. After a game update it can stop loading, and will need a new pack.

---

## 中文

### 第一步：用 Steam 准备好游戏

1. 在 Steam 里安装 Big Walk，并更新到 **1.6.0**。
2. 用 Steam 的 **Play** 正常启动一次，进入主菜单后退出。
3. 在 Steam 里右键 Big Walk → **管理** → **浏览本地文件**。这个文件夹就是游戏根目录，里面应该已经有 `Big Walk.app`。

默认路径：

```text
/Users/你的用户名/Library/Application Support/Steam/steamapps/common/Big Walk
```

### 第二步：下载并复制 macOS 预配置文件

在本仓库的 GitHub 页面选择 **Code → Download ZIP**，然后解压。

解压后的顶层文件夹名字是仓库名（例如 `big-walk-macos-main`），不是 `Big Walk`。把这个文件夹**里面的内容**复制到 Steam 的游戏根目录，也就是已经有 `Big Walk.app` 的那个文件夹。

不要替换、不要删除 `Big Walk.app`。

如果游戏目录里已经有别的模组，把文件夹合并进去，保留你原来的 `BepInEx/plugins`。不要用一份新的官方 BepInEx 覆盖这里的 `BepInEx/core`。

最终目录结构应该类似：

```text
Big Walk/
│
├── Big Walk.app/
│
├── BepInEx/
│   ├── config/
│   ├── core/
│   ├── interop/
│   └── plugins/
│       ├── BigWalk.DirectConnect/
│       ├── BigWalkLocalizationAPI/
│       └── ModSettingsMenu/
│
├── dotnet/
├── Data -> Big Walk.app/Contents/Resources/Data
├── GameAssembly.dylib -> Big Walk.app/Contents/Frameworks/GameAssembly.dylib
├── libdoorstop.dylib
└── run_bepinex.sh
```

`Data` 和 `GameAssembly.dylib` 必须是指向 `Big Walk.app` 里面的符号链接。压缩包有时会把链接变成普通文件。复制完成后，在终端执行下面的命令，把链接修好，并给启动脚本加上可执行权限。

Steam 库在默认位置时：

```sh
GAME="$HOME/Library/Application Support/Steam/steamapps/common/Big Walk"
cd "$GAME"

if [ -e Data ] && [ ! -L Data ]; then
  echo "Data 已经是真实文件夹，不要删除。确认这是游戏根目录后再继续。"
  exit 1
fi

rm -f Data GameAssembly.dylib
ln -s "Big Walk.app/Contents/Resources/Data" Data
ln -s "Big Walk.app/Contents/Frameworks/GameAssembly.dylib" GameAssembly.dylib
chmod +x "$GAME/run_bepinex.sh"
```

如果 Steam 库安装在其他磁盘，把 `GAME=` 换成实际路径。终端里可以用 `pwd` 查看当前目录。

### 第三步：配置 Steam 启动参数

打开 Steam：

**Big Walk → 属性 → 通用 → 启动选项**。

填入：

```text
"/Users/你的用户名/Library/Application Support/Steam/steamapps/common/Big Walk/run_bepinex.sh" %command%
```

这里必须使用绝对路径。保留两边的引号，最后要有 `%command%`。Steam **不会**展开 `~`。

在终端执行 `whoami` 可以看自己的用户名。例如用户名是 `mingzhefeng`：

```text
"/Users/mingzhefeng/Library/Application Support/Steam/steamapps/common/Big Walk/run_bepinex.sh" %command%
```

Steam 库不在默认位置时，把整段路径换成第二步里那个真实的游戏根目录。

### 第四步：启动游戏

通过 Steam 的 **Play** 按钮启动。不要直接双击 `Big Walk.app`。

如果安装成功，游戏主菜单或暂停菜单中应该出现：

**Mod Settings**

这说明 BepInEx 已经加载。

然后打开 **加入游戏**。在「通过代码加入」和好友列表之间，应该有一块 **通过 IP 加入**。这才说明 Direct Connect 已经加载。

没出现时，把游戏完整退出，再看游戏根目录里的 `BepInEx/LogOutput.log`。

### 第五步：通过 IP 加入

房主和加入的人都要完成本页的安装，并且 `SendRate` 保持一样。默认是 `60`。两边不一样时，角色移动可能会发飘。

1. 房主在游戏里照常开一局，并设置房间密码。这款游戏的主机需要密码。
2. 加入的人必须能访问房主电脑的 **UDP 7777**。
   - 同一局域网：填房主的局域网 IP，例如 `192.168.1.10`。Mac 上可以在终端执行 `ipconfig getifaddr en0` 查看。
   - 不在同一个网络：在房主路由器上把 **UDP 7777** 转发到房主电脑，或者两人都连接 Tailscale、ZeroTier 这类 VPN，然后填写 VPN 分配的地址。
3. 加入的人进入 **加入游戏**，在 **通过 IP 加入** 里填写房主的 IP（或域名）和房间密码，然后加入。

可以填写 IPv4（`192.168.1.10`）、IPv6 或域名。

按 **F5** 可以隐藏或显示游戏里的红色 `WARNING` 提示。那些提示来自 Epic 在线服务，不代表这次 IP 连接失败。

### 请注意

- 通过 IP 加入只适用于电脑玩家。主机玩家仍然使用房间码。
- 向游戏开发者报告问题之前，先卸掉全部模组，确认问题仍然存在。模组问题不要报给 House House。
- 本模组与 House House、Panic 无关。
- 更高的发送频率会多用一点带宽和房主的 CPU。

---

## English

### Step 1: Prepare the game in Steam

1. Install Big Walk in Steam and update it to **1.6.0**.
2. Start it once with Steam's **Play** button, reach the main menu, then quit.
3. In Steam, right-click Big Walk → **Manage** → **Browse local files**. That folder is the game root. It should already contain `Big Walk.app`.

The default path is:

```text
/Users/YOUR_USERNAME/Library/Application Support/Steam/steamapps/common/Big Walk
```

### Step 2: Download the macOS files and copy them in

On this repository's GitHub page, choose **Code → Download ZIP**, then unzip it.

The unzipped top folder is named after the repository (for example `big-walk-macos-main`), not `Big Walk`. Copy the **contents** of that folder into the Steam game root, the folder that already contains `Big Walk.app`.

Do not replace or delete `Big Walk.app`.

If you already have other mods, merge the folders and keep your existing `BepInEx/plugins`. Do not install a stock BepInEx package over `BepInEx/core`.

The final layout should look like this:

```text
Big Walk/
│
├── Big Walk.app/
│
├── BepInEx/
│   ├── config/
│   ├── core/
│   ├── interop/
│   └── plugins/
│       ├── BigWalk.DirectConnect/
│       ├── BigWalkLocalizationAPI/
│       └── ModSettingsMenu/
│
├── dotnet/
├── Data -> Big Walk.app/Contents/Resources/Data
├── GameAssembly.dylib -> Big Walk.app/Contents/Frameworks/GameAssembly.dylib
├── libdoorstop.dylib
└── run_bepinex.sh
```

`Data` and `GameAssembly.dylib` must be symbolic links into `Big Walk.app`. Some unzip tools turn those links into ordinary files. After copying, run this in Terminal. It recreates the links and marks the launcher executable.

For a Steam library in the default location:

```sh
GAME="$HOME/Library/Application Support/Steam/steamapps/common/Big Walk"
cd "$GAME"

if [ -e Data ] && [ ! -L Data ]; then
  echo "Data is a real folder. Do not delete it. Make sure this is the game root, then stop."
  exit 1
fi

rm -f Data GameAssembly.dylib
ln -s "Big Walk.app/Contents/Resources/Data" Data
ln -s "Big Walk.app/Contents/Frameworks/GameAssembly.dylib" GameAssembly.dylib
chmod +x "$GAME/run_bepinex.sh"
```

If the Steam library is on another disk, change the `GAME=` line to that path. `pwd` prints the current directory.

### Step 3: Set the Steam launch option

In Steam, open:

**Big Walk → Properties → General → Launch Options**.

Enter:

```text
"/Users/YOUR_USERNAME/Library/Application Support/Steam/steamapps/common/Big Walk/run_bepinex.sh" %command%
```

This must be an absolute path. Keep the quotes, and keep `%command%` at the end. Steam does **not** expand `~`.

Run `whoami` in Terminal to see the username. If the username is `mingzhefeng`:

```text
"/Users/mingzhefeng/Library/Application Support/Steam/steamapps/common/Big Walk/run_bepinex.sh" %command%
```

If the Steam library is somewhere else, use the real game-root path from Step 2.

### Step 4: Start the game

Start it with Steam's **Play** button. Do not double-click `Big Walk.app`.

When the install works, the main menu or the pause menu shows:

**Mod Settings**

That means BepInEx loaded.

Then open **Join Game**. Between join-by-code and the friends list, there should be a **Join via IP** section (在中文界面里是「通过 IP 加入」). That means Direct Connect loaded.

If it does not appear, quit the game completely and read `BepInEx/LogOutput.log` in the game root.

### Step 5: Join by IP

The host and the joining player both need this pack installed, with the same `SendRate`. The default is `60`. Different values can make movement look jittery.

1. The host starts a game as usual and sets a room password. A Big Walk host always requires a password.
2. The joining player must be able to reach the host on **UDP port 7777**.
   - Same local network: use the host's LAN address, such as `192.168.1.10`. On the host Mac, `ipconfig getifaddr en0` prints that address.
   - Different networks: forward **UDP 7777** on the host's router to the host computer, or connect both players with a VPN such as Tailscale or ZeroTier and use the VPN address.
3. The joining player opens **Join Game**, enters the host IP (or a domain name) and the room password under **Join via IP**, then joins.

IPv4 (`192.168.1.10`), IPv6, and domain names all work.

Press **F5** to hide or show the game's red `WARNING` banners. Those banners come from Epic Online Services. They do not mean the IP connection failed.

### Notes

- Joining by IP is for computer players. Console players still join with a room code.
- Before reporting a bug to the game's developers, remove every mod and check whether the problem remains. Report mod problems to the mod author, not to House House.
- This mod is not affiliated with or endorsed by House House or Panic.
- A higher send rate uses more bandwidth and a little more CPU on the host.

---

## 配置 / Configuration

第一次成功启动后会生成：

This file is created after the first successful launch:

`BepInEx/config/kamiloo13.bigwalk.direct-connect.cfg`

| 设置 / Setting | 默认 / Default | 说明 / Meaning |
|---|---|---|
| `Language` | `auto` | 跟随游戏语言，也可以写成 `zh-Hans` 或 `en` / Follows the game language, or set `zh-Hans` or `en` |
| `SendRateMode` | `DirectIpOnly` | 只在 IP 直连时提高发送频率 / Raise the send rate only for IP sessions |
| `SendRate` | `60` | 发送频率，30–120。所有人必须相同 / Send rate in Hz, from 30 to 120. Every player must match |
| `FlushKcpEveryFrame` | `true` | 每帧发送可靠数据 / Send reliable data every frame |
| `ToggleKey` | `F5` | 隐藏警告条的按键 / Key that hides the warning banners |
| `HiddenOnStart` | `false` | 一启动就隐藏警告条 / Hide the warning banners at startup |

修改这个文件之前先完全退出游戏。

Quit the game completely before editing this file.

`BepInEx/config/BepInEx.cfg` 里要保持 `UpdateInteropAssemblies = false`。不要改成 `true`，否则 macOS 上重新生成 interop 会失败。

Leave `UpdateInteropAssemblies = false` in `BepInEx/config/BepInEx.cfg`. Setting it to `true` makes interop regeneration fail on macOS.

## 更多说明 / More detail

- [TROUBLESHOOTING.md](TROUBLESHOOTING.md)：启动失败、日志位置、回退 / launch failures, logs, and rollback.
- [MACOS_MODDING_FIXES.md](MACOS_MODDING_FIXES.md)：macOS 修复记录 / the macOS fix history.
- [INTEROP_1.6.0.md](INTEROP_1.6.0.md)：1.6.0 的 interop 是怎么生成的 / how the 1.6.0 interop files were generated.
- Direct Connect 源码：<https://github.com/Kamiloo13/BigWalk.DirectConnect>。这个压缩包里只带编译好的 `BepInEx/plugins/BigWalk.DirectConnect/BigWalk.DirectConnect.dll`。 / Direct Connect source lives at that repository. This pack only includes the compiled plugin DLL.

## Credits

- **lajilaji** — original [N2N Direct Connect](https://thunderstore.io/c/big-walk/p/lajilaji/N2NDirectConnect/).
- **Ice Box Studio** — [Mod Settings Menu](https://github.com/ibox233/IceBox_Mods_Issues) and BigWalkLocalizationAPI.
- **BepInEx** — the mod loader.
- BIG WALK is by House House, published by Panic.

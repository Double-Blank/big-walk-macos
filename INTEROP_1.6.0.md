# Big Walk 1.6.0：重新生成 interop

**日期：** 2026-11-19  
**游戏：** Big Walk 1.6.0（`2609301508`），Unity `6000.3.17f1`  
**结果：** 保留 Steam 启动选项 `run_bepinex.sh`，游戏可以进主菜单，并加载 BigWalkLocalizationAPI 1.1.0 与 Mod Settings Menu 1.1.5。

这次没有单独运行 Cpp2IL 命令行。先把根目录的 `GameAssembly.dylib` 换成 x86_64 薄切片，打开 `UpdateInteropAssemblies`，再用 `./run_bepinex.sh` 启动。BepInEx 在游戏进程里自己调用 Cpp2IL 和 Il2CppInterop，把新的程序集写进 `BepInEx/interop`。

## 为什么要重做

游戏从文档里记录的 1.5.1 更新到了 1.6.0。`BepInEx/interop` 仍是 1.5.1 的缓存：

| 项目 | 值 |
|---|---|
| 旧 `assembly-hash.txt` | `2c03ec2e6a747e4916eb860aa07c8b35` |
| 当前通用 `GameAssembly.dylib` 的哈希 | `9d3c62327f7d316a0b2cda0700555a97` |
| 当时的配置 | `UpdateInteropAssemblies = false` |

模组按旧方法表挂钩。日志里出现 `Unable to find method`，随后 Dobby 对地址 `0x00` 做 detour，进程退出。Steam 记录的退出码是 `-1`。

更早的一次启动（2026-11-19 11:43）是另一个问题：Gatekeeper 拒绝加载 ad-hoc 签名的 `libdoorstop.dylib`（`library load disallowed by system policy`）。后面的启动已经能加载这个库，闪退发生在模组初始化阶段。

## UpdateInteropAssemblies

`BepInEx/config/BepInEx.cfg` 的 `[IL2CPP]` 段。

启动时 BepInEx 计算游戏根目录 `GameAssembly.dylib` 的哈希，和 `BepInEx/interop/assembly-hash.txt` 比较。

- `true`：哈希不一致时重新生成 `BepInEx/interop`。
- `false`：继续使用现有 interop，只打印过期警告。

这套 macOS 配置在正常游玩时保持 `false`。Cpp2IL 处理通用 Mach-O 时会向只读切片写入数据并失败（`Stream does not support writing`）。生成时临时使用 x86_64 薄切片，完成后恢复通用库符号链接，并把 `assembly-hash.txt` 写成通用库的哈希，避免下次启动再次生成。

## 生成时执行的命令

工作目录是游戏根目录，也就是 `Big Walk.app` 所在的目录。

```sh
mv BepInEx/plugins/ModSettingsMenu/ModSettingsMenu.dll \
   BepInEx/plugins/ModSettingsMenu/ModSettingsMenu.dll.disabled
mv BepInEx/plugins/BigWalkLocalizationAPI/BigWalkLocalizationAPI.dll \
   BepInEx/plugins/BigWalkLocalizationAPI/BigWalkLocalizationAPI.dll.disabled
mv BepInEx/interop BepInEx/interop.bak-1.5.1
mkdir -p BepInEx/interop
mv GameAssembly.dylib GameAssembly.dylib.universal-link
lipo "Big Walk.app/Contents/Frameworks/GameAssembly.dylib" \
    -thin x86_64 -output GameAssembly.dylib
```

同时把配置改为：

```ini
UpdateInteropAssemblies = true
```

然后启动：

```sh
./run_bepinex.sh
```

`Big Walk.app` 里的 `Contents/Frameworks/GameAssembly.dylib` 没有被修改。`lipo` 只在游戏根目录写出一个薄切片，供 BepInEx 的哈希计算和 Cpp2IL 读取。根目录原来的 `GameAssembly.dylib` 是指向 App 内通用库的符号链接。

## 生成结果

`BepInEx/native-launch.log` 里这次启动的关键记录：

- Cpp2IL 读取 `Data/il2cpp_data/Metadata/global-metadata.dat`
- IL2CPP metadata 版本 39
- Cpp2IL 约 14 秒结束
- Il2CppInterop 写出程序集
- 新的 `assembly-hash.txt` 是薄切片哈希 `d1958712cbbca399119cda5e1019a393`
- `BepInEx/interop` 中有 155 个 DLL

进程随后以退出码 139 结束。当时 dyld 加载的是根目录那份薄切片，不是正式游玩用的通用库。interop 文件在退出前已经写完。

## 恢复成可游玩状态

```sh
rm -f GameAssembly.dylib
mv GameAssembly.dylib.universal-link GameAssembly.dylib
printf '%s' '9d3c62327f7d316a0b2cda0700555a97' > BepInEx/interop/assembly-hash.txt
mv BepInEx/plugins/ModSettingsMenu/ModSettingsMenu.dll.disabled \
   BepInEx/plugins/ModSettingsMenu/ModSettingsMenu.dll
mv BepInEx/plugins/BigWalkLocalizationAPI/BigWalkLocalizationAPI.dll.disabled \
   BepInEx/plugins/BigWalkLocalizationAPI/BigWalkLocalizationAPI.dll
```

配置改回：

```ini
UpdateInteropAssemblies = false
```

`9d3c62327f7d316a0b2cda0700555a97` 来自更新前那次启动的 BepInEx 警告，是通用 `GameAssembly.dylib` 的哈希。`assembly-hash.txt` 没有换行，长度 32 字节。

恢复后的根目录入口：

```text
GameAssembly.dylib -> Big Walk.app/Contents/Frameworks/GameAssembly.dylib
```

再次执行 `./run_bepinex.sh`。两个模组都加载，游戏进入主菜单，`Base Manager` 完成初始化。日志里不再出现 `Unable to find method`，也没有对地址 `0x00` 的 detour。

## Mod Settings Menu

1.1.2 在 1.6.0 上能加载，但建不出主菜单入口：

```text
Resource path not found: SettingsRow_Binary Hud
```

1.6.0 的设置界面里没有这个物体名。Thunderstore 上的 1.1.5 在精确名字找不到时，会改用结构匹配的二选一设置行。

下载并替换的是：

```text
https://thunderstore.io/package/download/Ice_Box_Studio_BigWalk/ModSettingsMenu/1.1.5/
```

替换了 `BepInEx/plugins/ModSettingsMenu/` 中的：

- `ModSettingsMenu.dll`（1.1.5.0）
- `ModSettingsMenu.Localization.json`
- `manifest.json`
- `CHANGELOG.md`

旧 DLL 留在同目录，文件名是 `ModSettingsMenu.dll.1.1.2`。它不以 `.dll` 结尾，BepInEx 不会加载它。BigWalkLocalizationAPI 仍是 1.1.0，1.1.5 的依赖没有变。

换上 1.1.5 后再启动，日志是 `Loading [ModSettingsMenu 1.1.5]`，并且不再出现 `SettingsRow_Binary Hud` 错误。

## 留下的文件

| 路径 | 含义 |
|---|---|
| `BepInEx/interop/` | 按 1.6.0 的 x86_64 切片生成的 interop |
| `BepInEx/interop/assembly-hash.txt` | 通用库哈希 `9d3c62327f7d316a0b2cda0700555a97` |
| `BepInEx/interop.bak-1.5.1/` | 生成前移走的 1.5.1 interop，确认不需要后可以删除 |
| `BepInEx/plugins/ModSettingsMenu/ModSettingsMenu.dll.1.1.2` | 1.1.2 备份，不会被加载 |

Steam 启动选项没有改，仍然是用绝对路径调用 `run_bepinex.sh`，后面接 `%command%`。

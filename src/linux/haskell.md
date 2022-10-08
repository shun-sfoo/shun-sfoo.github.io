## haskell 环境配置

由于 hls 会通过项目中的 hie.yaml 文件查找依赖源文件
但使用 cabal 新建的项目下 hie.yaml 依赖总会导致错误，
而 stack 项目下的 hie.yaml 只需要简单配置成(根据 hls 提示的错误配置而来)

```yaml
cradle:
  stack:
    stackYaml: './stack.yaml'
```

就可以正常使用 hls。所以选择 stack 作为构建工具。

创建 `~/.cabal` 目录，并创建 `~/.cabal/config` 文件

```config
repository mirrors.ustc.edu.cn
  url: https://mirrors.ustc.edu.cn/hackage/
  secure: False
```

这是因为 GHCup 在安装 Cabal 时会进行初始化（会下载一个 100MB 的文件），
但此时我们还没有替换 Hackage 源！这一步首先替换 Hackage 源。

1. 使用科大源安装 GHCup 本体

```bash
curl --proto '=https' --tlsv1.2 -sSf \
https://mirrors.ustc.edu.cn/ghcup/sh/bootstrap-haskell \
| BOOTSTRAP_HASKELL_YAML=https://mirrors.ustc.edu.cn/ghcup/ghcup-metadata/ghcup-0.0.6.yaml sh
```

这里一路回车先不安装 hls 与 stack，等配置好源之后在用 ghcup tui 工具安装

`source .ghcup/env`

2. 配置 GHCup 使用科大源。编辑 `~/.ghcup/config.yaml` 增加如下配置：

```config
url-source:
    OwnSource: https://mirrors.ustc.edu.cn/ghcup/ghcup-metadata/ghcup-0.0.6.yaml
```

3. 配置 Stack 科大源

`vim ~/.stack/config.yaml`

Hackage 镜像

```config
package-indices:
- download-prefix: https://mirrors.ustc.edu.cn/hackage/
  hackage-security:
    keyids:
    - 0a5c7ea47cd1b15f01f5f51a33adda7e655bc0f0b0615baa8e271f4c3351e21d
    - 1ea9ba32c526d1cc91ab5e5bd364ec5e9e8cb67179a471872f6e26f0ae773d42
    - 280b10153a522681163658cb49f632cde3f38d768b736ddbc901d99a1a772833
    - 2a96b1889dc221c17296fcc2bb34b908ca9734376f0f361660200935916ef201
    - 2c6c3627bd6c982990239487f1abd02e08a02e6cf16edb105a8012d444d870c3
    - 51f0161b906011b52c6613376b1ae937670da69322113a246a09f807c62f6921
    - 772e9f4c7db33d251d5c6e357199c819e569d130857dc225549b40845ff0890d
    - aa315286e6ad281ad61182235533c41e806e5a787e0b6d1e7eef3f09d137d2e9
    - fe331502606802feac15e514d9b9ea83fee8b6ffef71335479a2e68d84adc6b0
    key-threshold: 3 # number of keys required

    # ignore expiration date, see https://github.com/commercialhaskell/stack/pull/4614
    ignore-expiry: true
```

stackage 镜像

```config
setup-info-locations:
  - http://mirrors.ustc.edu.cn/stackage/stack-setup.yaml
urls:
  latest-snapshot: http://mirrors.ustc.edu.cn/stackage/snapshots.json
```

## xmonad 作为窗口管理器

先下载需要的软件

```bash
pacman -S
xorg-server
xorg-apps
xorg-xinit
xorg-xmessage
xorg-xsetroot
libx11 libxft libxinerama libxrandr libxss libnotify
pkgconf
picom
feh
alsa-utils
xscreensaver
scort
trayer # 作为托盘工具
mpd
g++ gcc gmp make ncurses realpath xz-utils
ttf-firacode noto-fonts noto-fonts-emoji
firefox-developer-edition
```

下载 fourmolu 作为格式化工具

`stack install fourmulo`

安装 xmobar 作为状态栏

```bash
git clone https://github.com/jaor/xmobar.git
stack install --flag xmobar:all_extensions
```

安装 xmonad
已经配置好了，初始化配置参考官方教程，不同的是需要将 xmonad xmonad-contrib 作为子模块

```bash
git clone --recurse-submodules git@github.com:shun-sfoo/xmonad.git
cd xmonad
stack install
```

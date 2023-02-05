# macOS 环境配置

macOS下一直没有一个好的包管理，即使windows也有scoop可以用，这种情况下可以转换一下思路，
不再追求所有平台有相同的体验，而是根据当前平台的特点,打造相对舒适的环境。

## 包管理
macports

下载macports包，安装前断网，安装完成后修改镜像源`vim /opt/local/etc/macports/sources.conf`，
```bash
sudo port -v selfupdate # 更新
port deps emacs-mac-app
sudo port install emacs-mac-app
export PATH=/opt/local/bin:/opt/local/sbin:$PATH
port installed
port outdated
port info emacs-mac-app
sudo port upgrade outdated
```

## 终端

直接使用iterm2，bye kitty.
killer function status bar
hotkey 快捷打开
tmux 窗口分屏
macOS 默认shell是zsh
使用zi 配置，历史记录，和git信息功能插件。

## docker

##  编程开发

jetbrains toolbox

## emacs

配置文件编辑功能 org做笔记和生成博客

## vim

最新版macOS内置vim是0.9版，可以简单配置也用做配置修改功能

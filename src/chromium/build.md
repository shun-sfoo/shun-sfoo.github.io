# build

## require

arch

```bash
git repo openssh python-virtualenv python-oauth2-client
```

## build

| platform      | cpu      | core | spend-time |
| ------------- | -------- | ---- | ---------- |
| thinkpad E480 | 8 gen i5 | 4    | 739m       |
| pc            | i7-7700  | 7    | unkown     |

## source code

depot-tool

```bash
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git`
export PATH=/path/to/depot_tools:$PATH
```

chromium-os

```bash
cd ~/chromiumos
repo init -u https://chromium.googlesource.com/chromiumos/manifest
repo sync -j4
```

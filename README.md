# Rime 配置

个人 Rime 配置：白霜拼音 + 极点五笔86 + 五笔拼音混输。主要在 Linux（fcitx5-rime）上使用，`squirrel.custom.yaml` 供 macOS 鼠须管使用。

## 方案

| 菜单名称 | 方案 ID | 说明 |
|---|---|---|
| 白霜拼音 | `rime_frost` | 全拼，词库基于雾凇重做，字频词频更准 |
| 极点五笔86 | `wubi86_jidian` | 纯五笔，`z` 开头可用拼音反查五笔编码 |
| 拼音混输 | `wubi86_jidian_pinyin` | 五笔与拼音直接混输，无需前缀，拼音出的字会提示五笔编码 |

菜单顺序即 `default.custom.yaml` 中 `schema_list` 的顺序。Rime 会记住上次选择的方案，排第一的方案只在没有记录时作为默认。

## 安装

### Linux（fcitx5）

依赖：librime 需带 librime-lua 插件（白霜的日期、计算器、农历等功能用到）。Arch Linux 的 `librime` 包已自带。

```bash
git clone git@github.com:cheebun/rime-config.git
cd rime-config

# Rime 配置（fcitx5）
rsync -a --exclude='.git' --exclude='fcitx5/' --exclude='README.md' --exclude='LICENSE*' \
  ./ ~/.local/share/fcitx5/rime/

# fcitx5 候选框：Noto Sans CJK SC 10 号字、默认主题，附带玫红主题
cp fcitx5/conf/classicui.conf ~/.config/fcitx5/conf/
cp fcitx5/config ~/.config/fcitx5/config   # 清空 fcitx5 的左 Shift 临时切换，避免与 Rime 冲突
mkdir -p ~/.local/share/fcitx5/themes && cp -r fcitx5/themes/roseo ~/.local/share/fcitx5/themes/
```

复制完成后重新部署：

```bash
rime_deployer --build ~/.local/share/fcitx5/rime /usr/share/rime-data ~/.local/share/fcitx5/rime/build
dbus-send --session --dest=org.fcitx.Fcitx5 /controller org.fcitx.Fcitx.Controller1.ReloadAddonConfig string:rime
```

### macOS（鼠须管 Squirrel）

```bash
brew install --cask squirrel-app
```

系统设置 → 键盘 → 文字输入 → 输入法 → 编辑 → 添加 → 鼠须管。

用户目录是 `~/Library/Rime`，直接流式解压部署，不落地临时文件、不产生 `.git`（避免日后在这个目录误跑 `git clean -fd` 删掉用户词库）：

```bash
curl -fsSL https://codeload.github.com/cheebun/rime-config/tar.gz/refs/heads/master \
  | tar xz -C ~/Library/Rime --strip-components=1 \
    --exclude='*/fcitx5' --exclude='*/README.md' --exclude='*/LICENSE*'
```

重新部署：

```bash
/Library/Input\ Methods/Squirrel.app/Contents/MacOS/Squirrel --reload
```

升级白霜/合并新词后重新运行上面的 `curl | tar` 命令即可覆盖更新。

## 快捷键

### 中英文切换

| 按键 | 作用 |
|---|---|
| `Ctrl+Space` | fcitx5 在英文键盘与 Rime 之间切换 |
| 左 `Shift` | Rime 内切换中 / En，正在输入时已输入的编码以英文上屏 |
| 右 `Shift` | 同上 |
| `Ctrl+Shift+2` | Rime 内切换中 / En |
| `Caps Lock` | 直接输入大写英文，同时清空未上屏的编码 |

### 方案切换

| 按键 | 作用 |
|---|---|
| `F4` / `` Ctrl+` `` / `` Ctrl+Shift+` `` | 打开方案菜单，用数字键选择 |
| `Ctrl+Shift+1` | 直接切到下一个方案 |

### 其他

| 按键 | 作用 |
|---|---|
| `Ctrl+Shift+3` | 中英文标点切换 |
| `Ctrl+Shift+4` | 简繁切换 |
| `-` `=` / `,` `.` | 候选翻页 |
| `[` `]` | 以词定字（选词的首字 / 末字） |

fcitx5 默认把左 Shift 设为「临时切换输入法」，会与 Rime 冲突，`fcitx5/config` 已将其清空。

白霜自带的 `default.yaml` 去掉了 `F4`、`Ctrl+Shift+1/2` 和右 Shift 切换，`default.custom.yaml` 把它们补了回来。

## 白霜拼音常用功能

| 输入 | 作用 |
|---|---|
| `rq` / `sj` / `xq` / `dt` / `ts` | 日期 / 时间 / 星期 / 日期时间 / 时间戳 |
| `nl` | 今天的农历 |
| `N` + 日期 | 公历转农历，例如 `N20261028` |
| `V` + 算式 | 计算器，例如 `V3*7+1` |
| `R` + 数字 | 金额大写，例如 `R1234.5` |
| `U` + 十六进制 | Unicode 字符，例如 `U4e2d` → 中 |
| `u` + 部件拼音 | 拆字反查，例如 `umumu`（木+木）→ 林 |
| `/fh` | 符号；`/a` `/e` 等输入带调韵母 |
| `` ` `` | 开启墨奇辅助码（可选，用字形筛选候选） |

## 自定义词条

| 方案 | 文件 | 格式（中间是 Tab） |
|---|---|---|
| 五笔 / 混输 | `wubi86_jidian_user.dict.yaml` | `词<Tab>五笔编码`，`##` 开头为分组名 |
| 混输里的拼音 | `pinyin_simp.dict.yaml` 末尾 | `词<Tab>拼 音<Tab>权重` |

改完后重新部署。

## 候选框样式（fcitx5）

- 配置文件 `~/.config/fcitx5/conf/classicui.conf`，修改后执行
  `dbus-send --session --dest=org.fcitx.Fcitx5 /controller org.fcitx.Fcitx.Controller1.ReloadAddonConfig string:classicui` 立即生效
- 字体指定 `Noto Sans CJK SC`，避免中文回退到其他地区的字形
- 玫红主题：把 `Theme=` 改成 `roseo` 即可启用。它由 `squirrel.custom.yaml` 的 roseo 配色换算而来（鼠须管颜色是 BGR），fcitx5 不支持圆角和编码提示单独着色
- 删除 `classicui.conf` 可恢复 fcitx5 默认

## 升级白霜

```bash
git clone --depth 1 https://github.com/gaboolic/rime-frost /tmp/rime-frost
rsync -a --exclude='.git' --exclude='.github' --exclude='others/' --exclude='README.md' --exclude='LICENSE' \
  /tmp/rime-frost/ ~/.local/share/fcitx5/rime/
cp /tmp/rime-frost/LICENSE ./LICENSE-rime-frost
```

然后重新部署，再把 `~/.local/share/fcitx5/rime/` 同步回本仓库提交。`default.custom.yaml` 和个人词库不会被覆盖。

## 来源与许可

- 白霜拼音：[gaboolic/rime-frost](https://github.com/gaboolic/rime-frost)（commit `14e8580`），GPL-3.0，见 `LICENSE-rime-frost`。`rime_frost*`、`cn_dicts*`、`en_dicts/`、`lua/`、`opencc/` 等文件均来自该项目
- 极点五笔：[KyleBing/rime-wubi86-jidian](https://github.com/KyleBing/rime-wubi86-jidian)

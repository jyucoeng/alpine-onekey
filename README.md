# alpine-onekey

Alpine 小鸡的一键脚本，两个互相独立的脚本：

| 脚本 | 干什么 |
|---|---|
| `alpine-optimize-safe.sh` | 系统优化：apk 源、常用工具、时区、sshd、内核参数 |
| `alpine-ipv6-docker.sh` | 装 Docker + 配 IPv6（**会清空 `/var/lib/docker`**） |

改完之后写在哪：

| 文件 | 内容 |
|---|---|
| `/etc/sysctl.d/99-alpine-optimize.conf` | 内核参数 |
| `/etc/local.d/99-alpine-optimize.start` | 开机重放上面那份参数（`rc-update add local`） |
| `/etc/apk/repositories` | 锁到当前 Alpine 分支（备份 `repositories.bak.<时间>`） |

## 安装

不用安装，直接跑。想放本地反复用：

```sh
curl -fsSL https://raw.githubusercontent.com/jyucoeng/alpine-onekey/main/alpine-optimize-safe.sh -o /root/alpine-optimize-safe.sh
sh /root/alpine-optimize-safe.sh --dry-run
```

## 用法

```sh
# 先演练，只打印会改什么
curl -fsSL https://raw.githubusercontent.com/jyucoeng/alpine-onekey/main/alpine-optimize-safe.sh | sh -s -- --dry-run

# 正式执行
curl -fsSL https://raw.githubusercontent.com/jyucoeng/alpine-onekey/main/alpine-optimize-safe.sh | sh

# 不碰 sshd（密钥登录的机器推荐）
curl -fsSL https://raw.githubusercontent.com/jyucoeng/alpine-onekey/main/alpine-optimize-safe.sh | sh -s -- --no-ssh

# 最保守：只调内核参数
curl -fsSL https://raw.githubusercontent.com/jyucoeng/alpine-onekey/main/alpine-optimize-safe.sh | sh -s -- --sysctl-only
```

| 参数 | 作用 |
|---|---|
| `--dry-run` | 只打印，不改系统 |
| `--upgrade` | 额外 `apk upgrade` |
| `--no-ssh` | 跳过 sshd 段（不改配置、不重启） |
| `--no-tools` | 不装常用工具 |
| `--no-tz` | 不改时区 |
| `--sysctl-only` | 只调内核参数（= 上面三个一起跳过） |
| `-h` / `-V` | 帮助 / 版本 |

**默认会改 sshd**（`PermitRootLogin yes`、`PasswordAuthentication yes`），公网机器建议加 `--no-ssh`。

Docker + IPv6：

```sh
curl -fsSL https://raw.githubusercontent.com/jyucoeng/alpine-onekey/main/alpine-ipv6-docker.sh | sh
```

## 回滚

没有自动回滚，改动都在明面上，删两个文件即可（内核参数重启后回默认）：

```sh
rm -f /etc/sysctl.d/99-alpine-optimize.conf /etc/local.d/99-alpine-optimize.start
ls -t /etc/apk/repositories.bak.* | head -1     # 找到最近的源备份
cp -a <上面列出的备份> /etc/apk/repositories
```

不自动还原：时区、主机名、装上的包。

## 卸载

脚本不往系统里装任何自己的东西，卸载 = 上面「回滚」那三行，再删掉本地的 `alpine-optimize-safe.sh`。Docker 那支脚本装的 Docker 用 `apk del docker` 卸。

## 被外部调用

[vps-tune](https://github.com/jyucoeng/vps-tune) 的 Alpine 后端 `tune-alpine.sh` 就是本脚本（v2.2，加了 `-h`/`-V`）。链路：

```sh
sb opt          → vps-tune --apply --no-ssh → tune-alpine.sh
```

`vps-tune` 一侧默认带 `--no-ssh`，不会替你放开密码登录。

## 免责

按「现状」提供，无担保。会改 apk 源、sshd、内核参数，先跑 `--dry-run`。License: MIT

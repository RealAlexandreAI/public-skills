---
name: cn-mirror
description: 国内镜像换源指引 —— 当用户在国内需要给任何包管理器/下载源/SDK/系统仓库/网络基础设施配置加速时触发。覆盖 npm/pnpm/yarn、pip/uv/pipenv、cargo/crates.io、brew/bottles、docker、apt/debian/ubuntu、epel/centos/fedora、anaconda、github release/raw、gitlab、cocoapods、gem、go module、rustup、win-get、maven/gradle、nuget、alpine/arch、国内 DNS（阿里 223.5.5.5 / DNSPod 119.29.29.29 / 114DNS）、git 与 release 下载加速（ghfast.top / 1ms.run / gh-proxy / fastgit）、docker hub 加速（dockerproxy.cn）等。给出 MirrorZ Help 对应文档链接 + 探测环境后的适配命令（fish 用 set -gx / bash-zsh 用 export，无对应工具走官方通用流程）。触发词：换源、换镜像、加速、国内镜像、cn mirror、taobao 镜像、npmmirror、tuna、ustc、aliyun、tencent、清华源、镜像、DNS、ghfast、1ms、gh-proxy、dockerproxy、慢/超时/连不上 + 工具名（npm/pip/cargo/brew/docker/apt/git 等）。
description_zh: 国内镜像换源
description_en: CN mirror config
disable: false
agent_created: true
---

# cn-mirror

## When to use

- 用户提到"换源 / 换镜像 / 加速 / 国内镜像 / cn mirror / 清华源 / tuna / ustc / aliyun / npmmirror / rsproxy / goproxy.cn"等关键词
- 用户描述工具名 + "慢 / 失败 / 超时 / 连不上"（npm 慢、pip 慢、cargo 慢、brew 慢、docker pull 慢、apt update 慢、git clone 慢、rustup 慢）
- 用户直接问"X 怎么配镜像 / X 的国内源"
- 用户提到 MirrorZ、help.mirrorz.org、或希望参考某个具体镜像的官方文档

## 环境适配（推荐路径，非硬性假设）

> 本 skill 不预设用户环境。**先探测**（`echo $SHELL` / `which mise uv brew gh node`），命中则按对应风格出命令；探测不到就按**通用流程**（bash + 官方文档标准做法）输出。

- macOS ARM64 常见路径：`/opt/homebrew/bin/fish`
- Node.js：可能由 mise / nvm / volta / 系统 brew 管理 —— 用 `which node` 探测，**只配 registry 本身，不假设包管理器**
- Python：可能由 uv / conda / pyenv / 系统 pip 管理 —— uv 用 `UV_INDEX_URL`，pip 用 `pip.conf`，conda 用 `channels`
- 可能已装 Homebrew、gh CLI（`which brew gh` 探测）

## 决策流程

1. 识别用户提到的工具/包管理器/仓库类型
2. 查下方"镜像映射表"找对应 MirrorZ Help 文档与具体命令
3. **探测用户环境**（`echo $SHELL`、`which <tool>`），fish 给 `set -gx` 写法、bash/zsh 给 `export`、无该工具就按官方通用流程
4. **确认意图：临时一次 vs 持久生效** —— 默认给临时方案（当前会话/环境变量）；只有用户明确要"永久/以后都用"才写 rc 或配置文件，写前先备份
5. 输出配置命令时**必须同时给出回滚命令**（见"配置卫生与回滚"）
6. 提供 MirrorZ Help URL 作为权威参考
7. 给出验证步骤（curl / 配置文件 / install dry-run）

## 镜像映射表

> ⚠️ **slug 可能漂移或未收录**（如 winget/ohmyzsh-git/crates.io-sparse 已 404）。**使用规则**：
> 1. 优先直接验证：WebFetch `https://help.mirrorz.org/<slug>/`，200 → 用它作为权威参考；404 → 下一步
> 2. 404 时改用搜索：`https://help.mirrorz.org/` 首页搜索镜像名（页面提供搜索框），或抓 `https://mirrorz.org` 看镜像状态
> 3. 表内 slug 仅为**候选**，不是承诺；最终 URL 以验证结果为准

| 工具 / 仓库 | 候选 slug（需验证） | MirrorZ Help 入口 | 推荐镜像 |
|---|---|---|---|
| npm (任意 node 管理) | `npm` | `https://help.mirrorz.org/npm/` | npmmirror `registry.npmmirror.com` |
| pnpm | `npm` (复用) | `https://help.mirrorz.org/npm/` | npmmirror |
| yarn | `npm` (复用) | `https://help.mirrorz.org/npm/` | npmmirror |
| Python pip | `pypi` | `https://help.mirrorz.org/pypi/` | TUNA `pypi.tuna.tsinghua.edu.cn/simple` |
| uv (Python) | `pypi` | `https://help.mirrorz.org/pypi/` | uv 默认走 pypi，配置 `UV_INDEX_URL` |
| Rust cargo | `crates.io` + `crates.io-index` + `crates.io-sparse` | `https://help.mirrorz.org/crates.io-index/` | rsproxy.cn / USTC |
| Rustup | `rustup` | `https://help.mirrorz.org/rustup/` | rsproxy.cn / USTC |
| Docker CE | `docker-ce` | `https://help.mirrorz.org/docker-ce/` | 阿里云 / USTC |
| Docker Hub images | `dockerhub` | `https://help.mirrorz.org/dockerhub/` | dockerproxy.cn / 阿里云 |
| Homebrew (formula) | `homebrew` | `https://help.mirrorz.org/homebrew/` | TUNA / USTC |
| Homebrew bottles | `homebrew-bottles` | `https://help.mirrorz.org/homebrew-bottles/` | TUNA / 阿里云 |
| apt Debian | `debian` + `debian-security` | `https://help.mirrorz.org/debian/` | TUNA / USTC / 阿里云 |
| apt Ubuntu | `ubuntu` + `ubuntu-ports` | `https://help.mirrorz.org/ubuntu/` | TUNA / USTC |
| EPEL (RHEL/CentOS) | `epel` | `https://help.mirrorz.org/epel/` | TUNA / USTC / 阿里云 |
| CentOS | `centos` + `centos-stream` + `centos-vault` | `https://help.mirrorz.org/centos/` | TUNA / 阿里云 |
| Fedora | `fedora` | `https://help.mirrorz.org/fedora/` | TUNA / USTC |
| Anaconda | `anaconda` + `anaconda-extra` | `https://help.mirrorz.org/anaconda/` | TUNA / USTC |
| GitHub Release 下载 | `github-release` | `https://help.mirrorz.org/github-release/` | gh-proxy.com / fastgit |
| GitHub Raw (raw.githubusercontent.com) | `github-raw` | `https://help.mirrorz.org/github-raw/` | gh-proxy.com / fastgit |
| GitLab CE | `gitlab-ce` | `https://help.mirrorz.org/gitlab-ce/` | TUNA / USTC |
| GitLab Runner | `gitlab-runner` | `https://help.mirrorz.org/gitlab-runner/` | TUNA / USTC |
| Grafana | `grafana` | `https://help.mirrorz.org/grafana/` | TUNA |
| MySQL | `mysql` | `https://help.mirrorz.org/mysql/` | USTC |
| Mozilla (Firefox) | `mozilla` | `https://help.mirrorz.org/mozilla/` | TUNA / USTC |
| oh-my-zsh | `ohmyzsh-git` | `https://help.mirrorz.org/ohmyzsh-git/` | TUNA |
| WinGet (Windows) | `winget` | `https://help.mirrorz.org/winget/` | TUNA |
| RubyGems | `rubygems` | `https://help.mirrorz.org/rubygems/` | TUNA |
| CocoaPods | `cocoapods` | `https://help.mirrorz.org/cocoapods/` | TUNA / USTC |
| Go module | (非 MirrorZ) — 用 `goproxy.cn` | — | goproxy.cn |
| Maven (Java) | `maven` | `https://help.mirrorz.org/maven/` | 阿里云 `maven.aliyun.com` |
| Gradle (Java) | `gradle` | `https://help.mirrorz.org/gradle/` | 阿里云 / TUNA |
| NuGet (.NET) | `nuget` | `https://help.mirrorz.org/nuget/` | 华为云 / Azure China |
| Alpine Linux | `alpine` | `https://help.mirrorz.org/alpine/` | TUNA / USTC / 阿里云 |
| Arch Linux | `archlinux` + `archlinuxcn` | `https://help.mirrorz.org/archlinux/` | TUNA / USTC |
| CTAN (TeX) | `ctan` | `https://help.mirrorz.org/ctan/` | TUNA / USTC |

## 常用镜像站（推荐优先级）

- **TUNA（清华）** `mirrors.tuna.tsinghua.edu.cn` —— 覆盖最全，HTTPS，社区活跃，**默认推荐**
- **USTC（科大）** `mirrors.ustc.edu.cn` —— 老牌，覆盖广，HTTP/HTTPS
- **阿里云** `mirrors.aliyun.com` —— 商业级稳定，部分镜像独立域名
- **腾讯云** `mirrors.cloud.tencent.com` —— 腾讯云内网优
- **npmmirror** `registry.npmmirror.com` —— npm 专用，最快
- **rsproxy.cn** —— Rust crates 中国代理
- **goproxy.cn** —— Go module 中国代理
- **dockerproxy.cn** —— Docker Hub 镜像
- **华为云** `mirrors.huaweicloud.com` —— 覆盖面广，repo 路径与阿里类似
- **网易** `mirrors.163.com` —— apt 老牌镜像

## 关键工具的多源变体（阿里 / 腾讯 / 华为 等备选）

> 默认推荐 TUNA/USTC；以下为同一工具的国内多源，按用户网络（家用宽带 / 腾讯云内网）选择。
> 腾讯云内网域名 `mirrors.tencentyun.com` 仅在腾讯云 CVM 内网可达且免流量。

### PyPI（pip / uv 共用）

| 源 | URL |
|---|---|
| TUNA（默认） | `https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple`（或别名 `pypi.tuna.tsinghua.edu.cn/simple`） |
| 阿里云 | `https://mirrors.aliyun.com/pypi/simple/` |
| 腾讯云（公网） | `https://mirrors.cloud.tencent.com/pypi/simple/` |
| 腾讯云（内网） | `https://mirrors.tencentyun.com/pypi/simple/` |
| 华为云 | `https://mirrors.huaweicloud.com/repository/pypi/simple/` |
| USTC | `https://mirrors.ustc.edu.cn/pypi/simple/` |

pip 配置：`~/.pip/pip.conf` 的 `index-url` 换成上表任一。
uv 配置：`set -gx UV_INDEX_URL <url>`（uv 不读 pip.conf）。

### npm / pnpm / yarn（Node）

| 源 | URL |
|---|---|
| npmmirror（默认） | `https://registry.npmmirror.com` |
| 腾讯云（公网） | `https://mirrors.cloud.tencent.com/npm/` |
| 腾讯云（内网） | `https://mirrors.tencentyun.com/npm/` |
| 华为云 | `https://mirrors.huaweicloud.com/repository/npm/` |
| TUNA | `https://mirrors.tuna.tsinghua.edu.cn/npm/` |

配置：`npm config set registry <url>`（pnpm/yarn 同理）。

### Maven / Gradle（Java）

| 源 | URL |
|---|---|
| 阿里云（默认） | `https://maven.aliyun.com/repository/central` |
| 华为云 | `https://repo.huaweicloud.com/repository/maven/` |
| TUNA | `https://mirrors.tuna.tsinghua.edu.cn/maven/` |

配置：写入 `~/.m2/settings.xml` 的 `<mirror>` 或 Gradle `repositories`。

## 国内 DNS

| 服务商 | DNS 1 | DNS 2 | 备注 |
|---|---|---|---|
| 阿里 DNS（推荐） | `223.5.5.5` | `223.6.6.6` | DoH `https://dns.alidns.com/dns-query` |
| DNSPod（腾讯） | `119.29.29.29` | — | DoH `https://doh.pub/dns-query` |
| 114DNS | `114.114.114.114` | `114.114.115.115` | 纯净/安全/家庭多种模式 |
| 百度 DNS | `180.76.76.76` | — | — |

macOS 命令行配置（Wi-Fi 接口，`en0`）：

```fish
networksetup -setdnsservers Wi-Fi 223.5.5.5 223.6.6.6
# 验证
scutil --dns | grep nameserver
```

> 用途：GitHub/Docker/镜像站解析加速、抗 DNS 污染；配合 DoH（`dnscrypt-proxy` / Surge）效果更佳。

## Git / Release / Docker 下载加速（ghfast / 1ms / gh-proxy 类）

> GitHub 直连慢时用反向代理前缀。**格式统一**：`https://<proxy>/https://github.com/...`（保留完整原 URL）。

| 代理 | 域名 | 说明 |
|---|---|---|
| ghfast.top | `https://ghfast.top` | 综合代理（clone/raw/release），当前稳定 |
| 1ms.run | `https://1ms.run` | 综合代理（clone/raw/release） |
| gh-proxy.com | `https://gh-proxy.com` | 老牌，raw/release 稳定 |
| ghproxy.net | `https://ghproxy.net` | 备选 |
| fastgit | `https://raw.fastgit.org` | 仅 raw，老牌 |

### git clone 加速（全局 insteadOf，fish 生效于 git 配置）

```fish
git config --global url."https://ghfast.top/https://github.com/".insteadOf "https://github.com/"
# 撤销
git config --global --unset-all url."https://ghfast.top/https://github.com/".insteadOf
```

> 注意：insteadOf 会影响 push（推不上去），只在**拉取场景**临时设置或仅对 `git clone` 用完整代理 URL。

### raw.githubusercontent.com 加速

```fish
curl -L https://ghfast.top/https://raw.githubusercontent.com/user/repo/main/file
# 或设置别名
alias rawget='curl -L https://ghfast.top/https://raw.githubusercontent.com'
```

### GitHub Release 下载加速（gh release download 场景）

```fish
# gh 本身不走代理，手动拼代理前缀下载
gh release download -R user/repo -p '*.tar.gz' --clobber \
  -D /tmp/dl && curl -L -o /tmp/dl/x.tar.gz \
  "https://ghfast.top/https://github.com/user/repo/releases/download/v1.0.0/x.tar.gz"
```

### Docker Hub 镜像加速（国内镜像列表）

```json
{
  "registry-mirrors": [
    "https://dockerproxy.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://hub-mirror.c.163.com",
    "https://mirrors.ustc.edu.cn"
  ]
}
```

> ⚠️ 2024 年起国内多数公开 Docker Hub 加速器陆续失效（docker.io 被墙），`registry-mirrors` 方案已不可靠。**可靠替代**：dockerproxy.cn / 自建代理（nginx + docker-registry-proxy）/ 腾讯云内网 `mirror.ccs.tencentyun.com` / 阿里云个人加速器地址（登录容器镜像服务获取专属 URL）。失效时检查 `docker info` 的 Registry Mirrors 是否生效。

## 配置卫生与回滚（每次配置必须附带）

**通用原则**：
1. 先问用户：**临时一次**（当前会话/当前任务，推荐默认）还是**持久生效**（写 rc / 配置文件）？持久必须等用户明确确认
2. 改任何持久文件前**先备份**：`cp <file> <file>.bak.$(date +%s)`，或先记录原值（如 DNS）
3. 输出配置时**必带回滚命令**，让用户随时一键取消
4. 临时方案用环境变量（fish `set -gx` / bash `export`），只在当前会话生效，关终端自动消失

**回滚速查表**：

| 工具 | 设置方式 | 回滚 / 取消 |
|---|---|---|
| npm | `npm config set registry <url>` | `npm config delete registry`（或删 `~/.npmrc` 对应行） |
| pnpm | `pnpm config set registry <url>` | `pnpm config delete registry` |
| yarn | `yarn config set registry <url>` | `yarn config delete registry` |
| pip | `~/.pip/pip.conf` 的 `index-url` | 改回 `https://pypi.org/simple`，或删掉 pip.conf |
| uv | `set -gx UV_INDEX_URL <url>` | fish: `set -e UV_INDEX_URL`；bash: `unset UV_INDEX_URL` |
| cargo | `~/.cargo/config.toml` 的 `replace-with` | 还原备份文件，或删 `[source.crates-io]` 及 `[source.rsproxy*]` 块 |
| rustup | `set -gx RUSTUP_DIST_SERVER <url>` | `set -e RUSTUP_DIST_SERVER` / `unset` |
| brew | 4 个 `HOMEBREW_*` 变量 | 删 rc 文件里对应行（`set -e` / `unset` 每个） |
| docker | daemon.json `registry-mirrors` | 删除该字段或还原 `daemon.json.bak` |
| apt | `/etc/apt/sources.list.d/*.sources` | `cp <file>.bak <file>` 还原（改前必须备份） |
| git | `git config --global url.<proxy>.insteadOf` | `git config --global --unset-all url.<proxy>.insteadOf` |
| DNS | `networksetup -setdnsservers Wi-Fi <ip>` | `networksetup -setdnsservers Wi-Fi <原DNS>`（改前先 `scutil --dns` 记录原值） |
| alias | `alias rawget '...'` | fish: `functions -e rawget`；bash/zsh: `unalias rawget` |

> 持久化写入 rc 文件的命令，回滚即"删掉刚追加的那几行"；输出命令时注明具体行内容便于删除。

## 配置命令模板

> 先探测 `echo $SHELL`：fish 用 `set -gx`，bash/zsh 把同一命令换成 `export`；环境无对应工具/包管理器时，直接采用**官方通用流程**（写配置文件 / 环境变量，不依赖 shell 语法）。
> **每个命令的配套回滚见上方"配置卫生与回滚"速查表。**
> **默认给临时方案（当前会话环境变量）；要持久生效先问用户确认，确认后写 rc / 配置文件并先备份。**

### npm（registry 配置与包管理器无关）

```fish
npm config set registry https://registry.npmmirror.com
# 或全局 ~/.npmrc
echo 'registry=https://registry.npmmirror.com' >> ~/.npmrc
```

### pnpm

```fish
pnpm config set registry https://registry.npmmirror.com
```

### yarn

```fish
yarn config set registry https://registry.npmmirror.com
```

### Python pip（~/.pip/pip.conf）

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```

### Python uv（临时：环境变量；持久：uv.toml，官方推荐）

```fish
# 临时（当前会话）
set -gx UV_INDEX_URL https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
# 回滚：set -e UV_INDEX_URL
```

持久（MirrorZ 官方推荐，写入 `~/.config/uv/uv.toml`）：

```toml
[[index]]
url = "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
default = true
```

### Cargo（~/.cargo/config.toml）

```toml
[source.crates-io]
replace-with = "rsproxy-sparse"

[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"

[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"

[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true
```

### Rustup

```fish
set -gx RUSTUP_DIST_SERVER https://mirrors.tuna.tsinghua.edu.cn/rustup
# 永久：写入 ~/.config/fish/config.fish
```

### Homebrew（macOS ARM64，永久生效，~10 行）

```fish
set -gx HOMEBREW_API_DOMAIN https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api
set -gx HOMEBREW_BOTTLE_DOMAIN https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles
set -gx HOMEBREW_BREW_GIT_REMOTE https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
set -gx HOMEBREW_CORE_GIT_REMOTE https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
set -gx HOMEBREW_PIP_INDEX_URL https://pypi.tuna.tsinghua.edu.cn/simple
# 永久：把上面 set -gx 行追加到 ~/.config/fish/config.fish（bash/zsh 用 export 写入 ~/.bashrc / ~/.zshrc）
```

### Docker（macOS Desktop → Settings → Docker Engine；Linux/server → /etc/docker/daemon.json）

> 配置项与多源列表见上方"Git / Release / Docker 下载加速 → Docker Hub 镜像加速"一节，此处不重复。

### apt Debian/Ubuntu

`/etc/apt/sources.list.d/<mirror>.sources`（DEB822 格式，以 TUNA 为例）：

```
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/ubuntu
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

### EPEL（CentOS Stream 9 / RHEL）

```
dnf config-manager --set-enabled epel
sed -e 's|^metalink=|#metalink=|g' \
    -e 's|^#baseurl=https://download.example/pub/epel|baseurl=https://mirrors.tuna.tsinghua.edu.cn/epel|g' \
    -i.bak /etc/yum.repos.d/epel.repo
```

### RubyGems

```fish
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
bundle config mirror.https://rubygems.org https://mirrors.tuna.tsinghua.edu.cn/rubygems
```

### CocoaPods（macOS）

```fish
pod repo remove master
pod repo add master https://mirrors.tuna.tsinghua.edu.cn/git/CocoaPods/Specs.git
```

### Go module

```fish
set -gx GOPROXY https://goproxy.cn,direct
set -gx GOSUMDB off  # 国内拉取可能需要，国内校验用 sum.golang.google.cn
```

## 推荐镜像站 → MirrorZ 镜像对比

打开 https://mirrorz.org 看全网镜像状态（哪家同步全、哪家延迟低）。常用稳定梯队：

- TUNA、USTC、阿里云（多数仓库覆盖齐）
- npmmirror（npm 专项最快）
- rsproxy（crates 专项）
- goproxy.cn（Go 专项）
- dockerproxy.cn（Docker Hub 专项）

## 验证步骤（确保生效）

- `npm config get registry` → `https://registry.npmmirror.com/`
- `pip config list` → 显示 tuna/aliyun/tencent 镜像
- `cargo search serde --limit 1` → 应秒回
- `brew --repo` 指向 tuna 路径
- `docker info | grep -A5 "Registry Mirrors"` → 列出加速域名
- `apt update` 不再卡住
- `gh release download` / `curl https://raw.githubusercontent.com/...` 速度 < 1s
- DNS：`scutil --dns | grep nameserver` → 显示 223.5.5.5 等
- git clone 加速：`git ls-remote https://github.com/user/repo` 对比直连 vs ghfast 延迟

## Pitfalls

- **fish vs bash**：先 `echo $SHELL` 探测；fish 用 `set -gx`，bash/zsh 用 `export`，永久化写入各自 rc 文件
- **Node 包管理器差异**：先 `which node` 探测（mise/nvm/volta/brew 等），registry 配置与包管理器无关；不要建议 `brew install node`
- **架构差异**：Docker/Homebrew 按机器架构取 `arm64` / `x86_64` bottles；CentOS/RHEL 镜像区分 aarch64 vs x86_64
- **brew bottles 与 formula 分开配置**：HOMEBREW_BOTTLE_DOMAIN（binary 下载）+ HOMEBREW_CORE_GIT_REMOTE（formula git）
- **apt keyring**：Debian/Ubuntu 新版需 `Signed-By: /usr/share/keyrings/...`，不能用裸 https
- **Docker Desktop (macOS)**：daemon.json 修改走 Docker Desktop → Settings → Docker Engine；改 `~/.docker/daemon.json` 不生效
- **rustup** 与 **cargo** 是两个独立配置（环境变量 vs config.toml）
- **goproxy.cn** 配套 `GOSUMDB=sum.golang.google.cn` 才能做完整性校验（默认 sum.golang.org 国内可能不通）
- **uv** 走自己的 `UV_INDEX_URL`，不读 pip.conf
- **GitHub Release 加速**：用 gh-proxy 时注意部分 release 文件名含 `/`，URL 拼接需保留原路径
- **不要盲目换源**：先看官方源是否真的慢，再切；切错镜像可能引入 TLS/签名问题（特别是 apt、epel）
- **ghfast insteadOf 会拦截 push**：`git config url.<proxy>.insteadOf` 只对拉取安全；推代码前必须 unset，或 clone 时临时用完整代理 URL
- **Docker Hub 公开加速器大多失效**：2024 起 docker.io 被墙，`registry-mirrors` 里的公开地址（网易/USTC 等）可能已挂；用 dockerproxy.cn / 自建代理 / 云内网加速器，并用 `docker info` 验证
- **国内 DNS 是全局修改**：`networksetup -setdnsservers` 影响整机 DNS，改之前先记下原值（`scutil --dns`）；配合 DoH 可进一步抗污染
- **镜像 URL 尾部斜杠**：pip `index-url` 带尾 `/` 可省；npm `registry` 不需要尾 `/`；照抄表内 URL，别乱加/删斜杠

## 维护说明

- 镜像 URL 与加速器可用性变化快（尤其 Docker Hub 代理），使用前以 MirrorZ（https://mirrorz.org / https://help.mirrorz.org）实时状态为准
- 新增镜像时在"镜像映射表"追加一行（工具名 + 候选 slug + MirrorZ 入口 + 推荐镜像域名）

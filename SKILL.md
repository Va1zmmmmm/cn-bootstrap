---
name: cn-bootstrap
description: >
  中国大陆网络环境一键初始化工具。适用于全新安装的 Claude Code ——
  自动配置所有包管理器的国内镜像源、GitHub 代理、API 中转方案，
  让 Claude Code 在国内网络下开箱即用。
  Trigger: "初始化国内环境"、"配置镜像"、"装工具太慢了"、"下载又被墙了"、
  "帮我把开发环境配好"、"刚装的 Claude Code 什么都下不动"、
  npm/pip/git clone 失败或超时、全新机器首次使用 Claude Code。
  也适用于任何下载命令失败后用户说"想办法"、"怎么办"、"换个源"等场景。
---

# 🇨🇳 CN Bootstrap — 国内网络环境一键初始化

## 核心原则

你面对的是一台**刚开机、什么都没配、直连外网大概率不通**的机器。
你的任务不是一条条问用户，而是**自动检测 → 自动配置 → 一次搞定**。

```
检测网络 → 识别包管理器 → 配置镜像 → 验证 → 报告
  └── 能自动的绝不问，必须问的一次问完 ──┘
```

---

## 阶段零：网络环境检测

先跑这些命令摸清当前网络状况：

```bash
# 1. 测国内镜像连通性
ping -n 2 registry.npmmirror.com        # Windows
ping -c 2 registry.npmmirror.com        # Unix

# 2. 测 GitHub 连通性（大概率不通）
curl -s --connect-timeout 5 https://github.com > /dev/null && echo "GitHub OK" || echo "GitHub 不通"

# 3. 测 pip 官方源
curl -s --connect-timeout 5 https://pypi.org > /dev/null && echo "PyPI OK" || echo "PyPI 不通"

# 4. 检查是否已有代理
echo $HTTP_PROXY $HTTPS_PROXY $ALL_PROXY
```

**根据检测结果分级处理：**
- 全通 → 不用折腾，跳过
- GitHub 不通 → 必须配 Git 代理和 npm/pip 镜像
- 全不通 → 可能需要先配系统代理，提示用户

---

## 阶段一：npm / yarn / pnpm（JS/TS 生态）

### 1.1 检测是否有 npm
```bash
npm --version 2>/dev/null || echo "npm 未安装"
```

### 1.2 配置淘宝镜像（首选）
```bash
npm config set registry https://registry.npmmirror.com
```

备选镜像（按优先级）：
| 镜像 | URL |
|------|-----|
| 淘宝 | `https://registry.npmmirror.com` |
| 阿里云 | `https://npm.aliyun.com` |
| 腾讯云 | `https://mirrors.cloud.tencent.com/npm/` |
| 华为云 | `https://mirrors.huaweicloud.com/repository/npm/` |

### 1.3 配置常用二进制镜像
```bash
npm config set sass_binary_site       https://npmmirror.com/mirrors/node-sass/
npm config set electron_mirror        https://npmmirror.com/mirrors/electron/
npm config set puppeteer_download_host https://npmmirror.com/mirrors
npm config set chromedriver_cdnurl    https://npmmirror.com/mirrors/chromedriver
npm config set phantomjs_cdnurl       https://npmmirror.com/mirrors/phantomjs
npm config set node_sqlite3_binary_host_mirror https://npmmirror.com/mirrors
npm config set canvas_binary_host_mirror https://npmmirror.com/mirrors/node-canvas
```

### 1.4 yarn 用户
```bash
yarn config set registry https://registry.npmmirror.com 2>/dev/null
```

### 1.5 pnpm 用户
```bash
pnpm config set registry https://registry.npmmirror.com 2>/dev/null
```

---

## 阶段二：pip / uv（Python 生态）

### 2.1 检测 Python 环境
```bash
python3 --version 2>/dev/null || python --version 2>/dev/null || echo "Python 未安装"
pip3 --version 2>/dev/null || pip --version 2>/dev/null || echo "pip 未安装"
```

### 2.2 配置 pip 清华源（首选）
```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set global.trusted-host pypi.tuna.tsinghua.edu.cn
```

备选镜像（按优先级）：
| 镜像 | URL |
|------|-----|
| 清华 | `https://pypi.tuna.tsinghua.edu.cn/simple` |
| 阿里云 | `https://mirrors.aliyun.com/pypi/simple/` |
| 中科大 | `https://pypi.mirrors.ustc.edu.cn/simple/` |
| 豆瓣 | `https://pypi.douban.com/simple/` |
| 华为云 | `https://mirrors.huaweicloud.com/repository/pypi/simple` |

### 2.3 uv 用户
```bash
# 暂无永久配置命令，每次使用时加参数
# uv pip install --index-url https://pypi.tuna.tsinghua.edu.cn/simple <package>
```

### 2.4 conda 用户
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

---

## 阶段三：Git — GitHub 加速

### 3.1 检测 Git
```bash
git --version 2>/dev/null || echo "Git 未安装"
```

### 3.2 配置 URL 重写（推荐，一劳永逸）
```bash
# 所有 github.com 的请求自动走 kkgithub 镜像
git config --global url."https://kkgithub.com/".insteadOf "https://github.com/"
```

### 3.3 clone 时的域名替换速查
| 原始地址 | 替换地址 |
|----------|----------|
| `github.com/owner/repo.git` | `kkgithub.com/owner/repo.git` |
| GitHub Releases 下载 | `gh-proxy.org/https://github.com/...` |
| `raw.githubusercontent.com/...` | `raw.gitmirror.com/...` |
| 大文件加速 | `gitclone.com/github.com/...` |

### 3.4 如果有代理（梯子）
```bash
# 用户明确说有代理时才配，端口通常 7890(clash) 或 1080(ss)
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

---

## 阶段四：Go（Go Modules）

### 4.1 检测 Go
```bash
go version 2>/dev/null || echo "Go 未安装"
```

### 4.2 配置
```bash
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOSUMDB=sum.golang.google.cn
```

备选：
| 镜像 | GOPROXY 值 |
|------|-----------|
| 七牛云 | `https://goproxy.cn,direct` |
| 阿里云 | `https://mirrors.aliyun.com/goproxy/,direct` |

---

## 阶段五：Rust（Cargo）

### 5.1 检测 Rust
```bash
cargo --version 2>/dev/null || echo "Cargo 未安装"
rustup --version 2>/dev/null || echo "rustup 未安装"
```

### 5.2 配置 rustup 镜像
```bash
# 中科大源（首选）
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
```

### 5.3 配置 crates.io 镜像
创建或编辑 `~/.cargo/config.toml`：
```toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"

[source.tuna]
registry = "sparse+https://mirrors.tuna.tsinghua.edu.cn/crates.io-index/"
```

---

## 阶段六：RubyGems

```bash
gem sources --remove https://rubygems.org/
gem sources --add https://gems.ruby-china.com/
```

Bundler：
```bash
bundle config mirror.https://rubygems.org https://gems.ruby-china.com
```

---

## 阶段七：Homebrew（macOS / Linux）

仅 macOS 或 Linux 用户：
```bash
# 清华源
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
```

建议写入 `~/.bashrc` 或 `~/.zshrc` 持久化。

---

## 阶段八：Docker

### 8.1 Docker Desktop 用户（Windows/macOS）
在 Docker Desktop 设置 → Docker Engine 中添加：
```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.cn-hangzhou.aliyuncs.com"
  ]
}
```

### 8.2 Linux 用户
编辑 `/etc/docker/daemon.json`（同上），然后：
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 8.3 临时使用
```bash
docker pull docker.mirrors.ustc.edu.cn/<image>
```

---

## 阶段九：系统包管理器

### APT（Ubuntu/Debian）
```bash
sudo sed -i 's@//.*archive.ubuntu.com@//mirrors.tuna.tsinghua.edu.cn@g' /etc/apt/sources.list
sudo apt update
```

备选：阿里云 `mirrors.aliyun.com`、中科大 `mirrors.ustc.edu.cn`、163 `mirrors.163.com`

### YUM/DNF（CentOS/RHEL/Fedora）
```bash
sudo sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-*.repo
sudo sed -i 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' /etc/yum.repos.d/CentOS-*.repo
```

---

## 阶段十：Claude Code 本身

### 10.1 如果 Claude Code 还没装
```bash
# 走淘宝镜像安装
npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com
```

### 10.2 如果已装但 API 连不上

**方案 A：DeepSeek API 中转（当前在用，推荐）**
```json
// ~/.claude/settings.json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "你的DeepSeek API Key",
    "ANTHROPIC_MODEL": "deepseek-v4-pro"
  }
}
```
获取 Key：https://platform.deepseek.com → API Keys

**方案 B：其他可用中转**
| 平台 | BASE_URL 模板 |
|------|--------------|
| 硅基流动 | `https://api.siliconflow.cn/anthropic` |
| INFAI | `https://api.infi.ai/anthropic`（具体看平台文档） |

### 10.3 安装 skills-cn（用于后续装 skills）
```bash
npm install -g skills-cn --registry=https://registry.npmmirror.com
```

---

## 阶段十一：WebFetch 备选方案

Claude Code 的 WebFetch 工具访问部分网站会 403。遇到时：

| 场景 | 方案 |
|------|------|
| 网页被 Cloudflare 挡 | 搜 Google Cache：`webcache.googleusercontent.com/search?q=cache:<url>` |
| 查技术文章 | 先去 CSDN / 掘金 / 博客园 / 知乎搜同题文章 |
| GitHub README 打不开 | 用 `kkgithub.com` 在浏览器打开 |
| npm 包文档 | 上 `npmmirror.com` 搜同名包 |

---

## 配置优先级与容错

### 镜像选择策略
```
淘宝 npmmirror（首选）
  ↓ 不通
阿里云镜像
  ↓ 不通
腾讯云 / 华为云 / 中科大
  ↓ 都不通
换网络、开代理、或手动下载
```

### 执行原则
1. **临时优先** — 用 `--registry` / `-i` 参数而非永久配置，除非用户说"保存"
2. **验证每步** — 配完就跑一下 `npm install <小包>` 验证，别配一堆最后全报错
3. **清楚汇报** — 告诉用户配了哪个源，当前用的是什么

---

## 一键初始化脚本

当用户说"帮我全配好"时，一次性执行以下检查清单并汇报结果：

```
□ 网络检测：GitHub/PyPI/npm 连通性
□ npm 镜像 → registry.npmmirror.com
□ npm 二进制镜像 → electron/node-sass/puppeteer 等
□ pip 镜像 → pypi.tuna.tsinghua.edu.cn
□ git 镜像 → kkgithub.com
□ Go 镜像 → goproxy.cn（如有 Go）
□ Cargo 镜像 → mirrors.ustc.edu.cn（如有 Rust）
□ RubyGems 镜像 → gems.ruby-china.com（如有 Ruby）
□ Homebrew 镜像 → mirrors.tuna.tsinghua.edu.cn（macOS/Linux）
□ Docker 镜像 → docker.mirrors.ustc.edu.cn（如有 Docker）
□ Claude Code API → 当前使用的中转方案
□ skills-cn → 已安装（用于后续 skill 管理）
```

每完成一项汇报一项，最终输出汇总表。

---

## Do This

- 用户说"帮我把环境配好" → 跑阶段零检测网络 → 逐项配置 → 最后输出汇总表
- npm install 报 timeout → 自动加 `--registry=https://registry.npmmirror.com` 重试
- git clone GitHub 失败 → 自动换成 `kkgithub.com` 重试，不行再换 `gh-proxy.org`
- pip install 报 connection error → 自动加 `-i https://pypi.tuna.tsinghua.edu.cn/simple` 重试
- 下载 GitHub Release 文件 → 自动在前面拼 `https://gh-proxy.org/`
- 用户说"慢"、"卡"、"超时"、"被墙" → 立即检索引擎，切镜像重试
- 配完一项验证一项，别全部配完才发现都不通

## Not This

- 用户说 npm 装不上 → 叫用户"换个网络试试"（应该自动切镜像）
- 用户在国内 → 直接走官方源（明知会慢/失败还硬走）
- 镜像 1 挂了 → 直接报错放弃（应该降级到镜像 2、3）
- 永久修改用户配置 → 除非用户明确说"保存"、"记住"
- 配完不验证 → 用户下次用才发现配错了

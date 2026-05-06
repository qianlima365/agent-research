# AI Agent Research

AI Agent 领域研究内容站，涵盖开源项目、开发框架、技术栈与实践经验。

通过 GitHub Pages 发布：[访问地址](https://yafeiwang.github.io/agent-research/)

## 本地开发（macOS）

### 第一步：安装 Homebrew

Homebrew 是 macOS 的包管理工具。在终端执行：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成后，按终端提示运行类似下面的命令（将路径中的 `用户名` 替换为你的实际用户名）：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 第二步：安装 Ruby

macOS 自带 Ruby，但版本较旧，需要通过 Homebrew 安装新版本：

```bash
brew install ruby
```

安装完成后，将新版本的 Ruby 加入系统路径。在终端执行：

```bash
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

验证安装成功：

```bash
ruby -v
```

显示版本号（如 `ruby 3.4.x`）即表示成功。

### 第三步：安装 Bundler

Bundler 是 Ruby 项目的依赖管理工具：

```bash
gem install bundler
```

如果提示权限不足，尝试：

```bash
sudo gem install bundler
```

### 第四步：安装项目依赖

进入项目根目录，执行：

```bash
bundle install
```

此命令会读取 `Gemfile`，自动下载并安装 Jekyll 及所需插件。

### 第五步：启动本地服务

```bash
bundle exec jekyll serve
```

看到类似 `Server running... press ctrl-c to stop.` 的提示后，在浏览器打开：

```
http://localhost:4000/agent-research/
```

修改文件后保存，页面会自动重新构建刷新。

### 常用命令

| 命令 | 说明 |
|------|------|
| `bundle exec jekyll serve` | 启动开发服务器 |
| `bundle exec jekyll serve --livereload` | 启动并启用浏览器自动刷新 |
| `bundle exec jekyll build` | 仅构建站点，不启动服务器 |
| `bundle exec jekyll clean` | 清除构建缓存 |

### 常见问题

**1. 执行 `bundle install` 时提示缺少权限**

不要对 macOS 系统自带的 Ruby 目录执行 `sudo gem install`。请确保已通过 Homebrew 安装了独立 Ruby，并正确配置了 `PATH`。

**2. 提示 `command not found: bundle`**

说明 Bundler 没有安装成功，或安装路径未加入 `PATH`。尝试重新执行：

```bash
gem install bundler
```

**3. 端口被占用**

如果 `4000` 端口被占用，可指定其他端口：

```bash
bundle exec jekyll serve --port 4001
```

### 目录说明

| 目录 | 用途 |
|------|------|
| `_config.yml` | 站点配置 |
| `_layouts/` | 页面模板 |
| `assets/` | 样式、图片等静态资源 |
| `frameworks/` | Agent 框架研究 |
| `projects/` | 开源项目跟踪 |
| `development/` | 开发实践与模式 |
| `techstack/` | 技术栈选型 |
| `research/` | 论文与趋势研究 |

新增内容时，在对应目录下创建 markdown 文件即可，参考各目录下的 `index.md` 格式。

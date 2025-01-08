# Xmind 使用 Midscene 自动化测试脚本示例

> 本项目展示了如何使用 Midscene 框架对 Xmind 进行自动化测试。通过自然语言描述的方式，实现了包括登录、创建思维导图等常见操作的自动化，大大简化了测试脚本的编写和维护工作。

## 目录
- [简介](#什么是-midscene)
- [安装指南](#安装-midscene)
- [YAML 脚本自动化](#使用-yaml-脚本自动化的优势及场景)
  - [配置说明](#yaml-scripts-demo)
  - [运行方式](#运行脚本)
  - [调试方法](#debug)
- [桥接模式](#使用-midscene-桥接模式的优势及场景)
  - [简介及优势](#什么是-midscene-桥接模式)
  - [使用场景](#使用场景及优势)
  - [示例说明](#桥接模式-demo)
- [常见问题](#常见问题)
- [参考资料](#参考资料)

## 什么是 Midscene

Midscene.js 是一个采用多模态大语言模型的 UI 自动化框架，允许通过自然语言描述来执行交互、数据提取和断言。它通过简化传统的选择器密集型脚本，使得 UI 自动化维护变得更容易。Midscene.js 可以集成到各种测试环境如 Playwright 和 Puppeteer 中，并提供了一个用于调试的可视化报告和 Playground。此外，所有数据处理都在用户的自有环境中完成，确保数据的安全性。你可以通过其 Chrome 插件快速体验其功能，无需编写代码。

## 安装 Midscene

1. 安装 Chrome 扩展: [Midscene](https://chromewebstore.google.com/detail/midscene/gbldofcpkknbggpkmbdaefngejllnief)
2. 安装 Midscene 命令行界面：

```bash
npm i -g @midscene/cli
```

## 使用 YAML 脚本自动化的优势及场景

使用 YAML 脚本进行自动化的主要优势是其简洁性和可读性，使得非技术人员也能轻松编写和维护自动化脚本。此外，YAML 脚本通过自然语言进行任务定义，减少了对复杂API的依赖。适用场景包括进行基本的冒烟测试、验证关键用户路径等，尤其适合在不需要复杂编程的环境中快速实现自动化测试。

在 `.yaml` 文件中，有两个部分：`target` 和 `tasks`。

`target` 部分定义了任务的基本信息

```yaml
target:
  # 访问的 URL，必填。如果提供了 `serve` 参数，则提供相对路径
  url: <url>

  # 在本地路径下启动一个静态服务，可选
  serve: <root-directory>

  # 浏览器 UA，可选
  userAgent: <ua>

  # 浏览器视口宽度，可选，默认 1280
  viewportWidth: <width>

  # 浏览器视口高度，可选，默认 960
  viewportHeight: <height>

  # 浏览器设备像素比，可选，默认 1
  deviceScaleFactor: <scale>

  # JSON 格式的浏览器 Cookie 文件路径，可选
  cookie: <path-to-cookie-file>

  # 等待网络空闲的策略，可选
  waitForNetworkIdle:
    # 等待超时时间，可选，默认 10000ms
    timeout: <ms>
    # 是否在等待超时后继续，可选，默认 true
    continueOnNetworkIdleError: <boolean>

  # 输出 aiQuery 结果的 JSON 文件路径，可选
  output: <path-to-output-file>
```

`tasks` 部分是一个数组，定义了脚本执行的步骤。记得在每个步骤前添加 `-` 符号。

```yaml
tasks:
  - name: <name>
    flow:
      # 执行一个交互，`ai` 是 `aiAction` 的简写方式
      - ai: <prompt>

      # 执行一个交互
      - aiAction: <prompt>

      # 执行一个断言
      - aiAssert: <prompt>

      # 执行一个查询，返回一个 JSON 对象
      - aiQuery: <prompt> # 记得在提示词中描述输出结果的格式
        name: <name> # 查询结果在 JSON 输出中的 key
        
      # 等待某个条件满足，并设置超时时间(ms，可选，默认 30000)
      - aiWaitFor: <prompt>
        timeout: <ms>

      # 等待一定时间
      - sleep: <ms>

  - name: <name>
    flow:
      # ...
```

## Yaml Scripts Demo

### 安装及配置

1. 安装 Midscene 命令行界面：

   ```bash
   npm i -g @midscene/cli

2. 创建配置文件
   ```	bash
   touch .env
   ```

3. 配置 API Key
   ```bash
   export OPENAI_API_KEY="sk-abcdefghijklmnopqrstuvwxyz"
   ```

   配置其它AI服务商可参考 [自定义模型和服务商](https://midscenejs.com/zh/model-provider.html)

### 运行脚本

- 运行所有脚本

  > For windows, you need to replace `./` with `.\`, like `midscene .\yaml-scripts-demo\`.

  ```bash
  midscene ./yaml-scripts-demo/
  ```

- 运行 FW 登录并创建新思维导图脚本

  ```bash
  midscene ./yaml-scripts-demo/fw-demo.yaml
  ```

### Debug

- 运行脚本时使用有界面模式（即运行时可以看到浏览器窗口）

  ```bash
  midscene --headed ./yaml-scripts-demo/fw-demo.yaml
  ```

- 脚本执行完成后保持浏览器窗口打开

  ```bash
  midscene --keep-window ./yaml-scripts-demo/fw-demo.yaml
  ```

## 使用 Midscene 桥接模式的优势及场景

### 什么是 Midscene 桥接模式

Midscene 桥接模式是一种利用 Chrome 插件来实现本地脚本与桌面版 Chrome 浏览器直接连接的技术。它允许开发者在不离开当前浏览器环境的前提下，执行自动化脚本，使得可以直接利用现有的浏览器配置和页面状态。这种模式适合需要频繁与浏览器交互的自动化测试和开发任务。

### 使用场景及优势

使用 Midscene 桥接模式可以将你的本地脚本与桌面版 Chrome 直接连接，支持实时交互和自动化操作。这种模式允许你利用现有的浏览器设置，如 Cookies 和插件，以及保持当前的页面状态。桥接模式特别适用于需要直接与浏览器交互的自动化测试和开发场景，提供了高度的灵活性和控制能力。

## 桥接模式 demo

### 安装及配置

1. 安装 Chrome 扩展: [Midscene](https://chromewebstore.google.com/detail/midscene/gbldofcpkknbggpkmbdaefngejllnief)

2. 创建配置文件

   ```	bash
   touch .env
   ```

3. 配置 API Key

   ```bash
   export OPENAI_API_KEY="sk-abcdefghijklmnopqrstuvwxyz"
   ```

   配置其它AI服务商可参考 [自定义模型和服务商](https://midscenejs.com/zh/model-provider.html)

4. 安装依赖项：

   ```bash
   npm install
   ```

5. 运行示例以创建新标签页。运行时记得点击 Chrome 扩展中的"允许连接"按钮：

   ```bash
   npm run demo
   ```

6. 运行示例以连接到当前活动标签页：

   ```bash
   npm run demo-current-tab
   ```

## 常见问题

### 1. 如何处理动态加载的元素？
使用 `aiWaitFor` 命令等待元素出现：
```yaml
- aiWaitFor: 等待某个元素出现在页面上
  timeout: 5000
```

### 2. 如何处理弹窗和提示？
在脚本中添加相应的处理步骤：
```yaml
- ai: 如果出现弹窗，点击确认按钮
```

### 3. 如何调试执行失败的脚本？
- 使用 `--headed` 模式运行脚本查看执行过程
- 使用 `--keep-window` 参数保持浏览器窗口打开
- 检查 Midscene 的执行日志

### 4. 环境变量配置问题
确保在运行脚本前正确设置了环境变量：
- Windows: 使用 `set OPENAI_API_KEY=your-key`
- Linux/Mac: 使用 `export OPENAI_API_KEY=your-key`

## 最佳实践

1. **脚本组织**
   - 将相关的任务组织在同一个 YAML 文件中
   - 使用清晰的任务名称和注释
   - 将常用的操作序列抽取为单独的任务

2. **错误处理**
   - 添加适当的等待时间
   - 使用 `aiAssert` 验证关键步骤
   - 合理设置超时时间

3. **维护建议**
   - 定期更新 Midscene 版本
   - 保持脚本的简洁性
   - 做好版本控制

## 参考资料

- [Midscene 官方文档](https://midscenejs.com/)
- [Chrome 插件地址](https://chromewebstore.google.com/detail/midscene/gbldofcpkknbggpkmbdaefngejllnief)
- [自定义模型和服务商](https://midscenejs.com/zh/model-provider.html)
 

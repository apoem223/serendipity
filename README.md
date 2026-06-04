## serendipity

serendipity是一款简易的旅行地图打卡网站，用户可进入该网站，生成属于自己的旅行地图卡片。本项目是基于VS Code + Vue (Official) (and disable Vetur)所构建的纯前端网站，欢迎您发挥自己的创造力，为本项目增添更多功能。

## 开发者需要的运行环境

开发者克隆/解压项目后，需要确认本机是否配有以下环境：

必须安装：

- Node.js（版本要求为20.19.0 或 >=22.12.0）
- npm(随 Node.js 一起安装)

Q：如何确认自己是否安装：

A：在终端（PowerShell / CMD / Bash）中运行：

    # 检查 Node.js 版本
    node --version
    
    # 检查 npm 版本
    npm --version
    
    #若已安装，会跳出对应的版本号

可选安装：

- vscode编辑器（用于代码编辑）
- Vue Official 插件（vue开发辅助）

## 开发步骤

    # 1. 解压 zip 文件
    # 2. 进入项目目录
    cd serendipity
    
    # 3. 安装依赖
    npm install
    
    # 4. 启动开发服务器
    npm run dev
    
    # 5. （可选）运行 E2E 测试
    npx playwright install
    npm run test:e2e
    

祝您玩的愉快！


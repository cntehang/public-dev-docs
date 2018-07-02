# 前端项目指南

## 基本模版

README： 请参考通用的项目 [README.md 模版](../sample-project-readme.md).

.gitignore: 请参考[.gitignore template](./sample_dot_ignore).

## IDE 配置

前端 Javascript 和 TypeScript 的项目统一采用 VS Code 做为 IDE。按转 VS Code 后，根据不同具体项目安装常用的 Extensions.

### User Settings

The following are some commons user settings for VS Code IDE.

```json
"files.autoSave": "onFocusChange",
"editor.rulers": [
    100
],
"git.enableSmartCommit": true,
"editor.renderWhitespace": "all",
"editor.formatOnSave": true,
"editor.tabSize": 2,
"editor.minimap.enabled": false,
"files.trimFinalNewlines": true,
"files.insertFinalNewline": true
```

### 常用的 Extensions

- [Angular 6 Snippets](https://marketplace.visualstudio.com/items?itemName=Mikael.Angular-BeastCode): TypeScript, HTML, Angular Material, RxJS 等的 snippets.
- [Bracket Pair Colorizer](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer): 给予匹配的括号`()`, `{}`, `[]`不同颜色。
- [Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome): a debug tool for Chrome.
- [Kary Pro Colors](https://marketplace.visualstudio.com/items?itemName=karyfoundation.theme-karyfoundation-themes): 配置主题和颜色。
- [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint): 保证 markdown 文件风格的一致性。
- [Npm Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.npm-intellisense): 自动输入完成 package 的引用。
- [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense): 自动完成文件路径的输入。
- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode): 自动格式 JS/TS 代码。具体配置请参考代码标准文档。
- [PrintCode](https://marketplace.visualstudio.com/items?itemName=nobuhito.printcode): 用于打印代码。
- [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client): 发送和查看 REST API 的请求和响应。支持 CURL 命令。
- [TSLint](https://marketplace.visualstudio.com/items?itemName=eg2.tslint): TypeScript 代码检查工具。和 Preettier 一起用。具体配置参考代码标准文档。
- [TypeScript Hero](https://marketplace.visualstudio.com/items?itemName=rbbit.typescript-hero): 管理排列`imports`。
- [vscode-icons](https://marketplace.visualstudio.com/items?itemName=robertohuertasm.vscode-icons): 各种文件 Icon。

### 命令行运行 VS Code

In VS Code, open `Command Palette` (`Shift + CMD + P`), type `shell command` to find the "Shell Command: Install 'code' command in PATH" command. Run it and restart the terminal for the new `$PATH` value to take effect.

Another way to do it is to add the following to your `~/.bash_profile`: `export PATH="\$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"`. Restart the terminal for the new `$PATH` value to take effect.

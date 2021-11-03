---
title: VSCode设置FiraCode字体
abbrlink: '12238235'
date: 2021-09-28 15:04:19
tags: [工具, vscode]
---
## VSCode设置FiraCode字体
1. 进入FiraCode的[Github仓库](https://github.com/tonsky/FiraCode?_pjax=%23js-repo-pjax-container)下载字体
2. 安装`ttf`文件夹内所有字体
3. 更改vscode的`settings.json`中关于字体的设置参数
```
"editor.fontFamily": "'Fira Code', Consolas, Monaco, 'Courier New', monospace",
"editor.fontLigatures": true,
```
`fontLigatures` 为启用字体连字
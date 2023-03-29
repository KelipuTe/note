## 配置golang开发环境

- 2021-11-17
- VisualStudioCode-1.62

建议vscode的配置按工作区去配置，这样不同项目互不影响。

### 扩展

vscode的扩展主要就三个。

- Go
- CodeRunner
- Chinese

其他的工具是go提供的，vscode会在界面右下角用提示框提示安装。

### 工作区设置（settings.json）

菜单栏=>文件=>首选项=>设置。

在设置中点击`在settings.json中编辑`，就会自动在`.vscode`目录，创建settings.json设置文件。

```json
{
  "editor.fontSize": 16, //编辑器字体大小
  "debug.console.fontSize": 16, //调试控制台字体大小
  "terminal.integrated.fontSize": 16, //控制台字体大小
  "go.formatTool": "goformat", //go格式化工具
  "[go]": {
    "editor.detectIndentation": false, //文件打开时，不基于文件内容强制检测和转换空格和制表符
    "editor.renderWhitespace": "boundary", //显示空格和制表符
    "editor.tabSize": 2, //制表符占2个字符长度
    "editor.insertSpaces": true, //使用空格替代制表符
    "editor.formatOnSave": true, //保存文件时格式化代码
    "editor.codeActionsOnSave": {
      "source.organizeImports": true // 在保存时，自动调整import语句顺序按照字母顺序进行排列
    },
    "editor.suggest.snippetsPreventQuickSuggestions": false //这个属性和代码提示有关
  }
}
```

普通的修改设置不会出现上面的`"[go]": {}`这个参数。

在设置中搜索formatOnSave，然后点击`[go]`标签下面的`在settings.json中编辑`，然后`"[go]": {}`参数就会出现。
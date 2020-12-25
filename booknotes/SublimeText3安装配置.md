##### SublimeText3安装配置

###### 下载Sublime

https://download.sublimetext.com/Sublime%20Text%20Build%203211%20x64%20Setup.exe

###### Package Control组件安装

ctrl + ~ 在下方输入

```shell
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

###### 安装插件

按下Ctrl+Shift+P调出命令面板
输入install 调出 Install Package 选项并回车，然后在列表中选中要安装的插件。

###### 常用插件

Doc​Blockr  注释插件，生成幽美的注释。标准的注释，包括函数名、参数、返回值等，并以多行显示。

SideBarEnhancements  侧栏右键功能增强

Emmet  一种快速编写html/css的方法

JSFormat  JS代码格式化插件。 使用快捷键ctrl+alt+f

BracketHighlighter   类似于代码匹配，可以匹配括号，引号等符号内的范围。

Alignment  代码对齐，使用快捷键 Ctrl+Alt+A

SublimeLinter  一个支持lint语法的插件，可以高亮linter认为有错误的代码行，也支持高亮一些特别的注释，比如"TODO"。

html5  支持hmtl5规范的插件包

jQuery  支持JQuery规范的插件包


###### go开发环境插件

Golang Build  golang编译插件， Tools -> build 或者直接ctrl + b 即可选择通过go run或者go build运行.go文件。

配置Preferences -> package settings -> Golang Config -> Settings - Uesrs

```shell
{
    "PATH": "C:/Go/bin",
    "GOPATH": "C:/Go"
}
```

GoSublime  打造go语言的集成开发环境, 集成了go tool的一些命令如，golint, gocode goimport等工具集。

gosublime当前分支不支持自动安装, 无法使用Package Control组件安装，需要手动将gosublime的代码下载到sublime的package目录下。

进入sublime的package目录，可以通过菜单Preferences=》Browse Package，快速跳转package目录。

 使用git bash，cd到Package目录，运行 git clone https://github.com/DisposaBoy/GoSublime.git ，将代码下载到package目录。下载完后，sublime会自动检测到插件并进行安装。

配置Preferences -> package settings -> Golang Config -> Settings - Default

```shell
# 修改配置项
"gscomplete_enabled": true,

"fmt_enabled": true,

"comp_lint_enabled": true,
```

现在我们的编辑器配置完了，支持代码补全，代码高亮，错误提示，代码格式化等功能




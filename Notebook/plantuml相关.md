## plantuml导出图片方法（vscode）

● 方法1：

> Visual Studio code完成绘图后，通过Ctrl + Shift + P打开命令面板，输入 Export Current Diagram 命令后选择图片格式后即可导出

● 方法2 ：
	
> 在脚本处按快捷键Alt + D，打开预览页面，即可对图片进行复制。

---

## 查看参数列表

- 在 plantuml.jar 所在的文件夹中，使用CMD命令

	```bat
	java -jar plantuml.jar -language
	```

- 在 vscode 中使用如下命令产生一副有所有skinparam的图

	```
	@startuml
	help skinparams
	@enduml
	```
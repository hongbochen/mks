---
title: 在PyQt5中的菜单栏和工具栏
tags: PyQt5, 菜单栏， 工具栏
grammar_cjkRuby: true
---

在这一部分，我们学习创建状态栏，菜单栏和工具栏。一个菜单是位于菜单栏的一组命令。一个工具栏有一些按钮，这些按钮在应用程序中拥有一些常用命令。状态栏显示状态信息，通常位于应用窗口下方。

## QMainWindow

`QMainWindow`类提供了一个主应用窗口。这允许我们创建一个带有状态栏，工具栏和菜单栏的经典程序框架。

### Statusbar(状态栏)

一个状态栏是用于显示状态信息的一个组件。

```

#!/usr/bin/python3
# -*- coding:utf-8 -*- 

	import sys
	from PyQt5.QtWidgets import QMainWindow, QApplication

	class Example(QMainWindow):

		def __init__(self):
			super().__init__()

			self.initUI()

		def initUI(self):
			self.statusBar().showMessage("Ready")

			self.setGeometry(300,300,250,150)
			self.setWindowTitle("StatusBar")
			self.show()

	if __name__ == '__main__':
		app = QApplication(sys.argv)

		ex = Example()

		sys.exit(app.exec_())

```

状态栏在`QMainWindow`组件的帮助下被创建。

```

	self.statusBar().showMessage("Ready")

```

为了获取状态栏，我们调用类`QtGui.QMainWindow`的`statusBar()`方法。该方法的第一个调用创建一个状态栏。子序列调用返回状态栏对象。`showMessage()`展示在状态栏上的信息。

### 简单的菜单

菜单栏是GUI应用程序的通用组件。他是一组位于多个菜单的命令。（Mac OS以不同的方式对待菜单栏。为了获得相似的输出，我们可以添加下列一行:`menubar.setNativeMenubar(False)`）




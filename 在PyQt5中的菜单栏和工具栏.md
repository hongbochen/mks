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

下面是这个小例子程序的运行结果：
![enter description here][1]


### 简单的菜单

菜单栏是GUI应用程序的通用组件。他是一组位于多个菜单的命令。（Mac OS以不同的方式对待菜单栏。为了获得相似的输出，我们可以添加下列一行:`menubar.setNativeMenubar(False)`。）

```

#!/usr/bin/python3
# -*- coding:utf-8 -*-

import sys
from PyQt5.QtWidgets import QMainWindow,QAction, QApplication, qApp
from PyQt5.QtGui import QIcon

class Example(QMainWindow):
    def __init__(self):
        super().__init__()
        
        self.initUI()
        
    def initUI(self):
        exitAct = QAction(QIcon('exit.png'),'&Exit',self)
        exitAct.setShortcut('Ctrl+Q')
        exitAct.setStatusTip("Exit application")
        exitAct.triggered.connect(qApp.quit)
        
        self.statusBar()
        
        menubar = self.menuBar()
        
        fileMenu = menubar.addMenu("&File")
        fileMenu.addAction(exitAct)
        
        self.setGeometry(300,300,300,200)
        self.setWindowTitle("Simple menu")
        self.show()
        
if __name__ == '__main__':
    
    app = QApplication(sys.argv)
    
    ex = Example()
    
    sys.exit(app.exec_())

```

在上面的例子程序中，我们创建了一个带有一个菜单的菜单栏。这个菜单包含一个动作，如果选中的话，将会终止该应用程序。当然，也创建了一个状态栏。这个动作也可以使用`Ctrl+Q`快捷键。

```

exitAct = QAction(QIcon("exit.png"),"&Exit",self)
exitAct.setShortcut("Ctrl+Q")
exitAct.setStatusTip("Exit application")

```

`QAction`是一个运行在菜单栏，工具栏和定制键盘快捷键的抽象类。在上面三行中，我们使用特定的图标和一个'Exit'标签创建了一个行为。进一步说，一个快捷键为了这个行为被定义。第三行创建了一个状态提示，当鼠标经过该菜单选项的时候，被显示在状态栏上。

```

exitAct.triggered.connect(qApp.quit)

```

当我们选中这个特定的行为的时候，一个触发的信号被提交。该信号被连接到`QApplication`组件的`quit()`方法。这个会终止这个程序。

```

menubar = self.menuBar()
fileMenu = menubar.addMenu("&File")
fileMenu.addAction(exitAct)

```

`menuBar()`方法创建了一个菜单栏。我们使用`addMenu()`创建了一个文件按钮，并且使用`addAction()`方法添加一个行为。

下面是该小例子的截图：

![enter description here][2

### 子菜单

一个子菜单是位于另外一个菜单中的一个菜单。

```

#!/usr/bin/python3
# -*- coding:utf-8 -*-

import sys
from PyQt5.QtWidgets import QMainWindow, QAction, QMenu, QApplication

class Example(QMainWindow):
    
    def __init__(self):
        super().__init__()
        
        self.initUI()
        
    def initUI(self):
        menubar = self.menuBar()
        fileMenu = menubar.addMenu("File")
        
        impMenu = QMenu("Import",self)
        impAct = QAction("Import mail",self)
        impMenu.addAction(impAct)
        
        newAct = QAction("New", self)
        
        fileMenu.addAction(newAct)
        fileMenu.addMenu(impMenu)
        
        self.setGeometry(300,300,300,200)
        self.setWindowTitle("Submenu")
        
        self.show()
        
if __name__ == '__main__':
    
    app = QApplication(sys.argv)
    
    ex = Example()
    
    sys.exit(app.exec_())

```

在这个例子中，我们有两个菜单选项；一个位于文件菜单中，另一个位于文件的Import子菜单中。




  [1]: https://github.com/hongbochen/mks/blob/master/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20171224005701.png?raw=true
  [2]: https://github.com/hongbochen/mks/blob/master/images/simplemenu.png?raw=true
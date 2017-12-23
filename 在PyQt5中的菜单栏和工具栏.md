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

```

impMenu = QMenu("Import", self)

```

新的菜单使用`QMenu`创建。

```

impAct = QAction("Import mail", self)
impMenu.addAction(impAct)

```

一个行为通过使用`addAction()`被添加到子菜单中。

![enter description here][2]

### 选项菜单

在下面的例子中，我们创建了一个按钮可以被选中或者是不被选中。

```

#!/usr/bin/python3
# -*- coding:utf-8 -*-

import sys
from PyQt5.QtWidgets import QMainWindow,QApplication,QAction

class Example(QMainWindow):
    
    def __init__(self):
        super().__init__()
        
        self.initUI()
        
    def initUI(self):
        
        self.statusbar = self.statusBar()
        self.statusbar.showMessage("Ready")
        
        menubar = self.menuBar()
        viewMenu = menubar.addMenu("View")
        
        viewStatAct = QAction("View statusbar",self,checkable=True)
        viewStatAct.setStatusTip("View statusbar")
        viewStatAct.setChecked(True)
        viewStatAct.triggered.connect(self.toggleMenu)
        
        viewMenu.addAction(viewStatAct)
        
        self.setGeometry(300,300,300,200)
        self.setWindowTitle("Check menu")
        self.show()
        
    def toggleMenu(self,state):
        if state:
            self.statusbar.show()
        else:
            self.statusbar.hide()
            

if __name__ == "__main__":
    app = QApplication(sys.argv)
    
    ex = Example()
    
    sys.exit(app.exec_())

```

这个代码例子创建了带有一个行为的视图菜单。这个行为显示或者是隐藏状态栏。当状态栏可视的时候，菜单选项被选中。

```

viewStatAct = QAction('View statusbar', self, checkable=True)

```

使用`checkable`选项，我们创建了一个可选择菜单。

```

viewStatAct.setChecked(True)

```

因为状态栏在一开始的时候是可视的，我们使用`setChecked()`方法来设置该行为。

```

def toggleMenu(self, state):
	if state:
		self.statusbar.show()
	else:
		self.statusbar.hide()

```

依赖于行为选中的状态，我们设置状态栏是否显示。

![enter description here][3]

### 上下文菜单

一个上下文菜单，也被称作弹出菜单，一个出现在一些上下文中的一个命令列表。例如，在一个Opera网页浏览器中，当你在一个网页中右击的时候，我们获得一个上下文菜单。在这里我们可以重新加载一个页面，回退，或者是查看页面源码。如果我们右击一个工具栏，我们将会得到管理工具栏的另一个上下文菜单。

```

#!/usr/bin/python3
# -*- coding:utf-8 -*-

import sys
from PyQt5.QtWidgets import QMainWindow, qApp,QMenu,QApplication

class Example(QMainWindow):
    
    def __init__(self):
        super().__init__()
        
        self.initUI()
        
    def initUI(self):
        self.setGeometry(300,300,300,200)
        self.setWindowTitle("Context menu")
        
        self.show()
        
    def contextMenuEvent(self,event):
        cmenu = QMenu(self)
        
        newAct = cmenu.addAction("New")
        opnAct = cmenu.addAction("Open")
        quitAct = cmenu.addAction("Quit")
        action = cmenu.exec_(self.mapToGlobal(event.pos()))
        
        if action == quitAct:
            qApp.quit()
            
if __name__ == '__main__':
    app = QApplication(sys.argv)
    
    ex = Example()
    
    sys.exit(app.exec_())

```

为了能够使用上下文菜单，我们必须重新集成`contextMenuEvent()`方法。

```

action = cmenu.exec_(self.mapTpGlobal(event.pos()))

```

该上下文菜单被`exec_()`方法显示。他们从事件对象中获得鼠标指针的坐标。`mapToGlobal()`方法传递组件的坐标到全局的屏幕坐标。

```

if action == quitAct:
	qApp.quit()

```

  [1]: https://github.com/hongbochen/mks/blob/master/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20171224005701.png?raw=true
  [2]: https://github.com/hongbochen/mks/blob/master/images/submenu.png?raw=true
  [3]: https://github.com/hongbochen/mks/blob/master/images/checkmenu.png?raw=true
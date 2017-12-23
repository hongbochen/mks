在这一部分，我们学习PyQt5的一些基本功能。下面的例子展示了一个工具栏和一个图标，关闭窗口，显示一个消息窗口和将一个窗口放到桌面的中心。

### 简单的例子

这是一个显示一个小窗口的简单的例子。但是对于这个窗口我们可以做很多事情。我们可以重定义大小，最大化或最小化。这个需要很多编码。但是有些人已经将他们编写成了函数。因为这个在大多数应用中是重复的，没有必要一次又一次的编码。PyQt5是一个高层工具。如果我们使用一个底层的工具去编码的话，下面的编码例子将会很容易的到达成百上千行。

```

	#!/usr/bin/python3
	#-*- coding: utf-8 -*-
	
	import sys
	from PyQt5.QtWidgets import QApplication,QWidget
	
	if __name__ == '__main__':
	    
	    app = QApplication(sys.argv)
	    
	    w = QWidget()
	    w.resize(250,150)
	    w.move(300,300)
	    w.setWindowTitle("Simple")
	    w.show()
	    
	    sys.exit(app.exec_())

```

上面的编码例子在屏幕上显示了一个小的窗口。

```

import sys
	from PyQt5.QtWidgets import QApplication,QWidget

```

在这里我们提供了必要的引入。基础组件位于`PyQt5.QtWidgets`模块中。

```

	app = QApplication(sys.argv)

```

每一个PyQt5应用都必须要创建一个应用对象。`sys.argv`参数是来自命令行的一系列参数。Python脚本可以在shell中被运行。这就是一个我们控制我们脚本启动的一个方式。

```

w = QWidget()

```

`QWidget`是PyQt5的所用用户接口对象中的一个基础类。我们提供了`QWidget`的默认构造器。默认构造器没有父类。没有父类的widget被称为窗口。

```

w.resize(250,150)

```

`resize()`函数重新定义了widget的大小，他是250像素宽，150像素宽。

```

w.move(300,300);

```

`move()`函数移动组件的位置到x=300,y=300的位置上去。

```

w.setWindowTitle("Simple")

```

我们通过`setWindowTitle()`函数来这是窗口名称。这个名称被显示到名称栏里。

```

w.show()

```

`show()`方法在屏幕上显示该组件。组件首先在内存中创建，然后再屏幕上显示。

```

sys.exit(app.exec_())

```

最后，我们进入应用的主循环。时间的处理也就是从这里开始的。主循环接受来自窗口系统的时间，并且将他们分发到应用组件中。如果我们调用`exit()`函数或者是主组件被毁掉，主循环就会结束。`sys.exit()`方法确定是清理退出。环境将会指导应用结束。

`exec_()`函数有一个下划线。这是因为`exec`是Python的一个关键字，所以`exec_()`就被使用了。


### 应用图标

应用图标是一个小的图片，他被暂时在标题栏的左上角。在下面的例子中，我们将会展示如何在PyQt5中实现应用的图标。我们也将会介绍一些新的方法。

一些环境在标题栏中并不显示图标。我们需要使能他们。如果你看不到图标，可以在StackOverFlow中查看相关答案。

```

	#!/usr/bin/python3
	# -*- coding: utf-8 -*-
	
	import sys
	from PyQt5.QtWidgets import QApplication,QWidget
	from PyQt5.QtGui import QIcon
	
	class Example(QWidget):
	    
	    def __init__(self):
	        super().__init__()
	        
	        self.initUI()
	        
	    def initUI(self):
	        self.setGeometry(300,300,300,200)
	        self.setWindowTitle("Icon")
	        self.setWindowIcon(QIcon('head.png'))
	        
	        self.show()
	        
	if __name__ == '__main__':
	    app = QApplication(sys.argv)
	    ex = Example()
	    
	    sys.exit(app.exec_())

```

在之前的例子都是以面向过程的方式编码。Python编程语言既支持面向过程，也支持面向对象。在PyQt5编程意味着要进行面向对象的编程。

```

	class Example(QWidget):
	    
	    def __init__(self):
	        super().__init__()
	        
	        self.initUI()

```

在面向对象编程中有三个重要的事情，分别是类，数据和方法。在这里我们创建了一个新的类称作`Example`。`Example`类继承自`QWidget`类。这就意味着我们可以调用两个构造函数：第一个是`Example`类的和第二个是继承自的类的。`super()`函数返回了`Example`类的父类。`__init__()`方法是在Python语言中的构造器函数。

```

self.initUI()

```

GUI的创建被代理给了`initUI()`方法。

```

	self.setGeometry(300,300,300,200)
    self.setWindowTitle("Icon")
    self.setWindowIcon(QIcon('head.png'))

```

所有的三个方法都已经在`QWidget`类中被实现了。函数`setGeometry()`做了两件事情：他定位了在屏幕上的窗口并且设置了他的大小。前两个参数是窗口的x和y位置。第三个是窗口的宽度，第四个是高度。实际上，他组合了`resize()`和`move()`方法到了一个方法中。最后一个函数设置了应用的图标。为了实现，我们已经创建了一个`QIcon`对象。`QIcon`接收图标的位置并显示。

下面是实现演示：

![](https://github.com/hongbochen/mks/blob/master/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20171120223523.png?raw=true)

###显示一个工具栏

我们可以为任何一个组件提供我们的帮助。

```

	#!/usr/bin/python3
	# -*- coding: utf-8 -*- 
	
	import sys
	from PyQt5.QtWidgets import (QWidget, QToolTip,
	                            QPushButton, QApplication)
	from PyQt5.QtGui import QFont
	
	class Example(QWidget):
	    
	    def __init__(self):
	        super().__init__()
	        
	        self.initUI()
	    
	    def initUI(self):
	        QToolTip.setFont(QFont("SanaSerif",10))
	        
	        self.setToolTip("This is a <b>QWidget</b> widget")
	        
	        btn = QPushButton('Button',self)
	        btn.setToolTip('This is a <b>QPushButton</b> widget')
	        btn.resize(btn.sizeHint())
	        btn.move(50,50)
	        
	        self.setGeometry(300,300,300,200)
	        self.setWindowTitle("ToolTip")
	        self.show()
	        
	    
	if __name__ == '__main__':
	    
	    app =QApplication(sys.argv)
	    
	    ex = Example()
	    sys.exit(app.exec_())

```

在这个例子中，我们为两个PyQt5的组件显示了一个状态。

```

	QToolTip.setFont(QFont("SanaSerif",10))

```

这个静态方法设置用于去渲染提醒的字体。我们使用10px大小的SanaSerif字体。

```

	self.setToolTip("This is a <b>QWidget</b> widget")

```

为了创建一个提醒，我们调用`setToolTip()`方法。我们使用富文本格式。

```

 	btn = QPushButton('Button',self)
    btn.setToolTip('This is a <b>QPushButton</b> widget')
        

```	

我们创建了一个按钮组件并为他创建了提醒。

```

	btn.resize(btn.sizeHint())
	btn.move(50,50)

```

该按钮被重定义大小并且在窗口上移动。`sizeHint()`方法给了一个推荐的按钮的大小。

下面是这个例子的运行效果：

![](https://github.com/hongbochen/mks/blob/master/images/%E5%B0%9A%E6%9C%AA.png?raw=true)

### 关闭一个窗口

关闭窗口传统的方式就是点击位于标题栏上的x标示。在下面的例子中，我们将会显示我们如何通过编程方式关闭我们的窗口。我们将简单介绍信号和槽。

下面是我们在示例中使用的`QPushButton`组件的构造器：

```

	QPushButton(string text, QWidget parent = None)

```

`text`参数是展示在按钮上的文本。`parent`是我们将要把我们的按钮放到哪个组件里面。在我们的示例中，他将会是一个组件。应用程序的组件构成了一个体系结构。在这种等级制度下，大多数组件都有他们的父组件。没有父组件的组件是顶层组件。

```

	#!/usr/bin/python3
	# -*- coding: utf-8 -*-
	
	import sys
	from PyQt5.QtWidgets import QWidget,QApplication,QPushButton
	from PyQt5.QtCore import QCoreApplication
	
	class Example(QWidget):
	    
	    def __init__(self):
	        
	        super().__init__()
	        
	        self.initUI()
	        
	    def initUI(self):
	        qbtn = QPushButton("Quit",self)
	        
	        qbtn.clicked.connect(QCoreApplication.instance().quit)
	        qbtn.resize(qbtn.sizeHint())
	        qbtn.move(50,50)
	        
	        self.show()
	
	if __name__ == '__main__':
	    
	    app = QApplication(sys.argv)
	    
	    ex = Example()
	    
	    sys.exit(app.exec_())

```

在这个例子中，我们创建了一个退出按钮。当点击这个按钮的时候，应用退出了。

```

from PyQt5.QtCore import QCoreApplication

```

我们需要来自`QtCore`模块的`QCoreApplication`类。

```

qbtn = QPushButon("Quit",self)

```

我们创建了一个按钮。这个按钮是`QPushButton`类的实例。构造函数第一个参数是按钮的标签，第二个参数是父组件。父组件就是`Example`组件，继承自`QWidget`的组件。

```

	pbtn.clicked.connect(QCoreApplication.instance().quit)

```

在PyQt5中的事件处理系统是由信号和槽机制实现的。如果我们点击了按钮，信号`clicked`被激发。槽可以是Qt的槽或者是任何Python调用。`QCoreApplication`包含主要是事件循环－－他处理并且转发这些事件。`instance()`方法给我们当前他的实例。注意`QCoreApplication`是与`QApplication`被创建的。点击的信号被连接到`quit()`方法中，该方法结束整个程序。介于两个对象之间的通信已经完成了：发送者和接受者。发送者就是按钮，接受者就是应用对象。

![](https://github.com/hongbochen/mks/blob/master/images/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20171121001420.png?raw=true)

### 消息盒子

默认情况下，如果我们点击标题栏的x按钮，`QWidget`就被关闭了。有时候我们想要修改这个默认行为。例如，如果我们有一个文件在编辑器中被打开了，我们做了一些操作。我们需要显示一个对话框来确定操作。

```

	#!/usr/bin/python3
	# -*- coding: utf-8 -*-
	
	import sys
	from PyQt5.QtWidgets import QWidget,QMessageBox,QApplication
	
	class Example(QWidget):
	    
	    def __init__(self):
	        super().__init__()
	        
	        self.initUI()
	        
	    def initUI(self):
	        self.setGeometry(300,300,250,150)
	        self.setWindowTitle("Message Box")
	        self.show()
	        
	    def closeEvent(self,event):
	        reply = QMessageBox.question(self,"Message","Are you sure to quit?",QMessageBox.Yes|QMessageBox.No, QMessageBox.No)
	        
	        if reply == QMessageBox.Yes:
	            event.accept()
	        else:
	            event.ignore()
	            
	if __name__ == '__main__':
	    
	    app = QApplication(sys.argv)
	    
	    ex = Example()
	    
	    sys.exit(app.exec_())

```

如果我们关闭一个`QWidget`，`QCloseEvent`被生成。为了修改这个行为，我们需要重新实现`closeEvent()`事件处理。

```

	reply = QMessageBox.question(self,"Message","Are you sure to quit?",QMessageBox.Yes|QMessageBox.No, QMessageBox.No)
        

```

我们显示了一个带有两个按钮的消息盒子：是和否。第一个字符串是标题栏，第二个字符串是显示在对话框中的信息文本。第三个参数指定了显示在对话框中的按钮的组合。最后一个参数是默认按钮。也就是初始化时按键焦点所在的按钮。返回值被保存到`reply`变量中。

```

if reply == QtGui.QMessageBox.Yes:
    event.accept()
else:
    event.ignore()  

```

在这里我们测试了返回值。如果我们点击Yes按钮，我们将会接收事件然后关闭组件并终止应用。否则，我们将会忽略关闭事件。

![](https://github.com/hongbochen/mks/blob/master/images/wdqddq.png?raw=true)

### 将窗口放到屏幕中央

下面的代码展示了如何将一个窗口放到桌面中央。

```

	#!/usr/bin/python3
	# -*- coding:utf-8 -*-
	
	import sys
	from PyQt5.QtWidgets import QWidget,QDesktopWidget,QApplication
	
	class Example(QWidget):
	    def __init__(self):
	        super().__init__()
	        
	        self.initUI()
	        
	    def initUI(self):
	        
	        self.resize(250,150)
	        self.center()
	        
	        self.setWindowTitle("Center")
	        
	        self.show()
	        
	    def center(self):
	        
	        qr = self.frameGeometry()
	        cp = QDesktopWidget().availableGeometry().center()
	        qr.moveCenter(cp)
	        self.move(qr.topLeft())
	        
	if __name__ == '__main__':
	    
	    app = QApplication(sys.argv)
	    
	    ex = Example()
	    
	    sys.exit(app.exec_())

```

`QDesktopWidget`类提供了关于用户桌面的信息，包括屏幕大小。

```

	self.center()


```

将会至于窗口到中间的代码被放到定制化的`center()`方法中。

```

	qr = self.frameGeometry()

```

我们获得一个方形的特定的主窗口的图形。这包含所有的窗口帧。

```
	cp = QDesktopWidget().availableGeometry().center()

```

我们弄清楚了我们监视器的屏幕分辨率，然后通过这个分辨率，我们获得了中心点。

```

	qr.moveCenter(cp)

```

我们的矩形已经有了他的长和宽。现在我们设置矩形的中心为屏幕的中西。这个矩形的大小不能被改变。

```

	self.move(qr.topLeft())

```

我们移动应用窗口的左上角到矩形的左上角，这样窗口就能处在屏幕中间了。



# How to Create Custom RQT Plugin for Dummies

## Link to official tutorial and other useful resources
[Create your new rqt plugin](http://wiki.ros.org/rqt/Tutorials/Create%20your%20new%20rqt%20plugin#Install_.26_Run_your_plugin) from ROS RQT Tutorials which is missing a lot of pieces. 
Much of this is from [Lucas Walter's rqt_mypkg implementation](https://github.com/lucasw/rqt_mypkg/tree/master/rqt_example_py) and [How-Chen](https://github.com/how-chen/rqt_mypkg)

[Python QT Tutorial](http://wiki.ros.org/python_qt_binding) for when you want to create plugin with QT

To see some of complete rqt plugin examples, go [here](https://github.com/ros-visualization/rqt_common_plugins/tree/bd5efbb4f4f2e8ca40e54725e674e7ac5afdd0ba)

Creating Qt using Qt Designer example can be seen in this book, book is to big, so use your favorite site to download it: __Learning Robotics using Python: Design, simulate, program, and prototype an autonomous mobile robot using ROS, OpenCV, PCL, and Python__


## Overview Code Structure

* workspace
  * build
  * devel
  * src/rqt
    * rqt_mypkg
      * resource
        * MyPlugin.ui
      * scripts
        * rqt_mypkg
      * src/rqt_mypkg
        * __init\__.py
        * my_module.py
      * CMakeLists.txt
      * package.xml
      * plugin.xml
      * setup.py


## Summary of How to Create Custom Plugin / Dashboard

There are 3 main ways
1. Can just create everything from scratch by yourself (how the GUI will look ie buttons and stuff, how ). This will all be programming (so you program where to place the box like in javascript, also program interaction)
2. Use Qt Desginer to help design the GUI and interaction (what it will do when I press the button). Then convert it to python code later.
3. If you like RQT(ROS QT) and just want to combine them, could just run: " rosrun rqt_gui rqt_gui " Then pick whatever rqt plugin (or your own plugin is possible too I think) and drag them down to dashboard.

## Executing this program: 
Follow this if only this git repo have the desire Plugin, if not, follow the instruction below it about how to Create Custom Plugin
### 1. Clone the repository
```
% cd [catkin workspace]/src
```
then clone this directory
```
% git clone https://github.com/ChoKasem/rqt_tut.git
```

if it doesn't work, use the following directory instead as a starter
```
% git clone https://github.com/how-chen/rqt_mypkg.git
```

### 2. Specify execution script

source your workspace if haven't done it yet, 
```
% cd ~/[workspace_name]
% source ./devel/setup.bash
```

then make the script executable

```
% roscd rqt_mypkg/scripts
% chmod +x rqt_mypkg
```


### 3. Run Catkin Make
```
% cd [catkin workspace]
% catkin_make
```
### 4. Start roscore
In a new terminal window
```
% roscore
```

### 5. Run Script
```
% rosrun rqt_mypkg rqt_mypkg
```

## Creating rqt plugin from scratch
The following is largely from: [Create your rqt plugin package](http://wiki.ros.org/rqt/Tutorials/Create%20your%20new%20rqt%20plugin#Install_.26_Run_your_plugin)

### 1. Create your rqt plugin package 
Navigate to the source folder within your catkin workspace directory.
```
cd ~/catkin_ws/src
```

Create empty rqt package
```
catkin_create_pkg rqt_mypkg rospy rqt_gui rqt_gui_py
```

#### Edit the package.xml file.
Between the `<export>`  `</export>` tags, add the following:

```
<rqt_gui plugin="${prefix}/plugin.xml"/>
```

#### Create the plugin.xml file
create a plugin.xml file with the following code: 

```
<library path="src">
  <class name="My Plugin" type="rqt_mypkg.my_module.MyPlugin" base_class_type="rqt_gui_py::Plugin">
    <description>
      An example Python GUI plugin to create a great user interface.
    </description>
    <qtgui>
      <!-- optional grouping...
      <group>
        <label>Group</label>
      </group>
      <group>
        <label>Subgroup</label>
      </group>
      -->
      <label>My first Python Plugin</label>
      <icon type="theme">system-help</icon>
      <statustip>Great user interface to provide real value.</statustip>
    </qtgui>
  </class>
</library>
```
### 2. Create a python plugin
Based on the [rQT Python Plugin ROS Tutorial](http://wiki.ros.org/rqt/Tutorials/Writing%20a%20Python%20Plugin)

#### Create a GUI Layout
You can create a GUI using QT designer. This will output a .ui file. For this exercise, we will use a pre-made gui. Create a 'resource' folder inside the rqt_mypkg folder. 
```
% roscd rqt_mypkg
% mkdir resource
```

note: if roscd does not work, 
```
% cd ~/[workspace_name]
% source ./devel/setup.bash
```

within the resource folder, create a file called 'MyPlugin.ui' with the following contents
```
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Form</class>
 <widget class="QWidget" name="Form">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <widget class="QPushButton" name="Test">
   <property name="geometry">
    <rect>
     <x>120</x>
     <y>70</y>
     <width>98</width>
     <height>27</height>
    </rect>
   </property>
   <property name="text">
    <string>Original Name</string>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>
```

#### GUI Script 
make the src/rqt_mypkg/ folder inside the rqt_mypkg package directory
```
% roscd rqt_mypkg
% mkdir -p src/rqt_mypkg
```

create __init__.py
```
% cd src/rqt_mypkg/
% touch __init__.py
```

within the same folder, create my_module.py with the following contents:

```
import os
import rospy
import rospkg

from qt_gui.plugin import Plugin
from python_qt_binding import loadUi
from python_qt_binding.QtWidgets import QWidget

class MyPlugin(Plugin):

    def __init__(self, context):
        super(MyPlugin, self).__init__(context)
        # Give QObjects reasonable names
        self.setObjectName('MyPlugin')

        # Process standalone plugin command-line arguments
        from argparse import ArgumentParser
        parser = ArgumentParser()
        # Add argument(s) to the parser.
        parser.add_argument("-q", "--quiet", action="store_true",
                      dest="quiet",
                      help="Put plugin in silent mode")
        args, unknowns = parser.parse_known_args(context.argv())
        if not args.quiet:
            print 'arguments: ', args
            print 'unknowns: ', unknowns

        # Create QWidget
        self._widget = QWidget()
        # Get path to UI file which should be in the "resource" folder of this package
        ui_file = os.path.join(rospkg.RosPack().get_path('rqt_mypkg'), 'resource', 'MyPlugin.ui')
        # Extend the widget with all attributes and children from UI file
        loadUi(ui_file, self._widget)
        # Give QObjects reasonable names
        self._widget.setObjectName('MyPluginUi')
        # Show _widget.windowTitle on left-top of each plugin (when 
        # it's set in _widget). This is useful when you open multiple 
        # plugins at once. Also if you open multiple instances of your 
        # plugin at once, these lines add number to make it easy to 
        # tell from pane to pane.
        if context.serial_number() > 1:
            self._widget.setWindowTitle(self._widget.windowTitle() + (' (%d)' % context.serial_number()))
        # Add widget to the user interface
        context.add_widget(self._widget)

    def shutdown_plugin(self):
        # TODO unregister all publishers here
        pass

    def save_settings(self, plugin_settings, instance_settings):
        # TODO save intrinsic configuration, usually using:
        # instance_settings.set_value(k, v)
        pass

    def restore_settings(self, plugin_settings, instance_settings):
        # TODO restore intrinsic configuration, usually using:
        # v = instance_settings.value(k)
        pass

    #def trigger_configuration(self):
        # Comment in to signal that the plugin has a way to configure
        # This will enable a setting button (gear icon) in each dock widget title bar
        # Usually used to open a modal configuration dialog
```
note: `from python_qt_binding.QtGui import QWidget` has been modified to `from python_qt_binding.QtWidgets import QWidget`

#### Main script
Create a 'scripts' folder
```
% roscd rqt_mypkg
% mkdir scripts
```

inside the 'scripts' folder, make a file called rqt_mypkg with the following contents. This is the 'point of entry' for rqt. 

```
#!/usr/bin/env python

import sys

from rqt_mypkg.my_module import MyPlugin
from rqt_gui.main import Main

plugin = 'rqt_mypkg'
main = Main(filename=plugin)
sys.exit(main.main(standalone=plugin))
```


### 3. Install
navigate to the base directory
```
roscd rqt_mypkg
```

#### setup.py
create setup.py script with the following:
```
from distutils.core import setup
from catkin_pkg.python_setup import generate_distutils_setup
  
d = generate_distutils_setup(
    packages=['rqt_mypkg'],
    package_dir={'': 'src'},
)

setup(**d)
```

#### CMakeLists
open the CMakeLists
```
% roscd rqt_mypkg
% gedit CMakeLists.txt
```

uncomment (delete the '#' symbol) the following from the CMakeLists.txt
```
catkin_python_setup()
```
uncomment the following lines: 
```
install(PROGRAMS
  scripts/my_python_script
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```
and change `scripts/my_python_script` to `scripts/rqt_mypkg`. Note that it newer ros would be catkin_install_python, if that doesn't work just change it to install just like above.

Also, before 'Testing', add the following: 

```
install(DIRECTORY
  resource
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(FILES
  plugin.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
```

#### Specify main script 
Specify the main execution script 
```
% roscd rqt_mypkg
% cd scripts
% chmod +x rqt_mypkg
```

#### Run Catkin Make
```
% cd [catkin workspace]
% catkin_make
```

#### Start Roscore
In a new terminal window
```
% roscore
```

#### Run your RQT script
```
rosrun rqt_mypkg rqt_mypkg
```

## First Method: Make your own .ui file
Using qtdesigner, make a plain widget. Follow this [link](http://wiki.ros.org/python_qt_binding) to learn about Qt Widget python binding

### Reusing widget/GUI in .ui
Why do we have to recreate all widget if we could reuse the widget inside another widget by just referencing it. This will save time and me it more modular. Follow this [link](http://wiki.ros.org/rqt/Tutorials/Using%20.ui%20file%20in%20rqt%20plugin)

#### Inside the .ui file
To add another widget and reuse them, you simply add the widget class you want
```
<widget class="TopicWidget" name="_widget_topic" native="true">
  <property name="enabled">
  :
</widget>
```

BUT THEN at the end of your .ui file, you have to add this customwidget tag to tell that this is extension and where to find it
```
 <customwidgets>
  <customwidget>
    <class>TopicWidget</class>
    <extends>QWidget</extends>
    <header>rqt_topic.topic_widget</header>
  </customwidget>
 </customwidgets>
```

#### Inside my_module.py file (or the script file)
now you need to import whatever widget you use
```
from rqt_topic.topic_widget import TopicWidget
```
and tell rqt the custom class you use
```
loadUi(ui_file, self, {'TopicWidget': TopicWidget})
```
and reference tehm in self
```
self._widget_topic.set_selected_topics(self._selected_topics)
```

#### What you should do
All this is a bit complicated, best way is to just copy and paste then modify existing .ui and script from [here](https://github.com/ros-visualization/rqt_common_plugins/tree/51cef97fa059e68b9756058956db0f2f6ff8934f)


## Second Method: Using Qt Designer to Create Custom Plugin
This part refer to the book in the top of this readme
Download Qt with this command
```
sudo apt-get install python-qt4 pyqt4-dev-tools
```
An alternative to Qt is PySide, which can be download with the following command
```
sudo apt-get install python-pyside pyside-tools
```

Use Qt Designer to create your plugin. There you can create button, etc, and specify what signal it send when press, hold, etc.

After you have a .ui file from the designer, you can convert it to python file with the following command for PyQt and PySlide (they are almost equivalent)

For PyQt:
```
$ pyuic4 -x hello_world.ui -o hello_world.py
```
For PySlide:
```
$ pyside-uic -x hello_world.ui -o hello_world.py
```

It will looks like this:
```
from PyQt4 import QtCore, QtGui
try:
  _fromUtf8 = QtCore.QString.fromUtf8
except AttributeError:
  _fromUtf8 = lambda s: s
class Ui_Form(object):
  def setupUi(self, Form):
    Form.setObjectName(_fromUtf8("Form"))
    Form.resize(514, 355)
    self.pushButton = QtGui.QPushButton(Form)
    self.pushButton.setGeometry(QtCore.QRect(150, 80, 191, 61))
    self.pushButton.setObjectName(_fromUtf8("pushButton"))
    self.retranslateUi(Form)
    QtCore.QObject.connect(self.pushButton,
    QtCore.SIGNAL(_fromUtf8("clicked()")), Form.message)
    QtCore.QMetaObject.connectSlotsByName(Form)

  def retranslateUi(self, Form):
    Form.setWindowTitle(QtGui.QApplication.translate("Form", "Form",
    None, QtGui.QApplication.UnicodeUTF8))
    self.pushButton.setText( QtGui.QApplication.translate("Form",
    "Press", None, QtGui.QApplication.UnicodeUTF8))
    #This following code should be added manually
if __name__ == "__main__":
  import sys
    app = QtGui.QApplication(sys.argv)
    Form = QtGui.QWidget()
    ui = Ui_Form()
    ui.setupUi(Form)
    Form.show()
    sys.exit(app.exec_())
```
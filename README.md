# How to Create Custom RQT Plugin for Dummies
[Create your new rqt plugin](http://wiki.ros.org/rqt/Tutorials/Create%20your%20new%20rqt%20plugin#Install_.26_Run_your_plugin) from ROS rQT Tutorials which is missing a lot of pieces. 
Much of this is from [Lucas Walter's rqt_mypkg implementation](https://github.com/lucasw/rqt_mypkg/tree/master/rqt_example_py) and [How-Chen](https://github.com/how-chen/rqt_mypkg)

To see some of complete examples, go [here](https://github.com/ros-visualization/rqt_common_plugins/tree/bd5efbb4f4f2e8ca40e54725e674e7ac5afdd0ba)

## Executing this program: 
Follow this if only this git repo have the desire Plugin, if not, follow the instruction below it about how to Create Custom Plugin
### 1. Clone the repository
```
% cd [catkin workspace]/src
```
then clone this directory

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

## Make your own .ui file
Using qtdesigner, make a plain widget. 

# How to make Custom Plug for Dummy

clone this package into the src of the workspace

## Folder Structure
- rqt_mypkg
    - MyPlugin.ui
- script
    - rqt_mypkg.py
- src/my_mypkg
    - \_\_init__.py
    - my_module.py
- CMakeLists.txt
- package.xml
- plugin.xml

## Explaination
### .ui File
Pretty much tell how the GUI is going to look like. Each element (like button, slidebar, etc.) is call widget. There could be widget inside of widget

In the end, it's just an xml style file. Two ways you can make them
1. use QTcreator tool to create it, 
2. if you know xml and qt element, you can just do everything hardcode by just coding xml.

### rqt_mypkg.py
A python file that 

### plugin.xml
reference file containing meta data of your plugin (like name, type, base class, etc)

## Step to create custom plugin
1. create a new package
```catkin_create_pkg rqt_mypkg rospy rqt_gui rqt_gui_py```


2. Modify `package.xml`
add the folllowing to package.xml
```
<package>
  :
  <!-- all the existing tags -->
  <export>
    <rqt_gui plugin="${prefix}/plugin.xml"/>
  </export>
  :
</package>
```
3. Create `plugin.xml`
and put the following inside

```<library path="src">
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

4. Create directory as shown in the structure

5. Create `__init__.py` at the location shown in strucutre, it should be empty inside

6. Create `my_module.py`, put the following code as starter
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

7. create `rqt_mypkg.py` file, put the following code in there
```
#!/usr/bin/env python

import sys

from rqt_mypkg.my_module import MyPlugin
from rqt_gui.main import Main

plugin = 'rqt_mypkg'
main = Main(filename=plugin)
sys.exit(main.main(standalone=plugin))
```

8. Create the ui file by creating `MyPlugin.ui` in a location specify in structure. Put the following code in it as starter

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

9. test the code to see if everything is connect by sourcing the workspace, catkin make it, and run `rqt --standalone rqt_mypkg`
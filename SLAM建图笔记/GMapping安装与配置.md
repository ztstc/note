# GMapping安装与配置

https://zhuanlan.zhihu.com/p/262287388

1.安装包

```
sudo apt update
```

```
sudo apt-get install ros-noetic-gmapping 
```

> [!NOTE]
>
> 根据自己的<ros-distro>版本替换noetic

2.在自己的工作空间内添加gmapping.launch文件

```
<launch>
    <arg name="scan_topic" default="scan" />                  <!-- 根据自己发布scan名称进行修改 -->
    <node pkg="gmapping" type="slam_gmapping" name="slam_gmapping" output="screen" clear_params="true">
        <param name="base_frame" value="base_footprint"/>     <!-- 根据自己的基座标系名称进行修改 -->
        <param name="odom_frame" value="odom"/>               <!-- 根据自己的里程计坐标系名称进行修改 -->
        <param name="map_update_interval" value="4.0"/>
        <!-- Set maxUrange < actual maximum range of the Laser -->
        <param name="maxRange" value="5.0"/>
        <param name="maxUrange" value="4.5"/>
        <param name="sigma" value="0.05"/>
        <param name="kernelSize" value="1"/>
        <param name="lstep" value="0.05"/>
        <param name="astep" value="0.05"/>
        <param name="iterations" value="5"/>
        <param name="lsigma" value="0.075"/>
        <param name="ogain" value="3.0"/>
        <param name="lskip" value="0"/>
        <param name="srr" value="0.01"/>
        <param name="srt" value="0.02"/>
        <param name="str" value="0.01"/>
        <param name="stt" value="0.02"/>
        <param name="linearUpdate" value="0.5"/>
        <param name="angularUpdate" value="0.436"/>
        <param name="temporalUpdate" value="-1.0"/>
        <param name="resampleThreshold" value="0.5"/>
        <param name="particles" value="80"/>
        <param name="xmin" value="-1.0"/>
        <param name="ymin" value="-1.0"/>
        <param name="xmax" value="1.0"/>
        <param name="ymax" value="1.0"/>
        <param name="delta" value="0.05"/>
        <param name="llsamplerange" value="0.01"/>
        <param name="llsamplestep" value="0.01"/>
        <param name="lasamplerange" value="0.005"/>
        <param name="lasamplestep" value="0.005"/>
        <remap from="scan" to="$(arg scan_topic)"/>
    </node>
</launch>
```

3.启动gmapping

```
roslaunch <你的包> gmapping.launch
```

4.如果要用手柄控制

```
sudo apt-get install ros-<distro>-joy ros-noetic-teleop-twist-joy
```

新建launch文件 gmapping_joy.launch

```
<launch>
    <arg name="scan_topic" default="scan" />
    
    <!-- 手柄控制节点 -->
    <node pkg="joy" type="joy_node" name="joy_node" output="screen">
        <param name="dev" value="/dev/input/js0" />
        <param name="deadzone" value="0.3" />
        <param name="autorepeat_rate" value="20" />
    </node>

    <node pkg="teleop_twist_joy" type="teleop_node" name="teleop_twist_joy" output="screen">
        <param name="enable_button" value="0" />  <!-- 通常为A按钮 -->
        <param name="enable_turbo_button" value="-1" />  <!-- 禁用turbo模式 -->
        <param name="axis_linear" value="1" />    <!-- 通常为左摇杆上下 -->
        <param name="axis_angular" value="0" />   <!-- 通常为左摇杆左右 -->
        <param name="scale_linear" value="1.0" />  <!-- 线速度限制为1 嫌慢的可以提升到4--> 
        <param name="scale_angular" value="1.0" /> <!-- 角速度限制为1 嫌慢的可以提升到4-->
    </node>

    <!-- GMAPPING SLAM节点 -->
    <node pkg="gmapping" type="slam_gmapping" name="slam_gmapping" output="screen" clear_params="true">
        <param name="base_frame" value="base_footprint"/>
        <param name="odom_frame" value="odom"/>
        <param name="map_update_interval" value="4.0"/>
        <param name="maxRange" value="5.0"/>
        <param name="maxUrange" value="4.5"/>
        <param name="sigma" value="0.05"/>
        <param name="kernelSize" value="1"/>
        <param name="lstep" value="0.05"/>
        <param name="astep" value="0.05"/>
        <param name="iterations" value="5"/>
        <param name="lsigma" value="0.075"/>
        <param name="ogain" value="3.0"/>
        <param name="lskip" value="0"/>
        <param name="srr" value="0.01"/>
        <param name="srt" value="0.02"/>
        <param name="str" value="0.01"/>
        <param name="stt" value="0.02"/>
        <param name="linearUpdate" value="0.5"/>
        <param name="angularUpdate" value="0.436"/>
        <param name="temporalUpdate" value="-1.0"/>
        <param name="resampleThreshold" value="0.5"/>
        <param name="particles" value="80"/>
        <param name="xmin" value="-1.0"/>
        <param name="ymin" value="-1.0"/>
        <param name="xmax" value="1.0"/>
        <param name="ymax" value="1.0"/>
        <param name="delta" value="0.05"/>
        <param name="llsamplerange" value="0.01"/>
        <param name="llsamplestep" value="0.01"/>
        <param name="lasamplerange" value="0.005"/>
        <param name="lasamplestep" value="0.005"/>
        <remap from="scan" to="$(arg scan_topic)"/>
    </node>
</launch>
```

启动

```
roslaunch <你的包> gmapping_joy.launch
```


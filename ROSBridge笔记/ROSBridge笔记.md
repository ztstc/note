# 基于Vue和ROSBridge开发的机器人控制面板

## 启动

ROSBridge

```
roslaunch rosbridge_server rosbridge_websocket.launch
```

底盘节点（数据源）

```
cd ~/nailong_ws
source devel/setup.bash
roslaunch ros_uart_protocol launch_node.launch
```

网页

```
cd ~/robot-dashboard
npm run dev
```


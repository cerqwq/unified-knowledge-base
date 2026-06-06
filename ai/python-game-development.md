# Python 游戏开发知识库

## Web 游戏架构

### Flask + WebSocket 实时游戏
```python
from flask import Flask
from flask_socketio import SocketIO

app = Flask(__name__)
socketio = SocketIO(app)

@socketio.on('player_action')
def handle_action(data):
    # 处理玩家动作
    result = game.process(data)
    # 广播给所有玩家
    socketio.emit('game_update', result)
```

### 前后端通信协议
```javascript
// 客户端
socket.emit('action', {type: 'move', direction: 'up'});

socket.on('update', (state) => {
    renderGame(state);
});
```

## 游戏引擎选择

### Python 游戏引擎
| 引擎 | 特点 | 适用 |
|------|------|------|
| Pygame | 2D、简单 | 学习、原型 |
| Pyglet | OpenGL、多媒体 | 2D/3D |
| Arcade | 现代Pygame | 2D游戏 |
| Panda3D | 3D引擎 | 3D游戏 |
| Godot (Python) | 完整引擎 | 商业游戏 |

### Web 游戏引擎
| 引擎 | 特点 | 适用 |
|------|------|------|
| Phaser | 2D、成熟 | 2D网页游戏 |
| Three.js | 3D、WebGL | 3D网页游戏 |
| PixiJS | 2D、快速 | 渲染密集 |
| Babylon.js | 3D、完整 | 3D网页游戏 |

## 游戏设计模式

### ECS（实体-组件-系统）
```python
class Entity:
    def __init__(self):
        self.components = {}

class Component:
    pass

class System:
    def update(self, entities):
        for entity in entities:
            if self.required_components.issubset(entity.components):
                self.process(entity)
```

### 状态机
```python
class StateMachine:
    def __init__(self):
        self.current_state = None
        self.states = {}
    
    def update(self):
        if self.current_state:
            self.current_state.update()
```

### 游戏循环
```python
def game_loop():
    while running:
        handle_input()
        update_game_state()
        render()
        clock.tick(60)  # 60 FPS
```

## 性能优化

### 渲染优化
- 脏矩形渲染：只重绘变化区域
- 精灵批处理：合并绘制调用
- 纹理图集：减少纹理切换

### 逻辑优化
- 空间分区：四叉树、网格
- 对象池：重用游戏对象
- 脚本优化：热点代码用C扩展

### 网络优化
- 状态压缩：减少传输数据
- 预测回滚：减少延迟感
- 增量更新：只传变化部分

## 数据存储

### SQLite 游戏存档
```python
import sqlite3

def save_game(player_id, game_state):
    conn = sqlite3.connect('game.db')
    conn.execute(
        'INSERT INTO saves (player_id, state, timestamp) VALUES (?, ?, ?)',
        (player_id, json.dumps(game_state), datetime.now())
    )
    conn.commit()
```

### JSON 配置文件
```python
# 游戏配置
{
    "player": {
        "speed": 5,
        "health": 100,
        "inventory_size": 20
    },
    "world": {
        "width": 1000,
        "height": 1000,
        "tile_size": 32
    }
}
```

## 测试策略

### 单元测试
- 游戏逻辑函数
- 碰撞检测
- 状态转换

### 集成测试
- 完整游戏流程
- 网络通信
- 存档/读档

### 性能测试
- FPS 监控
- 内存占用
- 网络延迟

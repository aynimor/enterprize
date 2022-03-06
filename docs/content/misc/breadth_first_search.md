# 广度优先搜索

### 1、简介

​		**广度优先搜索** 或称 宽度优先搜索，英文全称为 **Breadth-First Search**，简称 `BFS`, 是一个针对图和树的盲目搜寻法，目的是系统地展开并检查图中的所有节点，以寻找结果。主要用于解决**迷宫最短路径和网络路由等问题**。

​		广度优先搜索算法主要使用队列的 **“先进先出”** 特性来实施算法过程：从初始结点结点开始，应用产生式规则和控制策略生成第一层结点，同时检查目标结点是否在这些生成的结点中。如果没有，继续使用产生式高泽将第一层的所有结点逐一拓展出第二层结点，并逐一检查第二层中的所有结点是否包含目标结点。如果没有在以此类推拓展所有结点并检查所有拓展出来的新结点，只到发现目标结点位置。如果拓展玩所有结点都没有发现目标结点，则说明该问题无解。

​	使用 **广度优先搜索** 来解决最短路径问题，一般需要两个步骤：

1. 使用图来建立问题模型；
2. 使用广度优先搜索解决问题。

> 图：Graph
>
> 定义：在计算机科学中，一个图就是一些顶点的集合，这些顶点通过一系列边结对（连接）。顶点用圆圈表示，边就是这些圆圈之间的连线。顶点之间通过边连接。
>
> 一个图可以表示一个社交网络，每一个人就是一个顶点，互相认识的人之间通过边联系。
>
> 图有各种形状和大小。边可以有权重（weight），即每一条边被分配一个整数或负数值。
>
> 考虑一个代表航线的图。各个城市就是顶点，航线就是边。那么边的权重可以是飞行时间，或者机票价格。
>
> 有了这样一张假设的航线图。便可以通过权重和路劲求出最便宜或是最快的路径。
>
> 来源：[数据结构：图（Graph）](https://www.jianshu.com/p/bce71b2bdbc8)
>

### 2、广度优先搜索算法

#### 2.1 广度优先搜索的工作过程和原理

有以下图结构

![image-20211226120157327](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261201377.png)

和一个队列数据结构

![image-20211226102731038](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261027063.png)

首先将结点 `1` 入队，并标记（标记的作用是做重复性检查，以防止遍历到重复的结点）。

![image-20211226103146168](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261031235.png)

然后访问队列的队头，并找出与队头一连接的同时没有被标记的结点，将其入队。然后将队头移动到 `2`，同时移动队尾

![image-20211226103451195](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261034263.png)

继续访问队头结点 `2` ，发现结点 `4`, `5` 未被标记，所以将结点 `4`, `5` 入队，并标记为已遍历，并将队头移动到 `3，同时移动队尾`

![image-20211226103749274](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261037336.png)

继续访问队头结点 `3`，发现结点 `6`, `7` 未被标记，所以将结点 `6`, `7` 入队，并标记为已遍历，并将队头移动到 `4`，同时移动队尾

![image-20211226103931069](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261039132.png)

继续访问队头结点 `4`，发现结点 `8`, 未被标记，所以将结点 `8` 入队，并标记为已遍历，并将队头移动到 `5`，同时移动队尾

![image-20211226104056763](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261040828.png)

继续访问结点 `5`，未发现未标记的结点，直接移动队头到 `6`，同时移动队尾

![image-20211226104152825](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261041896.png)

继续访问队头结点 `6`，发现结点 `9`, 未被标记，所以将结点 `9` 入队，并标记为已遍历，并将队头移动到 `7`，同时移动队尾

![image-20211226104318623](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261043692.png)

继续访问结点 `7`，未发现未标记的结点，直接移动队头到 `8`

![image-20211226104542457](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261045525.png)

继续访问结点 `8`，未发现未标记的结点，直接移动队头到 `9`

![image-20211226104630195](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261046261.png)

继续访问结点 `9`，未发现未标记的结点，直接移动队头到 `末尾`

![image-20211226104816963](https://raw.githubusercontent.com/aynimor/PicGo/main/img/202112261048027.png)

此时发现队头与队尾结点重合，即表示算法执行完毕，如果此时还没有找到目标结点，则说明该图无解。如果确保能够找到结点，则在某一步将结点入队时通过对新入队的结点目标结果相比对，如果符合结果则说明找到数据。此时即可结束算法。

#### 2.2 总结

广度优先搜索算法在整个搜索过程中，对于结点是沿着深度的断层拓展的。如果要拓展第 n+1 层结点，必须先全部拓展完第 n 层结点。对于同一层结点来说，他们对于问题的解的价值是相同的。因此，如果广度优先搜索算法有解，那第一个找到的解一定是最短路径的解序列。也就是说，第一个找到的目标结点，一定是使用产生式规则最少的解。

由此可以看出广度优先搜索算法适合求解**最少步骤**或者**最短解序列**这样的最优解问题

### 3. 算法代码

一个对于迷宫最短路径的算法的 Python 实现

定义一个迷宫的地图文件 `maze.in`

第一行为迷宫的行数和列数

0 代表可通行， 1 代表障碍物，不可通行

```
6 6
0 1 0 0 0 0
0 0 0 1 0 1
0 1 1 1 0 1
0 0 0 1 1 0
1 1 0 1 0 0
0 1 0 0 0 0
```

定义一个路径方向类，存储所有可能的方向

```Python
class Direction:
    UP    = (-1, 0)
    RIGHT = (0, 1)
    DOWN  = (1, 0)
    LEFT  = (0, -1)

    @classmethod
    def dirs(cls):
        return [cls.UP, cls.RIGHT, cls.DOWN, cls.LEFT]
```

定义一个路径点类，方便对迷宫坐标的比对

```Python
class Point(object):
    def __init__(self, x: int, y: int) -> None:
        self.x = x
        self.y = y

    def __eq__(self, __o: object) -> bool:
        return self.x == __o.x and self.y == __o.y
    
    def __add__(self, __o: List[int]):
        return Point(self.x + __o[0], self.y + __o[1])
    
    def __sub__(self, __o: List[int]):
        return Point(self.x - __o[0], self.y - __o[1])
    
    def at(self, grid: List[List[int]]) -> int:
        """判断当前坐标点的情况"""
        # 越界判断
        if self.x < 0 \
            or self.x >= len(grid):
            return 0, False
        if self.y < 0 \
            or self.y >= len(grid[self.x]):
            return 0, False
        return grid[self.x][self.y], True
    
    def __str__(self) -> str:
        return f"{self.x, self.y}"
```



定义一个迷宫类，用于初始化迷宫和路径寻找

```python
class Maze(object):
    def __init__(self, data: List[List[int]], row: int, col: int) -> None:
        self.data = data
        self.row = row
        self.col = col

    @classmethod
    def new(cls, path: str):
        """
        @param: path: 迷宫地图文件路径
        """
        if not os.path.exists(path):
            return None
        with open(path, "r", encoding="utf-8") as f:
            # 读取第一行 行数 和 列数
            line = f.readline().split()
            row, col = int(line[0]), int(line[0])
            # 填充迷宫
            data = []
            for _ in range(int(row)):
                column_data = []
                line = f.readline().split()
                for j in range(int(col)):
                    column_data.append(int(line[j]))
                data.append(column_data)
            return cls(data, row, col)

    @classmethod
    def blank_maze(cls, row: int, col: int):
        """空白地图， 用于对最短路径的存储"""
        return cls([[0 for _ in range(col)] for _ in range(row)], row, col)
    
    def print(self):
        """地图情况输出"""
        print("*" * 20)
        for row in self.data:
            print(row)
        print("*" * 20)
    
    def shortest_path(self, start: Point, end: Point):
        """通过最短路径图，得出最短路径的行进方式"""
        total_step, ok = end.at(self.data)
        queue = [end]
        current = queue[0]
        while len(queue) != total_step:
            current_step  = current.at(self.data)[0]
            start_queue_len = len(queue)
            for dir in Direction.dirs():
                next = current - dir
                step, _ = next.at(self.data)
                if step + 1 == current_step:
                    queue.append(next)
                    current = next
                    break
            # 如果在当前点旁没有找到其他的点，则说明没有最短路径
            end_queue_len = len(queue)
            if start_queue_len == end_queue_len:
                break
        queue.append(start)
        queue.reverse()
        return queue

    def walk(self, start: Point, end: Point):
        """最短路径寻找"""
        # 用于记录和标记路径点
        steps = self.blank_maze(self.row, self.col)
        # 可搜寻队列
        queue = [start]
        while len(queue) > 0:
            # 当前路径点
            current = queue[0]
            # 出队
            queue = queue[1:]

            # 到达终点，返回路径图
            if current == end:
                return steps, True
            for dir in Direction.dirs():
                
                next = current + dir
                value, ok = next.at(self.data)
                # 当前坐标越界，或当前坐标是障碍物
                if not ok or value == 1:
                    continue

                value, ok = next.at(steps.data)
                # 当前坐标已被标记
                if not ok or value != 0:
                    continue
                
                # 回到起点
                if next == start:
                    continue
                
                # 标记当前坐标，并记录行动步数
                steps.data[next.x][next.y] = current.at(steps.data)[0] + 1
                # 将当前坐标加入可可搜寻队列
                queue.append(next)
        # 未找到最短路径
        return steps, False

```

使用

```Python
if __name__ == "__main__":
    maze = Maze.new("./maze.in")
    start = Point(0, 0)
    end = Point(4, 5)
    steps, ok = maze.walk(start, end)
    steps.print()
    if ok:
        shortest_path = steps.shortest_path(start, end)
        for p in shortest_path:
            print(p)
```


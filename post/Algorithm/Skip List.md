# 简介

跳表（skip list）是一种有序数据结构，针对一个全序集的 key 进行排序。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/86/Skip_list.svg/800px-Skip_list.svg.png)

# 核心思想

对于一个有序的**单**链表（长度 $n$），如果直接搜索某个数字 $v$，需要扫描整个链表。一个想法是，==如果能够一次跳跃多个节点，那么搜索效率更高==。

如果根据特定的算法建立层级的链表，那么查询效率就会很高，比如根据二进制，但是这样建立的跳表不够灵活，无法灵活的插入和删除。

跳表的另一个核心是**随机化**：一个元素是否出现在下一层，随机判断。可以预设 $p$，那么一个元素的层高是：

| 最高层 | 概率                               |
| --- | -------------------------------- |
| 0   | $1-p$                            |
| 1   | $p(1-p)$                         |
| 2   | $p^2 (1-p)$                      |
| ... | ...                              |
| k   | $p^k (1-p)$                      |
| L   | $1 - \sum_{k=0}^{L-1} p^k (1-p)$ |

也就是说，我们随机确定一个元素的层高，然后插入。

# 操作

无论是插入、查询 还是 删除，一个核心操作是 找到大于等于 $v$ 的第一个节点 $p$，并且找到在每一层 $l$ 中 $p$ 的上一个节点 $pre^l$ 。

# 代码

```python
from dataclasses import dataclass
import random
from typing import Optional

MAX_LEVEL = 20
PROBABILITY = 0.5


def random_level() -> int:
    """Generate a random level for a new node"""
    level = 1
    while random.random() < PROBABILITY and level < MAX_LEVEL:
        level += 1
    return level


@dataclass(slots=True)
class Node:
    """Node of the skiplist."""

    key: int
    """The value of the node"""

    next_nodes: list[Optional["Node"]]
    """The next nodes of the node"""

    @property
    def level(self) -> int:
        """The level of the node"""
        return len(self.next_nodes)

    @staticmethod
    def new_node(val: int, level: int) -> "Node":
        """Create a new node with the given value and level"""
        return Node(val, [None] * level)


class Skiplist:
    """Skiplist implementation."""

    def __init__(self):
        self.head: Node = Node(0, [None] * MAX_LEVEL)
        """Dummy node, head of the skiplist"""

    def find_geq_node(self, val: int, pre: list[Node | None] | None) -> Node | None:
        """Find the node with the smallest key that is greater or equal to the given value.

        Args:
            val: The value to search for.
            pre: The list to store the previous nodes of the found node. If None, don't store them.
        """
        x = self.head
        level = MAX_LEVEL - 1
        while True:
            y = x.next_nodes[level]  # x -> y
            if y is None or y.key >= val:
                # x -> val -> y
                # store the previous node
                if pre is not None:
                    pre[level] = x

                # go down a level
                if level == 0:
                    return y
                else:
                    level -= 1
            else:
                x = y

    def search(self, target: int) -> bool:
        """Search for a value in the skiplist."""
        x = self.find_geq_node(target, None)
        return x is not None and x.key == target

    def add(self, num: int) -> None:
        """Add a value to the skiplist, allowing duplicates."""
        pre: list[Node | None] = [None] * MAX_LEVEL
        _ = self.find_geq_node(num, pre)
        new_node = Node.new_node(num, random_level())

        # insert the new node
        for level in range(new_node.level):
            last_node = pre[level]
            assert last_node is not None
            new_node.next_nodes[level] = last_node.next_nodes[level]
            last_node.next_nodes[level] = new_node

    def erase(self, num: int) -> bool:
        """Erase a value from the skiplist, return True if the value was found and removed."""
        pre: list[Node | None] = [None] * MAX_LEVEL
        x = self.find_geq_node(num, pre)

        # If not find
        if x is None or x.key != num:
            return False

        # remove `x`
        for level in range(x.level):
            last_node = pre[level]
            assert last_node is not None
            last_node.next_nodes[level] = x.next_nodes[level]

        del x
        return True

    def show(self):
        def show_level(level: int):
            x: Node | None = self.head
            while x is not None:
                print(x.key, end=" -> ")
                x = x.next_nodes[level]
            print()

        for level in range(MAX_LEVEL):
            print(f"Level {level}: ", end="")
            show_level(level)


```

# 复杂度

时间复杂度

| 操作            |         平均复杂度         | 最差复杂度  |
| :------------ | :-------------------: | :----: |
| 插入            |      $O(\log n)$      | $O(n)$ |
| 查询            |      $O(\log n)$      | $O(n)$ |
| 删除            |      $O(\log n)$      | $O(n)$ |
| 随机访问第 $k$ 个元素 | $O(n)$ or $O(\log n)$ | $O(n)$ |
> 如果多维护 **前向指针** 以及 **前向指针的长度** （和上一个元素的位置差），可以在 $O(\log n)$ 的复杂度完成随机访问。

空间复杂度：平均 $O(n)$ ，最差 $O(n \log n)$ 。

# 参考资料

- [LevelDB](https://selfboot.cn/2024/09/09/leveldb_source_skiplist/)
- [OI wiki](https://oi-wiki.org/ds/skiplist/)
- [Wikipedia](https://en.wikipedia.org/wiki/Skip_list)
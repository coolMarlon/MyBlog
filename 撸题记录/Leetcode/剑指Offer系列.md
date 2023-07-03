# 剑指 Offer 

## 剑指 Offer 09. 用两个栈实现队列

```python3
class CQueue:

    def __init__(self):
        self.Sa = []
        self.Sb = []


    def appendTail(self, value: int) -> None:
        self.Sa.append(value)
        return None


    def deleteHead(self) -> int:
        if len(self.Sb) != 0:
            result = self.Sb[-1]
            del self.Sb[-1]
            return result

        else:
            if len(self.Sa) != 0:
                for i in reversed(self.Sa):
                    self.Sb.append(i)
                self.Sa.clear()
                result = self.Sb[-1]
                del self.Sb[-1]
                return result
            else:
                return -1
            



# Your CQueue object will be instantiated and called as such:
# obj = CQueue()
# obj.appendTail(value)
# param_2 = obj.deleteHead()
```

## [剑指 Offer 30. 包含min函数的栈](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)

```python3
class MinStack:

    def __init__(self):
        """
        initialize your data structure here.
        """
        self.baseStack = []
        self.helpStackA = []
        self.helpStackB = []


    def push(self, x: int) -> None:
        self.baseStack.append(x)
        if len(self.helpStackA) == 0:
            self.helpStackA.append(x)
        else:
            if self.helpStackA[-1] >= x:
                self.helpStackA.append(x)
            else:
                self.helpStackB.append(x)

    def pop(self) -> None:
        if len(self.baseStack) == 0:
            return None
        top_value = self.baseStack[-1]
        del self.baseStack[-1]
        if top_value == self.helpStackA[-1]:
            del self.helpStackA[-1]
        else:
            del self.helpStackB[-1]

    def top(self) -> int:
        return self.baseStack[-1]


    def min(self) -> int:
        return self.helpStackA[-1]



# Your MinStack object will be instantiated and called as such:
# obj = MinStack()
# obj.push(x)
# obj.pop()
# param_3 = obj.top()
# param_4 = obj.min()
```


转载：https://blog.csdn.net/haha456487/article/details/106316116/
两种方式：

1. 第一种：

```py
import numpy as np
a = np.array([[1, 2], [3, 4], [9, 8]])

##使用库函数
from itertools import chain
a_a = list(chain.from_iterable(a))
print(a_a)
```

输出结果为：[1, 2, 3, 4, 9, 8]

2. 第二种：

```py
import numpy as np
a = np.array([[1, 2], [3, 4], [9, 8]])
b = a.flatten()
print(b)
```
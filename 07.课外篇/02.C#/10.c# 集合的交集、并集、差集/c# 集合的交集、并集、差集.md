转载：https://www.cnblogs.com/flywing/p/5912242.html

```c#
// Intersect 交集，Except 差集，Union 并集
int[] oldArray = { 1, 2, 3, 4, 5 };
int[] newArray = { 2, 4, 5, 7, 8, 9 };

var jiaoJi = oldArray.Intersect(newArray).ToList();//2,4,5
var oldChaJi = oldArray.Except(newArray).ToList();//1,3
var newChaJi = newArray.Except(oldArray).ToList();//7,8,9
var bingJi = oldArray.Union(newArray).ToList();//1,2,3,4,5,7,8,9
```
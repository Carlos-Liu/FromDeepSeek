`yield return` 是 C# 中用于**迭代器**的关键字，它让你可以逐个返回集合中的元素，而不需要一次性创建整个集合。

### 基本用法
- 在返回类型为 `IEnumerable<T>`、`IEnumerable`、`IEnumerator<T>` 或 `IEnumerator` 的方法中使用。
- 遇到 `yield return` 时，会返回当前值，并**记住当前位置**，下次迭代时从下一句继续执行。

### 示例
```csharp
public IEnumerable<int> GetNumbers()
{
    for (int i = 0; i < 5; i++)
    {
        yield return i; // 每次返回一个数字
    }
}

// 调用
foreach (int num in GetNumbers())
{
    Console.WriteLine(num); // 输出 0,1,2,3,4
}
```

### 与普通 `return` 的区别
- **普通 `return`**：直接返回整个集合（如 `List<int>`），一次性加载所有数据。
- **`yield return`**：按需生成数据，每次只返回一个元素，状态被保存。

### 优点
1. **延迟执行**：只有需要时才生成下一个元素，适合处理大数据或无限序列。
2. **节省内存**：不用预先存储所有数据，比如读取大文件时逐行返回。

### 注意事项
- 使用 `yield break` 可以提前终止迭代。
- 方法中不能同时包含普通 `return` 和 `yield return`。

简单说，`yield return` 让创建迭代器变得简单高效，特别适合需要按需生成序列的场景。

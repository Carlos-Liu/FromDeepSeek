# 问题：
a cocktail can include reagents and sub-cocktail (also a Cocktail entity), then sub-cocktail can include reagents and cocktail too, and so on.
I want to query all the sub-components (reagents and sub-cocktails, reagents of sub-cocktails and so on) of a root cocktail, revise the query.

`AsNoTracking()` 会禁用 EF Core 的 **identity map（身份映射）**，而正是这个机制负责在同一次查询结果中自动关联导航属性。

## 具体影响

|  | 有 `AsNoTracking()` | 无 `AsNoTracking()` |
| --- | --- | --- |
| EF Core identity map | ❌ 禁用 | ✅ 启用 |
| SubCocktail 的 CocktailItems 自动关联 | ❌ 不会关联（null） | ✅ 自动递归关联 |
| 查询到的子层级数据 | 只有第一层 | 所有层级 |

## 举例说明

假设结构为 `A → B → C`（A 包含子鸡尾酒 B，B 包含子鸡尾酒 C）：

- 加了 `AsNoTracking()`：`B.CocktailItems` 会是空的，**C 拿不到**
- 不加 `AsNoTracking()`：EF Core 从结果集中识别出 B 也是一个已加载的 `Cocktail`，
  自动把 B 的 `CocktailItems` 填充，**C 可以正常访问**

## 结论

对于这种递归结构，`AsNoTracking()` 会导致查询结果**不完整**，必须去掉。

#何时使用及有点

## 什么时候使用 `AsNoTracking()`

### 适用场景

**只读查询** —— 查询结果仅用于读取/展示，不需要修改后保存回数据库。

典型例子：

- 列表页、详情页的数据展示
- 报表、统计数据查询
- API 的 GET 接口返回 DTO

### 带来的好处

| 好处 | 说明 |
| --- | --- |
| **性能提升** | EF Core 不需要将实体注册到 `ChangeTracker`，减少内存分配和 CPU 开销 |
| **内存占用更低** | 不维护实体快照（snapshot），大量数据查询时效果明显 |
| **更快的查询速度** | 跳过变更跟踪的初始化逻辑，整体响应更快 |

### 不适合使用的场景

- 查询后需要 **修改实体并调用 `SaveChanges()`**
- 存在**递归/自引用导航属性**需要自动 fixup（如本项目的 `SubCocktail`）
- 需要 EF Core **延迟加载（lazy loading）** 的场景

### 总结建议
只读展示  →  用 AsNoTracking()       （性能优先）

需要修改  →  不用 AsNoTracking()     （正确性优先）

递归结构  →  不用 AsNoTracking()     （结果完整性优先）

在本项目 GetCocktailsAsync 的场景中，虽然是只读查询，但由于存在递归的 SubCocktail 结构，结果正确性优先于性能，所以去掉 AsNoTracking() 是正确的选择。

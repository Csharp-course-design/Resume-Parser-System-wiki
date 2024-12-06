> ### 即 数据访问层
> 用以提供与数据库交互的各种功能

包括以下功能

- 对`简历文件表`的 增加、查询 操作
- 对`简历信息表`的 增加、查询 操作
- 对 `关键字表`的 增加、查询 操作
- 建立数据库实体与数据库实体之间的关系
- 数据库层面的搜索引擎服务

## 类库编写大纲

在`DAL.DataControl`命名空间下，提供对所有表的 增加、查询 操作。

### 命名规范

- 数据库查询类：`<表名>Control`
- 实现的方法，参阅 **数据库操作接口说明**





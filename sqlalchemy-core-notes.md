# SQLAlchemy Core 使用笔记

## 创建 Engine

```python
from sqlalchemy import create_engine

# SQLite 示例
engine = create_engine('sqlite:///example.db')

# PostgreSQL 示例
# engine = create_engine('postgresql://user:password@localhost/dbname')

# MySQL 示例
# engine = create_engine('mysql+pymysql://user:password@localhost/dbname')

# 连接参数设置
engine = create_engine('sqlite:///example.db', echo=True, future=True)
```

参数说明：
- `echo`: 当设置为True时，引擎会将所有SQL语句和参数记录到标准输出，这对于调试非常有用
- `future`: 当设置为True时，使用2.0版本的语义，这将启用SQLAlchemy 2.0的新特性和行为

## 定义表结构 (Table)

```python
from sqlalchemy import MetaData, Table, Column, Integer, String, ForeignKey, DateTime

# 创建元数据对象
metadata = MetaData()

# 定义用户表
users = Table(
    'users',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50)),
    Column('email', String(100)),
    Column('created_at', DateTime)
)

# 定义文章表（与用户表关联）
posts = Table(
    'posts',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('title', String(100)),
    Column('content', String(500)),
    Column('user_id', Integer, ForeignKey('users.id')),
    Column('created_at', DateTime)
)

# 创建所有表
metadata.create_all(engine)
```

## 插入数据 (Create)

```python
from sqlalchemy import insert
from datetime import datetime

# 插入单条记录到 users 表
stmt = insert(users).values(
    name='Alice',
    email='alice@example.com',
    created_at=datetime.now()
)
result = engine.execute(stmt)

# 插入多条记录
stmt = insert(users).values([
    {'name': 'Bob', 'email': 'bob@example.com', 'created_at': datetime.now()},
    {'name': 'Charlie', 'email': 'charlie@example.com', 'created_at': datetime.now()}
])
result = engine.execute(stmt)

# 获取插入的主键值
print(result.inserted_primary_key)
```

参数说明：
- `values()`: 用于指定要插入的数据，可以是单个字典或字典列表
- `inserted_primary_key`: 返回插入行的主键值

## 查询数据 (Read)

```python
from sqlalchemy import select, and_, or_, not_, asc, desc

# 查询所有用户
stmt = select(users)
result = engine.execute(stmt)
for row in result:
    print(row)

# 条件查询
stmt = select(users).where(users.c.name == 'Alice')
result = engine.execute(stmt)
row = result.fetchone()

参数说明：
- `where()`: 指定查询条件，过滤结果集
- `fetchone()`: 获取结果集中的下一行，如果没有更多行则返回None

# 复杂条件查询
stmt = select(users).where(
    and_(
        users.c.name.like('%A%'),
        users.c.email.is_not(None)
    )
)
result = engine.execute(stmt)

参数说明：
- `and_()`: 用于组合多个条件，所有条件都必须满足
- `like()`: SQL LIKE操作符，用于模式匹配
- `is_not()`: 检查列值是否不为NULL

# 排序和限制结果数量
stmt = select(users).order_by(desc(users.c.created_at)).limit(10)
result = engine.execute(stmt)

参数说明：
- `order_by()`: 指定结果排序方式
- `desc()`: 降序排列
- `asc()`: 升序排列（默认）
- `limit()`: 限制返回的结果数量

# 聚合查询
from sqlalchemy import func
stmt = select(func.count(users.c.id))
result = engine.execute(stmt)
count = result.scalar()
```

## 更新数据 (Update)

```python
from sqlalchemy import update

# 更新特定用户的邮箱
stmt = update(users).where(
    users.c.name == 'Alice'
).values(email='alice.new@example.com')
result = engine.execute(stmt)

参数说明：
- `where()`: 指定要更新的行的条件
- `values()`: 指定要更新的列和新值

# 更新多列
stmt = update(users).where(
    users.c.id == 1
).values(
    name='Alice Cooper',
    email='alice.cooper@example.com'
)
result = engine.execute(stmt)
```

## 删除数据 (Delete)

```python
from sqlalchemy import delete

# 删除特定用户
stmt = delete(users).where(users.c.name == 'Bob')
result = engine.execute(stmt)

参数说明：
- `where()`: 指定要删除的行的条件

# 清空表（谨慎使用）
stmt = delete(users)
result = engine.execute(stmt)
```

## 表关联查询 (Joins)

```
# 内连接查询：获取用户及其文章
stmt = select(users, posts).select_from(
    users.join(posts)
)
result = engine.execute(stmt)

参数说明：
- `join()`: 执行内连接，默认基于外键关系

# 左外连接查询：获取所有用户及他们的文章（包括没有文章的用户）
stmt = select(users, posts).select_from(
    users.outerjoin(posts)
)
result = engine.execute(stmt)

参数说明：
- `outerjoin()`: 执行左外连接

# 显式指定连接条件
stmt = select(users, posts).select_from(
    users.join(posts, users.c.id == posts.c.user_id)
)
result = engine.execute(stmt)

参数说明：
- `users.join(posts, users.c.id == posts.c.user_id)`: 显式指定连接条件

# 只选择需要的字段
stmt = select(
    users.c.name,
    posts.c.title,
    posts.c.content
).select_from(
    users.join(posts)
).where(
    users.c.name == 'Alice'
)
result = engine.execute(stmt)

参数说明：
- `select(column1, column2, ...)`: 指定要查询的具体列
- `where()`: 添加过滤条件

```

## 子查询示例

```python
# 使用子查询查找有文章发布的用户
subq = select(posts.c.user_id).distinct().subquery()
stmt = select(users).where(users.c.id.in_(subq.select()))

result = engine.execute(stmt)
for row in result:
    print(row)

参数说明：
- `distinct()`: 去除重复值
- `subquery()`: 将查询结果作为子查询
- `in_()`: SQL IN操作符，检查值是否在子查询结果中
```

## 常用函数

```python
from sqlalchemy import func, distinct

# 计数
stmt = select(func.count(users.c.id))

参数说明：
- `func.count()`: SQL COUNT聚合函数，用于计算行数

# 去重计数
stmt = select(func.count(distinct(users.c.name)))

参数说明：
- `distinct()`: SQL DISTINCT关键字，去除重复值

# 最大值、最小值、平均值
stmt = select(
    func.max(users.c.id),
    func.min(users.c.id),
    func.avg(users.c.id)
)

参数说明：
- `func.max()`: SQL MAX函数，返回最大值
- `func.min()`: SQL MIN函数，返回最小值
- `func.avg()`: SQL AVG函数，返回平均值

# 字符串函数
stmt = select(users).where(func.upper(users.c.name).like('%ALICE%'))

参数说明：
- `func.upper()`: SQL UPPER函数，将字符串转换为大写
```

## 事务处理

```python
# 手动事务控制
with engine.connect() as conn:
    trans = conn.begin()
    try:
        # 执行多个操作
        conn.execute(insert(users), {'name': 'David', 'email': 'david@example.com'})
        conn.execute(insert(posts), {'title': 'Hello', 'content': 'World', 'user_id': 1})
        trans.commit()
    except:
        trans.rollback()
        raise

参数说明：
- `conn.begin()`: 开始一个事务
- `trans.commit()`: 提交事务，永久保存更改
- `trans.rollback()`: 回滚事务，撤销未提交的更改
```

## 高频方法总结

| 操作 | 方法 |
|------|------|
| 创建引擎 | `create_engine()` |
| 定义表 | `Table()` |
| 创建表 | `metadata.create_all()` |
| 插入数据 | `insert()` |
| 查询数据 | `select()` |
| 更新数据 | `update()` |
| 删除数据 | `delete()` |
| 表连接 | `table.join()` |
| 条件过滤 | `where()` |
| 排序 | `order_by()` |
| 限制结果 | `limit()` |
| 聚合函数 | `func.count()` 等 |

## 常用条件表达式

```python
# 等于
users.c.name == 'Alice'

# 不等于
users.c.name != 'Alice'

# 大于/小于
users.c.id > 5

# 包含
users.c.name.like('%Ali%')

# 在列表中
users.c.id.in_([1, 2, 3])

# 为空/不为空
users.c.email.is_(None)
users.c.email.is_not(None)

# 组合条件
and_(condition1, condition2)
or_(condition1, condition2)
not_(condition)
```

参数说明：
- `==`: 等于操作符
- `!=`: 不等于操作符
- `>`: 大于操作符
- `like()`: SQL LIKE操作符，用于模式匹配
- `in_()`: SQL IN操作符，检查值是否在列表中
- `is_(None)`: SQL IS NULL操作符，检查值是否为NULL
- `is_not(None)`: SQL IS NOT NULL操作符，检查值是否不为NULL
- `and_()`: SQL AND操作符，组合多个条件
- `or_()`: SQL OR操作符，满足任一条件
- `not_()`: SQL NOT操作符，否定条件

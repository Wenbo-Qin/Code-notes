# FastAPI 速查笔记

## 基础设置和运行

```python
from fastapi import FastAPI

# 创建应用实例
app = FastAPI()

# 简单路径操作
@app.get("/")
def read_root():
    return {"Hello": "World"}

# 运行应用
# uvicorn main:app --reload
```

参数说明：
- `FastAPI()`: 创建 FastAPI 应用实例
- `@app.get("/")`: 装饰器，定义 GET 请求的路径操作，"/" 是 URL 路径
- `uvicorn main:app --reload`: 命令行运行应用，main 是文件名，app 是应用实例名，--reload 启用热重载

## 路径参数和查询参数

```python
from fastapi import FastAPI

app = FastAPI()

# 路径参数
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}

# 带查询参数的路径
@app.get("/items/")
def read_item(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}

# 混合路径参数和查询参数
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

参数说明：
- `item_id: int`: 路径参数，类型为 int
- `q: str = None`: 可选查询参数，类型为 str，默认值为 None
- `skip: int = 0`: 带默认值的查询参数

## 请求体和数据模型

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 定义数据模型
class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

# 使用请求体
@app.post("/items/")
def create_item(item: Item):
    return item

# 访问模型属性
@app.post("/items/")
def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

参数说明：
- `BaseModel`: Pydantic 提供的基类，用于定义数据模型
- `item: Item`: 请求体参数，类型为 Item 模型
- `item.dict()`: 将模型转换为字典

## 查询参数和字符串校验

```python
from fastapi import FastAPI, Query

app = FastAPI()

# 字符串校验
@app.get("/items/")
def read_items(q: str = Query(None, max_length=50, min_length=3)):
    return {"q": q}

# 正则表达式校验
@app.get("/items/")
def read_items(
    q: str = Query(None, min_length=3, max_length=50, regex="^fixedquery$")
):
    return {"q": q}

# 列表查询参数
@app.get("/items/")
def read_items(q: list = Query([])):
    return {"q": q}
```

参数说明：
- `Query()`: 用于声明查询参数的额外信息
- `max_length`: 最大长度限制
- `min_length`: 最小长度限制
- `regex`: 正则表达式匹配
- `Query([])`: 默认值为列表

## 路径参数和数值校验

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()

# 路径参数数值校验
@app.get("/items/{item_id}")
def read_items(
    item_id: int = Path(..., gt=0, le=1000),
    q: str = Query(None, alias="item-query")
):
    return {"item_id": item_id, "q": q}

# 数值校验参数
# gt: 大于 (greater than)
# ge: 大于等于 (greater than or equal)
# lt: 小于 (less than)
# le: 小于等于 (less than or equal)
```

参数说明：
- `Path(...)`: 用于声明路径参数的额外信息，... 表示必需参数
- `gt`: 大于 (greater than)
- `le`: 小于等于 (less than or equal)
- `alias`: 参数别名

## 请求体部分更新

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

# 使用 Patch 进行部分更新
@app.patch("/items/{item_id}")
def update_item(item_id: int, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

参数说明：
- `@app.patch`: PATCH 请求方法，用于部分更新资源
- `exclude_unset=True`: 排除未设置的值
- `copy(update=update_data)`: 复制模型并更新特定字段

## 响应模型

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: str
    full_name: Optional[str] = None

class UserOut(BaseModel):
    username: str
    email: str
    full_name: Optional[str] = None

# 定义响应模型
@app.post("/user/", response_model=UserOut)
def create_user(user: UserIn):
    return user
```

参数说明：
- `response_model`: 指定响应的数据模型，会过滤掉不在模型中的字段
- `UserIn`: 输入模型，包含密码等敏感信息
- `UserOut`: 输出模型，不包含敏感信息

## 状态码和响应

```python
from fastapi import FastAPI, status

app = FastAPI()

# 设置响应状态码
@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(name: str):
    return {"name": name}

# 常用状态码
# status.HTTP_200_OK: 200 成功
# status.HTTP_201_CREATED: 2101 创建成功
# status.HTTP_404_NOT_FOUND: 404 未找到
# status.HTTP_422_UNPROCESSABLE_ENTITY: 422 请求体格式错误
```

参数说明：
- `status_code`: 设置 HTTP 响应状态码
- `status.HTTP_201_CREATED`: 创建成功的标准状态码

## 异常处理

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items/{item_id}")
def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}

# 添加自定义头信息
@app.get("/items-header/{item_id}")
def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"}
        )
    return {"item": items[item_id]}
```

参数说明：
- `HTTPException`: FastAPI 提供的异常类，用于返回 HTTP 错误响应
- `status_code`: HTTP 状态码
- `detail`: 错误详情信息
- `headers`: 自定义响应头

## 依赖注入

```python
from fastapi import Depends, FastAPI

app = FastAPI()

# 简单依赖
def common_parameters(q: str = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return commons

# 类作为依赖
class CommonQueryParams:
    def __init__(self, q: str = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
def read_items(commons: CommonQueryParams = Depends()):
    return commons
```

参数说明：
- `Depends()`: 声明依赖项
- `common_parameters`: 依赖函数
- `CommonQueryParams`: 依赖类

## 中间件

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# 添加 CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 自定义中间件
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

参数说明：
- `add_middleware()`: 添加中间件
- `CORSMiddleware`: CORS 中间件，处理跨域请求
- `@app.middleware("http")`: 自定义 HTTP 中间件
- `call_next(request)`: 调用下一个中间件或路由处理函数

## 高频方法总结

| 功能 | 方法 |
|------|------|
| 创建应用 | `FastAPI()` |
| 路径操作装饰器 | `@app.get()` `@app.post()` `@app.put()` `@app.delete()` |
| 数据模型 | `BaseModel` |
| 路径参数 | `/{item_id}` |
| 查询参数 | `Query()` |
| 路径参数校验 | `Path()` |
| 响应模型 | `response_model` |
| 状态码 | `status_code` |
| 异常处理 | `HTTPException` |
| 依赖注入 | `Depends()` |
| 中间件 | `add_middleware()` |

## 常用装饰器和方法

```python
# HTTP 方法装饰器
@app.get("/")
@app.post("/")
@app.put("/")
@app.delete("/")
@app.patch("/")

# 参数类型
def func(item_id: int):              # 路径参数
def func(q: str = None):             # 查询参数
def func(item: Item):                # 请求体
def func(commons: dict = Depends()): # 依赖

# 响应和异常
status.HTTP_200_OK
status.HTTP_201_CREATED
status.HTTP_404_NOT_FOUND
HTTPException(status_code=404, detail="Item not found")
```
---
title: SQLAlchemy中的JSONB和ARRAY
toc: true
date: 2021-03-04 10:31:00
description: SQLAlchemy中如何定义PostgreSQL中的JSONB和ARRAY，以及如何操作
tags:
- Python
- ORM
- SQLAlchemy
---

## Model定义

```python
from sqlalchemy import Column, Integer, String, ARRAY
from sqlalchemy.dialects import postgresql
from .base import Base

class Model1(Base):
  __tablename__ = "model1"
  id = Column(Integer, primary_key=True, index=True)
  attr_data = Column(postgresql.JSONB)
  tags = Column(postgresql.ARRAY(String, dimensions=1))
  def __repr__(self):
    return f"name: {self.name}"
```

## 操作

### 增

```python
m2 = Model1(name="m2", attr_data={"key1": "value1", "key2": "value2"}, tags=["tag1", "tag2", "tag3"])
session.add(m2)
session.commit()
```

### 删

```python
row = session.query(Model1).filter(Model1.id == 1).one()
session.delete(row)
session.commit()
```

### 查

为了方便测试，我们再添加了4条数据。

```python
session.add_all([
  Model1(name="m3", attr_data={
    "key1": "value1",
    "key2": "value2"
  }, tags=["tag1", "tag2", "tag3"]),
  Model1(name="m4", attr_data={
    "key1": "value11",
    "key2": "value22"
  }, tags=["tag1", "tag3"]),
  Model1(name="m5", attr_data={
    "key1": "value111",
    "key2": "value222"
  }, tags=["tag1", "tag2"]),
  Model1(name="m6")
])
session.commit()
```

查询attr_data不为null的数据

```python
rows = session.query(Model1).filter(Model1.attr_data.isnot(None)).all()
```

查询attr_data的key1的值为value1的数据

```python
rows = session.query(Model1).filter(Model1.attr_data["key1"].astext == "value1").all()
```

查询tags包含tag2的数据

```python
rows = session.query(Model1).filter(Model1.tags.any("tag2")).all()
```

查询tags包含tag2和tag3的数据

```python
rows = session.query(Model1).filter(Model1.tags.contains(cast(["tag2", "tag3"], ARRAY(String)))).all()
```

### 改

更新id为2的数据的attr_data的key2的值为vvv

```python
from sqlalchemy import func
rows = session.query(Model1).filter(Model1.id == 2).update(
    {Model1.attr_data: func.jsonb_set(Model1.attr_data, "{key2}", '"vvv"')}, synchronize_session=False)
```

更新所有数据的attr_data中key2的值为原值+后缀`_suffix`，如果attr_data为null, 则增加key2的键

```python
rows = session.query(Model1)
for row in rows:
  d = {}
  if row.attr_data:
    d = dict(row.attr_data)
  d["key2"] = f'{d["key2"] if "key2" in d else ""}_suffix'
  row.attr_data = d
session.commit()
```

用裸SQL

```python
sql = """
  UPDATE model1
  SET attr_data = jsonb_set(
    COALESCE(attr_data, '{}'::jsonb), 
    '{key2}', 
    concat('"', COALESCE(attr_data->>'key2', ''), '_suffix', '"')::jsonb
  )
  """
session.execute(sql)
session.commit()
```

## Reference

- https://docs.sqlalchemy.org/en/13/dialects/postgresql.html


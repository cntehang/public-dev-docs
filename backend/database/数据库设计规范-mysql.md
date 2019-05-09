# 数据库设计规范-mysql

## 0. 原则

## 1. 命名

- 表名，字段名全部小写，单词间以下划线分隔，如: flight_order。
- 主键建议命名为id。
- 编写sql脚本时，关键字大写, 如:

```sql
SELECT id, pay_mode FROM flight_order
```

## 2. 主键选择

## 3. 数据列设计

## 4. 索引

## 5. 最佳实践

- 每个版本的SQL语句要保证能够重复执行。

# 前后端协作规约

## 1 日期传输规范

对于系统中的两类日期时间，分别规定如下：

- 系统自己生成的时间：前后端交互过程中统一转为 [UTC 时间](https://zh.wikipedia.org/wiki/%E5%8D%8F%E8%B0%83%E4%B8%96%E7%95%8C%E6%97%B6)，格式上采用 [ISO 8601](https://zh.wikipedia.org/wiki/ISO_8601) 规定的 yyyy-MM-dd'T'HH:mm:ss.SSS'Z' 标准格式。
- 第三方传入的时间：前后端交互过程中不对日期格式做转换处理。
- 需要在接口文档中注明是标准格式的字符串还是其他格式，如果是其他格式，则注明具体格式（例如：yyyy-MM-dd)。

## 2 大数字传输规范

对于 Java 中的 long 型等大数字，当超过一定范围，JavaScript 无法精确展示，故后端给前端的 Long 型数字时需转换为 String 字符串返回。

## 3 列表传输规范

对于列表形式的数据，如多个 ID，前后端应以列表/数组传输，不要使用逗号分隔的字符串。

## 4 姓名字段规范

为了准确表达客户姓名，涉及到客户姓名展示的地方（不含历史快照部分），后端接口都会返回四个字段：

- 中文姓 surnameCn, 名givenNameCn
- 英文姓 surnameEn, 名givenNameEn

下述规则供前端参考：

- 在未做多语言国际化的情况下，前端展示姓名时需要根据如下原则：
  - 若有中文姓名则显示 surnameCn + givenNameCn，否则展示 surnameEn + “/” + givenNameEn
  
后端使用姓名相关字段原则：

- 涉及到员工姓名搜索的地方，应和 customer 表的 name_search_helper 字段进行模糊匹配
- 涉及到需要获取员工姓名的场景，如通知发送称谓等，使用 Customer 的 buildName() 方法获取姓名字符串
- 判断 Customer 的姓名和某一字符串是否相同，需要调用 Customer 的 sameName() 方法进行判断
- 修改了 Customer 任一姓名相关字段需要调用  Customer 的 buildNameSearchHelper() 方法重新构建这一辅助字段
- 在订单中一定要记录姓名快照（包含预定人姓名、客人姓名等），方便查询

前端使用姓名相关字段原则：

适用范围:webteyixing,tehang-system

- 前台机票，火车票下单根据用户搜索匹配中文名或者英文名，相关正则维护在constant文件，如果是英文名，需要用英文姓/英文名的形式，并且转换为大写传给后端，如果是中文名，则直接中文姓中文名
- 其他地方显示名字的地方，模板里面用showName的Pipe进行转换，ts里面依赖注入pipe，然后调用pipe的transform方法进行转换


# 前端代码审查列表

- [ ] 功能设计/代码结构是否合理
- [ ] 文件行数/函数行数是否超标(150/30)
- [ ] Http 请求的错误处理
- [ ] 关键步骤的日志
- [ ] 新增的组件都需要开启 OnPush 策略
- [ ] 组件销毁时的资源释放( unsubscribe, clearInterval 等)
  - 特别需要注意的是通过 service 得到的 observale
- [ ] 禁止魔数(number string)
- [ ] 变量命名需要有意义，禁止：i j str obj 等
- [ ] 类型准确，没有足够的理由禁止使用 any
- [ ] 对于 Array,  map/some/every/find/filter > forEach > for loop , 优先使用语义明确的方法
- [ ] 使用解构让代码变得更简洁
```diff
- const data = this.getData()
- if (data.xxx) { do something}
- if (data.yyy) { do something}
- return {
-   xxx: data.xxx,
-   yyy: data.yyy,
-   zzz: data.zzz,
- }
+ const {xxx, yyy, zzz} = this.getData()
+ if (xxx) { do something}
+ if (yyy) { do something}
+ return {xxx, yyy, zzz}
```
- [ ] 避免傻瓜代码
```diff
- const result = this.isValid()
- if (result) {
-   return true
- }
- return flase
+ return this.isValid()
```

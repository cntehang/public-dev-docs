# 基本原则:
- 必须非常清楚改动的意义以及带来的影响
- 把代码pull到本地进行review

# API 变动review原则

- 如果API的条件是由严变松（如：not null --> nullable ），那么必须把代码pull下来进行review，并增加null的测试（测试用例、测试脚本都可以）
如果API的条件是由松变严(如：nullable --> not null)，那么可以沿用之前的测试用例

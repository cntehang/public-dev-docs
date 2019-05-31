# 数据量比较大时，使用ion-virtual-scroll

> 该组件在数据量很大时的性能优势极为明显，接收的是一个数组，即使数据量很大，每次也只显示一部分（大概是稍大于屏幕显示范围的数量),如果数据量较小或者有分页时，不推荐使用。该组件本身也是有问题的，快速滑动时会有留白，但是为了应对大量数据的明显卡顿问题可以暂时忽略轻微的留白。

使用注意事项：

- 注意要给approxItemHeight，最好与每个item的实际渲染高度相差不大，可以加速与计算virtualstroll的滚动高度。此处要特别注意，如果预估的高度与实际高度相差特别大，在低性能设备上会卡顿的很明显， 如果很相近，则会很顺畅丝滑

- trackBy ，更改数据源（筛选，重新查询）后同一个元素可以重用，一般返回对应数据的唯一标识

- 如果有头部信息，需要注意headerFn的性能问题，该方法里最好只做简单操作，否则会很卡

- headerFn和trackBy方法里都不能直接访问this，所以可以让headerFn和trackBy返回一个匿名函数，在匿名函数中使用this的引用

  代码如下：

  ```typescript
   /**
     * 虚拟item 头部信息
     */
    virturalItemHeaderFn() {
      const thisReference = this
      return function(_: Airport, recordIndex: number, __: Airport[]) {
        return !thisReference.isSearching
          ? thisReference.keysMap.get(recordIndex)
          : null
      }
    }
  ```


## 参考资料

- [Ionic 4 | Implement Infinite Scroll List with Virtual Scroll List in Ionic 4 Application](https://www.freakyjolly.com/ionic-4-implement-infinite-scroll-list-with-virtual-scroll-list-in-ionic-4-application/)

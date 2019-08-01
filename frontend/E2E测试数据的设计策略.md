# E2E测试数据的设计策略

不管是做手工测试还是自动化测试，每一条测试用例，其实应该包含3个核心部分：

- 该用例依赖的数据
- 该用例的执行步骤
- 该用例的期望结果

如果把用例的执行过程看做是黑盒，那你喂给这个用例的数据就是输入，执行完这条用例之后你能得到输出，这个称之为实际结果，在最后你需要拿实际结果和期望结果进行比对，以此来判断该用例是否正确执行。所以测试的运行情况是严重依赖于测试数据，尤其对于自动化测试来说更是如此。

那么在写e2e测试用例的时候，我们要如何处理测试数据呢？一种最简单的办法是将测试数据hardcode到测试用例里面，像是下面这样：

```ts
it('should show return button', async () => {
  // 测试数据
  const orderNo = '310003375'
  // 执行之后的实际结果
  const rst = hotelOrdersPage.table.row
  // 预期结果
	const expectedRst = '退订'
  await hotelOrdersPage.enterOrderNo(orderNo).search()
  
  expect(rst).toContain(expectedRst)
})
```

上面这条测试用例依赖一个测试数据就是`orderNo`，这种方案简单明了，但是有个很大的弊端。在分析弊端之前我们先回到我们在实际开发过程中会如何去运行这些测试用例上来，当你把所有功能都完成了，相应的核心e2e测试用例也写好了，在提交测试验证之前我们需要自己先跑一遍所有的测试用例，那么问题来了：如果使用的hardcode方式，意味着我在测试环境要造好跟hardcode一样的数据，比如说上述代码中的orderNo；如果我下次因为某些原因要修改这些数据，我甚至还要找到我的测试用例里面去修改。

## 分离测试数据

自动化测试用例也是一堆代码，所以完全可以运用一些通用编程思维。对于上面测试数据hardcode的问题，解决方案就是将测试数据从测试用例里面分离出去，我们有一个地方统一去管理所有需要的测试数据，封装一个获取数据的api供测试用例调用，代码像是下面这样：

```ts
it('should show return button', async () => {
  // 测试数据
  const orderNo = getData().orderNo
  // 执行之后的实际结果
  const rst = hotelOrdersPage.table.row
  // 预期结果
	const expectedRst = '退订'
  await hotelOrdersPage.enterOrderNo(orderNo).search()
  
  expect(rst).toContain(expectedRst)
})
```

### 测试数据存在哪

那么问题又来了：每条测试数据所依赖的数据是不同，这个不同体现在几个方面：

- 数据的字段不一致
- 数据项的数量不一致

我要如何把这些不一致的数据统一起来管理呢？说到统一管理数据，可能一下子会冒出很多东西：

- 本地文件：Json | YAML | Excel
- 远程读取：数据库，Json

本地文件中，JSON和YAML虽然相比Excel更灵活，更容易去处理这些高度非结构化的数据，但是维护性却不如Excel；而本地文件相比远程文件，其优势是对于写代码的人来说更容易读写，劣势也是读写和可维护性，因为远程文件是对于团队协作更友好的方式。想象一下，你将测试用例代码化之后，很可能会有专门的测试人员去准备测试数据，那么这种时候远程无疑是一种更好的方式了。

另外，对于远程模式而言并非没有代价，如果要使用远程读取的方式，最好能有个图形化管理界面才能最大化发挥出它的价值。由于用例数据的结构化难度很大，所以推荐采取json这种灵活的模式来进行读写会更方便；在这种模式下，依然存在不同的方案：一种是MongoDB，一种是纯JSON文件形式，从长远考虑可维护性，MongoDB要优于纯文件的形式。

### 如何区分不同用例之间的数据

解决完读取问题之后，再来看另外一个问题：在用例里面如何通过统一的api去获取特定的某条数据呢？通常id会是一个选择，代码像是这样：

```ts
it('should show return button', () => {
  const orderNo = getData('123456').input
  hotelOrdersPage.enterOrderNo(orderNo).search()
})
```



但是如果直接在测试用例里面裸写某个id并不是个明智的选择，原因有二：

- id的来源是数据库，所以得先从数据库那边拿到id，耗时耗力，不利于解耦
- id在用例里面的可识别性太差

基于以上的考虑，我们不应该从数据源头考虑这个问题，而是采用一种约定方式，基于某个字段，在用例代码里面是这个值，在数据维护那边一样也是这个值，对于用例来说，it语句里面的描述其实就是个不错的识别码：首先它有特定的意思，其次，在某个特定的测试集里面它应该是唯一的（你不会希望在同一个测试集里面出现两个相同的测试用例说明），基于此我们可以构造如下代码：

```ts
const caseKey = {
  'shouldShowReturnButton': 'should show return button',
}
it(caseKey.shouldShowReturnButton, () => {
  const orderNo = getData(caseKey.shouldShowReturnButton).input
  hotelOrdersPage.enterOrderNo(orderNo).search()
})
```

而在数据维护那边我们如果要为这条用例添加数据或者修改这条用例的测试数据，则以`caseKey.shouldShowReturnButton`作为唯一的key值来识别，所以在这条用例的第一行读取数据就使用了这个key。

到这一步，我们基本上算是把测试数据和用例代码解耦了。

### 不同的测试集如何读取数据

前面讨论的是一条测试用例要如何获取属于自己的测试数据，而在实际的测试代码中，通常是以测试集来做用例的组织的，一个测试集以`describe`包裹起来，每个测试集里面会包含多条测试用例，代码如下：

```ts
describe('Hotel orders operations', () => {
  let hotelOrdersPage: HotelOrdersPage
  let caseKeys = {
    'shouldShowReturnButton': 'should show return button',
  }
  beforeAll(() => {
    hotelOrdersPage = new HotelOrdersPage()
    hotelOrdersPage.open()
  })
  
  it(caseKey.shouldShowReturnButton, () => {
    const orderNo = getData(caseKey.shouldShowReturnButton).input
    hotelOrdersPage.enterOrderNo(orderNo).search()
  })
})
```

由于前面讨论的是把数据放到远程数据库进行维护，那么我们是否要在每条测试用例里面去获取一次数据呢？很显然这是个很糟糕的做法，正确的做法应该是

- 运行这个测试集的开始拉取这个测试集对应的测试数据
- 运行所有测试集之前拉取所有的测试数据

虽然有两种策略，但是方案一是更适合的选择，因为有的时候我并不想运行所有的测试集，所以比较好的做法就是在运行某个特定测试集的开始去拉取该测试集所属的测试用例。

在前面的讨论中涉及了如何去获取某个测试用例所属的测试数据，同样的在获取测试集所属数据也要做同样的考虑和设计，我们会以测试集的名称（`describe`后面的描述）来区分不同测试集所属的数据，因为在整个项目中不同的测试集名称应该保持唯一，这样有利于管理，根据这些结论，我们的代码可以做如下调整：

```ts
// 统一的获取测试数据的方法，根据测试集的名字
// 实际的代码里面该方法会向服务器去请求数据，请求特定测试集对应的测试数据
const getTestData = (bySuiteName: string) => (byCaseKey: string) => any

const suiteName = {
  hotelOrderOperations: 'Hotel order operations'
}
describe(suiteName.hotelOrderOperations, () => {
  let caseKeys = {
    'shouldShowReturnButton': 'should show return button',
  }
  // 此处any仅用于演示
  let getData: (caseKey: string) => any
  beforeAll(() => {
    getData = getTestData(suiteName.hotelOrderOperations)
  })
  // 此处省去用例代码
})
```

这样构造以后，测试数据维护界面就应该根据这里的测试集的名称来管理测试数据。讨论到现在我们对于测试数据的设计有个大概的想法了：所有的测试数据先以测试集的名字做分类，在测试集下面又以测试用例的名称来管理每个测试用例自己的数据，最终的数据结构像是下面这样:

```json
{
  "hotelOrderOperations": {
    "shouldShowReturnButton": {
      "input": {
        "orderNo": "310003375"
      }
    },
    "shouldSupportFuzzySearchByGuestName": {
      "input": {
        "guestName": "张三"
      }
    }
  },
  "flightOrderOperations": {
    "shouldShowCancelButton": {
      "input": {
        "orderNo": "123456"
      }
    },
    "shouldSupportCombinatedSearch": {
      "input": {
        "guestName": "张三",
        "status": "已确认",
      }
    }
  }
}
```

## 分离预期结果

再回到前面测试用例里面将测试数据分离出去的点，还有上面的数据结构设计，其实这里故意设计了一个`input`字段，主要原因是因为我们之前只将测试数据分离出去了，但是预期结果仍然还留在测试用例里面，而实际预期结果是跟测试数据紧密管理，只有设计测试数据的人才知道自己想要的结果是什么，所以**预期结果**也要分离出去，直接跟自己所属测试用例的测试数据放在一起，也就是上面结构中的`input`平级，所以得到一个初步的方案如下：

```json
{
  "hotelOrderOperations": {
    "shouldShowReturnButton": {
      "input": {
        "orderNo": "310003375"
      },
      "expectedResult": {
        
      }
    },
  }
}
```

通常预期结果在一条测试用例里面会包含多个预期，且预期结果会有一个很口语化的描述，例如输入订单号：123456进行查询之后，我期望：搜索之后的列表不包含任何数据且页面显示无数据，这个期望结果其实包含两个信息：

1. 搜索结果数量为0
2. 列表显示：无数据

这个过程相当于是从一条口语化的内容里面去抽出我们想要的数据，这个转化过程是一定要有的，不然自动化测试的最后一个环节即：实际结果与预期结果的比较就无法进行了，我们没办法拿实际代码运行出来的结果与一条很口语化的句子去进行对比，所以针对分解出来的信息，我们需要对它们进行数据的结构化设计：

```json
{
  "hotelOrderOperations": {
    "shouldShowNoResult": {
      "input": {
        "orderNo": "310003375"
      },
      "expectedResult": {
        "listZero": {
          "value": 0,
          "description": "搜索之后的列表不包含数据"
        },
        "noContent": {
          "value": "无数据",
          "description": "表格中显示无数据"
        }
      }
    },
  }
}
```

上面结构中的**value**用于测试用例代码中取值和实际结果进行比较，**description**用于展示更易读的日志信息，结合该数据，最终代码如下：

```ts
// 省略若干代码
describe(suiteName.hotelOrderOperations, () => {
  let caseKeys = {
    'shouldShowNoResult': 'should show no result',
  }
  // 此处any仅用于演示
  let getData: (caseKey: string) => any
  beforeAll(() => {
    getData = getTestData(suiteName.hotelOrderOperations)
  })
  it('should show return button', async () => {
    // 测试数据
    const { input, expectedResult } = getData(caseKeys.shouldShowNoResult)
    // 执行之后的实际结果
    const table = hotelOrdersPage.table
    const orderCount = table.rows
    // 预期结果
    const { listZero, noContent } = expectedResult
    await hotelOrdersPage.enterOrderNo(input.orderNo).search()
		
    console.log(listZero.description)
    expect(orderCount).toBe(listZero.value)
    
    console.log(noContent.description)
    expect(table).toContain(noContent.value)
  })
})
```

## 环境与数据复用

到目前为止，数据分离出去了，数据结构也设计好了，我们还漏了一个重要的事情，假定我们把测试用例代码全部完成了，测试数据也设计好并且录入了系统，我执行一次之后就会将我原来设计的数据全部污染掉，下次再执行的时候，我可能几乎要全部重新设计数据，想象如果系统够复杂，测试代码足够多的话，这个重新设计的过程有多复杂？

所以好的方案是在首次设计好要使用的测试数据之前，将这份数据做一次备份，在每次执行测试用例之前用这份数据恢复一下再去跑测试用例，以后对于这份数据的维护只会发生在增加新的测试用例的情况下了。

## 更进一步

是不是到这一步就已经完美了？其实不然。回看上面会发现测试数据和测试用例**代码**之间并未完全解耦，未完全解耦的意思是，如果让一个测试人员只关心测试数据和测试用例的目的，能不能在没有开发人员的指导的情况完全跑起来这些用例？或者即便不是跨角色，是跨人员的，比如A写了这堆代码，B去维护测试数据，或者准备新的测试数据的时候，他知道如何填充这些数据吗？

在解决这个问题之前我们先来复盘一下整个测试的过程是如何进行的：

1. 设计测试用例（包含测试数据、测试步骤、预期结果）
2. 根据测试用例来实现测试代码
3. 准备测试环境
4. 执行自动化测试

在第二步里面，由于我们要依赖特定的数据字段名，例如：`orderNo`, `guestName`，所以到底是应该先在代码里面定义好我需要使用的测试数据对应的字段名称呢，还是应该先在测试数据维护系统里面先定义好，再根据测试数据维护系统里面去找这些字段来写到代码里面呢？这个过程其实跟前后端在定义**api**是很类似的：到底是先后端定义好前端直接去看文档再写入自己代码还是前端先写入代码再告知后端呢？我们知道通常做法是后端会先定义好，然后双方可能有一个协商的过程，最后再根据定稿的内容前端再去写入代码我要使用哪些特定的字段名。

那对于测试用例来说看起来两者都可行，但是在这里我们还是优先在测试数据维护系统中去定义好代码中所需要的字段，补上字段对应的说明然后可以交给他人去录入这些数据。因为如果先在代码中定义那就会出现如果设计测试数据的人和写测试代码的不是同一个人，导致沟通协作成本的大量增加。

我们还剩最后一件事，那就是根据上面的数据结构和需求去实现一个简单的图形化测试数据维护界面！Just do it!
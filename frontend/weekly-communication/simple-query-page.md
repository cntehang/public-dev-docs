# 一个简单查询展示页面

后台大部分页面都是简单查询展示页面，样式风格也是统一的，为了简化后台的开发，故做此简单的约定

## 页面布局

页面分 3 个部分：头部是一个页面名称，中部为搜索条件表单，下部分为数据展示表格，完整示例代码如下：

```html
<page-header title="企业账单" class="page-header-small"></page-header>

<form nz-form [formGroup]="form" class="form-small">
  <div shf-wrap="4" labelWidth="150">
    <shf-item label="出账日开始日期">
      <nz-date-picker formControlName="billDateStart"> </nz-date-picker>
    </shf-item>
    <shf-item label="出账日结束日期">
      <nz-date-picker formControlName="billDateEnd"> </nz-date-picker>
    </shf-item>
    <shf-item label="客户简称">
      <input nz-input formControlName="corpNameLike" />
    </shf-item>
    <shf-item label="状态">
      <shared-dict-select
        dictType="CorpCreditBillStatus"
        formControlName="status"
      >
      </shared-dict-select>
    </shf-item>
    <shf-item>
      <button
        nz-button
        nzType="primary"
        (click)="simpleTable.reset()"
        [disabled]="isSpinning$ | async"
      >
        <!-- 如果不是后台分页的 data 请改为 (click)="query()" -->
        查询
      </button>
      <button nz-button type="reset">重置</button>
    </shf-item>
  </div>
</form>

<nz-spin [nzTip]="[nzTip]" [nzSpinning]="isSpinning$ | async">
  <simple-table
    class="table-text-center table-small"
    showSizeChanger
    showPagination
    showTotal
    showQuickJumper
    [bordered]="true"
    [frontPagination]="false"
    (change)="query()"
    [data]="data"
    [columns]="columns"
  >
    <!--
      如果不是后台分页的 data 请去除  [frontPagination]="false" (change)="query()"
    -->
    <ng-template st-row="billDate" let-i>
      <span> {{ i.billDate | date: 'yyyy-MM-dd' }} </span>
    </ng-template>
    <ng-template st-row="status" let-i>
      <span> {{ i.status | dictValueTransform: 'CorpCreditBillStatus' }} </span>
    </ng-template>
  </simple-table>
</nz-spin>
```

**_注意：_**

- `shf-wrap="4" labelWidth="150"` 根据实际原型调整
- 统一使用 ReactiveForms 禁止使用 Template-driven forms
- 若操作按钮过多，建议另起一行 用 nz-row 包裹

## 页面样式

为了保持系统的整体性和统一行，非特殊的查询展示页统一使用: `class="page-header-small"` `class="form-small"` `class="table-text-center table-small"`  
以上样式定义在 index.less 中 便于统一修改
**_注意_**：本样式只适用于后台页面，前台请自行定义

```less
.page-header-small {
  padding: 6px 24px 0px 24px;
  margin-bottom: 6px;
}

.page-header-small.ad-ph .title,
.page-header-small.ad-ph .action {
  margin-bottom: 6px;
}

.form-small {
  border: 1px solid #d9d9d9;
  border-radius: 4px;
  padding: 6px;
  margin-bottom: 6px;

  .ant-input {
    padding: 1px 7px;
    height: 24px;
  }

  .ant-form-item {
    margin-bottom: 0;
  }

  .ant-select-selection--multiple,
  .ant-select-selection--single {
    height: 24px;
  }

  .ant-select-selection__rendered {
    line-height: 22px;
    margin: 0 7px;
  }
}

.table-text-center {
  table th,
  table td {
    text-align: center !important;
  }
}

.table-small {
  table th,
  table td {
    padding: 2px;
    height: 47px;
  }
  table td {
    background-color: white;
  }
}
```

## 基本逻辑代码

```ts
export class BillListComponent implements OnInit {
  isSpinning$ = new BehaviorSubject<boolean>(false);
  nzTip: string;
  form: FormGroup;

  @ViewChild(SimpleTableComponent)
  simpleTable: SimpleTableComponent;
  columns: SimpleTableColumn[] = [
    { title: "出账日", render: "billDate" },
    { title: "客户简称", index: "corpShortName" },
    { title: "账单起始日", render: "billStartDate" },
    { title: "是否逾期", render: "overdue" },
    { title: "状态", render: "status" },
    {
      title: "操作",
      buttons: [
        {
          text: "详情",
          click: (record: BillSummaryResponse) => {
            this.log.debug(record);
            this.routerService.toDetail(record.billId);
          }
        }
      ]
    }
  ];
  data: BillSummaryResponse[];

  private log: Logger;
  constructor(
    loggerService: LoggerService,
    private fb: FormBuilder,
    private httpService: BillListHttpService,
    private routerService: BillListRouterService,
    private handleErrorService: HandleErrorService
  ) {
    this.log = loggerService.getLogger("BillListComponent");
  }

  ngOnInit() {
    this.form = this.initForm();
    // 默认加载数据
    this.query();
  }

  /**
   * 分页获取企业账单数据
   *
   * @memberof BillListComponent
   */
  query() {
    const methodName = "query";
    const params = this.getParams();
    this.showSpin("查询中,请稍候...");
    this.httpService.getCreditBills(params).subscribe(
      res => {
        this.hideSpin();
        if (res instanceof AppError) {
          this.handleErrorService.handleAppError(this.log, res, methodName);
        } else {
          this.simpleTable.total = res.totalElements;
          this.data = res.content;
        }
      },
      err => {
        this.hideSpin();
        this.handleErrorService.handleAppError(this.log, err, methodName);
      }
    );
  }

  /**
   * 组装查询的参数
   *
   * @private
   * @returns {BillSearchRequest}
   * @memberof BillListComponent
   */
  private getParams(): BillSearchRequest {
    const methodName = "getParams";
    const params: BillSearchRequest = {
      ...this.form.value,
      ...this.formatDate(),
      pageNumber: this.simpleTable.pi,
      pageSize: this.simpleTable.ps
    };
    this.log.debug(methodName, params);
    return params;
  }

  /**
   * 起始时间 是 00：00：00
   * 截止时间是 +1天的 00：00：00
   *
   * @private
   * @returns {BillSearchRequest}
   * @memberof BillListComponent
   */
  private formatDate(): BillSearchRequest {
    const formValue = this.form.value;

    const billDateStart = formValue.billDateStart
      ? DateUtils.startOfDay(formValue.billDateStart)
      : undefined;
    const billDateEnd = formValue.billDateEnd
      ? DateUtils.addDays(DateUtils.startOfDay(formValue.billDateEnd), 1)
      : undefined;

    return {
      billDateStart,
      billDateEnd
    };
  }

  private initForm() {
    return this.fb.group({
      billDateStart: [],
      billDateStart: [],
      corpNameLike: [],
      status: []
    });
  }

  private showSpin(nzTip: string) {
    this.nzTip = nzTip;
    this.isSpinning$.next(true);
  }

  private hideSpin() {
    this.isSpinning$.next(false);
  }
}
```

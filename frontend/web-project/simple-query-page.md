# 一个简单查询展示页面

**Web 后台** 中大部分页面都是简单查询展示页面，样式风格也是统一的，为了简化后台的开发，故做此简单的约定

## 页面布局

页面分 3 个部分：头部是一个页面名称，中部为搜索条件表单，下部分为数据展示表格，完整示例代码如下：

```html
<page-header title="国内机票订单补录" class="page-header-small"></page-header>
<form
  nz-form
  [formGroup]="form"
  class="form-small"
  se-container="3"
  labelWidth="150"
>
  <se label="订单号">
    <input nz-input formControlName="orderNoLike" />
  </se>
  <se label="航班号">
    <input nz-input formControlName="flightNoLike" />
  </se>
  <se label="客户简称">
    <input nz-input formControlName="corpShortNameLike" />
  </se>
  <se label="预订日期">
    <nz-date-picker
      formControlName="bookingDateStart"
      class="width-45"
    ></nz-date-picker>
    ~
    <nz-date-picker
      formControlName="bookingDateEnd"
      class="width-45"
    ></nz-date-picker>
  </se>
  <se label="预订人">
    <input nz-input formControlName="bookingEmployeeNameLike" />
  </se>
  <se label="乘机人">
    <input nz-input formControlName="passengerNameLike" />
  </se>
  <se label="航空公司">
    <input nz-input formControlName="airlineLike" />
  </se>
  <se label="出发日期">
    <nz-date-picker
      formControlName="flightDateStart"
      class="width-45"
    ></nz-date-picker>
    ~
    <nz-date-picker
      formControlName="flightDateEnd"
      class="width-45"
    ></nz-date-picker>
  </se>
  <se label="状态">
    <shared-dict-select
      dictType="FlightManualOrderStatus"
      formControlName="status"
    ></shared-dict-select>
  </se>
  <se label="出票日期">
    <nz-date-picker
      formControlName="ticketingTimeStart"
      class="width-45"
    ></nz-date-picker>
    ~
    <nz-date-picker
      formControlName="ticketingTimeEnd"
      class="width-45"
    ></nz-date-picker>
  </se>
  <se>
    <button
      nz-button
      nzType="primary"
      [disabled]="isSpinning"
      (click)="simpleTable.reset()"
    >
      查询
    </button>
    <button nz-button type="reset">清空</button>
  </se>
  <se>
    <button nz-button nzType="primary" [disabled]="isSpinning">新增</button>
  </se>
</form>
<nz-spin [nzTip]="spinTip" [nzSpinning]="isSpinning">
  <st
    #simpleTable
    class="table-text-center table-small"
    [bordered]="true"
    [columns]="columns"
    [data]="data"
    (change)="onStChange($event)"
    [page]="{
      showSize: true,
      show: true,
      showTotal: true,
      showQuickJumper: true,
      front: false
    }"
  >
    <ng-template st-row="orderNo" let-i>
      <a routerLink="../order-detail/{{ i.orderId }}">{{ i.orderNo }}</a>
    </ng-template>
    <ng-template st-row="bookingTime" let-i>
      <span>{{ i.bookingTime | date: 'yyyy-MM-dd HH:mm' }}</span>
    </ng-template>
    <ng-template st-row="orderStatus" let-i>
      <span>{{ i.orderStatus | dictValueTransform: 'FlightOrderStatus' }}</span>
    </ng-template>
    <ng-template st-row="ticketingTime" let-i>
      <span>{{ i.ticketingTime | date: 'yyyy-MM-dd HH:mm' }}</span>
    </ng-template>
    <ng-template st-row="status" let-i>
      <span
        >{{ i.status | dictValueTransform: 'FlightManualOrderStatus' }}</span
      >
    </ng-template>
  </st>
</nz-spin>
```

**_注意：_**

- `se-container="3" labelWidth="150"` 根据实际原型调整
- 统一使用 ReactiveForms 禁止使用 Template-driven forms

## 基本逻辑代码

```ts
import {
  Component,
  OnInit,
  ChangeDetectionStrategy,
  ChangeDetectorRef,
  ViewChild,
} from '@angular/core'
import { FormBuilder, FormGroup } from '@angular/forms'
import { finalize } from 'rxjs/operators'

import { STColumn, STComponent } from '@delon/abc'

import { HandleErrorService } from '@core/services/handle-error.service'
import { Logger, LoggerService } from '@core/services/logger.service'
import { AppError } from '@shared/models/app-error'

import {
  ManualOrderListHttpService,
  FlightManualOrder,
  FlightManualOrderQueryParam,
} from './manual-order-list-http.service'
import { DateUtils } from '@shared/utils/date.utils'

@Component({
  selector: 'manual-order-list',
  templateUrl: './manual-order-list.component.html',
  styles: [],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ManualOrderListComponent implements OnInit {
  form: FormGroup
  spinTip = ''
  isSpinning = false

  data: FlightManualOrder[]
  columns: STColumn[] = [
    {
      title: '订单号',
      render: 'orderNo',
      width: '90px',
      sort: {
        compare: (a, b) => Number(a.orderNo) - Number(b.orderNo),
      },
    },
    { title: '客户简称', index: 'corpShortName' },
    {
      title: '预订时间',
      render: 'bookingTime',
      width: '90px',
      sort: {
        compare: (a, b) =>
          DateUtils.isBefore(a.bookingTime, b.bookingTime) ? 1 : -1,
      },
    },
    {
      title: '出发时间',
      index: 'flightDate',
      width: '90px',
      sort: {
        compare: (a, b) =>
          DateUtils.isBefore(a.flightDate, b.flightDate) ? 1 : -1,
      },
    },
    { title: '行程', index: 'cityNames' },
    { title: '航班号', index: 'flightNo' },
    { title: '预订人', index: 'bookingEmployeeName' },
    { title: '乘机人', index: 'passengerNames', width: '90px' },
    { title: '订单状态', render: 'orderStatus' },
    { title: '出票时间', render: 'ticketingTime', width: '90px' },
    { title: '金额(元)', index: 'paymentAmount' },
    { title: '订单状态', render: 'status' },
    {
      title: '操作',

      buttons: [
        {
          text: '添加',
          type: 'none',
          click: record => {
            console.log(record)
          },
        },
      ],
    },
  ]
  @ViewChild('simpleTable')
  private simpleTable: STComponent
  private log: Logger
  constructor(
    loggerService: LoggerService,
    private changeDetectorRef: ChangeDetectorRef,
    private formBuilder: FormBuilder,
    private httpService: ManualOrderListHttpService,
    private handleErrorService: HandleErrorService,
  ) {
    this.log = loggerService.getLogger('ManualOrderListComponent')
  }

  ngOnInit() {
    this.createForm()
    this.query()
  }

  onStChange(event: STChange) {
    const eventType = event.type
    if (eventType === 'pi' || eventType === 'ps') {
      this.query()
    }
  }

  private query() {
    const methodName = 'query'
    const queryParam = this.getQueryParam()
    this.log.debug(methodName, queryParam)
    this.startSpin('正在查询中,请稍候...')
    this.httpService
      .getManualOrders(queryParam)
      .pipe(
        finalize(() => {
          this.stopSpin()
          this.changeDetectorRef.markForCheck()
        }),
      )
      .subscribe(
        res => {
          if (res instanceof AppError) {
            this.handleErrorService.handleAppError(this.log, res, methodName)
          } else {
            this.data = res.content
            this.simpleTable.total = res.totalElements
          }
        },
        err =>
          this.handleErrorService.handleHttpError(this.log, err, methodName),
      )
  }

  private getQueryParam(): FlightManualOrderQueryParam {
    return {
      ...this.form.value,
      ...this.formatDate(),
      pageNumber: this.simpleTable.pi,
      pageSize: this.simpleTable.ps,
    }
  }

  private formatDate() {
    const formValue = this.form.value

    return {
      bookingDateStart: DateUtils.startOfDay(formValue.bookingDateStart),
      bookingDateEnd: DateUtils.startOfDay(
        DateUtils.addDays(formValue.bookingDateEnd, 1),
      ),
      flightDateStart: DateUtils.format(formValue.flightDateStart),
      flightDateEnd: DateUtils.format(
        DateUtils.addDays(formValue.flightDateEnd, 1),
      ),
      ticketingTimeStart: DateUtils.startOfDay(formValue.ticketingTimeStart),
      ticketingTimeEnd: DateUtils.startOfDay(
        DateUtils.addDays(formValue.ticketingTimeEnd, 1),
      ),
    }
  }

  private createForm() {
    this.form = this.formBuilder.group({
      orderNoLike: [''],
      flightNoLike: [''],
      corpShortNameLike: [''],
      bookingDateStart: [''],
      bookingDateEnd: [''],
      bookingEmployeeNameLike: [''],
      passengerNameLike: [''],
      airlineLike: [''],
      flightDateStart: [''],
      flightDateEnd: [''],
      status: ['WaitReview'],
      ticketingTimeStart: [''],
      ticketingTimeEnd: [''],
    })
  }

  private startSpin(spinTip: string) {
    this.spinTip = spinTip
    this.isSpinning = true
  }

  private stopSpin() {
    this.isSpinning = false
  }
}
```

## 页面样式

为了保持系统的整体性和统一行，非特殊的查询展示页统一使用: `class="page-header-small"` `class="form-small"` `class="table-text-center table-small"`  
以上样式定义在 `index.less` 中 便于统一修改

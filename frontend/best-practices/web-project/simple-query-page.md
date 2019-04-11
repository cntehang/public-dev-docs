# 一个简单查询展示页面

**Web 后台** 中大部分页面都是简单查询展示页面，样式风格也是统一的，为了简化后台的开发，故做此简单的约定

## 页面布局

页面分 3 个部分：头部是一个页面名称，中部为搜索条件表单，下部分为数据展示表格，完整示例代码如下：

```html
<page-header title="火车票任务处理" class="page-header-small"></page-header>
<form
  nz-form
  [formGroup]="form"
  class="form-small"
  (ngSubmit)="simpleTable.reset()"
>
  <se-container [se-container]="3" [labelWidth]="150">
    <se label="订单号">
      <input nz-input formControlName="orderNoLike" />
    </se>
    <se label="客户编码">
      <input nz-input formControlName="corpNoLike" />
    </se>
    <se label="预订日期">
      <nz-date-picker
        formControlName="bookingDateStart"
        nzPlaceHolder="开始日期"
        class="width-45"
      >
      </nz-date-picker>
      ~
      <nz-date-picker
        formControlName="bookingDateEnd"
        nzPlaceHolder="截止日期"
        class="width-45"
      >
      </nz-date-picker>
    </se>
    <se label="出发日期">
      <nz-date-picker
        formControlName="trainDateStart"
        nzPlaceHolder="开始日期"
        class="width-45"
      >
      </nz-date-picker>
      ~
      <nz-date-picker
        formControlName="trainDateEnd"
        nzPlaceHolder="截止日期"
        class="width-45"
      >
      </nz-date-picker>
    </se>
  </se-container>
  <div nz-row nzGutter="8">
    <div nz-col nzSpan="18">
      <button
        nz-button
        type="button"
        class="mx-lg"
        nzType="primary"
        [disabled]="!checkedRows.length"
        (click)="onClick()"
      >
        调出任务
      </button>
    </div>
    <div nz-col nzSpan="6" class="text-right">
      <button
        nz-button
        type="submit"
        nzType="primary"
        class="width-40"
        [disabled]="isSpinning"
      >
        查询
      </button>
      <button nz-button type="reset" class="width-40">清空</button>
    </div>
  </div>
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
    <ng-template st-row="bigOrderNo" let-i>
      <span class="text-blue point" click="'orderId'">{{ i.bigOrderNo }}</span>
    </ng-template>
    <ng-template st-row="bookingDateTime" let-i>
      <span>{{ i.bookingDateTime | date: 'yyyy-MM-dd HH:mm' }}</span>
    </ng-template>
    <ng-template st-row="createTime" let-i>
      <span>{{ i.createTime | date: 'yyyy-MM-dd HH:mm' }}</span>
    </ng-template>
  </st>
</nz-spin>
```

**_注意：_**

- `[se-container]="3" [labelWidth]="150"` 为默认值 根据实际原型调整
- 统一使用 `ReactiveForms` 禁止使用 `Template-driven forms`
- 对于表单内的 `button` , 请使用符合 `html5` 规范的写法：
  - 使用 `form` 的 `submit` 事件, 在 `Angular` 中即  `(ngSubmit)="onSubmit()"`
  - 给每个 `button` 指定具体的 `type`

## 基本逻辑代码

```ts
@Component({
  selector: 'processing-list',
  templateUrl: './processing-list.component.html',
  styles: [],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ProcessingListComponent implements OnInit, OnDestroy {
  private unsubscribe$ = new Subject()
  private refreshStaffInterval: NodeJS.Timer
  private log: Logger
  staffId: number
  spinTip: string
  isSpinning = false
  form: FormGroup

  data: AdminTrainTaskBo[] // 查询结果
  checkedRows: AdminTrainTaskBo[] = [] //被选中的行
  @ViewChild('simpleTable')
  simpleTable: STComponent
  columns: STColumn[] = [
    { title: '全选', index: 'id', type: 'checkbox' },
    {
      title: '订单号',
      render: 'bigOrderNo',
      width: '90px',
      sort: {
        compare: (a, b) => Number(a.bigOrderNo) - Number(b.bigOrderNo),
      },
    },
    { title: '客户简称', index: 'corpShortName' },
    { title: '行程', index: 'stationNames' },
    { title: '航班号', index: 'trainCode' },
    { title: '预订人', index: 'bookingCustomerName' },
  ]

  constructor(
    loggerService: LoggerService,
    private authService: AuthCacheService,
    private staffInfoService: StaffInfoService,
    private formBuilder: FormBuilder,
    private httpService: ProcessingListHttpService,
    private changeDetectorRef: ChangeDetectorRef,
    private handleErrorService: HandleErrorService,
  ) {
    this.log = loggerService.getLogger('ProcessingListComponent')
  }

  ngOnInit(): void {
    this.authService
      .getLoginStaff()
      .pipe(takeUntil(this.unsubscribe$))
      .subscribe(staff => {
        this.staffId = staff ? staff.id : undefined
        this.log.debug('staff: ', this.staffId)
      })

    this.createForm()

    this.query()
  }

  ngOnDestroy(): void {
    this.unsubscribe$.next()
  }

  onStChange(event: STChange): void {
    const eventType = event.type
    if (eventType === 'pi' || eventType === 'ps') {
      this.query()
    } else if (eventType === 'checkbox') {
      this.onCheckboxChange(event.checkbox as AdminTrainTaskBo[])
    }
  }

  private query(): void {
    const methodName = 'query'
    const params = this.getQueryParams()
    this.log.debug(methodName, params)
    this.startSpin('正在查询中,请稍候...')
    this.httpService
      .getTrainTasks(params)
      .pipe(
        finalize(() => {
          this.stopSpin()
          this.countDownConfig = { template: `$!s!秒`, leftTime: 30 }
          this.changeDetectorRef.markForCheck()
        }),
      )
      .subscribe(
        res => {
          if (res instanceof AppError) {
            this.handleErrorService.handleAppError(this.log, res, methodName)
          } else {
            this.simpleTable.total = res.totalElements
            this.data = res.content
          }
        },
        err =>
          this.handleErrorService.handleHttpError(this.log, err, methodName),
      )
  }

  private onCheckboxChange(checkedRows: AdminTrainTaskBo[]): void {
    this.checkedRows = checkedRows
  }

  private getQueryParams(): AdminTrainTaskQueryBo {
    const formValue = this.form.value
    return {
      ...formValue,
      bookingDateStart: DateUtils.startOfDay(formValue.bookingDateStart),
      bookingDateEnd: DateUtils.startOfDay(
        DateUtils.addDays(formValue.bookingDateEnd, 1),
      ),
      trainDateStart: DateUtils.format(formValue.trainDateStart),
      trainDateEnd: DateUtils.format(
        DateUtils.addDays(formValue.trainDateEnd, 1),
      ),
      pageNumber: this.simpleTable.pi,
      pageSize: this.simpleTable.ps,
    }
  }

  private createForm(): void {
    this.form = this.formBuilder.group({
      orderNoLike: [''],
      corpNoLike: [''],
      bookingDateStart: [''],
      bookingDateEnd: [''],
      trainDateStart: [''],
      trainDateEnd: [''],
    })
  }

  private startSpin(spinTip: string) : void{
    this.spinTip = spinTip
    this.isSpinning = true
  }

  private stopSpin() : void{
    this.isSpinning = false
  }
}

```

## 页面样式

注意：这里的样式是 `tehang-system` 的，其他项目请自定义  
为了保持系统的整体性和统一行，非特殊的查询展示页统一使用: `class="page-header-small"` `class="form-small"` `class="table-text-center table-small"`  
以上样式定义在 `index.less` 中 便于统一修改

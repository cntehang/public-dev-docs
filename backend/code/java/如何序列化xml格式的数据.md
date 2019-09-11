# 如何序列化 xml 格式的数据

统一采用这种方式来进行 API 的序列化和反序列化
gradle 依赖：`compile('com.fasterxml.jackson.dataformat:jackson-dataformat-xml:2.9.8')`

## dto 样例：

```java
@Data
@JacksonXmlRootElement
public class WechatPlaceOrderDto implements Serializable {
  private static final long serialVersionUID = 2738646911267887473L;

  /**
   * 是	String(32)	wxd678efh567hg6787	微信分配的小程序ID
   */
  @JacksonXmlProperty(localName = "appid")
  private String appId;
  /**
   * 是	String(32)	1230000109	微信支付分配的商户号
   */
  @JacksonXmlProperty(localName = "mch_id")
  private String mchId;
  /**
   * 否	String(32)	013467007045764	自定义参数，可以为终端设备号(门店号或收银设备ID)，PC网页或公众号内支付可以传"WEB"
   */
  @JacksonXmlProperty(localName = "device_info")
  private String deviceInfo;
}

```

`JacksonXmlRootElement`注解标志在 class 上，`JacksonXmlProperty`注解标注在属性上，这样就可以通过 API 正常接受 xml 和返回 xml 格式的数据了。

## API 的设置

```java
@PostMapping(path = "/test", produces = {"application/xml", "text/xml"})
public WechatPlaceOrderDto test2(@RequestBody String body) {}
```

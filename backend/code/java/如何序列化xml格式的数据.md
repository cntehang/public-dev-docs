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

## RestTemplate 设置

一般情况下，不用丹顿设置 RestTemplate，但是有时候第三方不按正规方法做，就需要自主设置一下了

请求发送设置：

```java
  HttpHeaders headers = new HttpHeaders();
  headers.setContentType(MediaType.APPLICATION_XML); // 表示自己发送的是xml格式数据，会按照xml来序列化
  HttpEntity<WechatPlaceOrderDto> request = new HttpEntity<WechatPlaceOrderDto>(dto, headers);
  *** resultDto = restTemplate.postForObject(url, request, ***.class);
```

解析 response 设置：

```java
  //正常情况不用设置，有对应的message converter，但也有例外，如微信服务端，返回的是xml数据，但是media type设置的是text/plain，这样就导致不能够自动解析到对象，只能以字符串接收，然后手工解析，但是也可以设置自己的message converter。 这个只有在确定对方的返回数据时才可以使用。
  MappingJackson2XmlHttpMessageConverter converter = new MappingJackson2XmlHttpMessageConverter();
    List<MediaType> types = new ArrayList<>();
    types.addAll(converter.getSupportedMediaTypes());
    types.add(0, new MediaType("text", "plain"));
    converter.setSupportedMediaTypes(types);

    List<HttpMessageConverter<?>> converters = restTemplate.getMessageConverters();
    converters.add(0, converter);
    restTemplate.setMessageConverters(converters);
```

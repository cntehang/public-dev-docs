# 如何使用 Java 根据 Html 生成 PDF 文档

百度/必应/谷歌一下，使用 Java 生成 PDF 文档的常用工具为

- [iText](https://github.com/itext/itext7)

但是最新的 `iText7` 使用 `AGPL` 协议，需要购买 license 才能够合理合法的在商业项目中使用。本着省钱的原则，使用 iText5 进行开发。

## 1 需求及痛点

调研 `iText` 的 Html 转 PDF 使用后，其主要问题如下

- 由于 `iText` 并非国人开发，内置的字体是不支持中文字符渲染，需要引入额外的字体依赖

网上有不少教程解决了这个问题，但是随之而来引入了另一个问题

- 由于大部分教程引入额外字体依赖时仅考虑了全中文的情况，使用的字体并不能对英文字体进行很好的渲染，造成英文字符错位难看

## 2 依赖配置

```text
  // for pdf rendering
  compile group: 'com.itextpdf', name: 'itextpdf', version: '5.5.13.1'

  // for pdf rendering
  compile group: 'com.itextpdf.tool', name: 'xmlworker', version: '5.5.13.1'

  // for chinese font in pdf rendering
  compile group: 'com.itextpdf', name: 'itext-asian', version: '5.2.0'
```

## 3 字体注册代码

```java
public class PdfFontProvider extends XMLWorkerFontProvider {

  private static final Logger LOG = LoggerFactory.getLogger(PdfFontProvider.class);

  public PdfFontProvider() {
    super(null, null);
  }

  @Override
  public Font getFont(final String fontName, String encoding, float size, final int style) {
    BaseFont font = null;
    try {
      if (StringUtils.equals(fontName, "STSong-Light")) {
        font = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
      } else {
        font = BaseFont.createFont(FontFactory.TIMES_ROMAN, FontFactory.defaultEncoding, true);
      }
    } catch (Exception e) {
      LOG.error("未找到 STSong-Light 字体库，可能是 com.itextpdf.itext-asian 依赖未载入");
    }
    return new Font(font, size, style);
  }

}
```

上述代码会根据 html 节点的 style 属性的 `font-family` 属性配置，对指明使用 `STSong-Light` 字体的内容使用宋体进行渲染，而其它部分则会使用其内置的 TIMES_ROMAN 这一英文字体进行渲染。根据对字体的更多需求，可对这一类进一步定制，搭配 Html 实现更美观的字体渲染。

## 4 PDF 生成代码

```java
  public static File html2Pdf(String html, String outputPath) {
    try {
      // step 1
      Document document = new Document(PageSize.A4);
      document.setMargins(20, 20, 0, 0);
      // step 2
      PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(path));
      // step 3
      document.open();
      // step 4
      InputStream cssInput = null;
      XMLWorkerHelper.getInstance().parseXHtml(writer, document, new ByteArrayInputStream(html.getBytes(StandardCharsets.UTF_8)), cssInput, new PdfFontProvider());
      // step 5
      document.close();
      LOG.info("PDF file: {} rendering successfully", outputPath);
      return new File(outputPath);
    } catch (IOException ex) {
      // do something
    } catch (DocumentException ex) {
      // do something
    }
  }

```

## 5 使用要点

- 必须引入 `itext-asian` 依赖以支持中文字体
- 待转换 Html 中的中文所在节点的属性一定要指定为 `style="font-family:STSong-Light"` 才能正常转换
- 对于中英文夹杂的部分，可以对个别文字使用 span 标签包裹并指定 `font-family` 以达到精确字体渲染的目的

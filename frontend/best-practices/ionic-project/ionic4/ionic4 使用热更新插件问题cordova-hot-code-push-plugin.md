# ionic4 使用热更新插件问题

> 依赖cordova-hot-code-push-plugin 热更新，在升级到ionic4之后都会遇到白屏的问题，经过调试代码，发现是由于cordova-plugin-ionic-webview升级导致的

cordova-plugin-ionic-webview升级到2.x以上会有如下影响

- iOS 最低兼容之10+
- android最低兼容 4.4+
- GCDWebServer 不支持带转义的字符，比如 **cordova-hot-code-push-plugin** 的存储位置是 **Application Support**, 在URL转义后变成  **Application%20Support** , 此时GCDWebServer无法找到该路径，所以会白屏
-  1.0时GCDWebServer的默认localserver 根目录是APP的根目录，而2.x之后改成了包中www目录，热更新后新的目录在 **Application Support**对应的www中，与localserver的根目录不符，所以此时也无法找到



针对以上基础改动，我们可以采取如下措施



## 1. iOS 打包target 改为10+

根据苹果官网的统计:<https://developer.apple.com/support/app-store/>

iOS 10 以上的版本占据了95%的市场份额，相对来说兼容10以下的性价比太低



## 2. android 兼容 至4.4+

根据安卓官方统计：<https://developer.android.com/about/dashboards?hl=zh-cn>

4.4以下的版本份额不足4%，所以低版本的也可以不给予考虑

## 3. 改动 cordova-hot-code-push-plugin的存储位置

将 **cordova-hot-code-push-plugin** 存储位置改为非 **Application Support** 的目录，比如cacheDirectory。此处只需要改动iOS对应的代码，如下：

在 **HCPFilesStructure.m**的 **pluginRootFolder**方法中做如下改动

```objective-c
  NSURL *supportDir = [fileManager applicationSupportDirectory];
```

改为

```objective-c
  NSURL *supportDir = [fileManager applicationCacheDirectory];
```



## 4. 在热更新插件启动完毕和热更新文件下载完毕后重设localserver根目录，并重启localserver

#### 安卓改动：

只需要改动 **redirectToLocalStorageIndexPage**方法，使用到了java的反射机制

```java
    private void redirectToLocalStorageIndexPage() {
        final String indexPage = getStartingPage();

        // remove query and fragment parameters from the index page path
        // TODO: cleanup this fragment
        String strippedIndexPage = indexPage;
        if (strippedIndexPage.contains("#") || strippedIndexPage.contains("?")) {
            int idx = strippedIndexPage.lastIndexOf("?");
            if (idx >= 0) {
                strippedIndexPage = strippedIndexPage.substring(0, idx);
            } else {
                idx = strippedIndexPage.lastIndexOf("#");
                strippedIndexPage = strippedIndexPage.substring(0, idx);
            }
        }

        // make sure, that index page exists
        String external = Paths.get(fileStructure.getWwwFolder(), strippedIndexPage);
        if (!new File(external).exists()) {
            Log.d("CHCP", "External starting page not found. Aborting page change.");
            return;
        }
        try {
            Log.d("CHCP", "begin restart app");
            String basePath = fileStructure.getWwwFolder();
            // 尝试重置本地服务器根目录为当前热更新后的外置存储路径
            Class[] cArg = new Class[1];
            cArg[0] = String.class;
            // 此处重置loacalserver的根目录
            webView.getEngine().getClass().getDeclaredMethod("setServerBasePath", cArg).invoke(webView.getEngine(),
                    basePath);
        } catch (NoSuchMethodException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (SecurityException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();

        }

        Log.d("CHCP", "Loading external page: " + external);
    }
```



#### iOS改动：

> iOS的改动有两处，一处是热更新拆件加载完毕后和热更新文件下载完成后都需要重定向localserver的根目录
>
> 为了与其他插件耦合，采用了objective-c的

添加如下方法：

```objective-c
/**
 * 切换本地服务根目录到外存储目录
 */
-(void) switchServerBaseToExternalPath{
    
    NSString * basePath = [_filesStructure.wwwFolder.absoluteString stringByReplacingOccurrencesOfString:@"file://" withString:@""];
    // 先要确保webVieEngine能响应以下两个方法
    if([self.webViewEngine respondsToSelector:@selector(setServerPath:)] && [self.webViewEngine respondsToSelector:@selector(basePath)]){
        // 先判断之前的本地服务根目录是否与将要切换的路径相同，如果不相同则切换，否则不切换
        NSString * preBasePath = [self.webViewEngine performSelector:@selector(basePath)];
        if( ![preBasePath isEqualToString:basePath] && [[NSFileManager defaultManager] fileExistsAtPath:basePath]){
            
            dispatch_async(dispatch_get_main_queue(), ^{
                // 此处需要在主线程中调用，否则会出现意想不到的错误
                [self.webViewEngine performSelector:@selector(setServerPath:) withObject:basePath];
            });
            
        }
        NSLog(@"reset the base server success, start reload app");
    }else{
        // 如果不能响应，则不需要再调用切换了，保持APP在未更新状态
        NSLog(@"cannot reset the base server, keep current page");
    }
}
```

并在改动**resetIndexPageToExternalStorage**的代码如下：

```objective-c
/**
 *  Redirect user to the index page that is located on the external storage.
 */
- (void)resetIndexPageToExternalStorage {
    NSString *indexPageStripped = [self indexPageFromConfigXml];
    
    NSRange r = [indexPageStripped rangeOfCharacterFromSet:[NSCharacterSet characterSetWithCharactersInString:@"?#"] options:0];
    if (r.location != NSNotFound) {
        indexPageStripped = [indexPageStripped substringWithRange:NSMakeRange(0, r.location)];
    }
    
    NSURL *indexPageExternalURL = [self appendWwwFolderPathToPath:indexPageStripped];
    if (![[NSFileManager defaultManager] fileExistsAtPath:indexPageExternalURL.path]) {
        return;
    }
    
    // rewrite starting page www folder path: should load from external storage
    if ([self.viewController isKindOfClass:[CDVViewController class]]) {
        // 在此处重置localserver
        [self switchServerBaseToExternalPath];
    } else {
        NSLog(@"HotCodePushError: Can't make starting page to be from external storage. Main controller should be of type CDVViewController.");
    }
}
```



fork了一份 **cordova-hot-code-push-plugin **的代码，并做了相应的改动，如果想用可以直接fork一下然后自己打npm的包，地址：<https://github.com/amosbaby/cordova-hot-code-push>



当然， 我也带了一个npm的包，名字叫做 **teh-hot-code-push-plugin**

安装方法：

```bash
cordova plugin add teh-hot-code-push-plugin	
```

其他与原来 插件没啥区别。亲测可用，如有问题，随时可以提issues或者评论。
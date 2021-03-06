AS 导入 Gallery2
1. 打开AS,选择Import Project(Eclipse ADT, Gradle, etc.)导入源码路径
2. 导入结束会看到`import-summary`文件,文件列出了导入的和未导入的文件列表.
3. 点击更新gradle,出现如下错误

  ```bash
  Error:(148, 32) error: Non-root compute kernel selectiveAdjust() is not supported in SDK levels 11-15
  ```

4. 在APP的Gradle中的`defaultConfig`下添加:

  ```bash
  renderscriptTargetApi 18
  ```

5. 继续更新Gradle,出现字串有重复定义错误:

  ```bash
  Error:Error: Found item String/no_storage more than one time
  ```

6. 我们先把__res/values/strings.xml__下的国际化字串删掉一部分,只留下中英文,减少修改工作量.
7. 然后在字串搜索`no_storage`,发现两个定义如下:

  ```xml
  <!-- alert to the user that USB storage must be available before using the camera [CHAR LIMIT=NONE] -->
      <string name="no_storage" product="nosdcard">Mount USB storage before using the camera.</string>
      <!-- alert to the user that an SD card must be installed before using the camera -->
      <string name="no_storage" product="default">Insert an SD card before using the camera.</string>
  ```
  看到字串有`product=`是对多个product的支持.为了快速解决问题,我们就在字串中搜索`product="nosdcard"`,把所有重复定义的都删掉.
8. 继续make project,报错

  ```bash
  Error:Error: The file name must end with .xml or .png
  ```
  资源里有一个没有类型的空文件__filtershow_state_button_background__,在代码里也搜不到使用它的地方,所以可以直接删掉.
9. make project,出现大量找不到包的错误.

  ```bash
  Error:(19, 36) error: package com.android.gallery3d.common does not exist
  ```
  这是因为导入的时候过滤掉了这些文件,现在我们要把他们从源码里加进来.你为了方便可以直接把文件拷贝到gallery3d下,但是我想梳理源码,所以打算保留原始的文件结构并且适应AS的目录结构,我新建了包.但代价是要把每个文件的import改掉.使用快捷键`ctrl+shift+r`全局替换
10. 还会遇到support库缺失,在Gradle添加

  ```gradle
      dependencies {
          compile 'com.android.support:support-v4:26.0.+'
          compile 'com.android.support:support-v13:26.0.+'
      }
  ```
11. 然后还缺少几个包

   ```bash
   Error:(19, 21) error: package com.adobe.xmp does not exist
   Error:(31, 25) error: package com.coremedia.iso does not exist
   Error:(33, 42) error: package com.googlecode.mp4parser.authoring does not exist
   ```
   把源码目录拷贝到项目java目录下

   ```bash
   /Oreo/external/xmp_toolkit/XMPCore/src/com
   /Oreo/external/mp4parser/isoparser/src/main/java/com
   /Oreo/external/apache-http/src/org
   ```
12. 剩下一些找不到包的小错误很好解决,主要看下最后的NDK错误.

   ```gradle
       externalNativeBuild {
           ndkBuild {
               path file("src/main/jni/Android.mk")
           }
       }
   ```

13. 运行安装时出现未知错误:

    ```Bash
    error Unknown failure (at android.os.Binder.execTransact(Binder.java:565)) Error while Installing APKs
    修改步骤:
    AndroidStudio
    File -> Settings -> Build, Execution, Deployment.
    Instant Run -> disable "Enable Instant Run to hot swap code/resource changes on deploy".
    Apply -> OK
    然后再运行,看到真正的错误:
    INSTALL_FAILED_NO_MATCHING_ABIS
    解决方案是: Build->Refresh linked C++ Projects
    ```


14. 如果导入项目报错：

    ```Bash
    A problem occurred configuring project ':app'.
    ```

    建议检查`File->Project Structure->SDK`的`NDK`的设置。

15. TBC

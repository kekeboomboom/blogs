对于安卓：

![image-20210831110150459](https://gitee.com/keke518/MarkDownPicture/raw/master/img/20210831110157.png)



> settings.gradle
>
> ```
> include ':app'
> rootProject.name = "wangke_android"
> ```

那么根项目是wangke_android，子项目是app

> 对于根项目的build.gradle：
>
> ```
> buildscript {
>     repositories {
>         google()
>         jcenter()
>     }
>     dependencies {
>         classpath "com.android.tools.build:gradle:4.1.1"
> 
>         // NOTE: Do not place your application dependencies here; they belong
>         // in the individual module build.gradle files
>     }
> }
> 
> allprojects {
>     repositories {
>         google()
>         jcenter()
>     }
> }
> 
> task clean(type: Delete) {
>     delete rootProject.buildDir
> }
> ```

所引用的库：google()，jcenter()，而jcenter在2021年声明将不能用，所以以后创建Android项目所用的库会改为mavencenter

有一个task，清理

> gradle.properties
>
> ```
> org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
> 
> android.useAndroidX=true
> 
> android.enableJetifier=true
> ```

配置gradle的jvm大小，配置安卓相关的是否用AndroidX等





> 子项目中的build.gradle
>
> 会有sdk版本，各种第三方依赖包


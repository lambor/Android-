# 自定义 Gradle 插件的
参考：http://unclechen.github.io/2015/11/17/%E8%87%AA%E5%AE%9A%E4%B9%89Android-Gradle%E6%8F%92%E4%BB%B6/
### 工程路径的建立
选择你的工程路径。`mkdir MyPlugin`,`cd MyPlugin`,`touch build.gradle`。

### 使用android studio打开你的工程
File -> Open -> your plugin project path  
然后选择YES,是为你的工程添加gradle wrapper。

### 建立工程结构
首先在 root directory下建一个src directory.然后根据下面的工程结构建立该工程：  
* src/main
* + groovy/your/package/name/(your groovy source)
* + resources/META-INF/gradle-plugins/(your_plugin_name.properties)

### 添加依赖（我没有添加好像也可以）
File -> Project Structure -> Modules -> Dependencies -> "plus button" -> select gradle lib path
我试了没有添加依赖也可以，好像build.gradle中compile gradleapi()就行了？

### 修改build.gradle内容
```
apply plugin: 'groovy'
apply plugin: 'maven'

//这三行是提交到本地maven库的配置，需要根据情况修改。此处的配置有关于使用时的引用。
version = '1.0.0'
group = 'com.dcnh35.gradle.plugin'
archivesBaseName = 'my-gradle-plugin'

repositories {
    mavenCentral()
}

dependencies {
    compile gradleApi() //难道是此处添加了依赖？
    compile localGroovy()
}

compileGroovy {
    sourceCompatibility = 1.7
    targetCompatibility = 1.7
    options.encoding = "UTF-8"
}

uploadArchives {
    repositories.mavenDeployer {
//        repository(url: "http://10.XXX.XXX.XXX:8080/nexus/content/repositories/releases/") {
//            authentication(userName: "admin", password: "admin123")
//        }
        repository(url: 'file:release/libs') //maven提交的位置为release/libs下
    }
}
```
修改完后，可以在android studio的右侧边栏gradle 中找到 upload/uploadArchives 任务。  
如果没有找到就点击左上角的refresh button（我在这里卡了很长时间。。。），经过点击refresh，你会发现工程中的groovy和resources等路径图标会修改。

### 完成gradle plugin的编写。
可根据参考编写。（参考中有一个类ProGuardTask一直找不到，不知道是不是已经过时了）
最后在META-INF/gradle-plugins/your_plugin_name.properties中添加
`implementation-class = your.package.name.YourPluginClass`

### 最后完事后就点击 uploadArchives 任务
任务完成后，会发现output中出现release/libs文件夹和文件夹下的maven库文件。

### 使用gradle plugin
讲上面所说的libs文件夹拷贝到需要的工程根目录下，然后在根目录的build.gradle中添加：
```
buildscript {
    repositories {
        ...
        maven {
            url 'libs'
        }
    }
    dependencies {
        ...
        classpath 'your.group.id:yourArchivesBaseName:your.plugin.version' 
        //比如 classpath 'com.dcnh35.gradle.plugin:my-gradle-plugin:1.0.0'
        //build.gradle 中的三行maven配置
        //version = '1.0.0'
        //archivesBaseName = 'my-gradle-plugins'
        //group = 'com.dcnh35.gradle.plugin'
    }
}
```

在需要使用该插件的 module 中的build.gradle中进行配置添加：  
`apply plugin: 'your_plugin_name'` //此处的plugin name是 META-INF/gradle-plugins/中properties文件的文件名所使用的。

如果你还允许用户配置自己的插件。你还需要提供相应的Extension类。详情见参考。  

在跟着教程摸索时，遇到了很多搞笑的事：  
比如我经常把META-INF写错成META-INFO。。。。。
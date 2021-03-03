# Gradle

**配置Gradle使用maven本地仓库**，这样Gradle就不会重新下载已经存在maven本地仓库的jar包，从而节省时间和空间。

　　在环境变量中加入新的系统变量：GRADLE_USER_HOME 变量值是maven本地仓库的路径，本文为例C:\Users\Administrator\.m2\repository

   此时，Gradle下载的文件将放到指定的仓库路径中。但是还需要修改build.gradle文件中加入mavenLocal() 引用本地仓库

　　

```
repositories { //repositories闭包
　　　　mavenLocal() //配置先从本地仓库寻找jar包，优先寻找上一个配置，找到不执行下面的配置
　　　　mavenCentral() //配置从中央仓库寻找
　　　　google() //第三方仓库
　　　　jcenter() //代码托管库：设置之后可以在项目中轻松引用jcenter上的开源项目
　　}
```


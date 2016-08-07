参考文档:gradle的官方userguide.pdf文档的chapter 55和chapter 56.
gradle的多模块或项目开发一定不会比maven差,在我看来!大的项目分成多个模块来开发是常事.下文就介绍一下怎么用gradle开发多模块项目.对于gradle,在Eclipse和IDEA开者之间,毫无疑问选择IDEA作为IDE.
testweb是一个简单例子,项目只分成了core和web两个模块.其中core模块是放一些基本的或公共的java类,web模块放的是web Controller,配置,页面.所以最终打包项目时,core应打成一个jar包,而web模块引用(依赖)core模块,对于web的java类也打起一个jar包,这两个jar包最后是放在lib包下面再打成war包.项目的主要结构如下:
testweb
  core
    src
      main
        java
      test
        java
        resources
    build.gradle
  web
    src
      main
        java
        resources
      test
        java
    build.gradle
  build.gradle
  settings.gradle
core主要使用spring+spring data jpa(hibernate实现)+mysql


一.根据上面的项目结构,新建必要的目录和文件.
1.settings.gradle.只有一个模块来说,此文件是可选的.对于多模块,此文件是必须的.
[plain] view plain copy 在CODE上查看代码片派生到我的代码片
include 'core','web'  

2.这里将core和web模块的gradle配置也放到了顶层的build.gradle
build.gradle

[java] view plain copy 在CODE上查看代码片派生到我的代码片
allprojects {  
    apply plugin: 'java'  
    group = 'org.exam'  
    version = '1.0'  
    sourceCompatibility = 1.8  
    targetCompatibility = 1.8  
}  
subprojects {  
    ext {  
        slf4jVersion = '1.7.7'  
        springVersion = '4.2.1.RELEASE'  
        hibernateVersion = '4.3.1.Final'  
    }  
    [compileJava, compileTestJava, javadoc]*.options*.encoding = 'UTF-8'  
    repositories {  
        mavenCentral()  
    }  
    configurations {  
        //compile.exclude module: 'commons-logging'  
        all*.exclude module: 'commons-logging'  
    }  
    dependencies {  
        compile(  
                "org.slf4j:jcl-over-slf4j:${slf4jVersion}",  
                "org.slf4j:slf4j-log4j12:${slf4jVersion}",  
                "org.springframework:spring-context:$springVersion",  
                "org.springframework:spring-orm:$springVersion",  
                "org.springframework:spring-tx:$springVersion",  
                "org.springframework.data:spring-data-jpa:1.5.2.RELEASE",  
                "org.hibernate:hibernate-entitymanager:$hibernateVersion",  
                "c3p0:c3p0:0.9.1.2",  
                "mysql:mysql-connector-java:5.1.26",  
                "commons-fileupload:commons-fileupload:1.3.1",  
                "com.fasterxml.jackson.core:jackson-databind:2.3.1"  
        )  
        testCompile(  
                "org.springframework:spring-test:$springVersion",  
                "junit:junit:4.11"  
        )  
    }  
}  
project(':core') {  
  
}  
project(':web') {  
    apply plugin: "war"  
    dependencies {  
        compile project(":core")  
        compile(  
                "org.springframework:spring-webmvc:$springVersion",  
                "org.apache.taglibs:taglibs-standard-impl:1.2.1"  
        )  
        providedCompile(  
                "javax.servlet:javax.servlet-api:3.1.0",  
                "javax.servlet.jsp:jsp-api:2.2.1-b03",  
                "javax.servlet.jsp.jstl:javax.servlet.jsp.jstl-api:1.2.1"  
        )  
    }  
    processResources{  
        /* 从'$projectDir/src/main/product'目录下复制文件到'WEB-INF/classes'目录下覆盖原有同名文件*/  
        from("$projectDir/src/main/product")  
    }  
  
    /*自定义任务用于将当前子项目的java类打成jar包,此jar包不包含resources下的文件*/  
    def jarArchiveName="${project.name}-${version}.jar"  
    task jarWithoutResources(type: Jar) {  
        from sourceSets.main.output.classesDir  
        archiveName jarArchiveName  
    }  
  
    /*重写war任务:*/  
    war {  
        dependsOn jarWithoutResources  
        /* classpath排除sourceSets.main.output.classesDir目录,加入jarWithoutResources打出来的jar包 */  
        classpath = classpath.minus(files(sourceSets.main.output.classesDir)).plus(files("$buildDir/$libsDirName/$jarArchiveName"))  
    }  
    /*打印编译运行类路径*/  
    task jarPath << {  
        println configurations.compile.asPath  
    }  
}  
  
/*从子项目拷贝War任务生成的压缩包到根项目的build/explodedDist目录*/  
task explodedDist(type: Copy) {  
    into "$buildDir/explodedDist"  
    subprojects {  
        from tasks.withType(War)  
    }  
}  

此项目包括core和web两个模块,其中core为普通java模块,web为web模块,并且web依赖core.


a.打包web时,会先将web\src\main\resources下的文件复制到web\build\resources\main目录,然后复制web\src\main\product下的文件到web\build\resources\main目录来覆盖同名文件.
b.编译web\src\main\java下的java文件到web\build\classes\main目录,然后将web\build\classes\main的文件打成jar包.
c.将所需依赖,包括core-${version}.jar和web-${version}.jar复制到war包下的\WEB-INF\lib目录.将web\src\main\product下的文件复制到WEB-INF\classes目录

二.将项目导入IDEA去开发
三.测试.
1.先测试core模块.主要参考core\src\test\java\com\exam\repository\UserRepositoryTest.java.
2.再测试web模块.
a.先用junit测试controller.主要参考web\src\test\java\com\exam\web\UserControllerTest.java,
b.参考<<配置简单的嵌入式jetty>>测试(可选)
c.打再成war包,部署到tomcat或jetty测试.


源码:http://download.csdn.net/detail/xiejx618/7736387

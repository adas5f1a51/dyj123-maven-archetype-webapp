# 自定义 Maven 骨架

## 1. 概述
Maven骨架(Maven Archetype) 官方定义：<br>
In short, Archetype is a Maven project templating toolkit. 
An archetype is defined as an original pattern or
model from which all other things of the same kind are made.
The name fits as we are trying to provide a system that 
provides a consistent means of generating Maven projects. 
Archetype will help authors create Maven project templates 
for users, and provides users with the means to generate 
parameterized versions of those project templates.<br>
骨架（又叫原型），没用过的估计不会来看这个，
用过的都知道这是什么东西了我就不说了。

## 2. 问题背景
我个人感觉原Maven的Web项目骨架不是很好，web.xml的文件头还是2.3版本的。
所以自己突发奇想自己写骨架了。

## 3. 创建自定义 Maven 骨架工程结构与配置文件
我以我自己写的web项目骨架为例介绍编写方式。
### 3.1 创建一个新的 Maven 项目
打开 idea，选择创建新项目，选择 Maven项目，直接创建就行。
不用使用骨架来创建，因为我们要自己写骨架。
### 3.2 自定义骨架项目结构
我是写的web项目的骨架，
项目的结构如下（可以根据此结构结合自己的需求自行扩展）：
```
root
├── src // 此处是这个maven项目的根目录
│   └── main
│       └── resources
│           ├── archetype-resources
│           │   ├── src // 此处是自定义骨架的根目录
│           │   │   ├── main
│           │   │   │   ├── java
│           │   │   │   ├── resources
│           │   │   │   └── webapp
│           │   │   │       └── WEB-INF
│           │   │   │           └── web.xml
│           │   │   └── test
│           │   │       ├── java
│           │   │       └── resource
│           │   └── pom.xml // 此处是自定义骨架的pom.xml
│           └── META-INF
│               └── maven
│                   └── archetype-metadata.xml
└── pom.xml // 此处是这个maven项目的pom.xml
```
archetype-resources目录是骨架的主目录，
我们使用此骨架生成的项目结构就基于此目录结构。<br>
### 3.3 配置文件
#### 3.3.1 骨架根目录/pom.xml
这个是骨架的pom.xml，我们使用此骨架生成的项目的pom.xml会使用此pom.xml。
内容大致如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 此处是pom.xml的头信息，每个都一样 -->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 这里的值是使用此骨架创建项目时，根据填写的信息进行自动替换 -->
    <!-- 所以使用占位符 -->
    <groupId>${groupId}</groupId>
    <artifactId>${artifactId}</artifactId>
    <version>${version}</version>
    <!-- 打包方式，我写的是web项目所以打包方式为war -->
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- 此处写此骨架会用到的依赖 -->
        <!-- 填写的依赖会在使用此骨架创建项目时自动导入 -->
        <!-- 我经常使用SpringMVC写web项目，所以这里我写SpringMVC的依赖 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.4</version>
        </dependency>
    </dependencies>

</project>
```
#### 3.3.2 archetype-metadata.xml
官方叫它骨架描述器(Archetype descriptor)，
此配置文件是用来描述骨架内的目录与文件信息，这个文件必须在
src/main/resources/META-INF/maven/目录下并且必须叫archetype-metadata.xml
这个名，不能改名。<br>
内容大致如下：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 骨架描述器的头信息，我这边报红也没问题 -->
<archetype-descriptor 
        xmlns="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/plugins/maven-archetype-plugin/archetype-descriptor/1.1.0 https://maven.apache.org/xsd/archetype-descriptor-1.1.0.xsd"
        name="dyj123-maven-archetype-webapp">
    <!-- 上面的name填骨架的名称，不写也没关系 -->
    
    <!-- 描述骨架的目录结构，在此描述的目录在用骨架创建项目时会自动生成-->
    <fileSets>
        <!-- 描述骨架的每一个目录 -->
        <!-- filtered：如果此目录及其子目录下include的文件中有maven占位符时，根据创建时填写的信息自动替换 -->
        <!-- packaged：在此目录自动添加和groupId一样的包 -->
        <fileSet filtered="true" packaged="true">
            <!-- 需要生成的目录或包 -->
            <!-- 此处的src指骨架目录的src，不是此maven项目的src -->
            <directory>src/main/java</directory>
        </fileSet>
        <!-- encoding：include内的文件的编码格式 -->
        <fileSet filtered="true" encoding="UTF-8">
            <directory>src/main/resources</directory>
        </fileSet>
        <fileSet filtered="true" encoding="UTF-8">
            <directory>src/main/webapp/WEB-INF</directory>
            <includes>
                <!-- 此处可使用通配符**/*.**包括所有文件 -->
                <include>web.xml</include>
            </includes>
        </fileSet>
        <fileSet filtered="true">
            <directory>src/test/java</directory>
        </fileSet>
        <fileSet filtered="true">
            <directory>src/test/resources</directory>
        </fileSet>
    </fileSets>
    
</archetype-descriptor>
```

### 3.4 安装我们写好的 Maven 骨架
在我们写好的 maven 项目下运行```mvn install```就可以把生成的骨架
安装到我们的本地 maven 仓库了。

## 4. 在idea中使用生成的骨架
打开 idea，创建 maven 工程，使用骨架创建，选择添加新骨架：

* GroupId：填我们写自定义骨架工程的maven项目的GroupId；
* ArtifactId：填我们写自定义骨架工程的maven项目的ArtifactId；
* Version：填我们写自定义骨架工程的maven项目的Version。

添加我们写的骨架之后，使用我们自定义骨架创建项目，看看效果如何。

## 5. 关于如何删除创建maven项目的骨架列表里面我们自己添加的骨架
C:\Users\${USER}\AppData\Local\JetBrains\IntelliJIdea2021.1\Maven\Indices\UserArchetypes.xml
此文件是保存我们自行添加的骨架列表，删除对应记录即可。
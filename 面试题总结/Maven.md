## Maven

---

### 1. Maven坐标

==参考答案==

一般maven使用 `[groupID, artifactId, version, packaging]` 来表示一个项目的某个版本，有时还会使用`classifier`来表示项目的附属构建，常见的附属构建有 javadoc 和 sources 包。



### 2. Maven 是如何寻找依赖的？

==参考答案==

首先会去**本地仓库**寻找，然后会去公司的**私服仓库**寻找，一般私服仓库存的都是公司自己开发的 jar 包，最后会去 由 Apache 团队来维护**中央仓库**寻找，一旦在一个地方找到就不再寻找。



### 3. Maven 常见的依赖范围 <scope>  有哪些?

==参考答案==

1）**compile**: 编译依赖，默认的依赖方式，在编译，测试，运行三个阶段都有效，典型地有 spring-core 等 jar。

2）**test**: 测试依赖，只在编译测试用例和运行测试用例有效，典型地有 JUnit。

3）**provided**: 对于编译和测试有效，不会打包进发布包中，典型的例子为 servlet-api，一般的 web 工程运行时都使用容器的 servlet-api。

4）**runtime**: 只在运行测试用例和实际运行时有效，典型地是 jdbc 驱动 jar 包。

5）**system**: 不从 maven 仓库获取该 jar ，而是通过 systemPath 指定该 jar 的路径。

6）**import**: `import`只能用在`dependencyManagement`块中，它将`spring-*-dependencies` 中`dependencyManagement`下的`dependencies`插入到当前工程的`dependencyManagement`中，所以不存在依赖传递。当没有`<scope>import</scope>`时，意思是将`spring-boot-dependencies` 的`dependencies`全部插入到当前工程的`dependencies`中，并且会依赖传递。



### 4. Maven的生命周期

==参考答案==

maven有三套生命周期，分别为：

- **clean 周期**：主要用于清理上一次构建产生的文件，可以理解为删除target目录

- **默认周期**：主要阶段包含:

  1）**process-resources** 默认处理 src/test/resources/ 下的文件，将其输出到测试的classpath目录中

  2）**compile** 编译 src/main/java下的java文件，产生对应的 class

  3）**process-test-resources** 默认处理src/test/resources/下的文件，将其输出到测试的classpath目录中

  4）**test-compile** 编译src/test/java下的java文件，产生对应的class

  5）**test** 运行测试用例

  6）**package** 打包构件，即生成对应的jar，war等

  7）**install** 将构件部署到本地仓库

  8）**deploy** 部署构件到远程仓库

- **site 周期**：主要阶段包含：

  1）**site** 产生项目的站点文档

  2）**site-deploy** 将项目的站点文档部署到服务器



### 5. 我们经常使用 `Mvn Clean Package` 命令进行项目打包，请问该命令执行了哪些动作来完成该任务？

==参考答案==

在这个命令中我们调用了 maven 的 clean 周期的 clean 阶段绑定的插件任务，以及 default 周期的 package 阶段绑定的插件任务。默认执行的任务有（maven 的术语叫 goal, 也有人翻译成目标，我这里用任务）：

```xml-dtd
maven-clean-plugin:clean->
maven-resources-plugin:resources->
maven-compile-plugin:compile->
mavne-resources-plugin:testResources->
maven-compile-plugin:testCompile->
maven-jar-plugin:jar
```



### 6. 依赖的解析机制

==参考答案==

- **解析 `RELEASE` 版本**：如果本地有，直接使用本地的，没有就向远程仓库请求。

- **解析 `SNAPSHOT` 版本**：合并本地和远程仓库的元数据文件 -- groupId / artifactId / version / maven-metadata.xml，这个文件存的版本是带时间戳的，将最新的一个改名为不带时间戳的格式供本次编译使用。



### 7. 插件的解析机制

==参考答案==

当我们输入`mvn dependency:tree`这样的指令，解析的步骤为：

 1）**解析 groupId**: maven 使用默认的 groupID:`org.apache.maven.plugins` 或者 `org.codehaus.mojo`

 2）**解析 artifactId** (maven的官方叫做插件前缀解析策略)

 3）**合并该 groupId 在所有仓库中的元数据库文件（maven-metadata-repository.xml）**，比如maven官方插件的元数据文件所在的目录为 orgapachemavenplugins ，该文件下有如下的条目

```xml
<plugin>
    <name>MavenDependencyPlugin</name>
    <prefix>dependency</prefix>
    <artifactId>maven-dependency-plugin</artifactId>
</plugin>
```


通过比较这样的条目，我们就将该命令的artifactId解析为maven-dependency-plugin

 4）**解析version**：如果你在项目的pom中声明了该插件的版本，那么直接使用该版本的插件，否则合并所有仓库中groupId/artifactId/maven-metadata-repository.xml，找到最新的发布版本。

对于非官方的插件，有如下两个方法可以选择：

1）使用 groupId:artifactId:version:goal 来运行

2）在Settings.xml中添加pluginGroup项，这样maven不能在官方的插件库中解析到某个插件，那么就可以去你配置的group下查找啦。



### 8. 模块的聚合与继承 

==参考答案==

#### 8.1 了解模块的聚合吗?

##### 8.1.1 聚合工程的作用是什么？

聚合工程主要是用来管理项目，聚合工程管理的项目在进行运行的时候，会按照项目与项目之间的依赖关系来自动决定执行的顺序和配置的顺序无关。



##### 8.1.2 模块是如何聚合的？

配置一个打包类型为 pom 的聚合模块，然后在该 pom 中使用 <module> 元素声明要聚合的模块



#### 8.2 了解模块的继承吗？

与java中的继承相似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承。从而达到**简化配置**和**减少版本冲突**。

要实现模块的继承，首先要将父模块打包为 pom 工程，然后在子模块中通过 <parent> 继承父模块的配置信息。

```xml
<!--配置当前工程继承自parent工程-->
<parent>
    <groupId>com.itheima</groupId>
    <artifactId>maven_01_parent</artifactId>
    <version>1.0-RELEASE</version>
    <!--设置父项目pom.xml位置路径-->
    <relativePath>../maven_01_parent/pom.xml</relativePath>
</parent>
```

### 8.3 聚合与继承的不同点和相同点

两种之间的作用:

* 聚合用于快速构建项目，对项目进行管理
* 继承用于快速配置和管理子项目中所使用的依赖以及依赖的的版本号

聚合和继承的相同点:

* 聚合与继承的 pom.xml 文件打包方式均为 pom ，可以将两种关系制作到同一个 pom 文件中
* 聚合与继承均属于设计型模块，并无实际的模块内容

聚合和继承的不同点:

* 聚合是在当前模块中配置关系，聚合可以感知到参与聚合的模块有哪些
* 继承是在子模块中配置关系，父模块无法感知哪些子模块继承了自己

### 9. dependencyManagement 依赖

==参考答案== 

#### 9.1 dependencyManagement 的作用是什么？

* <dependencyManagement> 标签不真正引入 jar 包，只是管理 jar 包的版本
* 子项目在引入的时候，只需要指定 <groupId> 和 <artifactId>，不需要加 <version>
* 当 <dependencyManagement> 标签中 jar 包版本发生变化，所有子项目中有用到该 jar 包的地方对应的版本会自动随之更新


#### 9.2 dependencyManagement 与 dependencies 的区别：

- **<dependencies>** 相对于 <dependencyManagement>，所有生命在 <dependencies> 里的依赖都会自动引入，并默认被所有的子项目继承。

- **<dependencyManagement>** 里只是声明依赖，并不自动实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且<version> 和 <scope> 都读取自父 pom；另外如果子项目中指定了版本号，那么会使用子项目中指定的 jar 版本。



### 10. 对于一个多模块项目，如何管理项目依赖的版本?

==参考答案==

通过在父模块中声明 <dependencyManagement> 和 <pluginManagement>，然后让子模块通过 <parent> 元素指定父模块，这样子模块在定义依赖是就可以只定义 <groupId> 和 <artifactId>，自动使用父模块的 <version>，这样统一整个项目的依赖的版本。



### 11. 一个项目的依赖来源于不同的组织，可能这些依赖还会依赖别的Jar包，如何保证这些传递依赖不会引起版本冲突？

==参考答案==

使用<dependency>的 <exclusion> 元素将会引起冲突的元素排除。



### 12. 常见的 Maven 私服的仓库类型有哪些？

==参考答案==

- （宿主仓库）hosted repository 

- （代理仓库）proxy repository

- （仓库组）group repository

### 13. 如何查询一个插件有哪些目标（Goal）？

==参考答案==

```shell
mvn help:describe -Dplugin=groupId:artifactId
```



### 14. Maven 是什么?

==参考答案==

答：Maven 主要服务于基于Java平台的**项目构建**、**依赖管理**和**项目信息管理**。Maven 的主要功能主要分为5点：**依赖管理**；**多模块构建**；**项目结构一致性**；**构建模型和插件机制的一致性**。



### 15. Maven仓库是什么？

==参考答案==

Maven仓库是基于简单文件系统存储的，集中化管理Java API资源（构件）的一个服务。仓库中的任何一个构件都有其唯一的坐标，根据这个坐标可以定义其在仓库中的唯一存储路径。得益于 Maven 的坐标机制，任何 Maven项目使用任何一个构件的方式都是完全相同的，Maven 可以在某个位置统一存储所有的 Maven 项目共享的构件，这个统一的位置就是仓库，项目构建完毕后生成的构件也可以安装或者部署到仓库中，供其它项目使用。
对于Maven来说，仓库分为两类：本地仓库和远程仓库。



### 16. Maven的工程类型有哪些？

==参考答案==

1）**POM工程**
POM工程是逻辑工程，通常是一个不具有业务功能的“空”工程（有且仅有一个pom文件）。标识该项目为聚合或继承项目

2）**JAR工程**
将会打包成jar用作jar包使用。即常见的本地工程 - Java Project。

3）**WAR工程**
将会打包成war，发布在服务器上的工程。如网站或服务。即常见的网络工程 - Dynamic Web Project。war工程默认没有WEB-INF目录及web.xml配置文件，IDE通常会显示工程错误，提供完整工程结构可以解决。



### 17. Maven常用命令有哪些？

==参考答案==

1）**install**
本地安装， 包含编译，打包，安装到本地仓库
编译 - javac
打包 - jar， 将java代码打包为jar文件
安装到本地仓库 - 将打包的jar文件，保存到本地仓库目录中。

2）**clean**
清除已编译信息。
删除工程中的target目录。

3）**compile**
只编译。javac命令

4）**deploy**
部署。常见于结合私服使用的命令。
相当于是install+上传jar到私服。
包含编译，打包，安装到本地仓库，上传到私服仓库。
5）**package**
打包。包含编译，打包两个功能。



### 18. 你们项目为什么选用maven进行构建？

==参考答案==

①首先，maven是一个优秀的项目构建工具。使用maven，可以很方便的对项目进行分模块构建，这样在开发和测试打包部署时，效率会提高很多。

②其次，maven可以进行依赖的管理。使用maven，可以将不同系统的依赖进行统一管理，并且可以进行依赖之间的传递和继承。



### 19. maven的依赖原则有什么?

==参考答案==

- **依赖路径最短优先原则**：一个项目 Demo 依赖了两个 jar 包，其中A-B-C-X(1.0) ， A-D-X(2.0)。由于X(2.0)路径最短，所以项目使用的是X(2.0)。
- **pom文件中声明顺序优先**：如果A-B-X(1.0) ，A-C-X(2.0) 这样的路径长度一样怎么办呢?这样的情况下，maven 会根据 pom 文件声明的顺序加载，如果先声明了B，后声明了C，那就最后的依赖就会是X(1.0)。
- **覆写优先原则**：当同级配置了相同资源的不同版本，后配置的覆盖先配置的。

### 
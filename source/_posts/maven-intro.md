title: About Maven
date: 2016-09-09 12:30:00
tags: [maven]
excerpt: 早期的 Maven 分享记录

---

## 序

> 在程序员的世界里， 有一个有驳常理的现象，优秀的程序员都是既懒又傻的。
>
> 因为懒，他才会写出各种各样的工具来替自己干活。
>
> 因为懒，他才会想办法避免去写无聊重复的代码——因此避免的代码的冗余，消减了维护的成本，使重构变得容易。
>
> 最终，这些由于懒惰激发出的动力而开发出的工具和最佳编程实践方法提升了产品的质量。

Maven 的诞生自然也难以摆脱这个既定的事实 - 只是因为程序员很懒 ╮(╯▽╰)╭, 还爱瞎折腾。

![title_pic](http://h.img.siblings.top/menu-restaurant-vintage-tab.jpg)

<!-- more -->

### Question?

Maven 是什么？ 

>  一种工具。 

工具又是用来干嘛的呢？

>  用来解决目前大家遇到的一些共性的、复杂的、耗时的问题。

任何一种工具的出现都是有原因的, 那么, Maven 又是用来解决哪些问题的呢？

## 过去

### 目录结构

在过去，我们新建项目时，需要添加项目的源码目录 `src`，包目录 `lib`，资源目录 `WebRoot/WebContent/web` 等等.

![web 源码目录](http://h.img.siblings.top/blog_maven_intro_web_src_dir.png)

我们可以发现， 不同项目的目录结构，各有各的风格。

这就会让我们在接触一个新的项目时，感到困惑。也不利于初学者的入门。

> 不同的源码目录， 不同的 web 资源目录，散落的 jar ，都会在项目构建初期带来比较大的麻烦

### `jar` 依赖

目录结构搭建完成后，我们往往还需要去不同的网站上下载项目所需要依赖的各种 jar 包。

>  `spring-2.5.6.jar`、`junit-4.9.jar` etc..
>
>  找 jar 包这个过程很痛苦， 做过的人都应该有所了解。 

在这个过程中， 我们需要找到合适的版本比如 `aspectjrt-1.5.1.jar`， 合适的依赖包 `aspectjweaver-1.5.1.jar`。
多个 `jar` 包之间版本要兼容等等相关的问题。比如 `spring-2.5.6.jar` 需要依赖 `slf4j-1.3.jar`， 而 `MyBatis` 可能需要依赖 `slf4j-1.5.jar`。这个时候还需要解决重复依赖的问题， 选择一个更兼容的版本。

jar 包找齐之后， 我们还需要把对于 `jar` 包的依赖，添加到项目的类路径依赖配置中「eg. `.classpath/.idea`」，不同的 `IDE` 下对于类路径的依赖也是不同的

> 比较有经验的工程师可能都会维护一个自己的 `jar` 库， 当然， `jar` 包的版本比较老也是难免的。
>
> 如果有必要，我们往往还需要下载项目的源码包，并导入到 `IDE` 环境中，`IDE` 源码配置也是相当烦人的一个操作

总而言之，解决 `jar` 包的依赖，是传统的项目开发中一项不容忽视的工作。



### 源码

在过去的开发过程中，如果我们想要查看某个依赖包的源码， 首先我们要去网上找到它的源码包，下载到本地， 然后在 `IDE` 环境中把源码包附加到项目中。

这个时候如果在另一个开发人员的机器上，我们希望查看这个依赖包的源码，只能去重复这个步骤。

这个流程是很费劲， 也很冗余的。



### 多项目管理

不知道大家过去有没有开发过存在多个项目依赖的 `Web` 应用。

假如， 现在有这样一个场景， 公司有多个项目组， 其中有架构组（F） 负责缓存 日志等公用服务，基础资源组（B） - 系统公用资源层的开发 负责用户 角色 资源的基础服务， `Web` 应用开发组（W）- 负责最终的商业项目开发， 还有负责不同业务线的开发小组。

现在， 我们的系统架构是这样:
![maven_multi_project_framework](http://h.img.siblings.top/maven_multi_project_framework_160917.png)

我们的 `Web` 应用 A， 假设目前的  `Release` 版本号是 2.0.   它所依赖的架构组件， 版本号是 0.1. 
现在因为发现了一个安全问题， 版本号要更新到最新的  0.2.

传统的步骤：
	1. 架构组打包项目， framework-0.2.jar
	2. 把打包好的组件发给其他项目组， 并通知其他项目组， 需要替换组件
	3. 所有涉及到的项目，备份 framework-0.1.jar , 升级到 framework-0.2.jar ， 如果 framework-0.2.jar 有了新的依赖 比如 aspect.jar， 那么我们的项目中也需要导入它所依赖的 jar 包
	4. 测试 - 如果组件 A 存在问题， 重复 1 - 4 步
	5. 成功发布项目

我们可以发现在项目依赖的场景中， 如果有底层依赖项目发生改变，人工参与成本很大。

那么，把其中的一些人工参与的部分抽象出来，就是 Maven 作为项目管理工具来处理的事情了。比如降低不同项目组之间的耦合，自动化构建项目组件（jar 包）等等

### VersionControl

在最终的版本库「`svn/git`」中，除了项目必要的源码，可能还会包含各种 jar 包以及 classpath 之类相关 IDE 配置。

> jar 包都可以当做是现有类库，理论上没有必要出现在项目之中
>
> 而 IDE 相关配置作为开发环境自身的配置，更不应该作为项目的一部分被版本管理工具管理起来

### Summary

目前来看，传统的项目开发，至少有以下几个不太好的地方：

1. 项目目录结构的生成，没有规范
2. 和 jar 包相关的管理，依赖问题
3. 源码查看比较费力
4. 多项目开发时，它们的耦合性，以及过多人工成本的损耗
5. 在版本库中包含了过多与项目自身无关的内容

## 现在

那么，Maven 是又通过哪些方式解决了这些烦人的问题呢？

### 目录结构

Maven 使用惯例优于配置的原则。

它要求在没有定制之前，所有的项目都有如下的结构 ( 以 web 项目为例 )：  

![maven_simple_web_structure](http://h.img.siblings.top/maven_simple_web_structure.png)



| 目录                            | 目的                              |
| :---------------------------- | ------------------------------- |
| ${basedir}                    | 存放 pom.xml和所有的子目录               |
| ${basedir}/src/main/java      | 项目的 java源代码                     |
| ${basedir}/src/main/resources | 项目资源，如各种配置文件                    |
| ${basedir}/src/test/java      | 项目的测试类，比如说 JUnit代码              |
| ${basedir}/src/test/resources | 测试使用的资源                         |
| ${basedir}/src/main/webapp    | 存放所有的 Web 资源, 比如 WEB-INF, 静态资源等 |

> 其中， ${basedir} : 代表项目根目录

Maven 为什么要使用惯例优于配置的原则？

> 一方面是为了统一项目结构，降低大家看到新项目时的学习成本
>
> 另一方面也方便初学者可以直接使用现成的项目结构



### jar 依赖

Maven 提供了一种方式， 让每一个组件都可以拥有自己的 location 坐标。
groupId + artifactId + version + [type]。
eg.  `org.springframework.spring-mvc-3.2.2.jar`

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-mvc</artifactId>
    <version>3.2.2</version>
  	<scope>jar</scope>
</dependency>
```
通过在项目中配置所需依赖包的 location， 我们就可以让项目自动依赖所需的 jar 包。
jar 包之间所有的依赖关系， Maven 会自动进行处理。当然我们可以通过手工修改配置的方式来制定不同的依赖版本， 但大多数情况下是不必要的。 



### 源码

Maven 对于已经把源码提交的仓库的源码，可以在你需要查看源码的时候，自动下载到本地环境，同时保证源码的版本版本与当前 jar 包一致。 而且 目前的 IDE 中都已经完美集成了这个功能。使用起来简直方便的没法说!

> 此处应有 Demo

### 多项目管理

在公司内部多项目依赖的场景下，Maven 的处理方式会显得优雅许多：

1. 修改项目版本号， deploy 到 maven 私有仓库（存放 jar 包的地方）
2. 邮件通知到相关项目组， 更新版本号
3. 涉及到的项目更新组件版本号
4. 测试 - 如果组件出现问题， 重复执行  deploy，通知相关人员，测试 即可， 不需要不断的发包给别人 也无需解决 jar 依赖的问题
5. 发布项目


### VersionControl

由于 jar 包全部通过配置文件 `pom.xml` 来控制，包括整个项目的结构，我们真正可以做到 `IDE` 与源码完全分离。所在我们不需要再提交 `IDE` 相关配置 & jar 包到版本管理工具中。只需要提交项目源码 & 一个 Maven 的核心配置文件 即可。



### Summary

综上所述，Maven 帮助我们处理了很多现有问题：

1. 项目的目录结构有了固定规范，便于理解与管理
2. 通过一个集中的仓库，管理所有 jar 包，也解决了 jar 包最令人头疼的依赖问题
3. 一键查看源码
4. 多项目开发时，降低了人工参与的成本
5. 版本库中只需要包含一个源码 & 一个 `xml` 配置文件即可

## 实战

###  Maven 安装

`brew install maven`

> `Note`: 安装完成后，如果找不到 `settings.xml` 请参考 `settings.xml` 小节的内容

![maven_install_demo](http://h.img.siblings.top/maven_intro_install_190919.gif)

> 在真实的开发过程中， `IDE` 已经为我们准备了足够好用的初始化环境。所以，本地环境不安装也不会影响我们的使用。
>
> 不过，`IDE` 通常会内置比较新的 `maven` 版本，但不一定稳定，而不同版本之间的 `maven` 在构建的使用过程中， 可能也会出现一些难以理解的问题。 怎么抉择就`仁者见仁智者见智`了。

### `~/.m2/` 目录

为了演示效果。我需要初始化一下 `~/.m2/` 目录。默认安装后，是不会自动创建此目录的。

但在真实的开发过程中，我们无需关心此目录的创建，`Maven` 会在第一次使用时自动处理。 

 `mvn help:system`

![mvn help:system](http://h.img.siblings.top/maven_intro_init_m2_dir_160919.gif)

> 第一次执行，可能需要下载一些 goal 插件。 默认会从中央仓库下载， 但是中央仓库的速度很慢，最好提前配置好自己的私有仓库。

命令执行完成后，用户目录下会自动创建一个 `.m2` 的隐藏文件夹。该文件夹下放置了 `Maven` 本地仓库  `.m2/repository`。 所有在本地被使用的 `artifact` 「即 jar 组件」都会被下载并存储到该仓库中， 方便重用。当然，这个位置也是可以更改的。

> 默认情况下， `~/.m2/ ` 目录下除了 `repository` 仓库之外就没有其他目录和文件了，不过大多数 `Maven` 用户需要复制`$M2_HOME/conf/settings.xml` 文件到 `~/.m2/settings.xml`。
>
> 这是一条最佳实践。

### setting.xml

`settings.xml` 是 maven 默认的配置文件，默认在 `$M2_HOME/conf`  的目录下 「这里  `$M2_HOME` 表示 `Maven` 的安装目录」。修改这个文件，可以在机器全局范围内定制 `Maven` 的行为。

> `Note`：如果通过 `brew` 安装的 `Maven` 文件 `settings.xml` 位置会有些不同。请在目录 `/usr/local/Cellar/maven/${version}/libexec/conf` 下查看。

一般情况下，我们更偏向于复制该文件至 `~/.m2/` 目录下「这里 `~` 表示用户目录」。修改该文件，在用户范围定制 `Maven` 的行为。

![simple_settings.xml](http://h.img.siblings.top/maven_intro_settings.xml.png)

> `<server>` 指定了我们私有仓库的用户名和密码，可以用来上传我们自己的 jar 组件到自己的仓库中
> `<repository>` 指定了我们的私有仓库的在网络上的位置，用于存储网络上已发布到中央仓库的 jar 包和我们自己发布的组件

除了上图中的配置项，我们还可以配置 `mirror` - 用于代替默认的中央仓库、 配置 `localRepository` - 用于指定本地仓库的位置，不使用默认的 `~/.m2/repository/`、配置 `profile` - 用于指定不同的运行环境。

比较常用的个人配置项基本只有这几个，还有很多其他的配置项, 在具体的使用过程中, 大家可以自行查阅相关资料, 比如：[Maven 的 settings.xml 配置文件](http://itfish.net/article/57514.html)。

### repository

前面说到，在本地被使用的 `artifact`  默认都会被存储到 `~/.m2/repository` 目录下。

由于 `Maven` 仓库是通过简单文件系统透明地展示给 `Maven` 用户的，所以在使用过程中, 我们可以绕过 `Maven` 直接查看或修改仓库文件，在遇到疑难问题时，这往往十分有用。

eg.  `org.springframework.spring-core-3.2.4.jar`

![repository_dir_structure](http://h.img.siblings.top/maven_intro_repository_dir_tree_160919.png)

除了本地仓库之外，`Maven` 还有上文中已提及到的 [中央仓库](https://repo1.maven.org/maven2/) 和 私有仓库「`Nexus` or `Artifactory`」

[中央仓库](https://repo1.maven.org/maven2/)，是由 `Maven` 社区管理的仓库， 无需配置， 直接通过网络即可访问。所有开源的社区都会把自己开发的项目发布到中央仓库， 缺点就是因为会提供给全世界使用，所以，慢的常常让人无法接受。

私有仓库，主要就是为了解决中央仓库访问缓慢的问题而出现的，它也可以用来存储一些公司内部发布的，非公开的  `artifact` 构件。

一般来说，每个使用 `Maven` 的组织都会搭建一个自己的 `Maven` 私服。 `Maven` 私服在搭建成功后，默认需要从中央仓库中拿到所有 `artifact` 构件的索引（这个时候并不会把真正的组件下载到服务器），方便开发人员可以直接在私服中检索是否存在项目所需要依赖的 `artifact`。

> `REF`: [如何搭建 `Nexus` 私服](https://my.oschina.net/liangbo/blog/195739)
>
> `Note` : 教程是 2.0+ 版本的 `Nexus`， 官方最新的版本已经到 3.0+ 了， 但是学习资源还很少。

当我们的项目需要具体依赖某一个 `artifact` 时，私服首先会在本地查看索引中是否已存在，如果索引中存在 `artifact`， 则查看是否存在具体的构件。 如果构件不存在，就先从中央服务器下载构件，并存储到私服自己的仓库中。

eg.  当我们在项目中配置 `spring-core-2.5.6.jar` 的依赖时， `Maven` 是如何找到这个构件，并下载到本地

![maven_intro_artifact_query_in_repository_s_order_190920.png](http://h.img.siblings.top/maven_intro_artifact_query_in_repository_s_order_190920.png)

> `Tips`: 由于中央仓库的访问速度太过缓慢， 我们还可以配置一些 [国内的 `Maven` 私服镜像](https://my.oschina.net/fdblog/blog/546938)
>    比如 `OSC` 的国内镜像 「一般会配置在 `~/.m2/settings.xml` 中」, 
> ```xml
> <mirror>
>   <id>CN</id>
>   <name>OSChina Central</name>                                   
>   <url>http://maven.oschina.net/content/groups/public/</url>
>   <mirrorOf>central</mirrorOf>
> </mirror>
> ```

`Note`: 在使用的过程中，我们还需要注意一些商业项目， 比如 `Oracle` 官方提供的 `JDBC Driver - ojdbc14.jar` 是不会出现在中央仓库中的。所以在使用过程中，我们需要单独把这些特殊的 `artifact` 手工上传到我们的私服上， 所幸这种类型的构件很少。

[Demo](http://h.img.siblings.top/maven_intro_3rd_u2_rep_160920.gif)

上传成功后， 我们便可以在私服的仓库中看到这个文件


![Nexus_3rd_in_rep](http://h.img.siblings.top/maven_intro_3rd_in_rep_160920_final.png)

>`Note` : 必须登录以后，才可以在私服上传构件

### IDE 集成

// `Tips`: `IDEA` 中默认已集成，足以满足我们的日常使用

### New Project in Maven

> 演示 `Maven` 新建一个简单の项目, 展示默认约定的目录结构

[Demo](http://h.img.siblings.top/maven_intro_new_in_idea_190920.gif)

### pom.xml

在演示中，我们可以发现，项目在新建之后默认会打开一个文件 - `pom.xml`。这个文件对于我们的日常的开发工作而言，使用是最频繁的。

下面是一个比较全面的 `pom` 文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 项目坐标 -->
    <!-- groupId - 一般会定义为公司域名倒序 -->
    <!-- artifactId - 一般会定义为项目名称/组件名称 -->
    <groupId>org.dxy</groupId>
    <artifactId>learn-mvn</artifactId>
    <!-- 项目版本号 -->
    <version>1.0-SNAPSHOT</version>
    <!-- 项目构建成功后的最终类型, 默认为 `jar`, 可忽略不写 -->
    <!-- 其他常用的值还有: war - web 应用, pom - 用于 `pom` 继承(Parent Project) 或者 多模块(Modules)项目合成 -->
    <packaging>war</packaging>

    <!-- 项目分发信息，在执行 `mvn deploy` 后表示要发布的位置。有了这些信息就可以把网站部署到远程服务器或者把构件部署到远程仓库。 -->
    <!-- Note: 必须在 `settings.xml` 已配置 <server> 节点, 并在节点中指定了私有仓库的用户名和密码 -->
    <distributionManagement>
        <repository>
            <uniqueVersion>true</uniqueVersion>
            <id>MAVEN-releases</id>
            <name>MAVEN-releases</name>
            <url>http://192.168.0.xxx/components/libs-release-local</url>
        </repository>
        <snapshotRepository>
            <uniqueVersion>true</uniqueVersion>
            <id>MAVEN</id>
            <name>MAVEN-snapshot</name>
            <url>http://192.168.0.xxx/components/libs-snapshot-local</url>
        </snapshotRepository>
    </distributionManagement>

    <!-- 常用于抽离一些 `pom` 配置中可能会发生变化的值 -->
    <properties>
        <!-- 项目属性 -->
        <!--
        <jdbc.driver.groupId>com.h2database</jdbc.driver.groupId>
        <jdbc.driver.artifactId>h2</jdbc.driver.artifactId>
        <jdbc.driver.version>${h2.version}</jdbc.driver.version>
        -->
        <jdbc.driver.groupId>mysql</jdbc.driver.groupId>
        <jdbc.driver.artifactId>mysql-connector-java</jdbc.driver.artifactId>
        <jdbc.driver.version>5.1.22</jdbc.driver.version>
        <!-- 构件的依赖版本 -->
        <junit.version>4.9</junit.version>
        <!-- Plugin的属性 -->
        <java.version>1.6</java.version>
        <!-- 一些默认的项目全局变量 -->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <!-- 还有其他许多 ${project.*} 的默认全局变量, 可以直接在 `pom` 的配置中使用. -->
    </properties>

    <!-- 构建依赖配置 -->
    <dependencies>
        <!-- JDBC -->
        <dependency>
            <groupId>${jdbc.driver.groupId}</groupId>
            <artifactId>${jdbc.driver.artifactId}</artifactId>
            <version>${jdbc.driver.version}</version>
        </dependency>
        <!-- Test -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
    </dependencies>

    <!-- 插件配置 -->
    <build>
        <plugins>
            <!-- compiler插件, 设定JDK版本 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <showWarnings>true</showWarnings>
                </configuration>
            </plugin>

            <!-- source attach plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- war打包插件, 设定war包名称不带版本号 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <!-- 这里使用了 maven 默认的全局变量 -->
                    <warName>${project.artifactId}</warName>
                </configuration>
            </plugin>

        </plugins>
    </build>
</project>
```



> 演示 parent 集成 & 多模块开发
>
> 

### The Life-Cycle

在 `pom.xml`  的配置中， 我们提到了 `packaging` ， 用于指定项目最终构建的文件类型。

那么, `maven` 项目是如何构建的呢？

![maven_intro_mvn_package_demo.gif](http://h.img.siblings.top/maven_intro_mvn_package_demo.gif)

`mvn package` 会根据 `pom.xml` 中描述的构建配置创建 `jar/war`。

其中， `package` 是 `maven` 构建生命周期的一部分。

那么， 什么是构建生命周期呢？

软件开发人员每天都在对项目进行清理、编译、测试 & 部署，这些就是一个项目构件的生命周期。

大家都在不停地做构建工作，每天不同的项目之间，都会用不同的方式来做类似的工作。

`Maven` 的生命周期就是为了对所有的构建过程进行抽象和统一。

生命周期是一个抽象的概念，它本身不需要做任何工作。实际的任务（如编译源代码）都交由插件来完成。类似设计模式中的模板方法（Template Method）。

构建生命周期是一组阶段的序列（sequence of phases），每个阶段定义了目标被执行的顺序。这里的阶段是生命周期的一部分。

Maven 一共包含三套相互独立的生命周期，分别为 `clean`、`default`、`site`。

- `clean`  - 用于清理项目已有构建
- `default` - 用于构建项目
- `site` - 用于建立项目站点

上面每个生命周期都包含一些阶段（phase）， 这些阶段是有序执行的。

`default`生命周期包含以下几个常见的阶段（表中已省略了一部分 `phase`） :

| 生命周期阶段            | 描述                                       |
| ----------------- | ---------------------------------------- |
| validate          | 检查工程配置是否正确，完成构建过程的所有必要信息是否能够获取到          |
| initialize        | 初始化构建状态，例如设置属性                           |
| process-sources   | 处理项目源码文件                                 |
| process-resources | 处理项目资源文件                                 |
| compile           | 编译工程源码                                   |
| test              | 使用适当的单元测试框架（例如JUnit）运行测试                 |
| package           | 获取编译后的代码，并按照可发布的格式进行打包，例如 JAR、WAR 或者 EAR 文件 |
| install           | 安装工程包到本地仓库中，该仓库可以作为本地其他工程的依赖             |
| deploy            | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程             |

构建 `Maven` 项目， 最主要的方式就是调用 `Maven` 的生命周期阶段。

以其中最常用的 `default` 为例。

`$mvn package` : 该命令调用 `default` 生命周期的 `package ` 阶段。但实际执行的阶段为 `default` 生命周期的 `validate`、`initialize`、… , 直到 `package` 的所有阶段。

### `Maven` 插件

上文中我们提到，生命周期的具体工作都是通过具体的插件来完成。

比如，

```xml
<!-- 插件配置 -->
<build>
  <plugins>
    <!-- compiler 插件, 设定JDK版本 -->
    <plugin>
      <!-- 可省略，如果插件是 Maven 的官方插件 -->
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.1</version>
      <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <showWarnings>true</showWarnings>
      </configuration>
    </plugin>
  </plugins>
</build>
```

在上面的配置中， 我们配置了用于编译的源代码的插件，并指定了编译时所使用的 `JDK` 版本。

在实际的使用过程中，为了能让用户不用任何配置就能构建 `Maven` 项目， `Maven` 默认已经为一些主要的生命周期阶段绑定了很多默认的插件目标（plugin:goal）。

下表中列出一些常见的插件目标

| 生命周期阶段                 | 插件目标                                 | 执行任务            |
| ---------------------- | ------------------------------------ | --------------- |
| process-resources      | maven-resources-plugin:resources     | 复制主资源文件到主输出目录   |
| compile                | maven-compile-plugin:compile         | 编译主代码到主输出       |
| process-test-resources | maven-resources-plugin:testResources | 复制测试资源文件到测试输出目录 |
| test-compile           | maven-compile-plugin:testCompile     | 编译测试代码到测试输出目录   |
| test                   | maven-surefire-plugin:test           | 执行测试用例          |
| package                | maven-jar-plugin:jar                 | 创建项目 jar 包      |
| install                | maven-install-plugin:install         | 将目录输出构件安装到本地仓库  |
| deploy                 | maven-deploy-plugin:deploy           | 将项目输出构件部署到远程仓库  |

`Maven` 插件非常多，但大多数没有完善的文档。

`Maven` 插件也支持自定义扩展，是一个比较大的体系。有兴趣的同学可以自己扩展一下。

### Maven or Ant



#### Apache Ant (2000)

- `Ant` 没有正式的约定项目的目录结构，你必须明确的告诉 `Ant` 去哪里找源代码，哪里放置输出。
- `Ant` 是程序化的，你必须明确的告诉 `Ant` 做什么， 什么时候做， 你必须告诉它去编译，然后去复制， 然后压缩。
- `Ant` 没有生命周期，你必须定义目标和目标之间的依赖。 你必须手工为每个目标附加一个任务序列

#### Apache Maven (2004)

- `Maven` 拥有默认的约定，如果你遵循了约定，它已经知道你的源代码在哪里。它把字节码放到 `target/classes`，然后在 `target目录` 生成一个 `jar` 文件。
- `Maven` 是声明式的。你需要做的只是创建一个 `pom.xml` 文件然后将源代码放到默认的目录。`Maven` 会帮你处理其它的事情。
- `Maven` 有一个生命周期，当你运行 `mvn install` 的时候被调用。这条命令告诉 `Maven` 执行一系列的有序的步骤，直到到达你指定的生命周期。在执行生命周期的过程中，`Maven`  会运行许多默认的插件目标，这些目标完成了像编译和创建一个 `jar` 文件这样的工作。

#### Gradle（2012）

一种更简洁、更易于理解、更灵活的构建方案。



## Maven 还可以做什么?

1. 构建
2. 文档生成
3. 报告
4. 依赖
5. SCMs
6. 发布
7. 分发
8. 邮件列表 
   总的来说，Maven 简化了工程的构建过程，并对其标准化。它无缝衔接了编译、发布、文档生成、团队合作和其他任务。Maven 提高了重用性，负责了大部分构建相关的任务

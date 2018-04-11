# 本地nexus3的搭建

**搭建私服**

私服是一种特殊的远程Maven仓库，它是架设在局域网内的仓库服务，私服一般被配置为互联网远程仓库的镜像，供局域网内的Maven用户使用。
当Maven需要下载构件的时候，先向私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，同时缓存在私服之上，然后为Maven下载请求提供下载服务，另外，对于自定义或第三方的jar可以从本地上传到私服，供局域网内其他maven用户使用。

**私服优点主要有：**

节省外网宽带

加速Maven构建

部署第三方构件

提高稳定性、增强控制：原因是外网不稳定

降低中央仓库的负荷：原因是中央仓库访问量太大


## 搭建步骤

**官网下载nexus压缩包**

https://www.sonatype.com/  —————— Products ————Nexus Repository OSS下载对应操作系统的压缩包

**安装步骤**

1. 解压

2. Win+R 进入E:\nexus3\nexus-3.9.0-01\bin”目录

3. 输入 nexus.exe /run，回车即可运行。

**如何安装**

1. 安装nexus.exe /install

2. 卸载nexus.exe /uninstall

**启动与关闭服务**

1. 启动 net start nexus
2. 关闭 net stop nexus

**登陆Nexus管理系统**

浏览器输入 localhost:8081 账户admin密码admin123

**如何修改端口号访问的根路径等**

编辑E:\nexus3\nexus-3.9.0-01\etc\nexus-default.properties

	## DO NOT EDIT - CUSTOMIZATIONS BELONG IN $data-dir/etc/nexus.properties
	##
	# Jetty section
	application-port=8088
	application-host=0.0.0.0
	nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
	nexus-context-path=/nexus
	
	# Nexus section
	nexus-edition=nexus-pro-edition
	nexus-features=\
	nexus-pro-feature

**如何修改数据存储的路径** 

编辑E:\nexus3\nexus-3.9.0-01\bin\nexus.vmoptions

	-Xms1200M
	-Xmx1200M
	-XX:MaxDirectMemorySize=2G
	-XX:+UnlockDiagnosticVMOptions
	-XX:+UnsyncloadClass
	-XX:+LogVMOutput 
	-XX:LogFile=../sonatype-work/nexus3/log/jvm.log
	-XX:-OmitStackTraceInFastThrow
	-Djava.net.preferIPv4Stack=true
	-Dkaraf.home=.
	-Dkaraf.base=.
	-Dkaraf.etc=etc/karaf
	-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
	-Dkaraf.data=../sonatype-work/nexus3
	-Djava.io.tmpdir=../sonatype-work/nexus3/tmp
	-Dkaraf.startLocalConsole=false

[参照网址](https://blog.csdn.net/sqzhao/article/details/70158999)

## Nexus3管理系统的使用

**进入管理系统有如下默认库（可自行增删）**

maven-central：maven中央库，默认从https://repo1.maven.org/maven2/拉取jar 

maven-releases：私库发行版jar 

maven-snapshots：私库快照（调试版本）jar 

maven-public：仓库分组，把上面三个仓库组合在一起对外提供服务，在本地maven基础配置settings.xml中使用。

**nexus里可以配置3种类型的仓库，分别是proxy、hosted、group**

proxy是远程仓库的代理。比如说在nexus中配置了一个central repository的proxy，当用户向这个proxy请求一个artifact，这个proxy就会先在本地查找，如果找不到的话，就会从远程仓库下载，然后返回给用户，相当于起到一个中转的作用

hosted是宿主仓库，用户可以把自己的一些构件，deploy到hosted中，也可以手工上传构件到hosted里。比如说oracle的驱动程序，ojdbc6.jar，在central repository是获取不到的，就需要手工上传到hosted里 

Hosted有三种方式，Releases、SNAPSHOT、Mixed

	Releases: 一般是已经发布的Jar包
	Snapshot: 未发布的版本
	Mixed：混合的

**注意事项：**
Deployment Pollcy: 我们需要把策略改成“Allow redeploy”。

group是仓库组，在maven里没有这个概念，是nexus特有的。目的是将上述多个仓库聚合，对用户暴露统一的地址，这样用户就不需要在pom中配置多个地址，只要统一配置group的地址就可以了 

[参照网址](https://blog.csdn.net/qq_36357820/article/details/76184400)

**关于快照版和正式版的区别**

maven的依赖管理是基于版本管理的，对于发布状态的artifact，如果版本号相同，即使我们内部的镜像服务器上的组件比本地新，maven也不会主动下载的。
maven会根据模块的版本号(pom文件中的version)中是否带有-SNAPSHOT来判断是快照版本还是正式版本。如果是快照版本，那么在mvn deploy时会自动发布到快照版本库中，而使用快照版本的模块，在不更改版本号的情况下，直接编译打包时，maven会自动从镜像服务器上下载最新的快照版本。如果是正式发布版本，那么在mvn deploy时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，编译打包时如果本地已经存在该版本的模块则不会主动去镜像服务器上下载。
所以，我们在开发阶段，可以将公用库的版本设置为快照版本，而被依赖组件则引用快照版本进行开发，在公用库的快照版本更新后，我们也不需要修改pom文件提示版本号来下载新的版本，直接mvn执行相关编译、打包命令即可重新下载最新的快照库了，从而也方便了我们进行开发。

## 操作

在distributionManagement段中配置的是snapshot快照库和release发布库的地址，我这里是采用nexus作为镜像服务器。对于版本库主要是id和url的配置，配置完成后就可以通过mvn deploy进行发布了，当然了，如果你的镜像服务器需要用户名和密码，那么还需要在maven的settings.xml文件中做如下配置：

**settings.xml**

	<server>  
	  <id>nexus-releases</id>  
	  <username>admin</username>  
	  <password>admin123</password>  
	</server>  
	  
	<server>  
	  <id>nexus-snapshots</id>  
	  <username>admin</username>  
	  <password>admin123</password>  
	</server>
**注意：**这里配置的server的id必须和pom文件中的distributionManagement对应仓库的id保持一致，maven在处理发布时会根据id查找用户名称和密码进行登录和文件的上传发布。

**pom.xml**
	
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  
	    <modelVersion>4.0.0</modelVersion>  
	    <groupId>net.aty.mybatis</groupId>  
	    <artifactId>mybatis-demo</artifactId>  
	    <packaging>jar</packaging>  
	    <version>${project.release.version}</version>  
	    <name>mybatis-demo</name>  
	    <url>http://maven.apache.org</url>  
	      
	    <properties>  
	        <project.release.version>0.1-SNAPSHOT</project.release.version>  
	    </properties>  
	      
	  
	    <profiles>  
	        <profile>  
	            <id>release</id>  
	        <properties>  
	            <project.release.version>0.1</project.release.version>  
	        </properties>  
	        </profile>  
	    </profiles>  
	      
	      
	    <!--定义snapshots库和releases库的nexus地址-->  
	    <distributionManagement>  
	        <repository>  
	            <id>nexus-releases</id>  
	            <url>  
	                http://172.17.103.59:8081/nexus/content/repositories/releases/  
	            </url>  
	        </repository>  
	        <snapshotRepository>  
	            <id>nexus-snapshots</id>  
	            <url>  
	                http://172.17.103.59:8081/nexus/content/repositories/snapshots/  
	            </url>  
	        </snapshotRepository>  
	    </distributionManagement>  
	</project>  

首先我们看到pom文件中version的定义是采用占位符的形式，这样的好处是可以根据不同的profile来替换版本信息，比如maven默认是使用0.1-SNAPSHOT作为该模块的版本。

1、如果在发布时使用mvn deploy -P release 的命令，那么会自动使用0.1作为发布版本，那么根据maven处理snapshot和release的规则，由于版本号后不带-SNAPSHOT故当成是正式发布版本，会被发布到release仓库；

2、如果发布时使用mvn deploy命令，那么就会使用默认的版本号0.1-SNAPSHOT，此时maven会认为是快照版本，会自动发布到快照版本库。

[参照网址](https://blog.csdn.net/aitangyong/article/details/53332091)

## settings.xml和pom.xml的配置

[参照不同版本Nexus的官方文档配置](https://help.sonatype.com/repomanager3/maven-repositories)

**settings.xml加入如下信息:**

	<settings>
	 <servers>
	    <server>
	      <id>nexus</id>
	      <username>admin</username>
	      <password>admin123</password>
	    </server>
	 </servers>
	  <mirrors>
	    <mirror>
	      <!--This sends everything else to /public -->
	      <id>nexus</id>
	      <mirrorOf>*</mirrorOf>
	      <url>http://localhost:8081/repository/maven-public/</url>
	    </mirror>
	  </mirrors>
	  <profiles>
	    <profile>
	      <id>nexus</id>
	      <!--Enable snapshots for the built in central repo to direct -->
	      <!--all requests to nexus via the mirror -->
	      <repositories>
	        <repository>
	          <id>central</id>
	          <url>http://central</url>
	          <releases><enabled>true</enabled></releases>
	          <snapshots><enabled>true</enabled></snapshots>
	        </repository>
	      </repositories>
	     <pluginRepositories>
	        <pluginRepository>
	          <id>central</id>
	          <url>http://central</url>
	          <releases><enabled>true</enabled></releases>
	          <snapshots><enabled>true</enabled></snapshots>
	        </pluginRepository>
	      </pluginRepositories>
	    </profile>
	  </profiles>
	  <activeProfiles>
	    <!--make the profile active all the time -->
	    <activeProfile>nexus</activeProfile>
	  </activeProfiles>
	</settings>

**pom.xml的内容**

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	
	    <groupId>com.cslvln.maven</groupId>
	    <artifactId>cl-parent</artifactId>
	    <version>${project.release.version}</version>
	    <!-- jar:java工程 war;web工程 pom;非java和web工程（父工程一般用这个,但是不生成target目录）-->
	    <packaging>pom</packaging>
	
	    <properties>
	        <project.release.version>0.1-SNAPSHOT</project.release.version>
	        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	        <maven.compiler.source>1.8</maven.compiler.source>
	        <maven.compiler.target>1.8</maven.compiler.target>
	    </properties>
	
	    <profiles>
	        <profile>
	            <id>release</id>
	            <properties>
	                <project.release.version>0.1</project.release.version>
	            </properties>
	        </profile>
	    </profiles>
	
	    <distributionManagement>
	        <repository>
	            <id>nexus</id>
	            <name>Releases</name>
	            <url>http://localhost:8088/nexus/repository/maven-releases</url>
	        </repository>
	        <snapshotRepository>
	            <id>nexus</id>
	            <name>Snapshot</name>
	            <url>http://localhost:8088/nexus/repository/maven-snapshots</url>
	        </snapshotRepository>
	    </distributionManagement>
	
	    <build>
	        <plugins>
	            <!-- maven 是个自动化构建工具，如果我们不告诉它我们的代码要使用什么样的jdk版本编译的话，它就会用maven-compiler-plugin默认的jdk版本来进行编译处理，这样就容易出现版本不匹配的问题，以至于可能导致编译不通过的问题-->
	            <plugin>
	                <groupId>org.apache.maven.plugins</groupId>
	                <artifactId>maven-compiler-plugin</artifactId>
	                <configuration>
	                    <source>${maven.compiler.source}</source>
	                    <target>${maven.compiler.target}</target>
	                    <encoding>${project.build.sourceEncoding}</encoding>
	                </configuration>
	            </plugin>
	            <!-- maven-source-plugin插件打出的包包含远吗，引用包含远吗的包，在IDEA等ide中查看源码文件时可以更容易读些。要生成带source的jar,需要在pom.xml中包含下面的插件
	            然后执行mvn install,打包成功的话，就可以在target文件夹中找到带source的jar包通常命名为xxx-version-sources.jar
	            注意：如果src/main/java中没有源码，则不生成-->
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
	        </plugins>
	    </build>
	</project>

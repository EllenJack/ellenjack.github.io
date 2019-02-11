---
title: Maven环境隔离
date: 2018-12-20 17:07:19
tags: Maven
category: 
- 工具
- Maven
---

> 在实际的开发过程中，我们往往需要有多个环境，如开发，测试，线上的环境，如果每次都修改配置文件后打包，那么也未免有些太过繁琐，也极其容易犯错，Maven为我们提供了响应的方式去隔离各种环境。下面就来看看这个环境的搭建。

## 步骤

1. 把项目中的

   ```
   src/main/resources
   ```

   的配置文件按照：开发，测试，线上的配置分别放在新建的不同的文件夹中，公共的保持其原有的不动即可 

   - 需要注意的是，因为需要分为三个环境，那么有些内容就应该单独隔离开来，如：db.properties，logback.xml等
   - 步骤： 
     1. 在`src/main`下分别新建文件夹①resources.develop②resources.test③resources.product
     2. 把需要分环境的配置文件分别拷贝到刚才创建的文件夹下

2. 在pom.xml文件中找到

   ```
   build
   ```

   节点，并在其内部增加如下内容 

   ```
   <resources>
       <resource>
           <directory>src/main/resources.${deploy.envtype}</directory>
       <excludes>
           <exclude>*.jsp</exclude>
           </excludes>
       </resource>
       <resource>
           <directory>src/main/resources</directory>
       </resource>
   </resources>
   ```

3. 在pom.xml中与build节点同级的位置增加如下内容: 

   ```
   <profiles>
       <profile>
           <id>develop</id>
           <activation>
               <activeByDefault>true</activeByDefault>
           </activation>
           <properties>
               <deploy.envtype>develop</deploy.envtype>
           </properties>
       </profile>
       <profile>
           <id>test</id>
           <properties>
               <deploy.envtype>test</deploy.envtype>
           </properties>
       </profile>
       <profile>
           <id>product</id>
           <properties>
               <deploy.envtype>product</deploy.envtype>
           </properties>
       </profile>
   </profiles>
   ```

4. 打包 

   - 打包命令 

     ```
     mvn clean package -Dmaven.test.skip=true [-P<env-name>]
     ```

   - 命令中的<env-name>指的是在配置文件中锁配置的环境标识

   - 如果省略了中括号中的参数，那么将会使用activeByDefault为true的环境进行打包

------

## Eclipse和Idea中的使用

1. Idea和Eclipse在开发中使用激活的配置进行开发 
   - Idea中有一个名为`Maven Projects`的视图，这个视图中就有我们配置的各种环境的选项，只要单选其中一个那么在使用该Idea进行开发的时候，就会使用这个环境进行编译运行
   - Eclipse中的设置在：项目右键 → properties → Maven，在右侧的`Active Maven Profiles(comma separated)`下面的框中填写激活的环境名称即可




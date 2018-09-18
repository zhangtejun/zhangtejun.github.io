---
layout: post
title:  "Jenkins 集成 SonarQube"
date:   2018-02-23 09:21:21
author: zhangtejun
categories: tools
---
#### 方案一 Jenkins构建前进行代码分析
##### Jenkins 安装 SonarQube 插件
* 登陆 jenkins，点击"系统管理"  => "管理插件" 如图。点击高级，通过.hpi文件来安装插件。[sonar.hpi]()
![](./image002.png)
  
* Jenkins 全局配置 SonarQube Server。 登陆 jenkins，点击"系统管理"  => "系统设置" 如图。
![](./003.PNG)

  ```javascript
  #Server URL
  http://192.168.41.32:9051
  #Server token => admin/admin
  ae92c934e5e3c23a715bc36be2e663d93af95d60
  ```

##### Jenkins 项目配置
1. 在 Jenkins 项目构建过程中加入 SonarScanner 进行代码分析。
首先需要在新建的 Jenkins 项目的构建环境标签页中勾选"Prepare SonarQube Scanner evironment"，如图 。
  ![](./image004.png)

2. 增加构建步骤"Execute SonarQube Scanner"，如图
  ![](./image005.png)

3. 配置 SonarQube Scanner 
  登陆 jenkins，点击"系统管理"  => "Global Tool Configuration" 如图。点击高级，
  ![](./005.PNG)
4. 配置 SonarQube Scanner 构建步骤，
  * 在 JDK 选择框中选择 SonarQube Scanner 使用的 JDK。
  
  * Path to project properties 是可选择的输入框，这里可以指定一个 sonar-project.properties 文件，
如果不指定的话会使用项目默认的 properties 文件；
  
  * Analysis properties 输入框，这里需要输入一些配置参数用来传递给 SonarQube，这里的参数优先
级高于 sonar-project.properties 文件里面的参数，所以可以在这里来配置所有的参数以替代 sonar-project.properties 文件，下面列出了一些参数，
sonar.language 指定了要分析的开发语言（特定的开发语言对应了特定的规则），sonar.sources 定义了需要分析的源代码位置（示例中的$WORKSPACE 
所指示的是当前 Jenkins 项目的目录），sonar.java.binaries 定义了需要分析代码的编译后 class 文件位置；
  
  * Additional arguments 
输入框中可以输入一些附加的参数，示例中的-X 意思是进入 SonarQube Scanner 的 Debug 模式，这样会输出更多的日志信息；
  
  * JVM Options 可以输入在执行 SonarQube Scanner 是需要的 JVM 参数。
    
  * 我们采用参数配置方式（针对java,js单独配置如下：）
   ```javascript
        # 在Analysis properties 配置参数 
        sonar.projectKey=spdb-pits-server-uat_java # 此项必须唯一，将在SonarQube中显示
        sonar.projectName=spdb-pits-server-uat_java
        sonar.projectVersion=1.0
        sonar.language=java #分析java文件
        sonar.sources=/home/source/.jenkins/workspace/pits/pitsEar4_97/spdb-pits-server/ #服务器项目打包路径
      
        # 在Analysis properties 配置参数 
        sonar.projectKey=spdb-pits-server-uat_js # 此项必须唯一，不同Jenkins服务器的key都不能重复，将在SonarQube中显示
        sonar.projectName=spdb-pits-server-uat_js
        sonar.projectVersion=1.0
        sonar.language=js #分析js文件
        sonar.sources=/home/source/.jenkins/workspace/pits/pitsEar4_97/spdb-pits-server/ #服务器项目打包路径
   ```

>注：默认只有java和js插件，如需要其他插件，请自行安装。
   
   
#####SonarQube服务器

> 所有jenkins服务器可公用一个SonarQube服务器(已安装)。

> 如果不需要安装SonarQube服务器可跳过以下步骤：

> SonarQube 服务器安装
  * 登陆服务器在命令行启 动 S onarQube：进 入 S onarQub e 安 装目录，进 入 b i n 目 录，运行`./sonar.sh start`， 打 开 `h ttp://192.168.41.32:9051`查看。
    
    ```javascript
    # 启动
    cd /home/sonarqube/sonarqube-5.5/bin/linux-x86-64
    nohup ./sonar.sh start &   
    
    # 停止
    cd /home/sonarqube/sonarqube-5.5/bin/linux-x86-64
    nohup ./sonar.sh stop & 
    
    # 重启
    cd /home/sonarqube/sonarqube-5.5/bin/linux-x86-64
    nohup ./sonar.sh restart &    
    ```

##### sonnar-runar

>登陆jenkins服务器，cd到任意目录，拷贝Sonar-Scanner到该目录。（上述Jenkins配置Sonar-Scanner 需要用到该目录）
```javascript
scp -r userName@xx.xx.xx.xx:/weblogic/sonnar .

#增加Sonar
export  SONAR_RUNNER_HOME=/weblogic/sonnar/SonarQubeScanner/sonar-runner-2.5-RC1 #该路径按需调整
export  PATH=${SONAR_RUNNER_HOME}/bin:${PATH}

# 测试sonnar-runar
可执行 sonar-runner -h 
```
> 至此所有配置已完成，可登陆jenkins构建项目及查看报告，
项目构建完成后可点击相应图标或者链接`http://xx.xx.xx.xx:xxxx`查看结果，默认管理员账户 `admin/admin`（注：代码有错误会导致构建失败）


#### 方案二  开发人员本地进行静态代码分析 
> 注：①需要是maven项目，②执行maven构建需要jdk1.8，eclipse可配置多版本jdk。
> 配置pom.xml，在<properties>节点增加sonar host参数：
```javascript
<properties>
  <sonar.host.url>http://xx.xx.xx.xx:xxxx/</sonar.host.url>
</properties>
```
>在pom.xml中增加<plugin>节点增加
```javascript
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artigactId>sonar-maven-plugin</artigactId>
  <version>3.0.1</version>
</plugin>
```
> 执行maven构建，`mvn sonar:sonar`

> 链接`http://xx.xx.xx.xx:xxxx`查看报告

#### 方案三 使用eclipse的sonarLint进行静态代码分析 
> 需要升级eclipse，暂时无法在sonarqube服务器上查看分析报告（需要sonarqube 5.6及以上版本和jdk1.8）,
可在eclipse控制台查看分析结果。

> SonarLint的使用
* Eclipse工具栏选择Window->Show View->other
[](./e1.jpg)

* 弹出“Show View”界面，输入Sonar，选择“SonarLintIssues”点击“OK”
[](./e2.jpg)

* 打开需要进行代码审查的java或js文件，SonarLint将会自动进行代码审查，在控制台输出审查结果
[](./e3.jpg)

* SonarLint默认在打开文件的时候自动进行代码审查。如果不想使用自动审查，设置方法：右键单击项目->Properties->SonarLint->取消“Run SonarLint automatically”->Apply->OK
[](./e4.jpg)

* 手动审查：右键审查文件->SonarLint->Analyze分析文件
[](./e5.jpg)

* 双击控制台的审查结果，可以自动定位到具体被审查内容的位置。如果修改代码，控制台将会自动刷新审查结果。
[](./e6.jpg)

* 右键审查结果，选择“Rule description”，查看针对单个问题的分析及改进建议
[](./e7.jpg)
[](./e8.jpg)

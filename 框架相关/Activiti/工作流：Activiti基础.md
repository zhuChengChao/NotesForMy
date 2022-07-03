# 工作流：Activiti基础

> 工作过程中接触到了工作流的概念，就觉得挺有趣的:wink:；就趁着这个五一抽空补充一下这方便的知识。
>
> > 换了工作后沉迷追剧打游戏...就是说还是得稍微学习一下哈:sweat_smile:
> >
> > 原视频地址：https://www.bilibili.com/video/BV1H54y167gf

在没有专门的工作流引擎之前，为了实现流程控制，通常的做法就是采用状态字段的值来跟踪流程的变化情况。这样不同角色的用户，通过状态字段的取值来决定记录是否显示。

针对有权限可以查看的记录，当前用户根据自己的角色来决定审批是否合格的操作。如果合格将状态字段设置一个值，来代表合格；当然如果不合格也需要设置一个值来代表不合格的情况。

这是一种最为原始的方式。通过状态字段虽然做到了流程控制，但是当流程发生变更的时候，这种方式所编写的代码也要进行调整。

那么有没有专业的方式来实现工作流的管理呢？并且可以做到业务流程变化之后，程序可以不用改变，如果可以实现这样的效果，那么业务系统的适应能力就得到了极大提升。

## 1. Activiti概述

官网地址：https://www.activiti.org/

### 1.1 概述

Activiti是一个工作流引擎， Activiti可以将业务系统中复杂的业务流程抽取出来，使用专门的建模语言BPMN2.0进行定义，业务流程按照预先定义的流程进行执行，实现了系统的流程由Activiti进行管理，减少业务系统由于流程变更进行系统升级改造的工作量，从而提高系统的健壮性，同时也减少了系统开发维护成本。

**BPM：**BPM（Business Process Management），即业务流程管理，是一种规范化的构造端到端的业务流程，以持续的提高组织业务效率。常见商业管理教育如EMBA、MBA等均将BPM包含在内。

**BPMN：**

* BPMN（Business Process Model And Notation，业务流程模型和符号）是由BPMI（BusinessProcess Management Initiative）开发的一套标准的业务流程建模符号，使用BPMN提供的符号可以创建业务流程。

* Activiti 就是使用 BPMN 2.0 进行流程建模、流程执行管理，它包括很多的建模符号，比如：

  * Event：事件，用一个圆圈表示，它是流程中运行过程中发生的事情：
  * Activity：活动，活动用圆角矩形表示，一个流程由一个活动或多个活动组成：

  ![BPMN符号](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400040.PNG)

* BPMN图形其实是通过xml表示业务流程，将.bpmn文件使用文本编辑器打开：

  ```xml
  <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
  <definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="http://www.activiti.org/testm1651322481124" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="m1651322481124" name="" targetNamespace="http://www.activiti.org/testm1651322481124" typeLanguage="http://www.w3.org/2001/XMLSchema">
    <process id="myEvection" isClosed="false" isExecutable="true" name="出差申请单" processType="None">
      <startEvent id="_2" name="StartEvent"/>
      <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="创建出差申请"/>
      <userTask activiti:assignee="jerry" activiti:exclusive="true" id="_4" name="经理审批"/>
      <userTask activiti:assignee="jack" activiti:exclusive="true" id="_5" name="总经理审批"/>
      <userTask activiti:assignee="rose" activiti:exclusive="true" id="_6" name="财务审批"/>
      <endEvent id="_7" name="EndEvent"/>
      <sequenceFlow id="_8" sourceRef="_2" targetRef="_3"/>
      <sequenceFlow id="_9" sourceRef="_3" targetRef="_4"/>
      <sequenceFlow id="_10" sourceRef="_4" targetRef="_5"/>
      <sequenceFlow id="_11" sourceRef="_5" targetRef="_6"/>
      <sequenceFlow id="_12" sourceRef="_6" targetRef="_7"/>
    </process>
    <bpmndi:BPMNDiagram documentation="background=#3C3F41;count=1;horizontalcount=1;orientation=0;width=842.4;height=1195.2;imageableWidth=832.4;imageableHeight=1185.2;imageableX=5.0;imageableY=5.0" id="Diagram-_1" name="New Diagram">
      <bpmndi:BPMNPlane bpmnElement="myEvection">
        <bpmndi:BPMNShape bpmnElement="_2" id="Shape-_2">
          <dc:Bounds height="32.0" width="32.0" x="210.0" y="80.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNShape>
        <bpmndi:BPMNShape bpmnElement="_3" id="Shape-_3">
          <dc:Bounds height="55.0" width="85.0" x="185.0" y="185.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNShape>
        <bpmndi:BPMNShape bpmnElement="_4" id="Shape-_4">
          <dc:Bounds height="55.0" width="85.0" x="185.0" y="290.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNShape>
        <bpmndi:BPMNShape bpmnElement="_5" id="Shape-_5">
          <dc:Bounds height="55.0" width="85.0" x="190.0" y="390.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNShape>
        <bpmndi:BPMNShape bpmnElement="_6" id="Shape-_6">
          <dc:Bounds height="55.0" width="85.0" x="190.0" y="490.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNShape>
        <bpmndi:BPMNShape bpmnElement="_7" id="Shape-_7">
          <dc:Bounds height="32.0" width="32.0" x="220.0" y="585.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNShape>
        <bpmndi:BPMNEdge bpmnElement="_12" id="BPMNEdge__12" sourceElement="_6" targetElement="_7">
          <di:waypoint x="236.0" y="545.0"/>
          <di:waypoint x="236.0" y="585.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNEdge>
        <bpmndi:BPMNEdge bpmnElement="_8" id="BPMNEdge__8" sourceElement="_2" targetElement="_3">
          <di:waypoint x="226.0" y="112.0"/>
          <di:waypoint x="226.0" y="185.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNEdge>
        <bpmndi:BPMNEdge bpmnElement="_9" id="BPMNEdge__9" sourceElement="_3" targetElement="_4">
          <di:waypoint x="227.5" y="240.0"/>
          <di:waypoint x="227.5" y="290.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNEdge>
        <bpmndi:BPMNEdge bpmnElement="_11" id="BPMNEdge__11" sourceElement="_5" targetElement="_6">
          <di:waypoint x="232.5" y="445.0"/>
          <di:waypoint x="232.5" y="490.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNEdge>
        <bpmndi:BPMNEdge bpmnElement="_10" id="BPMNEdge__10" sourceElement="_4" targetElement="_5">
          <di:waypoint x="230.0" y="345.0"/>
          <di:waypoint x="230.0" y="390.0"/>
          <bpmndi:BPMNLabel>
            <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
          </bpmndi:BPMNLabel>
        </bpmndi:BPMNEdge>
      </bpmndi:BPMNPlane>
    </bpmndi:BPMNDiagram>
  </definitions>
  ```

### 1.2 使用步骤

**部署Activiti：**Activiti是一个工作流引擎（其实就是一堆jar包API），业务系统访问（操作）Activiti的接口，就可以方便的操作流程相关数据，这样就可以把工作流环境与业务系统的环境集成在一起。

**流程定义：**使用Activiti流程建模工具（activity-designer）定义业务流程（.bpmn文件）。

> .bpmn文件就是业务流程定义文件，通过xml定义业务流程。

**流程定义部署：**

* activiti部署业务流程定义（.bpmn文件）；

* 使用activiti提供的api把流程定义内容存储起来，在Activiti执行过程中可以查询定义的内容;

* Activiti执行把流程定义内容存储在数据库中

**启动一个流程实例：**

* 流程实例也叫：ProcessInstance，启动一个流程实例表示开始一次业务流程的运行；

* 在员工请假流程定义部署完成后，如果张三要请假就可以启动一个流程实例，如果李四要请假也启动一个流程实例，两个流程的执行互相不影响。

**用户查询待办任务(Task)：**因为现在系统的业务流程已经交给activiti管理，通过activiti就可以查询当前流程执行到哪了，当前用户需要办理什么任务了，这些activiti帮我们管理了，而不需要开发人员自己编写在sql语句查询。

**用户办理任务：**用户查询待办任务后，就可以办理某个任务，如果这个任务办理完成还需要其它用户办理，比如采购单创建后由部门经理审核，这个过程也是由activiti帮我们完成了。

**流程结束：**当任务办理完成没有下一个任务结点了，这个流程实例就完成了。

### 1.3 开发环境

1. Activiti的Maven依赖：

   ```xml
   <dependency>
       <groupId>org.activiti</groupId>
       <artifactId>activiti-dependencies</artifactId>
       <version>7.0.0.Beta1</version>
       <scope>import</scope>
       <type>pom</type>
   </dependency>
   ```

2. 数据库支持：activiti运行需要有数据库的支持，支持的数据库有：h2, mysql, oracle, postgres, mssql, db2。

3. IDEA插件安装：actiBPM

## 2. Activiti使用

### 2.1 环境搭建

Activiti 在运行时需要数据库的支持，使用25张表，把流程定义节点内容读取到数据库表中，以供后续使用。

首先创建数据库：`CREATE DATABASE activiti DEFAULT CHARACTER SET utf8;`

工程框架搭建：

1. 创建Maven工程，这里取名：activiti01；

2. 加入Maven依赖包：

   ```xml
   <groupId>cn.xyc</groupId>
   <artifactId>activiti01</artifactId>
   <version>1.0-SNAPSHOT</version>
   
   <properties>
       <slf4j.version>1.6.6</slf4j.version>
       <log4j.version>1.2.12</log4j.version>
       <activiti.version>7.0.0.Beta1</activiti.version>
   </properties>
   
   <dependencies>
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-engine</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-spring</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <!-- bpmn 模型处理 -->
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-bpmn-model</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <!-- bpmn 转换 -->
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-bpmn-converter</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <!-- bpmn json数据转换 -->
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-json-converter</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <!-- bpmn 布局 -->
       <dependency>
           <groupId>org.activiti</groupId>
           <artifactId>activiti-bpmn-layout</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <!-- activiti 云支持 -->
       <dependency>
           <groupId>org.activiti.cloud</groupId>
           <artifactId>activiti-cloud-services-api</artifactId>
           <version>${activiti.version}</version>
       </dependency>
       <!-- mysql驱动 -->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>5.1.40</version>
       </dependency>
       <!-- mybatis -->
       <dependency>
           <groupId>org.mybatis</groupId>
           <artifactId>mybatis</artifactId>
           <version>3.4.5</version>
       </dependency>
       <!-- 链接池 -->
       <dependency>
           <groupId>commons-dbcp</groupId>
           <artifactId>commons-dbcp</artifactId>
           <version>1.4</version>
       </dependency>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.12</version>
       </dependency>
       <!-- log start -->
       <dependency>
           <groupId>log4j</groupId>
           <artifactId>log4j</artifactId>
           <version>${log4j.version}</version>
       </dependency>
       <dependency>
           <groupId>org.slf4j</groupId>
           <artifactId>slf4j-api</artifactId>
           <version>${slf4j.version}</version>
       </dependency>
       <dependency>
           <groupId>org.slf4j</groupId>
           <artifactId>slf4j-log4j12</artifactId>
           <version>${slf4j.version}</version>
       </dependency>
   </dependencies>
   ```

3. 添加log4j日志配置，为了能看到数据库的SQL信息：

   > 在resources 下创建log4j.properties

   ```
   # Set root category priority to INFO and its only appender to CONSOLE.
   #log4j.rootCategory=INFO, CONSOLE debug info warn error fatal
   log4j.rootCategory=debug, CONSOLE, LOGFILE
   # Set the enterprise logger category to FATAL and its only appender to CONSOLE.
   log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE
   # CONSOLE is set to be a ConsoleAppender using a PatternLayout.
   log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
   log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
   log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r[%15.15t] %-5p %30.30c %x - %m\n
   # 只要控制台输出就行了，就不需要输入到文件里了
   # LOGFILE is set to be a File appender using a PatternLayout.
   #log4j.appender.LOGFILE=org.apache.log4j.FileAppender
   #log4j.appender.LOGFILE.File=f:\act\activiti.log
   #log4j.appender.LOGFILE.Append=true
   #log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
   #log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r[%15.15t] %-5p %30.30c %x - %m\n
   ```

4. 添加activiti配置文件：

   使用activiti提供的默认方式来创建mysql的表；

   默认方式的要求是在 resources 下创建 activiti.cfg.xml 文件；

   > 注意：默认方式目录和文件名不能修改，因为activiti的源码中已经设置，到固定的目录读取固定文件名的文件

   默认方式要在在activiti.cfg.xml中bean的名字叫processEngineConfiguration，名字不可修改；

   在这里有2中配置方式：一种是单独配置数据源，一种是不单独配置数据源

   **配置方式1：单独配置数据源，直接配置processEngineConfiguration**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                       http://www.springframework.org/schema/beans/spring-beans.xsd
                       http://www.springframework.org/schema/contex
                       http://www.springframework.org/schema/context/spring-context.xsd
                       http://www.springframework.org/schema/tx
                       http://www.springframework.org/schema/tx/spring-tx.xsd">
       <!-- 默认id对应的值 为processEngineConfiguration -->
       <!-- processEngine Activiti的流程引擎 -->
       <bean id="processEngineConfiguration"
             class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
           <property name="jdbcDriver" value="com.mysql.jdbc.Driver"/>
           <property name="jdbcUrl" value="jdbc:mysql:///activiti"/>
           <property name="jdbcUsername" value="root"/>
           <property name="jdbcPassword" value="root"/>
           <!--actviti数据库表在生成时的策略
   				true：如果数据库中已经存在相应的表，那么直接使用，
   				false：如果不存在，那么会创建-->
           <property name="databaseSchemaUpdate" value="true"/>
       </bean>
   </beans>
   ```

   **配置方式2：不单独配置数据源，使用连接池**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
                       http://www.springframework.org/schema/beans/spring-beans.xsd
                       http://www.springframework.org/schema/contex
                       http://www.springframework.org/schema/context/spring-context.xsd
                       http://www.springframework.org/schema/tx
                       http://www.springframework.org/schema/tx/spring-tx.xsd">
   
       <!-- 这里可以使用 链接池 dbcp-->
       <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
           <property name="driverClassName" value="com.mysql.jdbc.Driver" />
           <property name="url" value="jdbc:mysql:///activiti" />
           <property name="username" value="root" />
           <property name="password" value="root" />
           <property name="maxActive" value="3" />
           <property name="maxIdle" value="1" />
       </bean>
   
       <bean id="processEngineConfiguration"
             class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
           <!-- 引用数据源 上面已经设置好了-->
           <property name="dataSource" ref="dataSource" />
           <!-- activiti数据库表处理策略 -->
           <property name="databaseSchemaUpdate" value="true"/>
       </bean>
   </beans>
   ```

5. Java类编写程序生成表：

   创建一个测试类，调用activiti的工具类，生成acitivti需要的数据库表。

   直接使用activiti提供的工具类ProcessEngines，会默认读取classpath下的activiti.cfg.xml文件，读取其中的数据库配置，创建 ProcessEngine，在创建ProcessEngine 时会自动创建表。 

   ```java
   package cn.xyc.activiti01.test;
   
   import org.activiti.engine.ProcessEngine;
   import org.activiti.engine.ProcessEngines;
   import org.junit.Test;
   
   public class TestDemo {
   
       /**
        * 生成 activiti的数据库表
        */
       @Test
       public void testCreateDbTable() {
           //使用classpath下的activiti.cfg.xml中的配置创建processEngine
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           System.out.println(processEngine);
       }
   }
   ```

   > 说明：
   >
   > 1. 运行以上程序段即可完成 activiti 表创建，通过改变 activiti.cfg.xml 中 databaseSchemaUpdate 参数的值执行不同的数据表处理策略；
   > 2. 上边的方法getDefaultProcessEngine方法在执行时，从activiti.cfg.xml 中找固定的名称 processEngineConfiguration。

   在输出日志中可以看到创建表的SQL语句：

   ```bash
   - --- starting SchemaOperationsProcessEngineBuild ---
   ...
   - SQL: create table ACT_GE_PROPERTY ( ...
   - SQL: create table ACT_GE_BYTEARRAY ( ...
   - SQL: create table ACT_RE_DEPLOYMENT
   ```

   执行完成后我们查看数据库，创建了 25 张表，结果如下： 

   ![表的创建](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400856.PNG)

到这，我们就完成activiti运行需要的数据库和表的创建。

### 2.2 表结构介绍

看到刚才创建的表，可以发现Activiti 的表都以 `ACT_` 开头。 

第二部分是表示表的用途的两个字母标识。 用途也和服务的 API 对应。

* **ACT_RE** ：'RE'表示 repository，这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
* **ACT_RU**：'RU'表示 runtime，这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。Activiti 只在流程实例执行过程中保存这些数据，在流程结束时就会删除这些记录。这样运行时表可以一直很小速度很快。
* **ACT_HI**：'HI'表示 history，这些表包含历史数据，比如历史流程实例，变量，任务等等。
* **ACT_GE** ：'GE'表示 general，通用数据，用于不同场景下。

| **表分类**   | **表名**              | **解释**                                           |
| ------------ | --------------------- | -------------------------------------------------- |
| 一般数据     |                       |                                                    |
|              | [ACT_GE_BYTEARRAY]    | 通用的流程定义和流程资源                           |
|              | [ACT_GE_PROPERTY]     | 系统相关属性                                       |
| 流程历史记录 |                       |                                                    |
|              | [ACT_HI_ACTINST]      | 历史的流程实例                                     |
|              | [ACT_HI_ATTACHMENT]   | 历史的流程附件                                     |
|              | [ACT_HI_COMMENT]      | 历史的说明性信息                                   |
|              | [ACT_HI_DETAIL]       | 历史的流程运行中的细节信息                         |
|              | [ACT_HI_IDENTITYLINK] | 历史的流程运行过程中用户关系                       |
|              | [ACT_HI_PROCINST]     | 历史的流程实例                                     |
|              | [ACT_HI_TASKINST]     | 历史的任务实例                                     |
|              | [ACT_HI_VARINST]      | 历史的流程运行中的变量信息                         |
| 流程定义表   |                       |                                                    |
|              | [ACT_RE_DEPLOYMENT]   | 部署单元信息                                       |
|              | [ACT_RE_MODEL]        | 模型信息                                           |
|              | [ACT_RE_PROCDEF]      | 已部署的流程定义                                   |
| 运行实例表   |                       |                                                    |
|              | [ACT_RU_EVENT_SUBSCR] | 运行时事件                                         |
|              | [ACT_RU_EXECUTION]    | 运行时流程执行实例                                 |
|              | [ACT_RU_IDENTITYLINK] | 运行时用户关系信息，存储任务节点与参与者的相关信息 |
|              | [ACT_RU_JOB]          | 运行时作业                                         |
|              | [ACT_RU_TASK]         | 运行时任务                                         |
|              | [ACT_RU_VARIABLE]     | 运行时变量表                                       |

### 2.3 Activiti类关系

**类关系图：**

![Activiti类关系图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400006.jpg)

> 在新版本中，通过实验可以发现IdentityService，FormService两个Serivce都已经删除了。

### 2.4 流程引擎配置类

> **activiti.cfg.xml：**activiti的引擎配置文件，包括：ProcessEngineConfiguration的定义、数据源定义、事务管理器等，此文件其实就是一个spring配置文件。

**流程引擎配置类：**流程引擎的配置类（ProcessEngineConfiguration），通过ProcessEngineConfiguration可以创建工作流引擎ProceccEngine，常用的两种方法如下： 

* StandaloneProcessEngineConfiguration：使用StandaloneProcessEngineConfigurationActiviti可以单独运行，来创建ProcessEngine，Activiti会自己处理事务。

  配置文件方式：通常在activiti.cfg.xml配置文件中定义一个id为 processEngineConfiguration 的bean；

  ```xml
  <bean id="processEngineConfiguration"
        class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
      <!--配置数据库相关的信息-->
      <!--数据库驱动-->
      <property name="jdbcDriver" value="com.mysql.jdbc.Driver"/>
      <!--数据库链接-->
      <property name="jdbcUrl" value="jdbc:mysql:///activiti"/>
      <!--数据库用户名-->
      <property name="jdbcUsername" value="root"/>
      <!--数据库密码-->
      <property name="jdbcPassword" value="root"/>
      <!--actviti数据库表在生成时的策略  true - 如果数据库中已经存在相应的表，那么直接使用，如果不存在，那么会创建-->
      <property name="databaseSchemaUpdate" value="true"/>
  </bean>
  ```

  还可以加入连接池：

  ```xml
  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql:///activiti"/>
      <property name="username" value="root"/>
      <property name="password" value="root"/>
      <property name="maxActive" value="3"/>
      <property name="maxIdle" value="1"/>
  </bean>
  <!--在默认方式下 bean的id  固定为 processEngineConfiguration-->
  <bean id="processEngineConfiguration"
        class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
      <!--引入上面配置好的 链接池-->
      <property name="dataSource" ref="dataSource"/>
      <!--actviti数据库表在生成时的策略  true - 如果数据库中已经存在相应的表，那么直接使用，如果不存在，那么会创建-->
      <property name="databaseSchemaUpdate" value="true"/>
  </bean>
  ```

  > 这部分上面已经贴过了。

* SpringProcessEngineConfiguration：通过**org.activiti.spring.SpringProcessEngineConfiguration**与Spring整合；

  创建spring与activiti的整合配置文件：

  activity-spring.cfg.xml（名称可修改）

  ```xml
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
         xsi:schemaLocation="http://www.springframework.org/schema/beans 
                             http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
                             http://www.springframework.org/schema/mvc 
                             http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd 
                             http://www.springframework.org/schema/context 
                             http://www.springframework.org/schema/context/spring-context-3.1.xsd 
                             http://www.springframework.org/schema/aop 
                             http://www.springframework.org/schema/aop/spring-aop-3.1.xsd 
                             http://www.springframework.org/schema/tx 
                             http://www.springframework.org/schema/tx/spring-tx-3.1.xsd ">
      <!-- 工作流引擎配置bean -->
      <bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
          <!-- 数据源 -->
          <property name="dataSource" ref="dataSource" />
          <!-- 使用spring事务管理器 -->
          <property name="transactionManager" ref="transactionManager" />
          <!-- 数据库策略 -->
          <property name="databaseSchemaUpdate" value="drop-create" />
          <!-- activiti的定时任务关闭 -->
          <property name="jobExecutorActivate" value="false" />
      </bean>
  
      <!-- 流程引擎 -->
      <bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
          <property name="processEngineConfiguration" ref="processEngineConfiguration" />
      </bean>
  
      <!-- 资源服务service -->
      <bean id="repositoryService" factory-bean="processEngine"
            factory-method="getRepositoryService" />
      <!-- 流程运行service -->
      <bean id="runtimeService" factory-bean="processEngine"
            factory-method="getRuntimeService" />
      <!-- 任务管理service -->
      <bean id="taskService" factory-bean="processEngine"
            factory-method="getTaskService" />
      <!-- 历史管理service -->
      <bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService" />
      <!-- 用户管理service -->
      <bean id="identityService" factory-bean="processEngine" factory-method="getIdentityService" />
      <!-- 引擎管理service -->
      <bean id="managementService" factory-bean="processEngine" factory-method="getManagementService" />
  
      <!-- 数据源 -->
      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
          <property name="driverClassName" value="com.mysql.jdbc.Driver" />
          <property name="url" value="jdbc:mysql://localhost:3306/activiti" />
          <property name="username" value="root" />
          <property name="password" value="mysql" />
          <property name="maxActive" value="3" />
          <property name="maxIdle" value="1" />
      </bean>
  
      <!-- 事务管理器 -->
      <bean id="transactionManager"
            class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource" />
      </bean>
  
      <!-- 通知 -->
      <tx:advice id="txAdvice" transaction-manager="transactionManager">
          <tx:attributes></tx:attributes>
          <!-- 传播行为 -->
          <tx:method name="save*" propagation="REQUIRED" />
          <tx:method name="insert*" propagation="REQUIRED" />
          <tx:method name="delete*" propagation="REQUIRED" />
          <tx:method name="update*" propagation="REQUIRED" />
          <tx:method name="find*" propagation="SUPPORTS" read-only="true" />
          <tx:method name="get*" propagation="SUPPORTS" read-only="true" />
          </tx:attributes>
      </tx:advice>
  
  <!-- 切面，根据具体项目修改切点配置 -->
  <aop:config proxy-target-class="true">
      <aop:advisor advice-ref="txAdvice"  pointcut="execution(* cn.xyc.ihrm.service.impl.*.(..))"* />
  </aop:config>
  </beans>
  ```

**创建processEngineConfiguration**

* 创建方式1：要求activiti.cfg.xml中必须有一个processEngineConfiguration的bean

  ```java
  ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("activiti.cfg.xml")
  ```

* 创建方式2：这种方式是需要修改bean 的名字

  ```java
  ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource, String beanName);
  ```

### 2.5 工作流引擎创建

工作流引擎（ProcessEngine），相当于一个门面接口，通过ProcessEngineConfiguration创建processEngine，通过ProcessEngine创建各个service接口。

* **默认创建方式：**将activiti.cfg.xml文件名及路径固定，且activiti.cfg.xml文件中有 processEngineConfiguration的配置，可以使用如下代码创建processEngine：

  ```java
  // 直接使用工具类 ProcessEngines，使用classpath下的activiti.cfg.xml中的配置创建processEngine
  ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
  System.out.println(processEngine);
  ```

* **一般创建方式**：

  ```java
  // 先构建ProcessEngineConfiguration
  ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource("activiti.cfg.xml");
  // 通过ProcessEngineConfiguration创建ProcessEngine，此时会创建数据库
  ProcessEngine processEngine = configuration.buildProcessEngine();
  ```

### 2.6 Servcie服务接口

Service是工作流引擎提供用于进行工作流部署、执行、管理的服务接口，使用这些接口可以就是操作服务对应的数据表。

* **Service创建方式：**通过ProcessEngine创建Service，方式如下：

  ```java
  RuntimeService runtimeService = processEngine.getRuntimeService();
  RepositoryService repositoryService = processEngine.getRepositoryService();
  TaskService taskService = processEngine.getTaskService();
  ```

* **Service总览：**

  | service名称       | service作用              |
  | ----------------- | ------------------------ |
  | RepositoryService | activiti的资源管理类     |
  | RuntimeService    | activiti的流程运行管理类 |
  | TaskService       | activiti的任务管理类     |
  | HistoryService    | activiti的历史管理类     |
  | ManagerService    | activiti的引擎管理类     |

  进一步说明：

  * **RepositoryService**

    是activiti的资源管理类，提供了管理和控制流程发布包和流程定义的操作。

    使用工作流建模工具设计的业务流程图需要使用此service将流程定义文件的内容部署到计算机。

    除了部署流程定义以外还可以：

    * 查询引擎中的发布包和流程定义。

    * 暂停或激活发布包，对应全部和特定流程定义。暂停意味着它们不能再执行任何操作了，激活是对应的反向操作。
    * 获得多种资源，像是包含在发布包里的文件，或引擎自动生成的流程图。
    * 获得流程定义的pojo版本， 可以用来通过java解析流程，而不必通过xml。

  * **RuntimeService**：Activiti的流程运行管理类。可以从这个服务类中获取很多关于流程执行相关的信息。
  * **TaskService**：Activiti的任务管理类。可以从这个类中获取任务的信息。
  * **HistoryService**：Activiti的历史管理类，可以查询历史信息，执行流程时，引擎会保存很多数据（根据配置），比如流程实例启动时间，任务的参与者，完成任务的时间，每个流程实例的执行路径，等等；这个服务主要通过查询功能来获得这些数据。
  * **ManagementService**：Activiti的引擎管理类，提供了对 Activiti 流程引擎的管理和维护功能，这些功能不在工作流驱动的应用程序中使用，主要用于 Activiti 系统的日常维护。

## 3. Activiti流程操作

### 3.1 流程创建

此处创建一个Activiti工作流，并启动这个流程。

创建Activiti工作流主要包含以下几步：

1. 定义流程，按照BPMN的规范，使用流程定义工具，用**流程符号**把整个流程描述出来

2. 部署流程，把画好的流程定义文件，加载到数据库中，生成表的数据
3. 启动流程，使用java代码来操作数据库表中的内容

**流程符号**

* BPMN 2.0是业务流程建模符号2.0的缩写。

* 它由Business Process Management Initiative这个非营利协会创建并不断发展。作为一种标识，BPMN 2.0是使用一些**符号**来明确业务流程设计流程图的一整套符号规范，它能增进业务建模时的沟通效率。

* 目前BPMN2.0是最新的版本，它用于在BPM上下文中进行布局和可视化的沟通。

* BPMN2.0的**基本符号**主要包含：

  * **事件 Event：**

    ![事件](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400873.png)

  * **活动 Activity：**活动是工作或任务的一个通用术语。一个活动可以是一个任务，还可以是一个当前流程的子处理流程；其次，你还可以为活动指定不同的类型。常见活动如下：

    ![活动](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400050.png)

  * **网关 GateWay：**网关用来处理决策

    ![网关](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400642.png)

    > 排他网关 (x) 
    >
    > * 只有一条路径会被选择。流程执行到该网关时，按照输出流的顺序逐个计算，当条件的计算结果为true时，继续执行当前网关的输出流；
    > * 如果多条线路计算结果都是 true，则会执行第一个值为 true 的线路。如果所有网关计算结果没有true，则引擎会抛出异常。
    > * 排他网关需要和条件顺序流结合使用，default 属性指定默认顺序流，当所有的条件不满足时会执行默认顺序流。
    >
    > 并行网关 (+) 
    >
    > * 所有路径会被同时选择
    >
    >   拆分：并行执行所有输出顺序流，为每一条顺序流创建一个并行执行线路。
    >
    >   合并：所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。
    >
    > 包容网关 (+) 
    >
    > * 可以同时执行多条线路，也可以在网关上设置条件
    >
    >   拆分：计算每条线路上的表达式，当表达式计算结果为true时，创建一个并行线路并继续执行
    >
    >   合并：所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。
    >
    > 事件网关 (+) 
    >
    > * 专门为中间捕获事件设置的，允许设置多个输出流指向多个不同的中间捕获事件。当流程执行到事件网关后，流程处于等待状态，需要等待抛出事件才能将等待状态转换为活动状态。

  * **流向 Flow：**流是连接两个流程节点的连线。常见的流向包含以下几种

    ![流向](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400304.png)

### 3.2 流程设计器使用

> 这里指的是IDEA的插件actiBPM

1. 在resources创建bpmn目录，在bpmn目录中点击菜单：New  → BpmnFile，创建BpmnFile，起名为evection，成功创建文件：evection.bpmn文件；

2. 绘制流程：利用右侧的流程符号绘制流程，绘制成功后给这个流程指定一个key，如下：

   ![绘制bpmn图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400981.PNG)

3. 关于其中的任务节点，需要指定一个负责人，即Assignee属性，如下：

   ![责任人指定](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400486.PNG)

   > 后面的责任人这边分别指定为：经理审批负责人为 jerry、总经理审批负责人为 jack、财务审批负责人为 rose

创建完成后即成功的生成了evection.bpmn文件，

> BPMN 2.0根节点是definitions节点。这个元素中，可以定义多个流程定义（不过建议每个文件只包含一个流程定义，可以简化开发过程中的维护难度）。注意，definitions元素最少也要包含xmlns 和 targetNamespace 的声明。targetNamespace可以是任意值，它用来对流程实例进行分类。
>
> 流程定义部分：定义了流程每个结点的描述及结点之间的流程流转。
>
> 流程布局定义：定义流程每个结点在流程图上的位置坐标等信息。

根据 .bpmn 文件生成 .png 文件：

1. 将刚刚的文件修改后缀名为 .xml 文件；
2. 右键点击这个 .xml 文件，选择Diagrams→Show BPMN 2.0 Designer；
3. 在展示的视图中，将其进行导出为 .png 格式的图片，然后再将这个图片放到resources下的bpmn目录；
4. 最后在将修改为 .xml 后缀的文件修改回来 .bpmn。

### 3.3 流程定义部署

#### 3.1.1 布署方式

将上面在设计器中定义的流程部署到activiti数据库中，就是流程定义部署。

通过调用activiti的api将流程定义的bpmn和png两个文件一个一个添加部署到activiti中，也可以将两个文件打成zip包进行部署。

**单个文件部署方式：**分别将bpmn文件和png图片文件部署。

```java
/**
 * 部署流程定义
 */
@Test
public void testDeployment(){

    // 1、创建ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2、得到RepositoryService实例
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3、使用RepositoryService进行部署
    Deployment deployment = repositoryService.createDeployment()
        .addClasspathResource("bpmn/evection.bpmn") // 添加bpmn资源
        .addClasspathResource("bpmn/evection.png")  // 添加png资源
        .name("出差申请流程")
        .deploy();
    // 4、输出部署信息
    System.out.println("流程部署id：" + deployment.getId());
    System.out.println("流程部署名称：" + deployment.getName());
}
```

执行此操作后activiti会将上边代码中指定的bpm文件和图片文件保存在activiti数据库；

> 后面会根据控制台输出的日志信息，依据DQL来分析其操作操作的数据表

**压缩包部署方式：**将evection.bpmn和evection.png压缩成zip包。

```java
@Test
public void deployProcessByZip() {
    // 定义zip输入流
    InputStream inputStream = this
        .getClass()
        .getClassLoader()
        .getResourceAsStream("bpmn/evection.zip");
    ZipInputStream zipInputStream = new ZipInputStream(inputStream);
    // 获取repositoryService
    RepositoryService repositoryService = processEngine
        .getRepositoryService();
    // 流程部署
    Deployment deployment = repositoryService.createDeployment()
        .addZipInputStream(zipInputStream)
        .deploy();
    System.out.println("流程部署id：" + deployment.getId());
    System.out.println("流程部署名称：" + deployment.getName());
}
```

执行此操作后activiti会将上边代码中指定的bpm文件和图片文件保存在activiti数据库。

#### 3.2.2 操作数据表

通过对控制台中输出的SQL语句分析，有如下：

```bash
--- starting SchemaOperationsProcessEngineBuild ---
...
# 表ACT_GE_PROPERTY
updating: PropertyEntity[name=next.dbid, value=2501]
==>  Preparing: update ACT_GE_PROPERTY SET REV_ = ?, VALUE_ = ? where NAME_ = ? and REV_ = ? 
==> Parameters: 2(Integer), 2501(String), next.dbid(String), 1(Integer)
<==    Updates: 1
...
# 表ACT_RE_PROCDEF
==>  Preparing: insert into ACT_RE_PROCDEF(ID_, REV_, CATEGORY_, NAME_, KEY_, VERSION_, DEPLOYMENT_ID_, RESOURCE_NAME_, DGRM_RESOURCE_NAME_, DESCRIPTION_, HAS_START_FORM_KEY_, HAS_GRAPHICAL_NOTATION_ , SUSPENSION_STATE_, TENANT_ID_, ENGINE_VERSION_) values (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) 
==> Parameters: myEvection:1:4(String), http://www.activiti.org/testm1651322481124(String), 出差申请单(String), myEvection(String), 1(Integer), 1(String), bpmn/evection.bpmn(String), bpmn/evection.png(String), null, false(Boolean), true(Boolean), 1(Integer), (String), null
<==    Updates: 1
# 表ACT_RE_DEPLOYMENT
==>  Preparing: insert into ACT_RE_DEPLOYMENT(ID_, NAME_, CATEGORY_, KEY_, TENANT_ID_, DEPLOY_TIME_, ENGINE_VERSION_) values(?, ?, ?, ?, ?, ?, ?) 
==> Parameters: 1(String), 出差申请流程(String), null, null, (String), 2022-04-30 20:53:03.07(Timestamp), null
<==    Updates: 1
# 表ACT_GE_BYTEARRAY
==>  Preparing: INSERT INTO ACT_GE_BYTEARRAY(ID_, REV_, NAME_, BYTES_, DEPLOYMENT_ID_, GENERATED_) VALUES (?, 1, ?, ?, ?, ?) , (?, 1, ?, ?, ?, ?) 
==> Parameters: 2(String), bpmn/evection.png(String), java.io.ByteArrayInputStream@435fb7b5(ByteArrayInputStream), 1(String), false(Boolean), 3(String), bpmn/evection.bpmn(String), java.io.ByteArrayInputStream@4e70a728(ByteArrayInputStream), 1(String), false(Boolean)
<==    Updates: 2
```

> 这里的话忽略了查询的表，只看了有数据修改的表

可以看到这里的话操作了4张表：

* ACT_GE_PROPERTY：系统相关属性，每次操作其实都会操作这个表，这里不进行分析

* ACT_RE_PROCDEF：流程定义表，部署每个新的流程定义都会在这张表中增加一条记录

  > ```mysql
  > -- 看下插入了什么：
  > ID_	REV_	CATEGORY_	NAME_	KEY_	VERSION_	DEPLOYMENT_ID_	RESOURCE_NAME_	DGRM_RESOURCE_NAME_	DESCRIPTION_	HAS_START_FORM_KEY_	HAS_GRAPHICAL_NOTATION_	SUSPENSION_STATE_	TENANT_ID_	ENGINE_VERSION_
  > myEvection:1:4	1	http://www.activiti.org/testm1651322481124	出差申请单	myEvection	1	1	bpmn/evection.bpmn	bpmn/evection.png	\N	0	1	1		\N
  > 
  > -- 注意：KEY 这个字段是用来唯一识别不同流程的关键字
  > ```

* ACT_RE_DEPLOYMENT：流程定义表，部署每个新的流程定义都会在这张表中增加一条记录

  ```mysql
  -- 看下插入了什么：
  ID_	NAME_	CATEGORY_	KEY_	TENANT_ID_	DEPLOY_TIME_	ENGINE_VERSION_
  1	出差申请流程	\N	\N		2022-05-01 16:15:29	\N
  ```

* ACT_GE_BYTEARRAY：流程资源表，刚刚的 .bpmn 文件和 .png 文件的保存

  ![ACT_GE_BYTEARRAY表数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400835.PNG)

act_re_deployment和act_re_procdef一对多关系：一次部署在流程部署表生成一条记录，但一次部署可以部署多个流程定义，每个流程定义在流程定义表生成一条记录。每一个流程定义在act_ge_bytearray会存在两个资源记录，bpmn和png。建议的话，一次部署一个流程，这样部署表和流程定义表是一对一有关系，方便读取流程部署及流程定义信息。

### 3.4 启动流程实例

流程定义部署在activiti后就可以通过工作流管理业务流程了，也就是说上边部署的出差申请流程可以使用了。

针对该流程，启动一个流程表示发起一个新的出差申请单，这就相当于java类与java对象的关系，类定义好后需要new创建一个对象使用，当然可以new多个对象。

对于请出差申请流程，张三发起一个出差申请单需要启动一个流程实例，出差申请单发起一个出差单也需要启动一个流程实例。

```java
/**
 * 启动流程实例
 */
@Test
public void testStartProcess(){

    // 1、创建ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2、获取RunTimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 3、根据流程定义Id启动流程
    ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myEvection");
    // 4、输出内容
    System.out.println("流程定义id：" + processInstance.getProcessDefinitionId());
    System.out.println("流程实例id：" + processInstance.getId());
    System.out.println("当前活动Id：" + processInstance.getActivityId());
}

// 输出：
// 流程定义id：myEvection:1:4
// 流程实例id：2501
// 当前活动Id：null
```

还是通过对控制台中输出的SQL语句分析，有如下：

```bash
# 操作表：ACT_HI_TASKINST
==>  Preparing: insert into ACT_HI_TASKINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, NAME_, PARENT_TASK_ID_, DESCRIPTION_, OWNER_, ASSIGNEE_, START_TIME_, CLAIM_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TASK_DEF_KEY_, FORM_KEY_, PRIORITY_, DUE_DATE_, CATEGORY_, TENANT_ID_ ) values ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
==> Parameters: 2505(String), myEvection:1:4(String), 2501(String), 2502(String), 创建出差申请(String), null, null, null, zhangsan(String), 2022-05-01 16:39:23.449(Timestamp), null, null, null, null, _3(String), null, 50(Integer), null, null, (String)
<==    Updates: 1
...
# 操作表：ACT_HI_PROCINST
==>  Preparing: insert into ACT_HI_PROCINST ( ID_, PROC_INST_ID_, BUSINESS_KEY_, PROC_DEF_ID_, START_TIME_, END_TIME_, DURATION_, START_USER_ID_, START_ACT_ID_, END_ACT_ID_, SUPER_PROCESS_INSTANCE_ID_, DELETE_REASON_, TENANT_ID_, NAME_ ) values ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
==> Parameters: 2501(String), 2501(String), null, myEvection:1:4(String), 2022-05-01 16:39:23.417(Timestamp), null, null, null, _2(String), null, null, null, (String), null
<==    Updates: 1
...
# 操作表：ACT_HI_ACTINST
==>  Preparing: insert into ACT_HI_ACTINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, ACT_ID_, TASK_ID_, CALL_PROC_INST_ID_, ACT_NAME_, ACT_TYPE_, ASSIGNEE_, START_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TENANT_ID_ ) values (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) , (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) 
==> Parameters: 2503(String), myEvection:1:4(String), 2501(String), 2502(String), _2(String), null, null, StartEvent(String), startEvent(String), null, 2022-05-01 16:39:23.426(Timestamp), 2022-05-01 16:39:23.428(Timestamp), 2(Long), null, (String), 2504(String), myEvection:1:4(String), 2501(String), 2502(String), _3(String), 2505(String), null, 创建出差申请(String), userTask(String), zhangsan(String), 2022-05-01 16:39:23.429(Timestamp), null, null, null, (String)
<==    Updates: 2
...
# 操作表：ACT_HI_IDENTITYLINK
==>  Preparing: insert into ACT_HI_IDENTITYLINK (ID_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_) values (?, ?, ?, ?, ?, ?)
==> Parameters: 2506(String), participant(String), zhangsan(String), null, null, 2501(String)
<==    Updates: 1
...
# 操作表：ACT_RU_EXECUTION
==>  Preparing: insert into ACT_RU_EXECUTION (ID_, REV_, PROC_INST_ID_, BUSINESS_KEY_, PROC_DEF_ID_, ACT_ID_, IS_ACTIVE_, IS_CONCURRENT_, IS_SCOPE_,IS_EVENT_SCOPE_, IS_MI_ROOT_, PARENT_ID_, SUPER_EXEC_, ROOT_PROC_INST_ID_, SUSPENSION_STATE_, TENANT_ID_, NAME_, START_TIME_, START_USER_ID_, IS_COUNT_ENABLED_, EVT_SUBSCR_COUNT_, TASK_COUNT_, JOB_COUNT_, TIMER_JOB_COUNT_, SUSP_JOB_COUNT_, DEADLETTER_JOB_COUNT_, VAR_COUNT_, ID_LINK_COUNT_) values (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) , (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?) 
==> Parameters: 2501(String), 2501(String), null, myEvection:1:4(String), null, true(Boolean), false(Boolean), true(Boolean), false(Boolean), false(Boolean), null, null, 2501(String), 1(Integer), (String), null, 2022-05-01 16:39:23.417(Timestamp), null, false(Boolean), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 2502(String), 2501(String), null, myEvection:1:4(String), _3(String), true(Boolean), false(Boolean), false(Boolean), false(Boolean), false(Boolean), 2501(String), null, 2501(String), 1(Integer), (String), null, 2022-05-01 16:39:23.425(Timestamp), null, false(Boolean), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer)
<==    Updates: 2
...
# 操作表：ACT_RU_TASK
inserting: Task[id=2505, name=创建出差申请]
==>  Preparing: insert into ACT_RU_TASK (ID_, REV_, NAME_, PARENT_TASK_ID_, DESCRIPTION_, PRIORITY_, CREATE_TIME_, OWNER_, ASSIGNEE_, DELEGATION_, EXECUTION_ID_, PROC_INST_ID_, PROC_DEF_ID_, TASK_DEF_KEY_, DUE_DATE_, CATEGORY_, SUSPENSION_STATE_, TENANT_ID_, FORM_KEY_, CLAIM_TIME_) values (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
==> Parameters: 2505(String), 创建出差申请(String), null, null, 50(Integer), 2022-05-01 16:39:23.429(Timestamp), null, zhangsan(String), null, 2502(String), 2501(String), myEvection:1:4(String), _3(String), null, null, 1(Integer), (String), null, null
<==    Updates: 1
...
# 操作表：ACT_RU_IDENTITYLINK
==>  Preparing: insert into ACT_RU_IDENTITYLINK (ID_, REV_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_, PROC_DEF_ID_) values (?, 1, ?, ?, ?, ?, ?, ?)
==> Parameters: 2506(String), participant(String), zhangsan(String), null, null, 2501(String), null
<==    Updates: 1
```

可以看到，这里总共操作了表：

* ACT_HI_TASKINST：流程任务历史信息

  ```
  ID_	PROC_DEF_ID_	TASK_DEF_KEY_	PROC_INST_ID_	EXECUTION_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	OWNER_	ASSIGNEE_	START_TIME_	CLAIM_TIME_	END_TIME_	DURATION_	DELETE_REASON_	PRIORITY_	DUE_DATE_	FORM_KEY_	CATEGORY_	TENANT_ID_
  2505	myEvection:1:4	_3	2501	2502	创建出差申请	\N	\N	\N	zhangsan	2022-05-01 16:39:23	\N	\N	\N	\N	50	\N	\N	\N	
  ```

  > 这里可以看到这个task为**创建出差申请**，且现在所处的状态为刚刚开始还没走完，因此只有开始时间，没有结束时间；

* ACT_HI_PROCINST：这条流程实例的历史信息

  ```mysql
  ID_	PROC_INST_ID_	BUSINESS_KEY_	PROC_DEF_ID_	START_TIME_	END_TIME_	DURATION_	START_USER_ID_	START_ACT_ID_	END_ACT_ID_	SUPER_PROCESS_INSTANCE_ID_	DELETE_REASON_	TENANT_ID_	NAME_
  2501	2501	\N	myEvection:1:4	2022-05-01 16:39:23	\N	\N	\N	_2	\N	\N	\N		\N
  ```

* ACT_HI_ACTINST：流程实例执行历史

  ```mysql
  ID_	PROC_DEF_ID_	PROC_INST_ID_	EXECUTION_ID_	ACT_ID_	TASK_ID_	CALL_PROC_INST_ID_	ACT_NAME_	ACT_TYPE_	ASSIGNEE_	START_TIME_	END_TIME_	DURATION_	DELETE_REASON_	TENANT_ID_
  2503	myEvection:1:4	2501	2502	_2	\N	\N	StartEvent	startEvent	\N	2022-05-01 16:39:23	2022-05-01 16:39:23	2	\N	
  2504	myEvection:1:4	2501	2502	_3	2505	\N	创建出差申请	userTask	zhangsan	2022-05-01 16:39:23	\N	\N	\N	
  ```

  > 两个节点，一个是开始StartEvent（已完成，有开始和结束时间），一个是创建出差申请（还没结束）；

* ACT_HI_IDENTITYLINK：流程的参与用户历史信息

  ```mysql
  ID_	GROUP_ID_	TYPE_	USER_ID_	TASK_ID_	PROC_INST_ID_
  2506	\N	participant	zhangsan	\N	2501
  ```

  > 可以看到这个流程的需要zhagnsan进行一个审批处理

* ACT_RU_EXECUTION：流程正在执行信息

  ```mysql
  ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
  2501	1	2501	\N	\N	myEvection:1:4	\N	2501	\N	1	0	1	0	0	1	\N		\N	2022-05-01 16:39:23	\N	\N	0	0	0	0	0	0	0	0	0
  2502	1	2501	\N	2501	myEvection:1:4	\N	2501	_3	1	0	0	0	0	1	\N		\N	2022-05-01 16:39:23	\N	\N	0	0	0	0	0	0	0	0	0
  ```

* ACT_RU_TASK：任务信息

  ```mysql
  ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
  2505	1	2502	2501	myEvection:1:4	创建出差申请	\N	\N	_3	\N	zhangsan	\N	50	2022-05-01 16:39:23	\N	\N	1		\N	\N
  ```

* ACT_RU_IDENTITYLINK：流程的参与用户信息

  ```mysql
  ID_	REV_	GROUP_ID_	TYPE_	USER_ID_	TASK_ID_	PROC_INST_ID_	PROC_DEF_ID_
  2506	1	\N	participant	zhangsan	\N	2501	\N
  ```

### 3.5 任务查询

流程启动后，任务的负责人就可以查询自己当前需要处理的任务，查询出来的任务都是该用户的待办任务。

```java
/**
 * 查询当前个人待执行的任务
 */
@Test
public void testFindPersonalTaskList(){

    // 任务负责人
    String assignee = "zhangsan";  // 从上面执行下来看的话，现在就是zhagnsan在处理
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    // 根据流程key 和 任务负责人 查询任务
    List<Task> list = taskService.createTaskQuery()
        .processDefinitionKey("myEvection") //流程Key
        .taskAssignee(assignee)//只查询该任务负责人的任务
        .list();

    for (Task task : list) {
        System.out.println("流程实例id：" + task.getProcessInstanceId());
        System.out.println("任务id：" + task.getId());
        System.out.println("任务负责人：" + task.getAssignee());
        System.out.println("任务名称：" + task.getName());
    }
}

// 输出结果如下：
// 流程实例id：2501
// 任务id：2505
// 任务负责人：zhangsan
// 任务名称：创建出差申请
```

还是老样子分析SQL：

```bash
==>  Preparing: select distinct RES.* from ACT_RU_TASK RES inner join ACT_RE_PROCDEF D on RES.PROC_DEF_ID_ = D.ID_ WHERE RES.ASSIGNEE_ = ? and D.KEY_ = ? order by RES.ID_ asc LIMIT ? OFFSET ? 
==> Parameters: zhangsan(String), myEvection(String), 2147483647(Integer), 0(Integer)
<==      Total: 1
```

对上述控制台输出的内容组合一下：

```mysql
-- 对应的sql：查了表ACT_RU_TASK（任务表），ACT_RE_PROCDEF（流程定义表）
select distinct RES.* from ACT_RU_TASK RES inner join ACT_RE_PROCDEF D on RES.PROC_DEF_ID_ = D.ID_ WHERE RES.ASSIGNEE_ = 'zhangsan' and D.KEY_ = 'myEvection' order by RES.ID_ asc LIMIT 2147483647 OFFSET 0;

-- 输出结果：
ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
2505	1	2502	2501	myEvection:1:4	创建出差申请	\N	\N	_3	\N	zhangsan	\N	50	2022-05-01 16:39:23	\N	\N	1		\N	\N
```

### 3.6 流程任务处理

任务负责人查询待办任务，选择任务进行处理，完成任务。

```java
/**
 * 任务负责人查询待办任务，选择任务进行处理，完成任务。
 */
@Test
public void completTask(){
    // 获取引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取taskService
    TaskService taskService = processEngine.getTaskService();

    // 根据流程key 和 任务的负责人 查询任务
    // 返回一个任务对象
    Task task = taskService.createTaskQuery()
        .processDefinitionKey("myEvection") // 流程Key
        .taskAssignee("zhangsan")   // 要查询的负责人
        .singleResult();

    // 完成任务,参数：任务id
    taskService.complete(task.getId());
}
```

处理完任务后，表的状态是有所修改的：

* ACT_HI_TASKINST：这里相比于上面，多了一条数据了，就是“经理审批”，然后上一数据的end_time已经有值了

  ```mysql
  ID_	PROC_DEF_ID_	TASK_DEF_KEY_	PROC_INST_ID_	EXECUTION_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	OWNER_	ASSIGNEE_	START_TIME_	CLAIM_TIME_	END_TIME_	DURATION_	DELETE_REASON_	PRIORITY_	DUE_DATE_	FORM_KEY_	CATEGORY_	TENANT_ID_
  2505	myEvection:1:4	_3	2501	2502	创建出差申请	\N	\N	\N	zhangsan	2022-05-01 16:39:23	\N	2022-05-01 17:13:28	2045436	\N	50	\N	\N	\N	
  5002	myEvection:1:4	_4	2501	2502	经理审批	\N	\N	\N	jerry	2022-05-01 17:13:28	\N	\N	\N	\N	50	\N	\N	\N	
  ```

* ACT_HI_IDENTITYLINK：同样，这里多了一条数据

  ```mysql
  ID_	GROUP_ID_	TYPE_	USER_ID_	TASK_ID_	PROC_INST_ID_
  2506	\N	participant	zhangsan	\N	2501
  5003	\N	participant	jerry	\N	2501
  ```

* ACT_HI_ACTINST：这里同样多了一条数据“经理审批”的数据，然后上一条“创建出差申请”的任务已经有了结束时间

  ```mysql
  ID_	PROC_DEF_ID_	PROC_INST_ID_	EXECUTION_ID_	ACT_ID_	TASK_ID_	CALL_PROC_INST_ID_	ACT_NAME_	ACT_TYPE_	ASSIGNEE_	START_TIME_	END_TIME_	DURATION_	DELETE_REASON_	TENANT_ID_
  2503	myEvection:1:4	2501	2502	_2	\N	\N	StartEvent	startEvent	\N	2022-05-01 16:39:23	2022-05-01 16:39:23	2	\N	
  2504	myEvection:1:4	2501	2502	_3	2505	\N	创建出差申请	userTask	zhangsan	2022-05-01 16:39:23	2022-05-01 17:13:28	2045454	\N	
  5001	myEvection:1:4	2501	2502	_4	5002	\N	经理审批	userTask	jerry	2022-05-01 17:13:28	\N	\N	\N	
  ```

* ACT_RU_EXECUTION：流程正在执行信息，这里的数据量是不变的，因为目前就是一条流程在执行，但可以看到 act_id变了，表示的就是下一个任务（刚刚是_3，现在是 _4）

  ```mysql
  ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
  2501	1	2501	\N	\N	myEvection:1:4	\N	2501	\N	1	0	1	0	0	1	\N		\N	2022-05-01 16:39:23	\N	\N	0	0	0	0	0	0	0	0	0
  2502	2	2501	\N	2501	myEvection:1:4	\N	2501	_4	1	0	0	0	0	1	\N		\N	2022-05-01 16:39:23	\N	\N	0	0	0	0	0	0	0	0	0
  ```

* ACT_RU_TASK：可以看到这个表的数据已经变成了下一个任务节点"经理审批"

  ```mysql
  ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
  5002	1	2502	2501	myEvection:1:4	经理审批	\N	\N	_4	\N	jerry	\N	50	2022-05-01 17:13:28	\N	\N	1		\N	\N
  ```

* ACT_RU_IDENTITYLINK：可以看到多了一个任务负责人的数据

  ```mysql
  ID_	REV_	GROUP_ID_	TYPE_	USER_ID_	TASK_ID_	PROC_INST_ID_	PROC_DEF_ID_
  2506	1	\N	participant	zhangsan	\N	2501	\N
  5003	1	\N	participant	jerry	\N	2501	\N
  ```

继续处理，把这个任务完成处理完：

```java
Task task = taskService.createTaskQuery()
                .processDefinitionKey("myEvection") //流程Key
                .taskAssignee("zhangsan")  //要查询的负责人
                .singleResult();
                
Task task = taskService.createTaskQuery()
                .processDefinitionKey("myEvection") //流程Key
                .taskAssignee("jerry")  //要查询的负责人
                .singleResult();

Task task = taskService.createTaskQuery()
                .processDefinitionKey("myEvection") //流程Key
                .taskAssignee("jack")  //要查询的负责人
                .singleResult();

Task task = taskService.createTaskQuery()
                .processDefinitionKey("myEvection") //流程Key
                .taskAssignee("rose")  //要查询的负责人
                .singleResult();
```

再看数据库的表信息：

- ACT_HI_TASKINST：历史的任务实例，可以看到一个流程包含了4个task；

  ![流程完成：ACT_HI_TASKINST数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400051.PNG)

- ACT_HI_IDENTITYLINK：历史的流程运行过程中用户关系，同样是有4个任务负责人；

  ![流程完成：ACT_HI_IDENTITYLINK数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400313.PNG)

- ACT_HI_ACTINST：历史的流程实例，这里还有开始和结束两个事件；

  ![流程完成：ACT_HI_ACTINST数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400955.PNG)

- ACT_HI_PROCINST：历史的流程实例，已经完成了

  ![流程完成：ACT_HI_PROCINST数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400543.PNG)

- ACT_RU_EXECUTION：为空了，因为没有流程已经执行完成，当前没有在执行的流程；

- ACT_RU_TASK：同样为空；
- ACT_RU_IDENTITYLINK：同样为空。

### 3.7 流程定义信息查询

查询流程相关信息，包含流程定义，流程部署，流程定义版本

```java
/**
* 查询流程定义
*/
@Test
public void queryProcessDefinition(){
    // 1.获取引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2.repositoryService
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3.得到ProcessDefinitionQuery 对象
    ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery();
    // 4.查询出当前所有的流程定义
    List<ProcessDefinition> definitionList = processDefinitionQuery
        .processDefinitionKey("myEvection") // 条件：processDefinitionKey=evection
        .orderByProcessDefinitionVersion()  // orderByProcessDefinitionVersion 按照版本排序
        .desc()                             // desc倒叙
        .list();                            // list 返回集合
    // 输出流程定义信息
    for (ProcessDefinition processDefinition : definitionList) {
        System.out.println("流程定义 id="+processDefinition.getId());
        System.out.println("流程定义 name="+processDefinition.getName());
        System.out.println("流程定义 key="+processDefinition.getKey());
        System.out.println("流程定义 Version="+processDefinition.getVersion());
        System.out.println("流程部署ID ="+processDefinition.getDeploymentId());
    }
}

// 输出：
// 流程定义 id=myEvection:1:4
// 流程定义 name=出差申请单
// 流程定义 key=myEvection
// 流程定义 Version=1
// 流程部署ID =1
```

> 下面就不分析SQL了，这毕竟是流程框架都做的事情

### 3.8 流程删除

```java
/**
     * 流程删除
     */
@Test
public void deleteDeployment() {
    // 流程部署id，这id就是表act_re_deployment表的id
    String deploymentId = "1";

    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 通过流程引擎获取repositoryService
    RepositoryService repositoryService = processEngine
        .getRepositoryService();
    // 删除流程定义，如果该流程定义已有流程实例启动则删除时出错
    repositoryService.deleteDeployment(deploymentId);
    // 设置true 级联删除流程定义，即使该流程有流程实例启动也可以删除，设置为false非级别删除方式，如果流程
    // repositoryService.deleteDeployment(deploymentId, true);
}
```

> 说明：
>
> 1. 使用repositoryService删除流程定义，历史表信息不会被删除；
>
> 2. 如果该流程定义下没有正在运行的流程，则可以用普通删除。
>
>    如果该流程定义下存在已经运行的流程，使用普通删除报错，可用级联删除方法将流程及相关记录全部删除。
>
>    先删除没有完成流程节点，最后就可以完全删除流程定义信息
>
>    项目开发中级联删除操作一般只开放给超级管理员使用.

### 3.9 流程资源下载

现在流程资源文件已经上传到数据库了，如果其他用户想要查看这些资源文件，可以从数据库中把资源文件下载到本地。

> 就是表ACT_GE_BYTEARRAY中的数据：
>
> ![ACT_GE_BYTEARRAY表数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261400374.PNG)

解决方案有：

1. jdbc对blob类型，clob类型数据读取出来，保存到文件目录；

2. 使用activiti的api来实现

> 使用commons-io.jar 解决IO的操作，需要引入commons-io依赖包
>
> ```xml
> <dependency>
>     <groupId>commons-io</groupId>
>     <artifactId>commons-io</artifactId>
>     <version>2.6</version>
> </dependency>
> ```

通过流程定义对象获取流程定义资源，获取bpmn和png

```java
/**
 * 流程资源下载
 */
@Test
public void  queryBpmnFile() throws IOException {
    // 1、得到引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2、获取repositoryService
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 3、得到查询器：ProcessDefinitionQuery，设置查询条件,得到想要的流程定义
    ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
        .processDefinitionKey("myEvection")
        .singleResult();
    // 4、通过流程定义信息，得到部署ID
    String deploymentId = processDefinition.getDeploymentId();
    System.out.println("流程布署ID：" + deploymentId);
    // 5、通过repositoryService的方法，实现读取图片信息和bpmn信息
    //   png图片的流
    InputStream pngInput = repositoryService.getResourceAsStream(deploymentId, processDefinition.getDiagramResourceName());
    //    bpmn文件的流
    InputStream bpmnInput = repositoryService.getResourceAsStream(deploymentId, processDefinition.getResourceName());
    // 6、构造OutputStream流
    File file_png = new File("C:\\Users\\ZhuCC\\Desktop\\evectionflow01.png");
    File file_bpmn = new File("C:\\Users\\ZhuCC\\Desktop\\evectionflow01.bpmn");
    FileOutputStream bpmnOut = new FileOutputStream(file_bpmn);
    FileOutputStream pngOut = new FileOutputStream(file_png);
    // 7、输入流，输出流的转换
    IOUtils.copy(pngInput,pngOut);
    IOUtils.copy(bpmnInput,bpmnOut);
    // 8、关闭流
    pngOut.close();
    bpmnOut.close();
    pngInput.close();
    bpmnInput.close();
}
```

> 说明：
>
> 1. deploymentId为流程部署ID
> 2. resource_name为act_ge_bytearray表中NAME_列的值
> 3. 使用repositoryService的getDeploymentResourceNames方法可以获取指定部署下得所有文件的名称
>
> 4. 使用repositoryService的getResourceAsStream方法传入部署ID和资源图片名称可以获取部署下指定名称文件的输入流
>
> 最后的将输入流中的图片资源进行输出。

### 3.10 流程历史信息的查看

即使流程定义已经删除了，流程执行的历史信息通过前面的分析，依然保存在activiti的act_hi_*相关的表中。

所以还是可以查询流程执行的历史信息，可以通过HistoryService来查看相关的历史记录。

```java
/**
     * 流程历史信息的查看
     */
@Test
public void findHistoryInfo(){
    // 获取引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取HistoryService
    HistoryService historyService = processEngine.getHistoryService();
    //  获取 actinst表的查询对象
    HistoricActivityInstanceQuery instanceQuery = historyService.createHistoricActivityInstanceQuery();
    //  查询 actinst表，条件：根据 InstanceId 查询
    //  instanceQuery.processInstanceId("2501");
    // 查询 actinst表，条件：根据 DefinitionId 查询
    instanceQuery.processDefinitionId("myEvection:1:4");
    // 增加排序操作,orderByHistoricActivityInstanceStartTime 根据开始时间排序 asc 升序
    instanceQuery.orderByHistoricActivityInstanceStartTime().asc();
    // 查询所有内容
    List<HistoricActivityInstance> activityInstanceList = instanceQuery.list();
    // 输出
    for (HistoricActivityInstance hi : activityInstanceList) {
        System.out.println(hi.getActivityId());
        System.out.println(hi.getActivityName());
        System.out.println(hi.getProcessDefinitionId());
        System.out.println(hi.getProcessInstanceId());
        System.out.println("<==========================>");
    }
}
```

输出结果：

```bash
_2
StartEvent
myEvection:1:4
2501
<==========================>
_3
创建出差申请
myEvection:1:4
2501
<==========================>
_4
经理审批
myEvection:1:4
2501
<==========================>
_5
总经理审批
myEvection:1:4
2501
<==========================>
_6
财务审批
myEvection:1:4
2501
<==========================>
_7
EndEvent
myEvection:1:4
2501
<==========================>
```

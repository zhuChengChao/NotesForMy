# 工作流：Activiti提升

> 工作流学习第二弹:open_mouth:
>
> > 原视频地址：https://www.bilibili.com/video/BV1H54y167gf

## 1. 流程实例

### 1.1 概述

**流程实例**（ProcessInstance）代表流程定义的执行实例。一个流程实例包括了所有的运行节点，可以利用这个对象来了解当前流程实例的进度等信息。

> 例如：用户或程序按照流程定义内容发起一个流程，这就是一个流程实例。

流程定义和流程实例的图解：

![流程定义和流程实例图解](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401116.png)

### 1.2 Businesskey

#### 1.2.1 概述

流程定义部署在activiti后，就可以在系统中通过activiti去管理该流程的执行，执行流程表示流程的一次执行。

> 比如部署系统出差流程后，如果某用户要申请出差这时就需要执行这个流程；如果另外一个用户也要申请出差则也需要执行该流程；每个执行互不影响，每个执行是单独的流程实例。

启动流程实例时，指定的businesskey，就会在act_ru_execution（流程实例的执行表）中存储businesskey。

**Businesskey：业务标识**，通常为业务表的主键，业务标识和流程实例一一对应，业务标识来源于业务系统，存储业务标识就是根据业务标识来关联查询业务系统的数据。

> 比如：出差流程启动一个流程实例，就可以将出差单的id作为业务标识businesskey存储到activiti中，将来查询activiti的流程实例信息就可以获取出差单的id从而关联查询业务系统数据库得到出差单信息。

```java
/**
 * 启动流程实例，添加businessKey
 */
@Test
public void addBussinessKey(){
    // 1、得到ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 2、得到RunTimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 3.1、启动流程实例，同时还要指定业务标识businessKey，也就是出差申请单id，这里是1001
    ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myEvection", "1001");
    // 3.2、再启动一个，这次的businessKey指定为2001
    // ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myEvection", "2001");
    // 4、输出processInstance相关属性
    System.out.println("业务id="+processInstance.getBusinessKey());
}
```

Activiti的act_ru_execution中存储业务标识：

![act_ru_execution中存储业务标识](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401577.PNG)

#### 1.2.2 数据库操作

启动流程实例，会操作如下数据库表：

**表：act_ru_execution**

```mysql
SELECT * FROM act_ru_execution WHERE business_key_ = 1001;  # 流程实例执行表，记录当前流程实例的执行情况

ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
17501	1	17501	1001	\N	myEvection:1:15004	\N	17501	\N	1	0	1	0	0	1	\N		\N	2022-05-01 21:01:45	\N	\N	0	0	0	0	0	0	0	0	0

-- 这里只有一个分支，主键id=17501和流程实例id=17501，相同
```

> 说明：流程实例执行，如果当前只有一个分支时，一个流程实例只有一条记录且执行表的主键id和流程实例id相同；如果当前有多个分支正在运行则该执行表中有多条记录，存在执行表的主键和流程实例id不相同的记录。**但，不论当前有几个分支总会有一条记录的执行表的主键和流程实例id相同**
>
> 一个流程实例运行完成，此表中与流程实例相关的记录删除。

**表：act_ru_task** 

```mysql
SELECT * FROM act_ru_task  # 任务执行表，记录当前执行的任务
```

![act_ru_task表数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401222.PNG)

> 说明：启动流程实例，流程当前执行到第一个任务结点，此表会插入一条记录表示当前任务的执行情况，如果任务完成则记录删除。

**表：act_ru_identitylink** 

```mysql
SELECT * FROM act_ru_identitylink  # 任务参与者，记录当前参与任务的用户或组
```

![act_ru_identitylink 表数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401447.PNG)

**表：act_hi_procinst** 

```mysql
SELECT * FROM act_hi_procinst  # 流程实例历史表
```

![act_hi_procinst数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401175.PNG)

> 说明：流程实例启动，会在此表插入一条记录，流程实例运行完成记录也不会删除。

**表：act_hi_taskinst**

```mysql
 SELECT * FROM act_hi_taskinst  #任务历史表，记录所有任务
```

![act_hi_taskinst表数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401879.PNG)

> 说明：开始一个任务，不仅在act_ru_task表插入记录，也会在历史任务表插入一条记录，任务历史表的主键就是任务id，任务完成此表记录不删除。

**表：act_hi_actinst**

```mysql
SELECT * FROM act_hi_actinst  # 活动历史表，记录所有活动
```

![act_hi_actinst表数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401620.PNG)

> 说明：活动包括任务，所以此表中不仅记录了任务，还记录了流程执行过程的其它活动，比如开始事件、结束事件。

#### 1.2.3 查询流程实例

流程在运行过程中可以查询流程实例的状态，当前运行结点等信息。

```java
/**
 * 查询流程实例
 */
@Test
public void queryProcessInstance() {
    // 流程定义key
    String processDefinitionKey = "myEvection";
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RunTimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    List<ProcessInstance> list = runtimeService
        .createProcessInstanceQuery()
        .processDefinitionKey(processDefinitionKey)//
        .list();

    for (ProcessInstance processInstance : list) {
        System.out.println("----------------------------");
        System.out.println("流程实例id："
                           + processInstance.getProcessInstanceId());
        System.out.println("所属流程定义id："
                           + processInstance.getProcessDefinitionId());
        System.out.println("是否执行完成：" + processInstance.isEnded());
        System.out.println("是否暂停：" + processInstance.isSuspended());
        System.out.println("当前活动标识：" + processInstance.getActivityId());
    }
}
```

输出内容：（在上面对启动流程实例，添加businessKey代码中，启动了两个流程实例）

```bash
----------------------------
流程实例id：17501
所属流程定义id：myEvection:1:15004
是否执行完成：false
是否暂停：false
当前活动标识：null
----------------------------
流程实例id：20001
所属流程定义id：myEvection:1:15004
是否执行完成：false
是否暂停：false
当前活动标识：null
```

#### 1.2.4 关联 BusinessKey

**需求：**在activiti实际应用时，查询流程实例列表时可能要显示出业务系统的一些相关信息，比如：查询当前运行的出差流程列表需要将出差单名称、出差天数等信息显示出来，出差天数等信息在业务系统中存在，而并没有在activiti数据库中存在，所以是无法通过activiti的api查询到出差天数等信息。

**实现：**

* 在查询流程实例时，通过businessKey（业务标识 ）关联查询业务系统的出差单表，查询出出差天数等信息。

* 通过下面的代码就可以获取activiti中所对应实例保存的业务Key。而这个业务Key一般都会保存相关联的业务操作表的主键，再通过主键ID去查询业务信息，比如通过出差单的ID，去查询更多的请假信息（出差人，出差时间，出差天数，出差目的地等）
* `String businessKey = processInstance.getBusinessKey();`

![act_ru_execution中存储业务标识](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401165.PNG)

### 1.3 挂起&激活流程实例

某些情况可能由于流程变更需要将当前运行的流程暂停而不是直接删除，流程暂停后将不会继续执行。

#### 1.3.1 全部流程实例挂起

操作流程定义为挂起状态，该流程定义下边所有的流程实例全部暂停；即：流程定义为挂起状态该流程定义将不允许启动新的流程实例，同时该流程定义下所有的流程实例将全部挂起暂停执行。

```java
/**
  * 全部流程实例挂起与激活
  */
@Test
public void SuspendAllProcessInstance(){

    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取repositoryService
    RepositoryService repositoryService = processEngine.getRepositoryService();
    // 查询流程定义的对象
    ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery().
        processDefinitionKey("myEvection").
        singleResult();
    // 得到当前流程定义的实例是否都为暂停状态
    boolean suspended = processDefinition.isSuspended();
    // 流程定义id
    String processDefinitionId = processDefinition.getId();
    // 判断是否为暂停
    if(suspended){
        // 如果是暂停，可以执行激活操作:param1 ：流程定义id ，param12：是否激活，param13：激活时间
        repositoryService.activateProcessDefinitionById(processDefinitionId, true, null
                                                       );
        System.out.println("流程定义："+processDefinitionId+",已激活");
    }else{
        // 如果是激活状态，可以暂停，参数1 ：流程定义id ，参数2：是否暂停，参数3：暂停时间
        repositoryService.suspendProcessDefinitionById(processDefinitionId,
                                                       true,
                                                       null);
        System.out.println("流程定义："+processDefinitionId+",已挂起");
    }
}
```

挂起前act_ru_execution&act_ru_task表的状态，即激活态：

![当前的激活状态：act_ru_execution](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401701.PNG)

![当前流程的激活状态：act_ru_task](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401111.PNG)

挂起后act_ru_execution&act_ru_task表的状态：

![当前的挂起状态：act_ru_execution](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401657.PNG)

![当前流程的挂起状态：act_ru_task](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261401617.PNG)

> 可以看到 `SUSPENSION_STATE_` 的状态

#### 1.3.2 单个流程实例挂起

操作流程实例对象，针对单个流程执行挂起操作，某个流程实例挂起则此流程不再继续执行，完成该流程实例的当前任务将报异常。

```java
/**
 * 单个流程实例挂起与激活
 */
@Test
public void suspendSingleProcessInstance(){

    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 查询流程定义的对象
    ProcessInstance processInstance = runtimeService
        .createProcessInstanceQuery()
        .processInstanceId("17501")
        .singleResult();
    // 得到当前流程定义的实例是否都为暂停状态,true:已暂停
    boolean suspended = processInstance.isSuspended();
    // 流程定义id，其实就是上面填写的id 17501
    String processDefinitionId = processInstance.getId();
    System.out.println(processDefinitionId); 
    // 判断是否为暂停
    if(suspended){
        // 如果是暂停，可以执行激活操作 ,参数：流程定义id
        runtimeService.activateProcessInstanceById(processDefinitionId);
        System.out.println("流程定义："+processDefinitionId+",已激活");
    }else{
        // 如果是激活状态，可以暂停，参数：流程定义id
        runtimeService.suspendProcessInstanceById( processDefinitionId);
        System.out.println("流程定义："+processDefinitionId+",已挂起");
    }
}
```

查看单个流程实例挂起后act_ru_execution&act_ru_task表的状态：

![当前的挂起状态-单流程：act_ru_execution](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402233.PNG)

![当前流程的挂起状态-单流程：act_ru_task](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402367.PNG)

> 可以看到实例id为17501的流程被挂起了，而流程id为20001的流程实例是没有被挂起的。

在流程实例被挂起的状态去执行流程：

```java
/**
 * 测试完成个人任务
 */
@Test
public void completTask(){

    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    // 完成任务,参数：流程实例id,完成zhangsan的任务
    Task task = taskService.createTaskQuery()
        .processInstanceId("17501")
        .taskAssignee("zhangsan")
        .singleResult();

    System.out.println("流程实例id="+task.getProcessInstanceId());
    System.out.println("任务Id="+task.getId());
    System.out.println("任务负责人="+task.getAssignee());
    System.out.println("任务名称="+task.getName());
    // 测试是否能完成任务
    taskService.complete(task.getId());
}
```

* 当流程被挂起的状态去执行任务，此时是无法执行的，控制台输出：

  ```
  org.activiti.engine.ActivitiException: Cannot complete a suspended task
  ```

* 再挂起的流程实例重新激活后再执行任务：

  ```bash
  # 调用方法suspendSingleProcessInstance
  流程定义：17501,已激活
  
  # 在调用方法completTask，成功执行
  流程实例id=17501
  任务Id=17505
  任务负责人=zhangsan
  任务名称=创建出差申请
  ```

## 2. 个人任务

### 2.1 分配任务负责人

#### 2.1.1 固定分配

在进行业务流程建模时指定固定的任务负责人， 如图：

![责任人指定](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402687.PNG)

#### 2.1.2 表达式分配

由于固定分配方式，任务只管一步一步执行任务，执行到每一个任务将按照 bpmn 的配置去分配任务负责人。 

**UEL 表达式：**Activiti 使用 UEL 表达式，UEL 是 java EE6 规范的一部分，UEL（Unified Expression Language，统一表达式语言）activiti 支持两个 UEL 表达式：UEL-value 和 UEL-method。 

**UEL-value 定义方式：**

* 定义方式1：`${...}`，在`{}`中填入相应的变量，即可以替换上图中Assignee中写死的值，如`${assignee _1}`
* 定义方式2：`${xxx.xxx}`，如`${user.assignee_1}`，表示 user 也是 activiti 的一个流程变量，`user.assignee`表示通过调用 user 的 getter 方法获取值

**UEL-method 方式：**

* `${xxxBean.getxxx}`，如：`${UserBean.getUserId()}`，其中userBean 是 spring 容器中的一个 bean，表示调用该 bean 的 `getUserId()` 方法。

**UEL-method 与 UEL-value 结合方式：**

* 比如：`${ldapService.findManagerForEmployee(emp)}`，其中 ldapService 是 spring 容器的一个 bean，findManagerForEmployee 是该 bean 的一个方法，emp 是 activiti 流程变量， emp 作为参数传到 ldapService.findManagerForEmployee 方法中。

**其它方式：**

* 表达式支持解析基础类型、bean、list、array 和 map，也可作为条件判断。如下：`${order.price > 100 && order.price < 250}`

> 注意事项：
>
> 由于使用了表达式分配，必须保证在任务执行过程表达式执行成功，比如：某个任务使用了表达式`${order.price > 100 && order.price < 250}`，当执行该任务时必须保证 order 在流程变量中存在，否则 activiti 异常。 

**使用UEL-value 定义方式来实现编写代码配置负责人：**

1. 定义任务分配流程变量，还是用了之前的这个出差流程，但是这里的责任人通过了 UEL 表达式进行指定：

   ![责任人指定-UEL](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402913.PNG)

   > 其他节点也和之前的相同，总共是4个节点：`${assignee_1}`~`${assignee_4}`

2. 布署该流程

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
           .addClasspathResource("bpmn/evection-uel.bpmn") // 添加bpmn资源(这里就生成png了)
           .name("出差申请流程")
           .deploy();
       // 4、输出部署信息
       System.out.println("流程部署id：" + deployment.getId());
       System.out.println("流程部署名称：" + deployment.getName());
   }
   
   // 输出内容：
   // 流程部署id：25001
   // 流程部署名称：出差申请流程
   ```

   > 查看布署的数据：
   >
   > * act_re_deployment
   >
   > ```mysql
   > ID_	NAME_	CATEGORY_	KEY_	TENANT_ID_	DEPLOY_TIME_	ENGINE_VERSION_
   > 25001	出差申请流程	\N	\N		2022-05-02 14:11:37	\N
   > ```
   >
   > * act_re_procdef
   >
   > ```msyql
   > ID_	REV_	CATEGORY_	NAME_	KEY_	VERSION_	DEPLOYMENT_ID_	RESOURCE_NAME_	DGRM_RESOURCE_NAME_	DESCRIPTION_	HAS_START_FORM_KEY_	HAS_GRAPHICAL_NOTATION_	SUSPENSION_STATE_	TENANT_ID_	ENGINE_VERSION_
   > myEvection-uel:1:25003	1	http://www.activiti.org/testm1651471112919	出差申请-uel	myEvection-uel	1	25001	bpmn/evection-uel.bpmn	\N	\N	0	1	1		\N
   > ```

3. 设置流程变量

   ```java
   /**
    * 设置流程负责人
    */
   @Test
   public void assigneeUEL(){
       // 获取流程引擎
       ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
       // 获取 RuntimeService
       RuntimeService runtimeService = processEngine.getRuntimeService();
       // 设置assignee的取值，用户可以在界面上设置流程的执行
       Map<String,Object> assigneeMap = new HashMap<>();
       assigneeMap.put("assignee_1","张三");
       assigneeMap.put("assignee_2","李经理");
       assigneeMap.put("assignee_3","王总经理");
       assigneeMap.put("assignee_4","赵财务");
       // 启动流程实例，同时还要设置流程定义的assignee的值
       runtimeService.startProcessInstanceByKey("myEvection-uel",assigneeMap);
       // 输出
       System.out.println(processEngine.getName());
   }
   ```

   执行成功后，可以在act_ru_variable表中看到刚才map中的数据：

   ![act_ru_variable数据](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402492.PNG)

#### 2.1.3 监听器分配

可以使用监听器来完成很多Activiti流程的业务。比如通过监听器的方式来指定负责人，那么在流程设计时就不需要指定assignee。

任务监听器是发生对应的任务相关事件时执行自定义 java 逻辑 或表达式。

![事件类型](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402788.PNG)

**任务事件event包括：**

* 创建时间：create，任务创建后触发；
* 分配事件：assignee，任务分配后触发；
* 删除事件：delete，任务完成后触发；
* 所有时间：all，所有事件发生都触发；

**任务类型type包括：**

* class：类，监听到事件找一个类来处理当然任务；选择class后，在后续的class栏中填入全限定类名；
* expression：表达式，监听到事件找一个表达式来处理当然任务；
* delegate expression：

**一个监听器的小Demo：**

1. 创建流程：

   ![监听流程配置](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402700.PNG)

   > 这里的话定义任务监听类：且类必须实现 org.activiti.engine.delegate.TaskListener 接口
   >
   > ```
   > package cn.xyc.demo.listener;
   > 
   > import org.activiti.engine.delegate.DelegateTask;
   > import org.activiti.engine.delegate.TaskListener;
   > 
   > public class MyTaskListener implements TaskListener{
   > 
   >     /**
   >      * 重写notify方法
   >      * @param delegateTask
   >      */
   >     @Override
   >     public void notify(DelegateTask delegateTask) {
   >         if(delegateTask.getName().equals("创建出差申请")&&
   >                 delegateTask.getEventName().equals("create")){
   >             //这里指定任务负责人
   >             delegateTask.setAssignee("张三");
   >         }
   >     }
   > }
   > ```
   >
   > 打个断点查看类DelegateTask中的内容：
   >
   > ![DelegateTask类的值](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402421.PNG)

2. 部署&启动流程实例：

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
           .addClasspathResource("bpmn/listen-demo.bpmn") // 添加bpmn资源
           .name("监听事件的Demo")
           .deploy();
       // 4、输出部署信息
       System.out.println("流程部署id：" + deployment.getId());
       System.out.println("流程部署名称：" + deployment.getName());
   }
   
   /**
    * 流程启动
    */
   @Test
   public void testListenerStart(){
       // 1、创建ProcessEngine
       ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
       // 2、得到runtimeService实例
       RuntimeService runtimeService = processEngine.getRuntimeService();
       // 3、启动（MyTaskListener中打断点查看）
       runtimeService.startProcessInstanceByKey("listen-demo");
   }
   ```

   启动完成后，可以在执行任务表act_ru_task中查看数据，可以看到其`ASSIGNEE_`属性即为上述设置的`张三`

   ```
   ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
   32505	1	32502	32501	listen-demo:1:30003	监听任务	\N	\N	_3	\N	张三	\N	50	2022-05-02 15:15:22	\N	\N	1		\N	\N
   ```

> 注意事项：使用监听器分配方式，按照监听事件去执行监听类的 notify 方法，方法如果不能正常执行也会影响任务的执行。 

### 2.2 查询任务

**查询任务负责人的待办任务：**

```java
/**
 * 查询当前个人待执行的任务
 */
@Test
public void findPersonalTaskList() {
    // 流程定义key
    String processDefinitionKey = "myEvection1";
    // 任务负责人
    String assignee = "张三";
    // 获取TaskService
    TaskService taskService = processEngine.getTaskService();
    List<Task> taskList = taskService.createTaskQuery()
    	.processDefinitionKey(processDefinitionKey)
    	.includeProcessVariables()
        .taskAssignee(assignee)
        .list();
    for (Task task : taskList) {
        System.out.println("----------------------------");
        System.out.println("流程实例id： " + task.getProcessInstanceId());
        System.out.println("任务id： " + task.getId());
        System.out.println("任务负责人： " + task.getAssignee());
        System.out.println("任务名称： " + task.getName());
    }
}
```

**关联 businessKey：**

* 需求：在 activiti 实际应用时，查询待办任务可能要显示出业务系统的一些相关信息。

  比如：查询待审批出差任务列表需要将出差单的日期、 出差天数等信息显示出来。

  出差天数等信息在业务系统中存在，而并没有在 activiti 数据库中存在，所以是无法通过 activiti 的 api 查询到出差天数等信息。

* 实现：在查询待办任务时，通过 businessKey（业务标识 ）关联查询业务系统的出差单表，查询出出差天数等信息。 

```java
/**
 * 关联 businessKey
 */
@Test
public void findProcessInstance(){
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取TaskService
    TaskService taskService = processEngine.getTaskService();
    // 获取RuntimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 查询流程定义的对象
    Task task = taskService.createTaskQuery()
        .processDefinitionKey("myEvection1")
        .taskAssignee("张三")
        .singleResult();
    // 使用task对象获取实例id
    String processInstanceId = task.getProcessInstanceId();
    // 使用实例id，获取流程实例对象
    ProcessInstance processInstance = runtimeService.createProcessInstanceQuery()
        .processInstanceId(processInstanceId)
        .singleResult();
    // 使用processInstance，得到 businessKey，然后就可以用bussinesskey去关联业务表了
    String businessKey = processInstance.getBusinessKey();

    System.out.println("businessKey="+businessKey);

}
```

### 2.3 办理任务

> 注意：在实际应用中，完成任务前需要校验任务的负责人是否具有该任务的办理权限 。

```java
/**
 * 完成任务，判断当前用户是否有权限
 */
@Test
public void completTask() {
    // 任务id
    String taskId = "15005";
    // 任务负责人
    String assingee = "张三";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    // 完成任务前，需要校验该负责人可以完成当前任务
    // 校验方法：根据任务id和任务负责人查询当前任务，如果查到该用户有权限，就完成
    Task task = taskService.createTaskQuery()
        .taskId(taskId)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        taskService.complete(taskId);
        System.out.println("完成任务");
    }
}
```

## 3. 流程变量

### 3.1 流程变量

**流程变量：**

* 流程变量在 activiti 中是一个非常重要的角色，流程运转有时需要靠流程变量，业务系统和 activiti 结合时少不了流程变量，流程变量就是 activiti 在管理工作流时根据管理需要而设置的变量。
* 比如：在出差申请流程流转时如果出差天数大于 3 天则由总经理审核，否则由人事直接审核，出差天数就可以设置为流程变量，在流程流转时使用。
* **注意：虽然流程变量中可以存储业务数据，可以通过activiti的api查询流程变量从而实现查询业务数据，但是不建议这样使用，因为业务数据查询由业务系统负责，activiti设置流程变量是为了流程执行需要而创建。**

**流程变量类型：**

![流程变量](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402572.png)

> 注意：如果将 pojo 存储到流程变量中，必须实现序列化接口 serializable，为了防止由于新增字段无法反序列化，需要生成 serialVersionUID。 

### 3.2 流程变量作用域

流程变量的作用域可以是一个流程实例(processInstance)；或一个任务(task)；或一个执行实例(execution)。

**globa变量**

* 流程变量的默认作用域是流程实例，当一个流程变量的作用域为流程实例时，可以称为 global 变量；

* 注意：

  如：Global变量：userId(变量名)：zhangsan(变量值)

  global 变量中变量名不允许重复，设置相同名称的变量，后设置的值会覆盖前设置的变量值。

**local变量**

* 任务和执行实例仅仅是针对一个任务和一个执行实例范围，范围没有流程实例大，称为 local 变量；

* Local 变量由于在不同的任务或不同的执行实例中，作用域互不影响，变量名可以相同没有影响；
* Local 变量名也可以和 global 变量名相同，没有影响。 

### 3.3 流程变量的使用方法 

**在属性上使用UEL表达式：**

* 可以在 assignee 处设置 UEL 表达式，表达式的值为任务的负责人，比如：`${assignee}`，assignee 就是一个流程变量名称；

* Activiti获取UEL表达式的值，即流程变量assignee的值 ，将assignee的值作为任务的负责人进行任务分配。

**在连线上使用UEL表达式：**

* 可以在连线上设置UEL表达式，决定流程走向，比如：`${price<10000}`，price就是一个流程变量名称，uel表达式结果类型为布尔类型；

* 如果UEL表达式是true，要决定流程执行走向。

### 3.4 使用Global变量控制流程

#### 3.4.1 需求

员工创建出差申请单，由部门经理审核：

* 部门经理审核通过后出差3天及以下由人财务直接审批；
* 3天以上先由总经理审核，总经理审核通过再由财务审批。

![条件-1](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402412.PNG)

#### 3.4.2 流程定义

1. 出差天数大于等于3连线条件：

   ![条件-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402689.PNG)

   也可以使用对象参数命名，如evection.num：

   ![条件-3](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402967.PNG)

2. 出差天数小于3连线条件，也可以使用对象参数命名，如：

   ![条件-4](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402368.PNG)

#### 3.4.3 流程布署

将刚刚设计好的流程进行部署：

```java
    /**
     * 流程部署
     */
    @Test
    public void testDeployment(){
        // 1、创建ProcessEngine
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 2、获取RepositoryServcie
        RepositoryService repositoryService = processEngine.getRepositoryService();
        // 3、使用service进行流程的部署，定义一个流程的名字，把bpmn和png部署到数据中
        Deployment deploy = repositoryService.createDeployment()
                .name("出差申请流程-variables")
                .addClasspathResource("bpmn/evection-global.bpmn")
                .deploy();
        // 4、输出部署信息
        System.out.println("流程部署id="+deploy.getId());
        System.out.println("流程部署名字="+deploy.getName());
    }
```

布署后的.bpmn文件

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="http://www.activiti.org/testm1651545432300" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="m1651545432300" name="" targetNamespace="http://www.activiti.org/testm1651545432300" typeLanguage="http://www.w3.org/2001/XMLSchema">
  <process id="myEvection-global" isClosed="false" isExecutable="true" name="出差申请-global" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="${assignee1}" activiti:exclusive="true" id="_3" name="创建出差申请"/>
    <userTask activiti:assignee="${assignee2}" activiti:exclusive="true" id="_4" name="部门经理审核"/>
    <userTask activiti:assignee="${assignee3}" activiti:exclusive="true" id="_5" name="总经理审批"/>
    <userTask activiti:assignee="${assignee4}" activiti:exclusive="true" id="_6" name="财务审批"/>
    <endEvent id="_7" name="EndEvent"/>
    <sequenceFlow id="_8" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_9" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_10" sourceRef="_4" targetRef="_5">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_11" sourceRef="_4" targetRef="_6">
      <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_12" sourceRef="_5" targetRef="_6"/>
    <sequenceFlow id="_13" sourceRef="_6" targetRef="_7"/>
  </process>
  <bpmndi:BPMNDiagram documentation="background=#3C3F41;count=1;horizontalcount=1;orientation=0;width=842.4;height=1195.2;imageableWidth=832.4;imageableHeight=1185.2;imageableX=5.0;imageableY=5.0" id="Diagram-_1" name="New Diagram">
    <bpmndi:BPMNPlane bpmnElement="myEvection-global">
      <bpmndi:BPMNShape bpmnElement="_2" id="Shape-_2">
        <dc:Bounds height="32.0" width="32.0" x="-5.0" y="15.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_3" id="Shape-_3">
        <dc:Bounds height="55.0" width="85.0" x="85.0" y="20.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_4" id="Shape-_4">
        <dc:Bounds height="55.0" width="85.0" x="220.0" y="20.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_5" id="Shape-_5">
        <dc:Bounds height="55.0" width="85.0" x="360.0" y="20.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_6" id="Shape-_6">
        <dc:Bounds height="55.0" width="85.0" x="365.0" y="120.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="55.0" width="85.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="_7" id="Shape-_7">
        <dc:Bounds height="32.0" width="32.0" x="505.0" y="145.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="32.0" width="32.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNEdge bpmnElement="_13" id="BPMNEdge__13" sourceElement="_6" targetElement="_7">
        <di:waypoint x="450.0" y="147.5"/>
        <di:waypoint x="505.0" y="161.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_12" id="BPMNEdge__12" sourceElement="_5" targetElement="_6">
        <di:waypoint x="405.0" y="75.0"/>
        <di:waypoint x="405.0" y="120.0"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_8" id="BPMNEdge__8" sourceElement="_2" targetElement="_3">
        <di:waypoint x="27.0" y="31.0"/>
        <di:waypoint x="85.0" y="47.5"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_9" id="BPMNEdge__9" sourceElement="_3" targetElement="_4">
        <di:waypoint x="170.0" y="47.5"/>
        <di:waypoint x="220.0" y="47.5"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_11" id="BPMNEdge__11" sourceElement="_4" targetElement="_6">
        <di:waypoint x="305.0" y="47.5"/>
        <di:waypoint x="365.0" y="147.5"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
      <bpmndi:BPMNEdge bpmnElement="_10" id="BPMNEdge__10" sourceElement="_4" targetElement="_5">
        <di:waypoint x="305.0" y="47.5"/>
        <di:waypoint x="360.0" y="47.5"/>
        <bpmndi:BPMNLabel>
          <dc:Bounds height="0.0" width="0.0" x="0.0" y="0.0"/>
        </bpmndi:BPMNLabel>
      </bpmndi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

#### 3.4.4 设置global流程变量

在部门经理审核前设置流程变量，变量值为出差单信息（包括出差天数），部门经理审核后可以根据流程变量的值决定流程走向。

在设置流程变量时，**可以在启动流程时设置，也可以在任务办理时设置**。

**创建POJO对象**：创建出差申请pojo对象

```java
/**
 * 出差申请 pojo
 */
@Data
public class Evection implements Serializable {

    /**
     * 主键id
     */
    private Long id;
    /**
     * 出差申请单名称
     */
    private String evectionName;
    /**
     * 出差天数
     */
    private Double num;
    /**
     * 预计开始时间
     */
    private Date beginDate;
    /**
     * 预计结束时间
     */
    private Date endDate;
    /**
     * 目的地
     */
    private String destination;
    /**
     * 出差事由
     */
    private String reson;
}
```

##### 3.4.4.1 启动流程时设置变量

在启动流程时设置流程变量，变量的作用域是整个流程实例。

通过Map<key,value>设置流程变量，map中可以设置多个变量，这个key就是流程变量的名字。

```java
/**
 * 启动流程实例,设置流程变量的值
 */
@Test
public void startProcess(){
    // 创建ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取runtimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 流程定义key
    String processKey = "myEvection-global";
    // 创建变量集合
    Map<String, Object> map = new HashMap<>();
    // 创建出差pojo对象，设置出差天数
    Evection evection = new Evection();
    evection.setNum(2d);
    // 定义流程变量，把出差pojo对象放入map
    map.put("evection", evection);
    // 设置assignee的取值，用户可以在界面上设置流程的执行
    map.put("assignee1","张三");
    map.put("assignee2","李经理");
    map.put("assignee3","王总经理");
    map.put("assignee4","赵财务");
    // 启动流程实例，并设置流程变量的值（把map传入）
    ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processKey, map);
    // 输出
    System.out.println("流程实例名称="+processInstance.getName());
    System.out.println("流程定义id="+processInstance.getProcessDefinitionId());
}

// 输出：
// 流程实例名称=null
// 流程定义id=myEvection-global:1:35003
```

> 说明：`startProcessInstanceByKey(processDefinitionKey, variables)`，流程变量作用域是一个流程实例，**流程变量使用Map存储**，同一个流程实例设置变量map中key相同，后者覆盖前者。

查看表act_ru_variable中的数据，可以看到上述设置的数据：

![variables：act_ru_variable](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402525.PNG)

> 列说明：`Id_：主键  / Type_：变量类型 / Name_：变量名称  / Execution_id_：所属流程实例执行id，global和local变量都存储 / Proc_inst_id_：所属流程实例id，global和local变量都存储  / Task_id_：所属任务id，local变量存储 / Bytearray_：serializable类型变量存储对应act_ge_bytearray表的id / Double_：double类型变量值 / Long_：long类型变量值/ Text_：text类型变量值 `

##### 3.4.4.2 任务办理时设置变量

在完成任务时设置流程变量，该流程变量只有在该任务完成后其它结点才可使用该变量，它的作用域是整个流程实例，如果设置的流程变量的key在流程实例中已存在相同的名字则后设置的变量替换前边设置的变量。

这里需要在创建出差单任务完成时设置流程变量。

```java
/**
 * 完成任务，即在任务办理时设置变量值
 */
@Test
public void completTask() {
    // 任务id
    String key = "myEvection-global";
    // 任务负责人
    String assingee = "张三";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    // 创建变量集合
    Map<String, Object> map = new HashMap<>();
    // 创建出差pojo对象
    Evection evection = new Evection();
    // 设置出差天数
    evection.setNum(2d);
    // 定义流程变量
    map.put("evection",evection);
    // 完成任务前，需要校验该负责人可以完成当前任务
    // 校验方法：根据任务id和任务负责人查询当前任务，如果查到该用户有权限，就完成
    Task task = taskService.createTaskQuery()
        .processDefinitionKey(key)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        // 完成任务是，设置流程变量的值，放入这map
        taskService.complete(task.getId(),map);
        System.out.println("任务执行完成");
    }
}
```

> 说明：通过当前任务设置流程变量，需要指定当前任务id，如果当前执行的任务id不存在则抛出异常。任务办理时也是通过`map<key,value>`设置流程变量，一次可以设置多个变量。

##### 3.4.4.3 通过当前流程实例设置

通过流程实例id设置全局变量，**该流程实例必须未执行完成**。

```java
@Test
public void setGlobalVariableByExecutionId(){
    // 当前流程实例执行 id，通常设置为当前执行的流程实例
    String executionId="47501";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RuntimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 创建出差pojo对象
    Evection evection = new Evection();
    // 设置天数
    evection.setNum(3d);
    // 通过流程实例 id设置流程变量
    runtimeService.setVariable(executionId, "evection", evection);
    // 也可以一次设置多个值，就是之前的map形式
    // runtimeService.setVariables(executionId, variables)
}
```

从表act_ru_execution中获取到的流程实例id：

```
ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
47506	1	47501	\N	47501	myEvection-global:1:35003	\N	47501	_3	1	0	0	0	0	1	\N		\N	2022-05-03 12:01:29	\N	\N	0	0	0	0	0	0	0	0	0
```

> 注意：executionId 必须当前未结束流程实例的执行id，通常此id设置流程实例的id。也可以通`runtimeService.getVariable()` 获取流程变量。

##### 3.4.4.4 通过当前任务设置

```java
@Test
public void setGlobalVariableByTaskId(){

    // 当前待办任务id，这个任务id可以从act_ru_task中获取
    String taskId="47509";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    Evection evection = new Evection();
    evection.setNum(3);
    // 通过任务设置流程变量
    taskService.setVariable(taskId, "evection", evection);
    // 一次设置多个值，就是之前的map形式
    // taskService.setVariables(taskId, variables)
}
```

从表act_ru_task中获取到的任务id：

```
ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
47509	1	47506	47501	myEvection-global:1:35003	创建出差申请	\N	\N	_3	\N	张三	\N	50	2022-05-03 12:01:29	\N	\N	1		\N	\N
```

> 注意：任务id必须是当前待办任务id，act_ru_task中存在。如果该任务已结束，会报错也可以通过`taskService.getVariable()` 获取流程变量。

#### 3.4.5 分支测试

**分支1：出差天数小于3天的**

```java
/**
 * 完成任务-节点1：创建出差申请
 */
@Test
public void completTask() {
    // 任务id
    String processKey = "myEvection-global";
    // 任务负责人
    String assingee = "张三";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    // 完成任务前，需要校验该负责人可以完成当前任务
    // 校验方法：根据任务id和任务负责人查询当前任务，如果查到该用户有权限，就完成
    Task task = taskService.createTaskQuery()
        .processDefinitionKey(processKey)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        taskService.complete(task.getId());
        System.out.println("任务执行完成");
    }
}

/**
 * 完成任务-节点2：部门经理审核
 */
String assingee = "李经理";
```

这里设置的出差天数为：`evection.setNum(2d);`，因此当`节点2：部门经理审核`完成后，应该走的是`财务审批节点`，查看数据库中表 act_ru_task 中的任务节点：

```
ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
42502	1	37509	37501	myEvection-global:1:35003	财务审批	\N	\N	_6	\N	赵财务	\N	50	2022-05-03 11:26:55	\N	\N	1		\N	\N
```

可以看到，已经到了`财务审批节点`，没有走这个`总经理审批`节点。

```java
/**
 * 完成任务-节点4：财务审批
 */
String assingee = "赵财务";
```

查看表 act_hi_procinst 数据，可以看到这条流程已经走完了：

![条件-5](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402445.PNG)

**分支2：出差天数大于等于3天的**

> 前面的操作还是一样：开启流程&执行任务，不过这里不同的就是在开启流程时，不设置出差的pojo对象，而是通过当前流程实例来进行设置。

```java
/**
 * 启动流程实例,设置流程变量的值-分支2
 */
@Test
public void startProcess2(){
    // 创建ProcessEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取runtimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 流程定义key
    String processKey = "myEvection-global";
    // 创建变量集合
    Map<String, Object> map = new HashMap<>();
    // 创建出差pojo对象，设置出差天数
    map.put("assignee1","张三");
    map.put("assignee2","李经理");
    map.put("assignee3","王总经理");
    map.put("assignee4","赵财务");
    // 启动流程实例，并设置流程变量的值（把map传入）
    ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processKey, map);
    // 输出
    System.out.println("流程实例名称="+processInstance.getName());
    System.out.println("流程定义id="+processInstance.getProcessDefinitionId());
}
```

然后通过当前流程实例来设置pojo对象：

> 其中的流程实例的id可以在表act_ru_execution中的列`PROC_INST_ID_`中找到

```java
@Test
public void setGlobalVariableByExecutionId(){
    // 当前流程实例执行 id，通常设置为当前执行的流程实例
    String executionId="47501";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RuntimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 创建出差pojo对象
    Evection evection = new Evection();
    // 设置天数
    evection.setNum(3d);
    // 通过流程实例 id设置流程变量
    runtimeService.setVariable(executionId, "evection", evection);
    // 一次设置多个值
    // runtimeService.setVariables(executionId, variables)
}
```

执行完成后，创建的出差pojo对象就设置成功了，这时再继续完成后续任务节点：

```java
/**
     * 完成任务
     */
@Test
public void completTask() {
    // 任务id
    String processKey = "myEvection-global";
    // 任务负责人
    String assingee = "张三";
    //  String assingee = "李经理";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    // 完成任务前，需要校验该负责人可以完成当前任务
    // 校验方法：根据任务id和任务负责人查询当前任务，如果查到该用户有权限，就完成
    Task task = taskService.createTaskQuery()
        .processDefinitionKey(processKey)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        taskService.complete(task.getId());
        System.out.println("任务执行完成");
    }
}
```

当`部门经理审核`审核完成后，这时由于设置的出差天数为：`evection.setNum(3d);`后续走的就是`总经理审批`节点吗，查看数据库中表 act_ru_task 中的任务节点：

```
ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
55002	1	47506	47501	myEvection-global:1:35003	总经理审批	\N	\N	_5	\N	王总经理	\N	50	2022-05-03 13:36:51	\N	\N	1		\N	\N
```

可以看到确实走是`总经理审批`，继续执行：

```java
// 替换上述的任务责任人：
String assingee = "王总经理";
String assingee = "赵财务";
```

当节点`财务审批`完成审批后，整个实例就执行完成，可以查看表 act_hi_procinst：

```
ID_	PROC_INST_ID_	BUSINESS_KEY_	PROC_DEF_ID_	START_TIME_	END_TIME_	DURATION_	START_USER_ID_	START_ACT_ID_	END_ACT_ID_	SUPER_PROCESS_INSTANCE_ID_	DELETE_REASON_	TENANT_ID_	NAME_
47501	47501	\N	myEvection-global:1:35003	2022-05-03 12:01:29	2022-05-03 13:39:38	5889759	\N	_2	_7	\N	\N		\N
```

#### 3.4.6 注意事项

1. 如果UEL表达式中流程变量名不存在则报错；

2. 如果UEL表达式中流程变量值为空NULL，流程不按UEL表达式去执行，而流程结束；

3. 如果UEL表达式都不符合条件，流程结束；

4. 如果连线不设置条件，会走flow序号小的那条线。

   > 可以查看定义的流程文件信息：
   >
   > ```xml
   > <sequenceFlow id="_10" sourceRef="_4" targetRef="_5">
   >     <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
   > </sequenceFlow>
   > <sequenceFlow id="_11" sourceRef="_4" targetRef="_6">
   >     <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
   > </sequenceFlow>
   > ```
   >
   > 如果没定义这个conditionExpression的话，默认走的就是flow序号较小的，即`id="_10"`的这条。
   >
   > > **实测：会生成所有下一个节点的任务！**

### 3.5 设置local流程变量

#### 3.5.1 任务办理时设置

任务办理时设置local流程变量，当前运行的流程实例只能在该任务结束前使用，任务结束该变量无法在当前流程实例使用，可以通过查询历史任务查询。

> 其他的流程开始啥的都和上面一致

```java
/**
 * 处理任务时设置local流程变量
 */
@Test
public void completTaskLocal() {
    // 任务id，从表act_ru_task中获取
    String taskId = "62509";  // 第一个节点
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    // 定义流程变量
    Map<String, Object> variables = new HashMap<>();
    Evection evection = new Evection ();
    evection.setNum(3d);
    // 变量名是holiday，变量值是holiday对象
    variables.put("evection", evection);
    // 设置local变量，作用域为该任务，注意这里是setVariablesLocal，而全局的是setVariable
    taskService.setVariablesLocal(taskId, variables);
    // 完成任务
    taskService.complete(taskId);
}
```

>  说明：设置作用域为任务的local变量，每个任务可以设置同名的变量，互不影响。

#### 3.5.2 通过当前任务设置

```java
/**
 * 通过当前任务设置local变量
 */
@Test
public void setLocalVariableByTaskId(){
	// 当前待办任务id
    String taskId="65005";  // 第二个节点
	// 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    Evection evection = new Evection ();
    evection.setNum(3d);
	// 通过任务设置流程变量
    taskService.setVariableLocal(taskId, "evection", evection);
	// 一次设置多个值 
    // taskService.setVariablesLocal(taskId, variables)
}
```

> 注意：任务id必须是当前待办任务id，act_ru_task中存在。

#### 3.5.3 Local变量测试

**测试1：**

若上边例子中设置global变量改为设置local变量是否可行？为什么？

Local变量在任务结束后无法在当前流程实例执行中使用，如果后续的流程执行需要用到此变量则会报错。

> 即上述将Evection设置成立local变量，后续在流程分支判断时是无法进行进行的，因为流程结束后就不认识这个变量了，报错信息：
>
> ```
> org.activiti.engine.ActivitiException: Unknown property used in expression: ${evection.num>=3}
> ```

**测试2：**

在部门经理审核、总经理审核、财务审核时设置local变量，可通过historyService查询每个历史任务时将流程变量的值也查询出来，代码如下：

```java
/**
 * 查询每个历史任务的流程变量
 */
@Test
public void testGetHistoryLocalVariable(){
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建历史任务查询对象
    HistoryService historyService = processEngine.getHistoryService();
    HistoricTaskInstanceQuery historicTaskInstanceQuery = historyService.createHistoricTaskInstanceQuery();
    // 查询结果包括 local变量
    HistoricTaskInstanceQuery taskInstanceQuery = historicTaskInstanceQuery.includeTaskLocalVariables();
    List<HistoricTaskInstance> list = taskInstanceQuery.processInstanceId("62501").list();
    for (HistoricTaskInstance historicTaskInstance : list) {
        System.out.println("==============================");
        System.out.println("任务id：" + historicTaskInstance.getId());
        System.out.println("任务名称：" + historicTaskInstance.getName());
        System.out.println("任务负责人：" + historicTaskInstance.getAssignee());
        System.out.println("任务local变量：" + historicTaskInstance.getTaskLocalVariables());
    }
}

// 输出：
// ==============================
// 任务id：62509
// 任务名称：创建出差申请
// 任务负责人：张三
// 任务local变量：{evection=cn.xyc.demo.listener.pojo.Evection@5f6722d3}
// ==============================
// 任务id：65005
// 任务名称：部门经理审核
// 任务负责人：李经理
// 任务local变量：{evection=cn.xyc.demo.listener.pojo.Evection@2c532cd8}
```

> 注意：查询历史流程变量，特别是查询pojo变量需要经过反序列化，不推荐使用。

## 4. 组任务

### 4.1 组任务设置

**需求：**

* 在流程定义中在任务结点的 assignee 固定设置任务负责人，在流程定义时将参与者固定设置在 .bpmn 文件中，如果临时任务负责人变更则需要修改流程定义，系统可扩展性差。
* 针对这种情况可以给任务设置多个候选人，**可以从候选人中选择参与者来完成任务**。 

**设置任务候选人：** 

* 在流程图中任务节点的配置中设置 `candidate-users`(候选人)，多个候选人之间用逗号分开。 

  ![多候选人的bpmn图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402058.PNG)

* 查看bpmn文件

  ```xml
  <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="创建出差申请"/>
  <!-- 可以看到部门经理的审核人已经设置为 lisi,wangwu 这样的一组候选人，
  	可以使用activiti:candiateUsers=”用户 1,用户 2,用户 3”的这种方式来实现设置一组候选人-->
  <userTask activiti:candidateUsers="lisi,wangwu" activiti:exclusive="true" id="_4" name="经理审批"/>   
  <userTask activiti:assignee="jack" activiti:exclusive="true" id="_5" name="总经理审批"/>
  <endEvent id="_6" name="EndEvent"/>
  <userTask activiti:assignee="rose" activiti:exclusive="true" id="_7" name="财务审批"/>
  ```

### 4.2 组任务

#### 4.2.1 组任务办理流程

1. **查询组任务**

   指定候选人，查询该候选人当前的待办任务。

   候选人不能立即办理任务。

2. **拾取(claim)任务**

   该组任务的所有候选人都能拾取。

   将候选人的组任务，变成个人任务，原来候选人就变成了该任务的负责人。

   如果拾取后不想办理该任务？需要将已经拾取的个人任务归还到组里边，将个人任务变成了组任务。

3. **查询个人任务**

   查询方式同个人任务部分，根据assignee查询用户负责的个人任务。

4. **办理个人任务**

#### 4.2.2 组任务操作

> 布署任务：
>
> ```java
> package cn.xyc.activiti01.test;
> 
> import org.activiti.engine.ProcessEngine;
> import org.activiti.engine.ProcessEngines;
> import org.activiti.engine.RepositoryService;
> import org.activiti.engine.repository.Deployment;
> import org.junit.Test;
> 
> public class CandidateTest {
> 
>     /**
>      * 部署流程定义
>      */
>     @Test
>     public void testDeployment(){
> 
>         // 1、创建ProcessEngine
>         ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
>         // 2、得到RepositoryService实例
>         RepositoryService repositoryService = processEngine.getRepositoryService();
>         // 3、使用RepositoryService进行部署
>         Deployment deployment = repositoryService.createDeployment()
>                 .addClasspathResource("bpmn/evection-cadidate.bpmn")
>                 .name("出差申请流程-多候选人")
>                 .deploy();
>         // 4、输出部署信息
>         System.out.println("流程部署id：" + deployment.getId());
>         System.out.println("流程部署名称：" + deployment.getName());
>     }
> }
> ```
>
> 开启一个流程实例：
>
> ```java
> /**
> * 启动流程实例
> */
> @Test
> public void testStartProcess(){
> 
>     // 1、创建ProcessEngine
>     ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
>     // 2、获取RunTimeService
>     RuntimeService runtimeService = processEngine.getRuntimeService();
>     // 3、根据流程定义Id启动流程（这里的key忘记该了，就这样吧）
>     ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess_1");
>     // 4、输出内容
>     System.out.println("流程定义id：" + processInstance.getProcessDefinitionId());
>     System.out.println("流程实例id：" + processInstance.getId());
>     System.out.println("当前活动Id：" + processInstance.getActivityId());
> }
> ```
>
> 完第一节点任务，流转到第二个节点，即多候选人节点：
>
> ```java
> 
> /**
> * 任完第一节点任务，流转到第二个节点，即多候选人节点
> */
> @Test
> public void completTask(){
>     // 获取引擎
>     ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
>     // 获取taskService
>     TaskService taskService = processEngine.getTaskService();
> 
>     // 根据流程key 和 任务的负责人 查询任务
>     // 返回一个任务对象
>     Task task = taskService.createTaskQuery()
>         .processDefinitionKey("myProcess_1") //流程Key
>         .taskAssignee("zhangsan")  //要查询的负责人
>         .singleResult();
> 
>     // 完成任务,参数：任务id
>     taskService.complete(task.getId());
> }
> ```
>
> 当流转到多候选人节点时，由于有多个候选人还未分配责任人，因而在 act_ru_task表中，ASSIGNEE_ 字段为null，如下：
>
> ```
> ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
> 75002	1	72502	72501	myProcess_1:1:70003	经理审批	\N	\N	_4	\N	\N	\N	50	2022-05-08 12:01:58	\N	\N	1		\N	\N
> ```
>
> 然后可以查看一下表 act_ru_identitylink（任务参与者）中的数据：
>
> ```
> ID_	REV_	GROUP_ID_	TYPE_	USER_ID_	TASK_ID_	PROC_INST_ID_	PROC_DEF_ID_
> 75003	1	\N	candidate	lisi	75002	\N	\N
> 75004	1	\N	participant	lisi	\N	72501	\N
> 75005	1	\N	candidate	wangwu	75002	\N	\N
> 75006	1	\N	participant	wangwu	\N	72501	\N
> ```
>
> 该表中记录当前参考任务用户或组，当前任务如果设置了候选人，会向该表插入候选人记录，有几个候选就插入几个。
>
> > 与act_ru_identitylink对应的还有一张历史表act_hi_identitylink，向act_ru_identitylink插入记录的同时也会向历史表插入记录。

**查询组任务：** 根据候选人查询组任务

```java
/**
* 根据候选人查询组任务
*/
@Test
public void findGroupTaskList() {
    // 流程定义key
    String processDefinitionKey = "myProcess_1";
    // 任务候选人
    String candidateUser = "lisi";
    //  获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    // 查询组任务
    List<Task> list = taskService.createTaskQuery()
        .processDefinitionKey(processDefinitionKey)
        .taskCandidateUser(candidateUser)  // 根据候选人查询
        .list();
    for (Task task : list) {
        System.out.println("----------------------------");
        System.out.println("流程实例id：" + task.getProcessInstanceId());
        System.out.println("任务id：" + task.getId());
        System.out.println("任务负责人：" + task.getAssignee());
        System.out.println("任务名称：" + task.getName());
    }
}

// 输出：
// ----------------------------
// 流程实例id：72501
// 任务id：75002
// 任务负责人：null
// 任务名称：经理审批
```

**拾取组任务： **候选人员拾取组任务后该任务变为自己的个人任务

```java
/**
 * 拾取组任务
 */
@Test
public void claimTask(){
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    // 要拾取的任务id（上面可知，也可以从表act_ru_task中获取）
    String taskId = "75002";
    // 任务候选人id
    String userId = "lisi";
    // 拾取任务
    //    即使该用户不是候选人也能拾取(建议拾取时校验是否有资格)
    //    校验该用户有没有拾取任务的资格
    Task task = taskService.createTaskQuery()
        .taskId(taskId)
        .taskCandidateUser(userId)  // 根据候选人查询
        .singleResult();
    if(task!=null){
        // 拾取任务
        taskService.claim(taskId, userId);
        System.out.println("任务拾取成功");
    }
}
```

> 说明：
>
> * 即使该用户不是候选人也能拾取，建议拾取时校验是否有资格
>
> * 组任务拾取后，该任务已有负责人，通过候选人将查询不到该任务
>
> 拾取成功后，act_ru_task表中，ASSIGNEE_ 字段不再是null了，如下：
>
> ```
> ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
> 75002	2	72502	72501	myProcess_1:1:70003	经理审批	\N	\N	_4	\N	lisi	\N	50	2022-05-08 12:01:58	\N	\N	1		\N	2022-05-08 12:12:01
> ```

**查询个人待办任务：** 查询方式同个人任务查询

```java
/**
 * 查询个人待办任务
 */
@Test
public void findPersonalTaskList() {
    // 流程定义key
    String processDefinitionKey = "myProcess_1";
    // 任务负责人
    String assignee = "lisi";
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 创建TaskService
    TaskService taskService = processEngine.getTaskService();
    List<Task> list = taskService.createTaskQuery()
        .processDefinitionKey(processDefinitionKey)
        .taskAssignee(assignee)
        .list();
    for (Task task : list) {
        System.out.println("----------------------------");
        System.out.println("流程实例id：" + task.getProcessInstanceId());
        System.out.println("任务id：" + task.getId());
        System.out.println("任务负责人：" + task.getAssignee());
        System.out.println("任务名称：" + task.getName());
    }
}

// 输出：
// ----------------------------
// 流程实例id：72501
// 任务id：75002
// 任务负责人：lisi
// 任务名称：经理审批
```

**归还组任务：** 如果个人不想办理该组任务，可以归还组任务，归还后该用户不再是该任务的负责人

```java
/**
 * 归还组任务，由个人任务变为组任务，还可以进行任务交接
 */
@Test
public void setAssigneeToGroupTask() {
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 查询任务使用TaskService
    TaskService taskService = processEngine.getTaskService();
    // 当前待办任务
    String taskId = "75002";
    // 任务负责人
    String userId = "lisi";
    // 校验userId是否是taskId的负责人，如果是负责人才可以归还组任务
    Task task = taskService
        .createTaskQuery()
        .taskId(taskId)
        .taskAssignee(userId)
        .singleResult();
    if (task != null) {
        // 如果设置为null，归还组任务,该 任务没有负责人
        taskService.setAssignee(taskId, null);
    }
}
```

> 说明：
>
> * 建议归还任务前校验该用户是否是该任务的负责人
>
> * 也可以通过setAssignee方法将任务委托给其它用户负责，注意被委托的用户可以不是候选人（建议不要这样使用）
>
> 任务归还后，act_ru_task表中，ASSIGNEE_ 字段再次置为null值；且上述查询个人待办任务执行结果为空了；

**任务交接：**任务交接，任务负责人将任务交给其它候选人办理该任务

> 交接前候选人lisi先去拾取一下任务。

```java
/**
 * 任务交接
 */
@Test
public void setAssigneeToCandidateUser() {
    // 获取processEngine
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 查询任务使用TaskService
    TaskService taskService = processEngine.getTaskService();
    // 当前待办任务
    String taskId = "75002";
    // 任务负责人
    String userId = "lisi";
    // 将此任务交给其它候选人办理该 任务
    String candidateuser = "wangwu";
    // 校验userId是否是taskId的负责人，如果是负责人才可以归还组任务
    Task task = taskService
        .createTaskQuery()
        .taskId(taskId)
        .taskAssignee(userId)
        .singleResult();
    if (task != null) {
        taskService.setAssignee(taskId, candidateuser);
    }
}
```

> 交接完成后，可以看到表 act_ru_task中的ASSIGNEE_字段发生改变。
>
> ```
> ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
> 75002	5	72502	72501	myProcess_1:1:70003	经理审批	\N	\N	_4	\N	wangwu	\N	50	2022-05-08 12:01:58	\N	\N	1		\N	2022-05-08 12:19:15
> ```

**办理个人任务：**任务拾取完成后，就可以有拾取人去办理任务了

```java
/**
 * 完成任务
 */
@Test
public void completeTask(){
    // 任务ID
    String taskId = "75002";
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    processEngine.getTaskService().complete(taskId);
    System.out.println("完成任务："+taskId);
}
```

> 任务完成，流程流转到下一个节点，任务 act_ru_task 表数据如下：
>
> ```
> ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
> 82502	1	72502	72501	myProcess_1:1:70003	总经理审批	\N	\N	_5	\N	jack	\N	50	2022-05-08 12:23:02	\N	\N	1		\N	\N
> ```

## 5. 网关

**网关作用：网关用来控制流程的流向**

![网关](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402338.PNG)

### 5.1 排他网关-ExclusiveGateway

#### 5.1.1 概述

排他网关：用来在流程中实现决策；当流程执行到这个网关，所有分支都会判断条件是否为true，如果为true则执行该分支。

> 注意：排他网关只会选择一个为true的分支执行。**如果有两个分支条件都为true，排他网关会选择id值较小的一条分支去执行。**

为什么要用排他网关？

* 不用排他网关也可以实现分支，如：在连线的condition条件上设置分支条件；
* 在连线设置condition条件的缺点：如果条件都不满足，流程就结束了(是异常结束)；但如果从网关出去的线所有条件都不满足则系统**抛出异常**。

#### 5.1.2 流程定义

![排他网关图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402943.PNG)

```xml
<process id="evection-exclusive" isClosed="false" isExecutable="true" name="出差审批-排他网关" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="填写出差单"/>
    <userTask activiti:assignee="lisi" activiti:exclusive="true" id="_4" name="部门经理审批"/>
    <exclusiveGateway gatewayDirection="Unspecified" id="_5" name="ExclusiveGateway"/>
    <userTask activiti:assignee="wangwu" activiti:exclusive="true" id="_6" name="总经理审批"/>
    <userTask activiti:assignee="zhaoliu" activiti:exclusive="true" id="_7" name="财务审批"/>
    <endEvent id="_8" name="EndEvent"/>
    <sequenceFlow id="_9" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_10" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_11" sourceRef="_4" targetRef="_5"/>
    <sequenceFlow id="_12" name="出差天数大于等于3天" sourceRef="_5" targetRef="_6">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_13" name="出差天数小于3天" sourceRef="_5" targetRef="_7">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_14" sourceRef="_6" targetRef="_7"/>
    <sequenceFlow id="_15" sourceRef="_7" targetRef="_8"/>
</process>
```

#### 5.1.3 流程测试

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
        .addClasspathResource("bpmn/evection-exclusive.bpmn") // 添加bpmn资源
        .name("出差审批-排他网关")
        .deploy();
    // 4、输出部署信息
    System.out.println("流程部署id：" + deployment.getId());
    System.out.println("流程部署名称：" + deployment.getName());
}
```

**分支1：**即 `exection.num>=3`

```java
/**
 * 启动流程实例,设置流程变量的值
 */
@Test
public void startProcess(){
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RunTimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 流程定义key
    String key = "evection-exclusive";
    // 创建变量集合
    Map<String, Object> map = new HashMap<>();
    // 创建出差pojo对象
    Evection evection = new Evection();
    // 设置出差天数
    evection.setNum(3d);
    // 定义流程变量，把出差pojo对象放入map
    map.put("evection", evection);
    // 启动流程实例，并设置流程变量的值（把map传入）
    ProcessInstance processInstance = runtimeService
        .startProcessInstanceByKey(key, map);
    // 输出
    System.out.println("流程实例名称："+processInstance.getName());
    System.out.println("流程定义id："+processInstance.getProcessDefinitionId());
}

/**
 * 完成任务
 */
@Test
public void completTask(){
    // 流程定义的Key
    String key = "evection-exclusive";
    // 任务负责人
    //   节点1:填写出差单
    // String assingee = "zhangsan";  
    //   节点2：部门经理审批
    // String assingee = "lisi";
    //   节点3：根据排他网关（exection.num=3)，现在应该执行的是节点3，即总经理审批
    // String assingee = "wangwu";  
    //   节点4：财务审批
    String assingee = "zhaoliu";
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取taskservice
    TaskService taskService = processEngine.getTaskService();
    // 查询任务
    Task task = taskService.createTaskQuery()
        .processDefinitionKey(key)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        // 根据任务id来   完成任务
        taskService.complete(task.getId());
    }
}
```

> 当执行完`部门经理审批`节点后，根据排网关，此时走的就是`总经理审批`节点，查看表act_ru_task：
>
> ```
> 总经理审批ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
> 7503	1	2505	2501	evection-exclusive:1:3	总经理审批	\N	\N	_6	\N	wangwu	\N	50	2022-05-08 13:38:27	\N	\N	1		\N	\N
> ```

**分支2：**即 `exection.num<3`

```java
// 上述方法startProcess中：
evection.setNum(2d);

// 上述方法completTask中：
// 任务负责人
//   节点1:填写出差单
// String assingee = "zhangsan";  
//   节点2：部门经理审批
// String assingee = "lisi"; 
//   节点4：财务审批，根据排他网关（exection.num=3)，现在应该执行的是节点4，即财务审批
String assingee = "zhaoliu";
```

> 当执行完`部门经理审批`节点后，根据排网关，此时走的就是`财务审批`节点，查看表act_ru_task：
>
> ```
> ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
> 20003	1	15005	15001	evection-exclusive:1:3	财务审批	\N	\N	_7	\N	zhaoliu	\N	50	2022-05-08 13:48:42	\N	\N	1		\N	\N
> ```

**分支3：**都满足的情况

> 重新定义流程（删库重来）

将流程定义为如下：

```xml
<process id="evection-exclusive" isClosed="false" isExecutable="true" name="出差审批-排他网关" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="填写出差单"/>
    <userTask activiti:assignee="lisi" activiti:exclusive="true" id="_4" name="部门经理审批"/>
    <exclusiveGateway gatewayDirection="Unspecified" id="_5" name="ExclusiveGateway"/>
    <userTask activiti:assignee="wangwu" activiti:exclusive="true" id="_6" name="总经理审批"/>
    <userTask activiti:assignee="zhaoliu" activiti:exclusive="true" id="_7" name="财务审批"/>
    <endEvent id="_8" name="EndEvent"/>
    <sequenceFlow id="_9" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_10" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_11" sourceRef="_4" targetRef="_5"/>
    <sequenceFlow id="_12" name="出差天数大于等于3天" sourceRef="_5" targetRef="_6">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_13" name="出差天数大于等于3天" sourceRef="_5" targetRef="_7">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_14" sourceRef="_6" targetRef="_7"/>
    <sequenceFlow id="_15" sourceRef="_7" targetRef="_8"/>
</process>
```

> 依据执行上方代码：`evection.setNum(3d);`

在完成节点2：`部门经理审批`时，流转到了节点3：`总经理审批`，因为此时`总经理审批`节点的id小于`财务审批`的节点id；体现出：**如果有两个分支条件都为true，排他网关会选择id值较小的一条分支去执行。**

**分支4：**异常情况，即都不满足的情况

> 还是用上面分支3的流程定义，还是执行上方代码：`evection.setNum(2d);`
>
> 在完成节点2：`部门经理审批`时，抛出异常：
>
> ```java
> org.activiti.engine.ActivitiException: No outgoing sequence flow of the exclusive gateway '_5' could be selected for continuing the process
> ```

### 5.2 并行网关-ParallelGateway

#### 5.2.1 概述

**并行网关：**并行网关允许将流程分成多条分支，也可以把多条分支汇聚到一起，并行网关的功能是基于进入和外出顺序流的。

**fork分支：**并行后的所有外出顺序流，为每个顺序流都创建一个并发分支。

**join汇聚：** 所有到达并行网关，在此等待的进入分支，直到所有进入顺序流的分支都到达以后，流程就会通过汇聚网关。

> 注意：
>
> * 如果同一个并行网关有多个进入和多个外出顺序流，它就同时具有分支和汇聚功能；这时，网关会先汇聚所有进入的顺序流，然后再切分成多个并行分支。
>
> * **与其他网关的主要区别是，并行网关不会解析条件。** **即使顺序流中定义了条件，也会被忽略。**

#### 5.2.2 流程定义

![并行网关图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402345.PNG)

```xml
<process id="evection-parallel" isClosed="false" isExecutable="true" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="创建出差单"/>
    <parallelGateway gatewayDirection="Unspecified" id="_4" name="ParallelGateway"/>
    <userTask activiti:assignee="lisi" activiti:exclusive="true" id="_5" name="技术经理"/>
    <userTask activiti:assignee="wangwu" activiti:exclusive="true" id="_6" name="项目经理"/>
    <parallelGateway gatewayDirection="Unspecified" id="_7" name="ParallelGateway"/>
    <exclusiveGateway gatewayDirection="Unspecified" id="_8" name="ExclusiveGateway"/>
    <userTask activiti:assignee="zhaoliu" activiti:exclusive="true" id="_9" name="总经理"/>
    <endEvent id="_10" name="EndEvent"/>
    <sequenceFlow id="_11" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_12" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_13" name="出差天数大于等于3天(不起作用)" sourceRef="_4" targetRef="_5">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_14" name="出差天数小于3天(不起作用)" sourceRef="_4" targetRef="_6">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_15" sourceRef="_5" targetRef="_7"/>
    <sequenceFlow id="_16" sourceRef="_6" targetRef="_7"/>
    <sequenceFlow id="_17" sourceRef="_7" targetRef="_8"/>
    <sequenceFlow id="_18" name="出差天数大于等于3天" sourceRef="_8" targetRef="_9">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[#{evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_19" sourceRef="_9" targetRef="_10"/>
    <sequenceFlow id="_20" name="出差天数小于3天" sourceRef="_8" targetRef="_10">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
    </sequenceFlow>
</process>
```

> 说明：
>
> - 技术经理和项目经理是两个execution分支，在act_ru_execution表有两条记录分别是技术经理和项目经理，act_ru_execution还有一条记录表示该流程实例。
>
> - 待技术经理和项目经理任务全部完成，在汇聚点汇聚，通过parallelGateway并行网关。
>
> - 并行网关在业务应用中常用于会签任务，**会签任务即多个参与者共同办理的任务**。

#### 5.2.2 流程测试

**定义流程：**

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
        .addClasspathResource("bpmn/evection-parallel.bpmn")
        .name("出差申请-并行网关")
        .deploy();
    // 4、输出部署信息
    System.out.println("流程部署id：" + deployment.getId());
    System.out.println("流程部署名称：" + deployment.getName());
}
```

**启动流程实例：**

```java
/**
 * 启动流程实例,设置流程变量的值
 */
@Test
public void startProcess(){
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RunTimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 流程定义key
    String key = "evection-parallel";
    // 创建变量集合
    Map<String, Object> map = new HashMap<>();
    // 创建出差pojo对象
    Evection evection = new Evection();
    // 设置出差天数
    evection.setNum(4d);
    // 定义流程变量，把出差pojo对象放入map
    map.put("evection",evection);
    // 启动流程实例，并设置流程变量的值（把map传入）
    ProcessInstance processInstance = runtimeService
        .startProcessInstanceByKey(key, map);
    // 输出
    System.out.println("流程实例名称："+processInstance.getName());
    System.out.println("流程定义id："+processInstance.getProcessDefinitionId());
}
```

**执行流程：**节点1`创建出差单`

```java
/**
 * 执行流程
 */
@Test
public void completTask(){
    // 流程定义的Key
    String key = "evection-parallel";
    // 任务负责人
    String assingee = "zhangsan";
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取taskservice
    TaskService taskService = processEngine.getTaskService();
    // 查询任务
    Task task = taskService.createTaskQuery()
        .processDefinitionKey(key)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        // 根据任务id来   完成任务
        taskService.complete(task.getId());
    }

}
```

> 执行完第一个任务节点后，进入并行网关，查看表数据：
>
> * act_ru_task，在任务执行表中，存在两条任务（根据上面设置的条件，可以看出并行网关的条件设定是不起作用的）
>
>   ```
>   ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
>   5004	1	2505	2501	evection-parallel:1:3	技术经理	\N	\N	_5	\N	lisi	\N	50	2022-05-08 14:35:31	\N	\N	1		\N	\N
>   5007	1	5002	2501	evection-parallel:1:3	项目经理	\N	\N	_6	\N	wangwu	\N	50	2022-05-08 14:35:31	\N	\N	1		\N	\N
>   ```
>
> * act_ru_identitylink，任务执行人表中，可以看到有3条数据
>
>   ```
>   ID_	REV_	GROUP_ID_	TYPE_	USER_ID_	TASK_ID_	PROC_INST_ID_	PROC_DEF_ID_
>   2509	1	\N	participant	zhangsan	\N	2501	\N
>   5005	1	\N	participant	lisi	\N	2501	\N
>   5008	1	\N	participant	wangwu	\N	2501	\N
>   ```
>
> * act_ru_execution，查询流程实例执行表，可以看到有两个分支在执行：
>
>   ```
>   ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
>   2501	2	2501	\N	\N	evection-parallel:1:3	\N	2501	\N	1	0	1	0	0	1	\N		\N	2022-05-08 14:33:17	\N	\N	0	0	0	0	0	0	0	0	0
>   2505	2	2501	\N	2501	evection-parallel:1:3	\N	2501	_5	1	0	0	0	0	1	\N		\N	2022-05-08 14:33:17	\N	\N	0	0	0	0	0	0	0	0	0
>   5002	1	2501	\N	2501	evection-parallel:1:3	\N	2501	_6	1	0	0	0	0	1	\N		\N	2022-05-08 14:35:31	\N	\N	0	0	0	0	0	0	0	0	0
>   ```

**执行流程：**继续执行，随意执行一个分支节点：

```java
// 上述方法：completTask
// 任务负责人：技术经理lisi
String assingee = "lisi";
```

> 对并行任务的执行：并行任务执行不分前后，由任务的负责人去执行即可。
>
> 当执行完成后，act_ru_task表中就少了一条数据。
>
> 在流程实例执行表act_ru_execution中有中多个分支存在且有并行网关的汇聚结点：（第二条数据）
>
> ```
> ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
> 2501	3	2501	\N	\N	evection-parallel:1:3	\N	2501	\N	1	0	1	0	0	1	\N		\N	2022-05-08 14:33:17	\N	\N	0	0	0	0	0	0	0	0	0
> 2505	3	2501	\N	2501	evection-parallel:1:3	\N	2501	_7	0	0	0	0	0	1	\N		\N	2022-05-08 14:33:17	\N	\N	0	0	0	0	0	0	0	0	0
> 5002	1	2501	\N	2501	evection-parallel:1:3	\N	2501	_6	1	0	0	0	0	1	\N		\N	2022-05-08 14:35:31	\N	\N	0	0	0	0	0	0	0	0	0
> ```
>
> 有并行网关的汇聚结点：说明有一个分支已经到汇聚，等待其它的分支到达。

**执行流程：**继续执行剩下的一个节点

```java
String assingee = "wangwu";
```

> 当所有分支任务都完成，都到达汇聚结点后：流程实例执行表act_ru_execution，执行流程实例已经变为总经理审批（由于之前条件设置为了`evection.num=4`)，说明流程执行已经通过并行网关
>
> ```
> ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
> 2501	4	2501	\N	\N	evection-parallel:1:3	\N	2501	\N	1	0	1	0	0	1	\N		\N	2022-05-08 14:33:17	\N	\N	0	0	0	0	0	0	0	0	0
> 5002	2	2501	\N	2501	evection-parallel:1:3	\N	2501	_9	1	0	0	0	0	1	\N		\N	2022-05-08 14:35:31	\N	\N	0	0	0	0	0	0	0	0	0
> ```

**总结：所有分支到达汇聚结点，并行网关执行完成。** 

### 5.3 包含网关-InclusiveGateway

#### 5.3.1 概述

**包含网关：**可以看做是排他网关和并行网关的结合体。 

* 和排他网关一样，可以在外出顺序流上定义条件，包含网关会解析它们；
* 但是主要的区别是包含网关可以选择多于一条顺序流，这和并行网关一样。

包含网关的功能是基于进入和外出顺序流的：

* 分支：所有外出顺序流的条件都会被解析，结果为true的顺序流会以并行方式继续执行，会为每个顺序流创建一个分支。

* 汇聚：所有并行分支到达包含网关，会进入等待状态，直到每个包含流程token的进入顺序流的分支都到达；这是与并行网关的最大不同。换句话说，**包含网关只会等待被选中执行了的进入顺序流**。 在汇聚之后，流程会穿过包含网关继续执行。

#### 5.3.2 流程定义

> 出差申请大于等于3天需要由项目经理审批，小于3天由技术经理审批，出差申请必须经过人事经理审批。

![包含网关图](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261402834.PNG)

```xml
<process id="evection-inclusive" isClosed="false" isExecutable="true" name="出差申请-包含网关" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="创建出差申请"/>
    <inclusiveGateway gatewayDirection="Unspecified" id="_4" name="InclusiveGateway"/>
    <userTask activiti:assignee="lisi" activiti:exclusive="true" id="_5" name="项目经理"/>
    <userTask activiti:assignee="wangwu" activiti:exclusive="true" id="_6" name="人事经理"/>
    <userTask activiti:assignee="zhaoliu" activiti:exclusive="true" id="_7" name="技术经理"/>
    <inclusiveGateway gatewayDirection="Unspecified" id="_8" name="InclusiveGateway"/>
    <exclusiveGateway gatewayDirection="Unspecified" id="_9" name="ExclusiveGateway"/>
    <userTask activiti:assignee="qianqi" activiti:exclusive="true" id="_10" name="总经理"/>
    <endEvent id="_11" name="EndEvent"/>
    <sequenceFlow id="_12" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_13" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_14" name="出差大于等于3天" sourceRef="_4" targetRef="_5">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_15" sourceRef="_4" targetRef="_6"/>
    <sequenceFlow id="_16" name="出差小于3天" sourceRef="_4" targetRef="_7">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_17" sourceRef="_5" targetRef="_8"/>
    <sequenceFlow id="_18" sourceRef="_6" targetRef="_8"/>
    <sequenceFlow id="_19" sourceRef="_7" targetRef="_8"/>
    <sequenceFlow id="_20" sourceRef="_8" targetRef="_9"/>
    <sequenceFlow id="_21" name="出差大于等于3天" sourceRef="_9" targetRef="_10">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num>=3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_22" name="出差小于3天" sourceRef="_9" targetRef="_11">
        <conditionExpression xsi:type="tFormalExpression"><![CDATA[${evection.num<3}]]></conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="_23" sourceRef="_10" targetRef="_11"/>
</process>
```

#### 5.3.3 流程测试

**定义流程：**

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
        .addClasspathResource("bpmn/evection-inclusive.bpmn")
        .name("出差流程-包含网关")
        .deploy();
    // 4、输出部署信息
    System.out.println("流程部署id：" + deployment.getId());
    System.out.println("流程部署名称：" + deployment.getName());
}
```

**启动流程实例：**

```java
/**
 * 启动流程实例,设置流程变量的值
 */
@Test
public void startProcess(){
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RunTimeService
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 流程定义key
    String key = "evection-inclusive";
    // 创建变量集合
    Map<String, Object> map = new HashMap<>();
    // 创建出差pojo对象
    Evection evection = new Evection();
    // 设置出差天数
    evection.setNum(4d);
    // 定义流程变量，把出差pojo对象放入map
    map.put("evection",evection);
    // 启动流程实例，并设置流程变量的值（把map传入）
    ProcessInstance processInstance = runtimeService
        .startProcessInstanceByKey(key, map);
    // 输出
    System.out.println("流程实例名称："+processInstance.getName());
    System.out.println("流程定义id："+processInstance.getProcessDefinitionId());
}
```

**执行流程：**节点1`创建出差单`，执行完成后就进入了包含网关

```java
/**
 * 执行流程
 */
@Test
public void completTask(){
    // 流程定义的Key
    String key = "evection-inclusive";
    // 任务负责人
    String assingee = "zhangsan";
    // 获取流程引擎
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取taskservice
    TaskService taskService = processEngine.getTaskService();
    // 查询任务
    Task task = taskService.createTaskQuery()
        .processDefinitionKey(key)
        .taskAssignee(assingee)
        .singleResult();
    if(task != null){
        // 根据任务id来   完成任务
        taskService.complete(task.getId());
    }

}
```

> 当流程执行到第一个包含网关后，会根据条件判断，当前要走哪几个分支：
>
> - act_ru_task，在任务执行表中，存在两条任务，即满足条件的一条`项目经理`，另一条即为都会执行的`人事经理`
>
>   ```
>   ID_	REV_	EXECUTION_ID_	PROC_INST_ID_	PROC_DEF_ID_	NAME_	PARENT_TASK_ID_	DESCRIPTION_	TASK_DEF_KEY_	OWNER_	ASSIGNEE_	DELEGATION_	PRIORITY_	CREATE_TIME_	DUE_DATE_	CATEGORY_	SUSPENSION_STATE_	TENANT_ID_	FORM_KEY_	CLAIM_TIME_
>   5004	1	2505	2501	evection-inclusive:1:3	项目经理	\N	\N	_5	\N	lisi	\N	50	2022-05-08 15:06:45	\N	\N	1		\N	\N
>   5007	1	5002	2501	evection-inclusive:1:3	人事经理	\N	\N	_6	\N	wangwu	\N	50	2022-05-08 15:06:45	\N	\N	1		\N	\N
>   ```
>
> - act_ru_execution，查询流程实例执行表，可以看到有两个分支在执行：
>
>   ```
>   ID_	REV_	PROC_INST_ID_	BUSINESS_KEY_	PARENT_ID_	PROC_DEF_ID_	SUPER_EXEC_	ROOT_PROC_INST_ID_	ACT_ID_	IS_ACTIVE_	IS_CONCURRENT_	IS_SCOPE_	IS_EVENT_SCOPE_	IS_MI_ROOT_	SUSPENSION_STATE_	CACHED_ENT_STATE_	TENANT_ID_	NAME_	START_TIME_	START_USER_ID_	LOCK_TIME_	IS_COUNT_ENABLED_	EVT_SUBSCR_COUNT_	TASK_COUNT_	JOB_COUNT_	TIMER_JOB_COUNT_	SUSP_JOB_COUNT_	DEADLETTER_JOB_COUNT_	VAR_COUNT_	ID_LINK_COUNT_
>   2501	2	2501	\N	\N	evection-inclusive:1:3	\N	2501	\N	1	0	1	0	0	1	\N		\N	2022-05-08 15:05:48	\N	\N	0	0	0	0	0	0	0	0	0
>   2505	2	2501	\N	2501	evection-inclusive:1:3	\N	2501	_5	1	0	0	0	0	1	\N		\N	2022-05-08 15:05:48	\N	\N	0	0	0	0	0	0	0	0	0
>   5002	1	2501	\N	2501	evection-inclusive:1:3	\N	2501	_6	1	0	0	0	0	1	\N		\N	2022-05-08 15:06:45	\N	\N	0	0	0	0	0	0	0	0	0
>   ```
>
>   其中：第一条记录：包含网关分支；
>
>   后两条记录代表两个要执行的分支：
>
>   * `ACT_ID = "_5"` 代表 `项目经理` 审批节点
>
>   * `ACT_ID = "_6"` 代表 `人事经理` 审批节点

**执行流程：**后续执行流程基本就和并网关相同了；

**小结：**在分支时，需要判断条件，**符合条件的分支，将会执行**，符合条件的分支最终才进行汇聚。

### 5.4 事件网关-EventGateway

> 不甚理解，没有测试。

#### 5.4.1 概述

事件网关允许根据事件判断流向，网关的每个外出顺序流都要连接到一个**中间捕获事件**。 

当流程到达一个基于事件网关，网关会进入等待状态：会暂停执行；与此同时，会为每个外出顺序流创建相对的事件订阅。

事件网关的外出顺序流和普通顺序流不同，这些顺序流不会真的"执行"，相反它们让流程引擎去决定执行到事件网关的流程需要订阅哪些事件。

事件网关要考虑以下条件：

1. 事件网关必须有两条或以上外出顺序流；
2. 事件网关后，只能使用intermediateCatchEvent类型（activiti不支持基于事件网关后连接ReceiveTask）
3. 连接到事件网关的中间捕获事件必须只有一个入口顺序流。

![事件类型-2](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261403483.PNG)

#### 5.4.2 流程定义

![事件网关](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206261403798.PNG)

```xml
<process id="evection-event" isClosed="false" isExecutable="true" name="出差申请-事件网关" processType="None">
    <startEvent id="_2" name="StartEvent"/>
    <userTask activiti:assignee="zhangsan" activiti:exclusive="true" id="_3" name="提交申请"/>
    <eventBasedGateway eventGatewayType="Exclusive" gatewayDirection="Unspecified" id="_4" instantiate="false" name="EventGateway"/>
    <intermediateCatchEvent id="_5" name="IntermediateCatchingEvent">
        <messageEventDefinition id="_5_ED_1"/>
    </intermediateCatchEvent>
    <intermediateCatchEvent id="_6" name="IntermediateCatchingEvent">
        <signalEventDefinition id="_6_ED_1"/>
    </intermediateCatchEvent>
    <intermediateCatchEvent id="_7" name="IntermediateCatchingEvent">
        <timerEventDefinition id="_7_ED_1"/>
    </intermediateCatchEvent>
    <userTask activiti:assignee="lisi" activiti:exclusive="true" id="_8" name="消息任务"/>
    <userTask activiti:assignee="wangwu" activiti:exclusive="true" id="_9" name="信号任务"/>
    <userTask activiti:assignee="zhaoliu" activiti:exclusive="true" id="_10" name="定时任务"/>
    <exclusiveGateway gatewayDirection="Unspecified" id="_11" name="ExclusiveGateway"/>
    <endEvent id="_12" name="EndEvent"/>
    <sequenceFlow id="_13" sourceRef="_2" targetRef="_3"/>
    <sequenceFlow id="_14" sourceRef="_3" targetRef="_4"/>
    <sequenceFlow id="_15" sourceRef="_4" targetRef="_5"/>
    <sequenceFlow id="_16" sourceRef="_4" targetRef="_6"/>
    <sequenceFlow id="_17" sourceRef="_4" targetRef="_7"/>
    <sequenceFlow id="_18" sourceRef="_5" targetRef="_8"/>
    <sequenceFlow id="_19" sourceRef="_6" targetRef="_9"/>
    <sequenceFlow id="_20" sourceRef="_7" targetRef="_10"/>
    <sequenceFlow id="_21" sourceRef="_8" targetRef="_11"/>
    <sequenceFlow id="_22" sourceRef="_9" targetRef="_11"/>
    <sequenceFlow id="_23" sourceRef="_10" targetRef="_11"/>
    <sequenceFlow id="_24" sourceRef="_11" targetRef="_12"/>
</process>
```


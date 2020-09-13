# 整合Spring

前面我们介绍的例子没有用到Spring，实际开发中，Activiti一般都是和Spring整合在一起的。这篇笔记中我们介绍下Activiti和Spring整合的相关配置。

## applicationContext.xml

要使用Activiti，就需要配置`processEngine`。除此之外，Activiti的七大接口也可以通过Spring自动装配。下面是相关配置：

```xml
<!-- activiti -->
<bean id="processEngineConfiguration"
    class="org.activiti.spring.SpringProcessEngineConfiguration">
    <property name="dataSource" ref="dataSource" />
    <property name="transactionManager" ref="transactionManager" />
    <property name="databaseSchemaUpdate" value="true" />
    <property name="jobExecutorActivate" value="false" />
</bean>
<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
    <property name="processEngineConfiguration" ref="processEngineConfiguration"/>
</bean>
<bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService" />
<bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService" />
<bean id="taskService" factory-bean="processEngine" factory-method="getTaskService" />
<bean id="formService" factory-bean="processEngine" factory-method="getFormService" />
<bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService" />
<bean id="managementService" factory-bean="processEngine" factory-method="getManagementService" />
<bean id="identityService" factory-bean="processEngine" factory-method="getIdentityService" />
```

`processEngineConfiguration`中，我们主要配置了数据源`dataSource`和事务管理器`transactionManager`，这两个在Spring中基本是必须配置的。`databaseSchemaUpdate`用于自动建表，如果不需要，设置为`false`比较好。`jobExecutorActivate`用于计划任务，如果有流程涉及定时任务，需要配置该字段为`true`。

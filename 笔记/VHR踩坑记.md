# VHR踩坑记

> 跨域请求问题

这本来不是问题，源码已经加了前端解决方案在`vue.config.js`中，但是手贱自己再去加了一个后端跨域请求配置类，当时不知道前端的解决方案，反正把我搞迷糊了。具体就去看笔记

> 1. elementui表格中与属性绑定的是prop不是type

```xml
<el-table :data="perEc" border stripe>
    <el-table-column label="序号" prop="id"></el-table-column>
    <el-table-column label="员工编号" prop="eid"></el-table-column>
    <el-table-column label="员工姓名" prop="name"></el-table-column>
    <el-table-column label="员工姓名" prop="name"></el-table-column>
</el-table>
```

> 2. 做之前要想好每一步都要干些什么，最好先写伪代码 注释 ，最好想到我有哪些参数，最后输出什么

> 3. 在数据传输是，要考虑写的对象是否序列化过，例如这次rabbitmq生产者要传输数据但是没有序列化EmployeeecCustom但是不会执行任何报错，最后用try catch检查出了是没有序列化

```java
 Map<String, Object> map = new HashMap();
            Employee employee = employeeMapper.getEmployeeById(employeeecCustom.getEid());
            map.put(MailConstants.EMPLOYEEEC_CUSTOM, lastEmpec);
map.put(MailConstants.EMPLOYEE, employee);
try {
    rabbitTemplate.convertAndSend(MailConstants.EMPEC_MAIL_EXCHANGE_NAME, MailConstants.EMPEC_MAIL_ROUTING_KEY_NAME, map, new CorrelationData(msgId));
} catch (AmqpException e) {
    e.printStackTrace();
}
```


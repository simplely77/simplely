# Junit单元测试框架

___注意事项 :___ 
- 声明测试类
- 测试方法必须公共、无参、无返回值
- 必须声明`@Test`注解

___断言机制 :___ 
```Java
Assert.assertEquals("方法内部有bug"，expected，actual)
```

___常见注解 :___
```Java
@Test
@Before
@After
@BeforeClass
@AfterClass
```

后面四个常用于__资源初始化和释放__,全局资源使用`@BeforeClass`、`@AfterClass`,方法内资源使用`@Before`、`@After`,新版本__junit__已经改名


---
title: Java基础-JUnit单元测试
abbrlink: 9cc058c7
date: 2020-01-30 16:00:29
categories:
  - Java
  - Junit
tags: [Java, Junit]
---

## 简介

单元测试, 是对代码可执行性的一种测试方式,测试的是代码的可执行性,不对代码逻辑进行测试.
常见的单元测试放在 `test` 目录下,和 `src` 同一层级,目录层次与 `src` 目录保持一致,便于测试.
在项目正式发布时,删除或者跳过 `test` 目录



## 测试类规范

 1. 测试方法上必须使用 `@Test` 进行修饰
 2. 测试方法必须使用 `public void` 进行修饰，不能待任何的参数
 3. 新建一个源代码目录
 4. 测试类的包应该和被测试类保持一致
 5. 测试单元中的每个方法必须可以独立测试，测试方法间不能有任何的依赖
 6. 测试类使用 `-Test`作为类名的后缀
 7. 测试方法使用 `test` 作为方法名的前缀





## 包结构

* `src`  模拟源代码目录
* `test`  测试源代码目录



## 子包描述

### `src` 包
封装了一个用来测试的 `Compute` 类,其中只有一个 `division()` 方法,实现传入两个数的除法运算

``` java
/**	单元测试的带基本类型输入/输出的方法测试*/
public int division (int i,int j) {
	return i/j;
}
```

### `test` 包
- `ComputeTest`  , `Compute` 类的断言测试类
- `JunitException`  具有忽略异常的测试类
- `Junitlifecycle`  单元测试的生命周期
- `ParameterTest`  参数化测试,适合匹配大量测试数据的测试
- `SuitTest`  测试套件,本身为空类,一次运行多个测试类

#### 断言测试
断言测试是对具有返回结果的方法进行测试,验证输入参数后,返回结果是否和预期一致

``` java
@Test
public void testdivision() {
	//简单除法计算, 3 ?= 6 / 2 
	assertEquals(3, new Compute().division (6, 2));
	System.out.println("计算成功...");
}
```



#### 忽略运行时异常测试

针对运行时可能出现的异常,为了避免该异常打断我们的测试,可以在 `@Test` 注解中声明 `expected` 属性,指定忽略的运行期异常; `timeout` 设定超时失败时间.

``` java
	/**	使用单元测试,运行异常*/
	@Test(expected=ArithmeticException.class,timeout=100)
	public void testAdd() {
		//抛出一个零除异常
		assertEquals(1, new Compute().division (1, 0));
	}
```


#### 单元测试运行周期

| 注解         | 执行时期             | 说明                                       |
| ------------ | -------------------- | ------------------------------------------ |
| @BeforeClass | 类加载前执行         | 静态方法,在类初始化时执行,适合加载各种配置 |
| @Before      | 每一个测试方法前执行 | 适合放一些多个测试类执行前均会经过的方法   |
| @Test        | 测试方法             | 测试方法必须是共有的无声明异常的方法       |
| @Ignore      | 忽略测试             | 该方法将不会被测试                         |
| @After       | 每一个测试方法后执行 | 适合放一些多个测试类执行后均会经过的方法   |
| @AfterClass  | 类执行完后执行       | 适合资源的清理,关闭各种连接                |

**示例**

``` java
/**单元测试的运行周期 */
public class Junitlifecycle {
	
	/**静态方法,在类初始化时执行,适合加载各种配置*/
	@BeforeClass
	public static void setUpBeforeClass() throws Exception {
		System.out.println("类实初始化前创建,只执行一次");
	}

	/**前面的通用代码块*/
	@Before
	public void setUp() throws Exception {
		System.out.println("测试方法前执行,执行多次");
	}

	@Test
	public void test1() {
		System.out.println("测试代码1....");
	}
	
	@Ignore("不执行该方法")				//被该注解修饰后,不再执行
	@Test
	public void test2() {
		System.out.println("测试代码2....");
	}
	/**后面的通用代码块*/
	@After
	public void tearDown() throws Exception {
		System.out.println("测试方法后执行,执行多次");
	}

	/**对资源的清理,关闭各种连接*/
	@AfterClass
	public static void tearDownAfterClass() throws Exception {
		System.out.println("类销毁时创建,只执行一次");
	}
	
}
```



#### 参数化测试

 1. 设定为 `@RunWith(Parameterized.class)` 运行环境
 2. 声明变量存放预期值和参数
 3. 使用变量和预期值作为构造函数,并赋值
 4. 声明返回值为集合类的静态方法,作为参数,并 `@Parameters` 修饰
 5. 传入类中的变量,完成赋值,进行断言

``` java
/**	参数化测试*/
@RunWith(Parameterized.class)		//1.改变运行环境
public class ParameterTest {

	//2.声明变量存放预期值和参数
	int result = 0;
	int param1 = 4;
	int param2 = 2;
	//3.使用变量和预期值作为构造函数,并赋值
	public ParameterTest(int result, int param1, int param2) {
		super();
		this.result = result;
		this.param1 = param1;
		this.param2 = param2;
	}
	
	
	//4.声明返回值为集合类的静态方法,作为参数,并修饰
	@Parameters
	public static Collection<Object[]> returnList() {
		
		List<Object[]> list = new ArrayList<Object[]>();		//作为返回值
		Object[][] objects = new Object[][]{{2,2,1},{3,9,3}};	//创建一个二维数组,保存为变量和结果	2=2/1  3=9/3
		list = Arrays.asList(objects);
		return list;
	}

	//5.传入类中的变量,完成赋值,进行断言
	@Test
	public void testdivision() {
		//使用有参的方式,传入一个List
		assertEquals(result, new Compute().division (param1, param2));
		System.out.println("计算成功...");
	}
}
```



#### 测试套件

对多个测试方法进行统一测试的类,本身为空类,作为入口类.

 1. `@RunWith(Suite.class)` 注解该类为测试套件类
 2. `@SuiteClasses({})` 存放将要运行的各种测试类

``` java
@RunWith(Suite.class)			//套件运行专用类
@SuiteClasses({ComputeTest.class,Junitlifecycle.class})		//多个运行的测试类...
public class SuitTest {

}
```
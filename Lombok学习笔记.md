1.Lombok的安装


- Intellij 设置支持注解处理
    
- - 点击 File > Settings > Build > Annotation Processors
        
    - 勾选 Enable annotation processing（这里一般来说项目都会默认开启我感觉，实际开发的时候如果不生效记得看一看是不是这里的问题）
        
- 安装插件
    
- - 点击 Settings > Plugins > Browse repositories
        
    - 查找 Lombok Plugin 并进行安装
        
    - 重启 IntelliJ IDEA
        

- 将 lombok 添加到 pom 文件
    
```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.8</version>
</dependency>
```
这里的话只需要把依赖导入到pom文件中即可

2.Lombok的基本使用：

Lombok的主要作用就是提供注解api来修饰制定的类。

这里我们先来介绍其基本注解，并给出例子说明如何作用

	2.1- `val`：用在局部变量前面，相当于将变量声明为 final
	
	public static void main(String[] args) { 
	val sets = new HashSet<String>(); 
	val lists = new ArrayList<String>(); 
	val maps = new HashMap<String, String>(); 
	//=>相当于如下 
	final Set<String> sets2 = new HashSet<>(); 
	final List<String> lists2 = new ArrayList<>(); 
	final Map<String, String> maps2 = new HashMap<>(); }
	
	2.2- `@NonNull`：给方法参数增加这个注解，会自动在方法内对该参数进行是否为空的校验，如果为空，则抛出 NPE（NullPointerException）
	public void notNullExample(@NonNull String string) { string.length(); 
	} 
	//=>相当于 
	public void notNullExample(String string)
	 { if (string != null) 
	 { string.length(); } 
	 else {
	  throw new NullPointerException("null"); }
	   }
	   
	  
	2.3- `@Cleanup`：自动管理资源，用在局部变量之前，在当前变量范围内即将执行完毕退出之前会自动清理资源，自动生成 try-finally 这样的代码来关闭流
	public static void main(String[] args) {
    try {
        @Cleanup InputStream inputStream = new FileInputStream(args[0]);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }

    // => 相当于

    InputStream inputStream = null;
    try {
        inputStream = new FileInputStream(args[0]);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } finally {
        if (inputStream != null) {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
	}
	2.4- `@Getter/@Setter`：用在属性上，再也不用自己手写 setter 和 getter 方法了，还可以指定访问范围
	@Setter(AccessLevel.PUBLIC)
	@Getter(AccessLevel.PROTECTED)
	private int id;
	private String shap;
	
	2.5- `@ToString`：用在类上，可以自动覆写 toString 方法，当然还可以加其他参数，例如@ToString(exclude=”id”)排除 id 属性，或者@ToString(callSuper=true, includeFieldNames=true)调用父类的 toString 方法，包含所有属性
	
	
	@ToString(
    exclude = "id",
    callSuper = true,
    includeFieldNames = true)
	public class LombokDemo {
	
    private int id;
    private String name;
    private int age;

    public static void main(String[] args) {
        // 输出：LombokDemo(super=LombokDemo@48524010, name=null, age=0)
        System.out.println(new LombokDemo());
    }
}
	2.6- `@EqualsAndHashCode`：用在类上，自动生成 equals 方法和 hashCode 方法
	
	 @EqualsAndHashCode(exclude = {"id", "shape"}, callSuper = false) public class LombokDemo { private int id; private String shap; }
	2.7- `@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor`：用在类上，自动生成无参构造和使用所有参数的构造函数以及把所有@NonNull 属性作为参数的构造函数，如果指定 staticName = “of”参数，同时还会生成一个返回类对象的静态工厂方法，比使用构造函数方便很多
						@NoArgsConstructor
						@RequiredArgsConstructor
						@AllArgsConstructor
						public class User {

						    @NonNull
						    private String name;
						
						    private int age;
						}
	### **生成效果说明：**

- @NoArgsConstructor
    
    → User()
    
- @RequiredArgsConstructor
    
    → User(String name)（只包含 @NonNull 或 final 字段）
    
- @AllArgsConstructor
    
    → User(String name, int age)
    
	
	
	2.8- `@Data`：注解在类上，相当于同时使用了@ToString、@EqualsAndHashCod- e、@Getter、@Setter 和@RequiredArgsConstrutor 这些注解，对于 POJO 类十分有用
	
	这个就不给例子了。
	
	2.9- `@Value`：用在类上，是@Data 的不可变形式，相当于为属性添加 final 声明，只提供 getter 方法，而不提供 setter 方法
			@Value
			public class Book {
			    String title;
			    int price;
			}
		说明：

		- 类默认为 final
		    
		- 字段默认为 private final
		    
		- 只生成 getter
    
		- 适合不可变对象（Immutable Object）
	2.10- `@Builder`：用在类、构造器、方法上，为你提供复杂的 builder APIs，让你可以像如下方式一样调用 Person.builder().name("Adam Savage").city("San Francisco").job("Mythbusters").job("Unchained Reaction").build();更多说明参考 Builder
				@Builder
			public class Person {
			    private String name;
			    private int age;
			    private String city;
			}
	使用示例如下：
		Person person = Person.builder()
	    .name("Adam Savage")
	    .age(50)
	    .city("San Francisco")
	    .build();
	
	
	
	2.11- `@SneakyThrows`：自动抛受检异常，而无需显式在方法上使用 throws 语句
			@SneakyThrows
		public void readFile() {
		    FileInputStream inputStream = new FileInputStream("test.txt");
		    inputStream.read();
		}
	2.12- `@Synchronized`：用在方法上，将方法声明为同步的，并自动加锁，而锁对象是一个私有的属性 或LOCK，而 java 中的 synchronized 关键字锁对象是 this，锁在 this 或者自己的类对象上存在副作用，就是你不能阻止非受控代码去锁 this 或者类对象，这可能会导致竞争条件或者其它线程错误
					public class OrderService {
			
			    private int orderCount = 0;
			
			    @Synchronized
			    public void createOrder() {
			        orderCount++;
			    }
					}
		
		#### **Lombok 实际生成的效果（等价概念）：**
		public class OrderService {

		    private final Object $lock = new Object();
		    private int orderCount = 0;
		
		    public void createOrder() {
		        synchronized ($lock) {
		            orderCount++;
		        }
		}
}
	2.13- `@Log`：根据不同的注解生成不同类型的 log 对象，但是实例名称都是 log，有六种可选实现类
	    - `@CommonsLog` Creates log =   org.apache.commons.logging.LogFactory.getLog(LogExample.class);
	    - `@Log` Creates log = java.util.logging.Logger.getLogger(LogExample.class.getName());
	    - `@Log4j` Creates log = org.apache.log4j.Logger.getLogger(LogExample.class);
	    - `@Log4j2` Creates log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
	    - `@Slf4j` Creates log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
	    - `@XSlf4j` Creates log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
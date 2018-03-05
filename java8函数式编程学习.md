# JAVA8函数式编程学习

## 第1章 简介

### 1.1 为什么需要再次修改JAVA

面向对象编程是对数据进行抽象；函数式编程是对行为进行抽象

### 1.2 什么是函数式编程

核心：在思考问题时，使用不可变值和函数，函数对一个值进行处理，映射成另一个值

### 1.3 示例

## 第2章 Lambda表达式

### 2.1 第一个lambda表达式

使用匿名内部类将行为和按钮单击进行关联

	button.addActionListener(new ActionListener() { 
		public void actionPerformed(ActionEvent event) {             System.out.println("button clicked");       }	});
这实际上是代码即数据的例子，我们给按钮传递了一个代表某种行为的对象
缩写后的lambda表达式
	button.addActionListener(event -> System.out.println("button clicked"));
event是参数名，->将参数和lambda表达式主体分开；使用匿名内部类时需要显示的声明参数类型，lambda表达式可以根据上下文在后台推断参数类型
### 2.2 如何辨别Lambda表达式
	Runnable noArguments = () -> System.out.println("Hello World");

Lamda表达式不包含参数，使用空括号()表示没有参数，该lambda表达式实现了Runnable接口，也只有一个方法run，没有参数。		ActionListener oneArgument = event -> System.out.println("button clicked");

包含且只包含一个参数，可省略参数的括号
	Runnable multiStatement = () -> {
		System.out.print("Hello");          System.out.println(" World");    };
Lambda表达式的主体可以是一段代码块
    BinaryOperator<Long> add = (x, y) -> x + y;
Lambda表达式也可以表示包含多个参数的方法，这行代码创建了一个函数
    BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;
可以显式声明参数类型
### 2.3 引用值，而不是变量
匿名内部类使用所在方法的变量时，需要变量为final局部变量
	final String name = getUserName(); 
	button.addActionListener(new ActionListener() {		public void actionPerformed(ActionEvent event) {
			 System.out.println("hi " + name);		} 	});

Java8要求是既成事实上的final，即只能给该变量复制一次，换而言之，lamda表达式引用的是值，而不是变量

	String name = getUserName();	button.addActionListener(event -> System.out.println("hi " + name));
Lambda表达式也被称为闭包
### 2.4 函数接口
函数接口是只有一个抽象方法的接口，用作lambda表达式的类型
ActionListener 接口:接受 ActionEvent 类型的参数,返回空
	public interface ActionListener extends EventListener { 
		public void actionPerformed(ActionEvent event);	}这就是函数接口，接口中单一方法命名并不重要，只要方法签名和lambda表达式中的类型匹配就可以。
用图形表示不同类型的函数接口，指向函数接口的箭头表示参数
### 2.5 函数接口

Lambda表达式中的类型推断，实际上是Java 7开始引入的目标类型推断的扩展。

使用菱形操作符，根据变量类型做推断

	Map<String, Integer> oldWordCounts = new HashMap<String, Integer>(); 	Map<String, Integer> diamondWordCounts = new HashMap<>();
变量diamondWordCounts使用了菱形操作符，不明确声明泛型类型，编译器可以自己推断出来。
将构造函数直接传递给一个方法，也可以根据方法签名来判断类型
使用菱形操作符，根据方法签名做推断
	useHashmap(new HashMap<>());	...	private void useHashmap(Map<String, String> values);
程序仍然要经过类型检查来保证运行的安全性，但不用再显示声明类型，这就是所谓的类型推断
使用Lambda表达式检测一个Integer是否大于5
类型推断
	Predicate<Integer> atLeast5 = x -> x > 5;
Predicate接口的源码，接受一个对象，返回一个布尔值
	public interface Predicate<T> { 		boolean test(T t);	}
略显复杂的函数接口： BinaryOperator,该接口接受两个参数，返回一个值
	BinaryOperator<Long> addLongs = (x, y) -> x + y;
### 2.6 要点回顾
- Lambda表达式是一个匿名方法，将行为像数据一样传递- Lambda表达式的常见解构: BinaryOperator<Integer> add = (x, y) → x + y
- 函数接口指仅具有单个抽象方法的接口，用了表示Lambda表达式的类型
## 第三章 流
对核心类库的改进主要包括集合类的API和新引入的流(Stream)
### 3.1 从外部迭代到内部迭代
通用模式的迭代代码
	int count = 0;	for (Artist artist : allArtists) {		if (artist.isFrom("London")) { 
			count++;		}
	}
	
for循环实际上是一个封装了迭代的语法糖，使用外部迭代方式，使用迭代方式的代码如下

	int count = 0;	Iterator<Artist> iterator = allArtists.iterator(); 	while(iterator.hasNext()) {		Artist artist = iterator.next(); 
		if (artist.isFrom("London")) {			count++; 
		}	}
较难抽象，本质上是一种串行化操作
另一种方法是内部迭代，stream返回内部迭代中的相应接口
	long count = allArtists.stream()									.filter(artist -> artist.isFrom("London"))									.count();
									
Stream是用函数式编程方式在集合类上进行复杂操作的工具

### 3.2 实现机制

整个过程被分解为两个操作：过滤和计数，但只需要对列表迭代一次

只过滤，不计数

	allArtists.stream()               .filter(artist -> artist.isFrom("London"));
filter只刻画出了stream，但没有产生新的集合。只描述Stream，不产生新集合的方法叫做惰性求值方法，count之类最终会从Stream中产生值的方法叫做及早求值方法
由于使用了惰性求值，没有输出艺术家的名字
	
	allArtists.stream()               .filter(artist -> {                   System.out.println(artist.getName());				   return artist.isFrom("London");				});
加入一个用于终止操作的流，艺术家的名字就会被输出
输出艺术家的名字
	long count = allArtists.stream()                            .filter(artist -> {                                System.out.println(artist.getName());								return artist.isFrom("London"); 								})								.count();输出如下
	John Lennon    Paul McCartney    George Harrison    Ringo Starr

如果返回值是Stream，那么是惰性求值，如果是另一个只为空，那么是及早求值，形成一个惰性求值的链，最后用一个及早求值返回结果。

类似建造者模式，使用一系列操作设置属性和配置，最后调用build方法，此时对象才被真正创建

### 3.3 常用的流操作

#### 3.3.1 collect(toList())

- collect(toList())方法由Stream里的值生成一个列表，是一个及早求值操作

Stream的of方法使用一组初始值生成新的Stream，下面是使用collect方法的一个例子

	List<String> collected = Stream.of("a", "b", "c") 
											 .collect(Collectors.toList());	assertEquals(Arrays.asList("a", "b", "c"), collected);
使用collect(toList())方法从Stream中生成一个列表，在调用Stream上一系列惰性操作后，再调用一个类似collect的及早求值方法。
#### 3.3.2 map
- 如果有一个函数可以将一种类型的值转换成另外一种类型，map操作就可以使用该函数，将一个流中的值转换成一个新的流。
使用for循环将字符串转换为大写
	List<String> collected = new ArrayList<>();	for (String string : asList("a", "b", "hello")) {          String uppercaseString = string.toUpperCase();          collected.add(uppercaseString);    }	assertEquals(asList("A", "B", "HELLO"), collected);
使用map操作替代for循环将字符串转换为大写形式
	List<String> collected = Stream.of("a", "b", "hello")											.map(string -> string.toUpperCase())                                    		.collect(toList());    assertEquals(asList("A", "B", "HELLO"), collected);
传给map的Lambda表达式只接受一个String类型的参数，返回一个新的String。Lamba表达式必须是Function接口的一个实例
#### 3.3.3 filter
- 遍历数据并检查其中的元素时，可尝试使用Stream中提供的新方法filter
使用循环遍历列表，使用条件语句做判断
	List<String> beginningWithNumbers = new ArrayList<>();
	for(String value : asList("a", "1abc", "abc1")) {		if (isDigit(value.charAt(0))) { 			beginningWithNumbers.add(value);		} 
	}	assertEquals(asList("1abc"), beginningWithNumbers);
函数式风格
	List<String> beginningWithNumbers       = Stream.of("a", "1abc", "abc1")               .filter(value -> isDigit(value.charAt(0)))               .collect(toList());
	assertEquals(asList("1abc"), beginningWithNumbers);
	
和map很像，filter接受一个函数作为参数，返回值为布尔型。for循环中存在if语句，说明可以用filter方法进行重构

#### 3.3.4 flatMap

- flatMap方法可用Stream替换值，然后将多个Stream连接成一个Stream

包含多个列表的Stream

	List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))                                    .flatMap(numbers -> numbers.stream())                                    .collect(toList());    assertEquals(asList(1, 2, 3, 4), together);
#### 3.3.5 max和min
使用Stream查找最短曲目
	List<Track> tracks = asList(new Track("Bakai", 524),								new Track("Violets for Your Furs", 378),								new Track("Time Was", 451));    Track shortestTrack = tracks.stream()                                 .min(Comparator.comparing(track -> track.getLength()))                                 .get();    assertEquals(tracks.get(1), shortestTrack);

为了让Stream对象按照曲目长度进行排序，需要传给它一个Comparator对象。

max和min返回一个Optional对象，通过调用get方法可以取出Optional对象中的值。

#### 3.3.6 通用模式

使用for循环重写min，查找最短曲目

	List<Track> tracks = asList(new Track("Bakai", 524),	new Track("Violets for Your Furs", 378),new Track("Time Was", 451));	Track shortestTrack = tracks.get(0);
	for (Track track : tracks) {		if (track.getLength() < shortestTrack.getLength()){
			 shortestTrack = track;		}
	}    assertEquals(tracks.get(1), shortestTrack);

reduce模式的伪代码

	Object accumulator = initialValue;
	for(Object element : collection) {        accumulator = combine(accumulator, element);    }
首先赋给accumulator一个初始值，在循环体中，通过调用combine函数，拿accumulator和集合中的每一个元素做运算，再将运算结果赋给accumulator,最后accumulator的值就是想要的结果。
#### 3.3.7 reduce
reduce操作可以实现从一组值中生成一个值，count,min,max方法都是reduce操作。
使用reduce求和，Lambda表达式就是reducer，acc是累加器
	int count = Stream.of(1, 2, 3)                       .reduce(0, (acc, element) -> acc + element);    assertEquals(6, count);

- 4.2节将介绍另外一种标准类库内置的求和方法，在实际生产环境中，应该使用那种方式

展开reduce操作

	BinaryOperator<Integer> accumulator = (acc, element) -> acc + element;
	int count = accumulator.apply(                     accumulator.apply(                         accumulator.apply(0, 1),									2),
				3);
				
使用命令式编程方式求和的JAVA代码

	int acc = 0;	for (Integer element : asList(1, 2, 3)) {         acc = acc + element;    }    assertEquals(6, acc);在命令式编程方式下，每一次循环将集合中的元素和类将其相加，用相加后的结果更新累加器的值。对集合来说，循环在外部，且需要手动更新变量。
#### 3.3.8 整合操作
如何将问题分解为建单的Stream操作
- 将问题分解步骤
- 找出问题对应的Stream API
整合所有操作得到如下代码
	Set<String> origins = album.getMusicians()                                .filter(artist -> artist.getName().startsWith("The"))                                .map(artist -> artist.getNationality())                                .collect(toSet());
通过Stream暴露集合的最大优点在于，它很好地封装了内部实现的数据结构，仅暴露一个Stream接口，用户在实际操作中如何使用，都不会影响内部的List或Set。
### 3.4重构遗留代码
将一段使用循环进行集合操作的代码，重构成基于Stream的操作
遗留代码：找出长度大于1分钟的曲目
	public Set<String> findLongTracks(List<Album> albums) {
		Set<String> trackNames = new HashSet<>();		for(Album album : albums) {			for (Track track : album.getTrackList()) {
				if (track.getLength() > 60) {                     String name = track.getName();                     trackNames.add(name);                }			}
		}        return trackNames;    }

重构第一步：找出长度大于1分钟的曲目

	public Set<String> findLongTracks(List<Album> albums) { 
		Set<String> trackNames = new HashSet<>();
		albums.stream()               .forEach(album -> {                   album.getTracks()				   		.forEach(track -> {							if (track.getLength() > 60) {							}); 						});		return trackNames;	 }将内部的forEach方法使用Stream操作拆分后
	public Set<String> findLongTracks(List<Album> albums) {
		Set<String> trackNames = new HashSet<>(); 
		albums.stream()        	.forEach(album -> {            	album.getTracks()                   .filter(track -> track.getLength() > 60)                   .map(track -> track.getName())                   .forEach(name -> trackNames.add(name));			});		return trackNames;	}使用flatMap操作替换外层的forEach
	public Set<String> findLongTracks(List<Album> albums) {         Set<String> trackNames = new HashSet<>();         albums.stream()               .flatMap(album -> album.getTracks())               .filter(track -> track.getLength() > 60)               .map(track -> track.getName())               .forEach(name -> trackNames.add(name));		 return trackNames;	}
使用collect操作，删掉变量trackNames
	public Set<String> findLongTracks(List<Album> albums) {
		return albums.stream()			.flatMap(album -> album.getTracks())            .filter(track -> track.getLength() > 60)            .map(track -> track.getName())            .collect(toSet());    }

### 3.5 多次调用流操作

用户也可以选择每一步强制对函数求值，但最好不要如此操作

	List<Artist> musicians = album.getMusicians()                                   .collect(toList());    List<Artist> bands = musicians.stream()                                   .filter(artist -> artist.getName().startsWith("The"))                                   .collect(toList());    Set<String> origins = bands.stream()                                .map(artist -> artist.getNationality())                                .collect(toSet());
	
符合Stream使用习惯的链式调用

	Set<String> origins = album.getMusicians()                                .filter(artist -> artist.getName().startsWith("The"))                                .map(artist -> artist.getNationality())                                .collect(toSet());                                

和流的链式调用相比有如下缺点
- 代码可读性差，样板代码太多，隐藏了真正的业务逻辑
- 效率差，每一步都要对流及早求值，生成新的集合
- 代码充斥一堆垃圾变量，它们只用来保存中间结果，除此之外毫无用处
- 难于自动并行化处理

*尚未习惯不能成为拆开链式操作的理由，和建造者模式一样，按规则写出每一行代码，可以帮助用户慢慢习惯这种链式操作*

### 3.6 高阶函数

高阶函数是指接受另外一个函数作为参数，或返回一个函数的函数。如果函数的参数列表里包含函数接口，或该函数返回一个函数接口，那么该函数就是高阶函数。

### 3.7 正确使用Lambda表达式

明确要说明什么转化，而不是说明如何转化。

没有副作用的函数不会改变程序或外界的状态。

有副作用的代码，将参数保存到了成员变量

	private ActionEvent lastEvent;	private void registerHandler() {
		button.addActionListener((ActionEvent event) -> {			this.lastEvent = event;			}); 
	}
	
无论何时，将Lambda表达式传给Stream上的高阶函数，都应该尽量避免副作用(forEach方法除外)

### 3.8 要点回顾

- 内部迭代将更多控制权交给集合类
- 和Iterator类似，Stream是一种内部迭代方法
- 将Lambda表达式和Stream上方法结合起来，可以完成很多常见的集合操作

### 3.9 练习

## 4 类库

如何使用Lambda表达式

Java 8 引入了默认方法和接口的静态方法

### 4.1 在代码中使用Lambda表达式

使用isDebugEnabled方法降低日志性能开销

	Logger logger = new Logger();
	if (logger.isDebugEnabled()) {        logger.debug("Look at this: " + expensiveOperation());    }
使用Lambda表达式简化日志代码
	Logger logger = new Logger();    logger.debug(() -> "Look at this: " + expensiveOperation());
从类库的角度，使用内置的Supplier函数接口，只有一个get方法

启用Lambda表达式实现的日志记录器

	public void debug(Supplier<String> message) { if (isDebugEnabled()) {             debug(message.get());         }	}
调用get方法，相当于调用传入的Lambda表达式，这种方式也能和匿名内部类一起工作，可以实现向后兼容。
不同函数接口有不同的方法。	
### 4.2 基本类型
基本类型(int)内建在语言和运行环境中，是基本的程序构建模块；装箱类型(Integer)属于普通的Java类，是对基本类型的一种封装。
Java的泛型是基于对泛型参数类型的擦除——换句话说，假设它是Object对象的实例——因此只有装箱类型才能作为泛型参数。
装箱类型是对象，在内存中存在额外开销
将基本类型转换为装箱类型，成为装箱，反之称为拆箱，两者需要额外的计算开销。
为了减少性能开销，Stream类的某些方法对基本类型和装箱类型做了区分
- 如果方法返回类型为基本类型，则在基本类型前加To，ToLongFunction
- 如果参数是基本类型，则不加前缀，LongFunction
- 如果高阶函数使用基本类型，则在操作后加后缀To再加基本类型，如MapToLong
基本类型都有与之对应的Stream，以基本类型名为前缀，如LongStream。通过一些高阶函数装箱方法如mapToObj，可以从一个基本类型的Stream得到一个装箱后的Stream，如Stream<Long>
尽量使用对基本类型做过特殊处理的方法从而改善性能。这些特殊的Stream还提供额外的方法。
使用summaryStatistics方法统计曲目长度
	public static void printTrackLengthStatistics(Album album) { 

		IntSummaryStatistics trackLengthStats                 = album.getTracks()                        .mapToInt(track -> track.getLength())                        .summaryStatistics();        System.out.printf("Max: %d, Min: %d, Ave: %f, Sum: %d",                           trackLengthStats.getMax(),                           trackLengthStats.getMin(),                           trackLengthStats.getAverage(),                           trackLengthStats.getSum());	}
对基本类型进行特殊处理方法mapToInt，将每首曲目映射为曲目长度，返回一个IntStream对象，包含summaryStatistic方法，返回统计值。
### 4.3 重载解析
Java中可以重载方法，造成多个方法由相同的方法名，但签名却不一样。这在退单参数类型时可能会出现问题。
两个重载方法
	private void overloadedMethod(Object o)	{ 
		System.out.print("Object");	}	private void overloadedMethod(String s) 	{ 		System.out.print("String");	}
方法调用
	overloadedMethod("abc");
输出String而不是Object
BinaryOperator是一种特殊的BIFunction类型，参数的类型和返回值的类型相同
Lambda表达式的类型就是对应的函数接口
两个可供选择的重载方法
	private interface IntegerBiFunction extends BinaryOperator<Integer> {		}	private void overloadedMethod(BinaryOperator<Integer> Lambda){ 		System.out.print("BinaryOperator");	}	private void overloadedMethod(IntegerBiFunction Lambda) {         System.out.print("IntegerBinaryOperator");    }

另一个重载方法调用

	overloadedMethod((x, y) -> x + y);
	
输出IntegerBinaryOperator机最具体的类型

同时存在多个重载方法时，哪个是最具体的方法可能并不明确

Lambda表达式作为参数时，其类型由其目标类型推到得出

- 如果只有一个可能的目标类型，由相应函数接口里的参数类型推到得出

- 如果有多个可能的目标类型，由最具体的类型推导得出

- 如果有多个可能的目标类型且最具体的类型不明确，则需人为指定类型

### 4.4 @FunctionalInterface

每个用作函数接口的接口都应该添加该注释。会强制javac检查一个接口是否符合函数接口的标准，如果该注释添加一个枚举类型、类或另一个注释，或者接口包含不止一个抽象方法，javac就会报错，重构代码时使用它很容易发现问题。

和Closeable和Comparable接口不通，提高Stream对象可操作性而引入的各种新接口都添加了该注释

### 4.5 二进制接口的兼容性

向后兼容；Java 8为Collection接口增加了stream方法。为实现向后兼容，Java 8添加了新的语言特性：默认方法

### 4.6 默认方法

接口定义了"如果你没有实现A方法，那么就用我的吧"，接口中这样的方法叫做默认方法。

默认方法示例：forEach实现方式:

	default void forEach(Consumer<? super T> action) { 
		  for (T t : this) {              action.accept(t);          }	}
default关键字，默认方法只能通过调用子类的方法来修改子类本身
#### 默认方法和子类
Parent接口，其中welcome是一个默认方法
	public interface Parent {		public void message(String body);		public default void welcome() {			message("Parent: Hi!");		}		public String getLastMessage();
	 }

在客户代码中使用默认方法

	@Test	public void parentDefaultUsed() {         Parent parent = new ParentImpl();         parent.welcome();         assertEquals("Parent: Hi!", parent.getLastMessage());	}
新建一个接口Child，继承自Parent接口，Child接口实现了自己默认的welcome方法，同样的Child实现类不会实现welcome方法，因此也继承了接口的默认方法
继承了Parent接口的Child接口

	public interface Child extends Parent {
			@Override		public default void welcome() {             message("Child: Hi!");        }	}调用该接口，最后输出的字符串时"Child:Hi!"
调用Child接口的客户代码
	@Test
	public void childOverrideDefault(){
		Child child = new ChildImpl();		child.welcome();		assertEquals("Child: Hi!", child.getLastMessage());	}
任何时候，一旦与类(如实现类)中定义的昂发产生冲突，都要优先选择类中定义的方法
重写welcome默认实现的父类
	public class OverridingParent extends ParentImpl {		@Override		public void welcome() {             message("Class Parent: Hi!");         }	}
调用的是类中的具体方法，而不是默认方法
	@Test	public void concreteBeatsDefault() {		Parent parent = new OverridingParent();		parent.welcome();		assertEquals("Class Parent: Hi!", parent.getLastMessage());	}
子接口重写了父接口中的默认方法
	public class OverridingChild extends OverridingParent implements Child {	}
类中重写的方法优先级高于接口中定义的默认方法
	@Test	public void concreteBeatsCloserDefault() {		Child child = new OverridingChild();		child.welcome();		assertEquals("Class Parent: Hi!", child.getLastMessage());	}	简而言之，类中重写的方法胜出。为了实现向后兼容，不破坏已有的实现。
#### 4.7 多重继承
接口允许多重继承，因此有可能碰到两个接口包含签名相同的默认方法的情况。
此时javac并不明确应该继承哪个接口中的方法，因此编译器会报错，在类中实现具体方法就能解决这个问题。
##### 三定律
- 1.类胜于接口。如果在继承链中有方法体或抽象的方法声明，那么就可以忽略接口中定义的方法。
- 2.子类胜于父类。如果一个接口继承了另一个接口，且两个接口都定义了一个默认方法，那么子类中定义的方法生出。
- 3.没有规则三。如果前两条规则不适用，子类要么实现该方法，要么将该方法声明为抽象方法。
#### 4.8 权衡
接口和抽象类之间还是存在明显的区别。接口允许多重继承，却没有成员变量;抽象类可以继承成员变量，却不能多重继承。
#### 4.9 接口的静态方法
放置工具方法
#### 4.10 Optional
Optional是为核心类库新设计的一个数据类型，用了替换null值。1. Optional对象鼓励程序员适时检查变量是否为空 2.将一个类的API中可能为空的值文档化。
创建某个值的Optional对象
	Optional<String> a = Optional.of("a");    assertEquals("a", a.get());
Optional对象也可能为空，因此有对应的工厂方法empty，另外一个工厂方法ofNullable则可将一个控制转换为Optional对象
创建一个空的Optional对象，并检查其是否有值
	Optional emptyOptional = Optional.empty();    Optional alsoEmpty = Optional.ofNullable(null);    assertFalse(emptyOptional.isPresent());	// 例 4-22 中定义了变量 a 	assertTrue(a.isPresent());
	
使用orElse和orElseGet方法

	assertEquals("b", emptyOptional.orElse("b"));    assertEquals("c", emptyOptional.orElseGet(() -> "c"));
当试图避免空值相关的缺陷时，可以考虑使用Optional对象
#### 4.11 要点回顾
- 使用为基本类型定制的Lambda表达式和Stream，如IntStream可以显著提升系统性能
- 默认方法是指接口中定义的包含方法体的方法，方法名有default关键字做前缀
- 在一个值可能为空的建模情况下，使用Optional对象能替代使用null值
### 5 高级集合类和收集器
#### 5.1 方法引用
Lambda表达式如下
	artist -> artist.getName()
方法引用重写后的Lambda表达式
	Artist::getName标准语法为Classname:methodName，凡是使用Lambda表达式的地方，就可以使用方法引用
Lambda表达式
	(name, nationality) -> new Artist(name, nationality)
方法引用
	Artist::new
方法引用自动支持多个参数
			

	
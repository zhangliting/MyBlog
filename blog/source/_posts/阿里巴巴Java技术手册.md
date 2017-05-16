---
title: 阿里巴巴技术手册
date: 2017-03-21 23:26:23
tags: [阿里巴巴]
categories: [Java]
---


# Java开发手册(
---
版本号 | 制作团队 | 更新日期 | 备注 
------ | ---------|----------|-----
1.0.0|阿里巴巴集团技术部 | 2016.12.7| 首次公开
---
## 编程规约

### (一) 命名规约

1. 【强制】类名使用UpperCamelCase风格，必须遵从驼峰形式，但以下情形例外：（领域模型的相关命名）DO / DTO / VO / DAO等。

    正例：MarcoPolo / UserDO / XmlService / TcpUdpDeal / TaPromotion    
    反例：macroPolo / UserDo / XMLService / TCPUDPDeal / TAPromotion
   
2. 【强制】常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。 

    正例： MAX_STOCK_COUNT 
    反例： MAX_COUNT
    
3. 【强制】抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类命名以它要测试的类的名称开始，以Test结尾。

4. 【强制】POJO类中的任何布尔类型的变量，都不要加is，否则部分框架解析会引起序列化错误。 

    反例：定义为基本数据类型boolean isSuccess；的属性，它的方法也是isSuccess()，RPC框架在反向解析的时候，“以为”对应的属性名称是      success，导致属性获取不到，进而抛出异常。
 
5. 【强制】杜绝完全不规范的缩写，避免望文不知义。 

    反例：<某业务代码>AbstractClass“缩写”命名成AbsClass；condition“缩写”命名成 condi，此类随意缩写严重降低了代码的可阅读性。
    
6. 【推荐】接口类中的方法和属性不要加任何修饰符号（public 也不要加），保持代码的简洁性，并加上有效的javadoc注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定是与接口方法相关，并且是整个应用的基础常量。 正例：接口方法签名：void f(); 接口基础常量表示：String COMPANY = "alibaba";

    反例：接口方法定义：public abstract void f(); 说明：JDK8中接口允许有默认实现，那么这个default方法，是对所有实现类都有价值的默认实现。
    
7. 【参考】各层命名规约： A) Service/DAO层方法命名规约 1） 获取单个对象的方法用get做前缀。 2） 获取多个对象的方法用list做前缀。 3） 获取统计值的方法用count做前缀。 4） 插入的方法用save（推荐）或insert做前缀。 5） 删除的方法用remove（推荐）或delete做前缀。 6） 修改的方法用update做前缀。 B) 领域模型命名规约 1） 数据对象：xxxDO，xxx即为数据表名。 2） 数据传输对象：xxxDTO，xxx为业务领域相关的名称。 3） 展示对象：xxxVO，xxx一般为网页名称。 4） POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。
<!--more-->

### (二) 常量定义
---
1. 【强制】不允许出现任何魔法值（即未经定义的常量）直接出现在代码中。 反例： String key="Id#taobao_"+tradeId； cache.put(key, value);
2. 【强制】long或者Long初始赋值时，必须使用大写的L，不能是小写的l，小写容易跟数字1混淆，造成误解。 说明：Long a = 2l; 写的是数字的21，还是Long型的2?
3. 【推荐】不要使用一个常量类维护所有常量，应该按常量功能进行归类，分开维护。如：缓存相关的常量放在类：CacheConsts下；系统配置相关的常量放在类：ConfigConsts下。 说明：大而全的常量类，非得ctrl+f才定位到修改的常量，不利于理解，也不利于维护。

### (三) 格式规约
---
1. 【强制】大括号的使用约定。如果是大括号内为空，则简洁地写成{}即可，不需要换行；如果是非空代码块则： 1） 左大括号前不换行。 2） 左大括号后换行。 3） 右大括号前换行。 4） 右大括号后还有else等代码则不换行；表示终止右大括号后必须换行。
2. 【强制】单行字符数限制不超过120个，超出需要换行，换行时，遵循如下原则： 1） 换行时相对上一行缩进4个空格。 2） 运算符与下文一起换行。 3） 方法调用的点符号与下文一起换行。 4） 在多个参数超长，逗号后进行换行。 5） 在括号前不要换行，见反例。 正例： StringBuffer sb = new StringBuffer(); //超过120个字符的情况下，换行缩进4个空格，并且方法前的点符号一起换行 sb.append("zi").append("xin")… .append("huang"); 反例： StringBuffer sb = new StringBuffer(); //超过120个字符的情况下，不要在括号前换行 sb.append("zi").append("xin")…append ("huang"); //参数很多的方法调用也超过120个字符，逗号后才是换行处 method(args1, args2, args3, ... , argsX);
3. 【推荐】方法体内的执行语句组、变量的定义语句组、不同的业务逻辑之间或者不同的语义之间插入一个空行。相同业务逻辑和语义之间不需要插入空行。 说明：没有必要插入多行空格进行隔开。
### (四) OOP规约
---
1. 【强制】所有的覆写方法，必须加@Override注解。 反例：getObject()与get0bject()的问题。一个是字母的O，一个是数字的0，加@Override可以准确判断是否覆盖成功。另外，如果在抽象类中对方法签名进行修改，其实现类会马上编译报错。
2. 【强制】对外暴露的接口签名，原则上不允许修改方法签名，避免对接口调用方产生影响。接口过时必须加@Deprecated注解，并清晰地说明采用的新接口或者新服务是什么。
3. 【强制】不能使用过时的类或方法。 说明：java.net.URLDecoder 中的方法decode(String encodeStr) 这个方法已经过时，应该使用双参数decode(String source, String encode)。接口提供方既然明确是过时接口，那么有义务同时提供新的接口；作为调用方来说，有义务去考证过时方法的新实现是什么。
4. 【强制】Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用equals。 正例： "test".equals(object);
    反例： object.equals("test"); 说明：推荐使用java.util.Objects#equals （JDK7引入的工具类）
5. 【强制】所有的相同类型的包装类对象之间值的比较，全部使用equals方法比较。 说明：对于Integer var=?在-128至127之间的赋值，Integer对象是在IntegerCache.cache产生，会复用已有对象，这个区间内的Integer值可以直接使用==进行判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑，推荐使用equals方法进行判断。
6. 【强制】关于基本数据类型与包装数据类型的使用标准如下： 1） 所有的POJO类属性必须使用包装数据类型。 2） RPC方法的返回值和参数必须使用包装数据类型。 3） 所有的局部变量推荐使用基本数据类型。 说明：POJO类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何NPE问题，或者入库检查，都由使用者来保证。 正例：数据库的查询结果可能是null，因为自动拆箱，用基本数据类型接收有NPE风险。 反例：某业务的交易报表上显示成交总额涨跌情况，即正负x%，x为基本数据类型，调用的RPC服务，调用不成功时，返回的是默认值，页面显示：0%，这是不合理的，应该显示成中划线-。所以包装数据类型的null值，能够表示额外的信息，如：远程调用失败，异常退出。
7. 【强制】构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在init方法中。
8. 【推荐】 类内方法定义顺序依次是：公有方法或保护方法 > 私有方法 > getter/setter方法。 说明：公有方法是类的调用者和维护者最关心的方法，首屏展示最好；保护方法虽然只是子类关心，也可能是“模板设计模式”下的核心方法；而私有方法外部一般不需要特别关心，是一个黑盒实现；因为方法信息价值较低，所有Service和DAO的getter/setter方法放在类体最后。
9. 【推荐】慎用Object的clone方法来拷贝对象。 说明：对象的clone方法默认是浅拷贝，若想实现深拷贝需要重写clone方法实现属性对象的拷贝。

### (五) 集合处理
---
1. 【强制】Map/Set的key为自定义对象时，必须重写hashCode和equals。 正例：String重写了hashCode和equals方法，所以我们可以非常愉快地使用String对象作为key来使用。
2. 【强制】ArrayList的subList结果不可强转成ArrayList，否则会抛出ClassCastException异常：java.util.RandomAccessSubList cannot be cast to java.util.ArrayList ; 说明：subList 返回的是 ArrayList 的内部类 SubList，并不是 ArrayList ，而是 ArrayList 的一个视图，对于SubList子列表的所有操作最终会反映到原列表上。
3. 【强制】在subList场景中，高度注意对原集合元素个数的修改，会导致子列表的遍历、增加、删除均产生ConcurrentModificationException 异常。
4. 【强制】使用工具类Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的add/remove/clear方法会抛出UnsupportedOperationException异常。 说明：asList的返回对象是一个Arrays内部类，并没有实现集合的修改方法。Arrays.asList体现的是适配器模式，只是转换接口，后台的数据仍是数组。 String[] str = new String[] { "a", "b" }; List list = Arrays.asList(str); 第一种情况：list.add("c"); 运行时异常。 第二种情况：str[0]= "gujin"; 那么list.get(0)也会随之修改。
5. 【强制】不要在foreach循环里进行元素的remove/add操作。remove元素请使用Iterator方式，如果并发操作，需要对Iterator对象加锁。
反例： 
```java
List<String> a = new ArrayList<String>();
a.add("1"); a.add("2"); for (String temp : a) { if("1".equals(temp)){ a.remove(temp); } }
```
说明：这个例子的执行结果会出乎大家的意料，那么试一下把“1”换成“2”，会是同样的结果吗？ 
正例： 
```java
Iterator<String> it = a.iterator(); while(it.hasNext()){
String temp = it.next(); if(删除元素的条件){ it.remove(); } }
```
6. 【强制】在JDK7版本以上，Comparator要满足自反性，传递性，对称性，不然Arrays.sort，Collections.sort会报IllegalArgumentException异常。 说明： 1） 自反性：x，y的比较结果和y，x的比较结果相反。 2） 传递性：x>y,y>z,则x>z。 3） 对称性：x=y,则x,z比较结果和y，z比较结果相同。 反例：下例中没有处理相等的情况，实际使用中可能会出现异常： 
```jvava
new Comparator<Student>() { @Override public int compare(Student o1, Student o2) { return o1.getId() > o2.getId() ? 1 : -1; } }
```
7. 【推荐】集合初始化时，尽量指定集合初始值大小。 说明：ArrayList尽量使用ArrayList(int initialCapacity) 初始化。
8. 【推荐】使用entrySet遍历Map类集合KV，而不是keySet方式进行遍历。 说明：keySet其实是遍历了2次，一次是转为Iterator对象，另一次是从hashMap中取出key所对应的value。而entrySet只是遍历了一次就把key和value都放到了entry中，效率更高。如果是JDK8，使用Map.foreach方法。 正例：values()返回的是V值集合，是一个list集合对象；keySet()返回的是K值集合，是一个Set集合对象；entrySet()返回的是K-V值组合集合。
11. 【推荐】高度注意Map类集合K/V能不能存储null值的情况

集合类|Key|Value|Super|说明
-----|---|-----|------|----
Hashtable|不允许为null|不允许为null|Dictionary|线程安全
ConcurrentHashMap|不允许为null|不允许为null|AbstractMap|线程局部安全
TreeMap|不允许为null|允许为null|AbstractMap|线程不安全
HashMap|允许为null|允许为null|AbstractMap|线程不安全

### (六) 并发处理
---
1. 【强制】获取单例对象要线程安全。在单例对象里面做操作也要保证线程安全。 说明：资源驱动类、工具类、单例工厂类都需要注意。
2. 【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。 说明：使用线程池的好处是减少在创建和销毁线程上所花的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。
3. 【强制】SimpleDateFormat 是线程不安全的类，一般不要定义为static变量，如果定义为static，必须加锁，或者使用DateUtils工具类。 正例：注意线程安全，使用DateUtils。亦推荐如下处理：
```java
private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>() { @Override protected DateFormat initialValue() { return new SimpleDateFormat("yyyy-MM-dd"); } };
```
说明：如果是JDK8的应用，可以使用instant代替Date，Localdatetime代替Calendar，Datetimeformatter代替Simpledateformatter，官方   给出的解释：simple beautiful strong immutable thread-safe。

4. 【强制】高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能锁区块，就不要锁整个方法体；能用对象锁，就不要用类锁。
5. 【强制】对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造成死锁。 说明：线程一需要对表A、B、C依次全部加锁后才可以进行更新操作，那么线程二的加锁顺序也必须是A、B、C，否则可能出现死锁。
6. 【强制】并发修改同一记录时，避免更新丢失，要么在应用层加锁，要么在缓存加锁，要么在数据库层使用乐观锁，使用version作为更新依据。
说明：如果每次访问冲突概率小于20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于3次。
7. 【强制】多线程并行处理定时任务时，Timer运行多个TimeTask时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，使用ScheduledExecutorService则没有这个问题。
8. 【强制】线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
9. 说明：Executors各个方法的弊端： 1）newFixedThreadPool和newSingleThreadExecutor: 主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。 2）newCachedThreadPool和newScheduledThreadPool: 主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。
10. 【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。 
正例： 
```java
public class TimerTaskThread extends Thread { public TimerTaskThread(){ super.setName("TimerTaskThread"); … }
```
11. 【推荐】使用CountDownLatch进行异步转同步操作，每个线程退出前必须调用countDown方法，线程执行代码注意catch异常，确保countDown方法可以执行，避免主线程无法执行至countDown方法，直到超时才返回结果。 说明：注意，子线程抛出异常堆栈，不能在主线程try-catch到。
12. 【推荐】避免Random实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一seed 导致的性能下降。 说明：Random实例包括java.util.Random 的实例或者 Math.random()实例。 正例：在JDK7之后，可以直接使用API ThreadLocalRandom，在 JDK7之前，可以做到每个线程一个实例。
13. 【参考】volatile解决多线程内存不可见问题。对于一写多读，是可以解决变量同步问题，但是如果多写，同样无法解决线程安全问题。如果想取回count++数据，使用如下类实现：AtomicInteger count = new AtomicInteger(); count.addAndGet(1); count++操作如果是JDK8，推荐使用LongAdder对象，比AtomicLong性能更好（减少乐观锁的重试次数）。
14. 【参考】ThreadLocal无法解决共享对象的更新问题，ThreadLocal对象建议使用static修饰。这个变量是针对一个线程内所有操作共有的，所以设置为静态变量，所有此类实例共享此静态变量 ，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。

### (七) 控制语句
---
1. 【强制】在if/else/for/while/do语句中必须使用大括号，即使只有一行代码，避免使用下面的形式：if (condition) statements;
2. 【推荐】推荐尽量少用else， if-else的方式可以改写成： if(condition){ … return obj; } // 接着写else的业务逻辑代码; 说明：如果使用要if-else if-else方式表达逻辑，【强制】请勿超过3层，超过请使用状态设计模式。

### (八) 注释规约
---
1. 【强制】类、类属性、类方法的注释必须使用javadoc规范，使用/**内容*/格式，不得使用//xxx方式。 说明：在IDE编辑窗口中，javadoc方式会提示相关注释，生成javadoc可以正确输出相应注释；在IDE中，工程调用方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高阅读效率。
2. 【强制】所有的抽象方法（包括接口中的方法）必须要用javadoc注释、除了返回值、参数、异常说明外，还必须指出该方法做什么事情，实现什么功能。 说明：如有实现和调用注意事项，请一并说明。
3. 【参考】对于注释的要求：第一、能够准确反应设计思想和代码逻辑；第二、能够描述业务含义，使别的程序员能够迅速了解到代码背后的信息。完全没有注释的大段代码对于阅读者形同天书，注释是给自己看的，即使隔很长时间，也能清晰理解当时的思路；注释也是给继任者看的，使其能够快速接替自己的工作。

### (九) 其它
---
1. 【强制】在使用正则表达式时，利用好其预编译功能，可以有效加快正则匹配速度。 说明：不要在方法体内定义：Pattern pattern = Pattern.compile(规则);
2. 【强制】避免用Apache Beanutils进行属性的copy。 说明：Apache BeanUtils性能较差，可以使用其他方案比如Spring BeanUtils, Cglib BeanCopier。
3. 【强制】注意 Math.random() 这个方法返回是double类型，注意取值范围 0≤x<1（能够取到零值，注意除零异常），如果想获取整数类型的随机数，不要将x放大10的若干倍然后取整，直接使用Random对象的nextInt或者nextLong方法。
4. 【强制】获取当前毫秒数：System.currentTimeMillis(); 而不是new Date().getTime(); 说明：如果想获取更加精确的纳秒级时间值，用System.nanoTime。在JDK8中，针对统计时间等场景，推荐使用Instant类。
5. 【推荐】任何数据结构的使用都应限制大小。 说明：这点很难完全做到，但很多次的故障都是因为数据结构自增长，结果造成内存被吃光。
6. 【推荐】对于“明确停止使用的代码和配置”，如方法、变量、类、配置文件、动态配置属性等要坚决从程序中清理出去，避免造成过多垃圾。清理这类垃圾代码是技术气场，不要有这样的观念：“不做不错，多做多错”。

## 二、异常日志
### (一) 异常处理
---
1. 【强制】不要捕获Java类库中定义的继承自RuntimeException的运行时异常类，如：IndexOutOfBoundsException / NullPointerException，这类异常由程序员预检查来规避，保证程序健壮性。
2. 【强制】异常不要用来做流程控制，条件控制，因为异常的处理效率比条件分支低。
3. 【强制】对大段代码进行try-catch，这是不负责任的表现。catch时请分清稳定代码和非稳定代码，稳定代码指的是无论如何不会出错的代码。对于非稳定代码的catch尽可能进行区分异常类型，再做对应的异常处理。
4. 【强制】捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。
5. 【强制】finally块必须对资源对象、流对象进行关闭，有异常也要做try-catch。 说明：如果JDK7，可以使用try-with-resources方法。
6. 【强制】不能在finally块中使用return，finally块中的return返回后方法结束执行，不会再执行try块中的return语句。
7. 【推荐】方法的返回值可以为null，不强制返回空集合，或者空对象等，必须添加注释充分说明什么情况下会返回null值。调用方需要进行null判断防止NPE问题。 说明：本规约明确防止NPE是调用者的责任。即使被调用方法返回空集合或者空对象，对调用者来说，也并非高枕无忧，必须考虑到远程调用失败，运行时异常等场景返回null的情况。
8. 【推荐】防止NPE，是程序员的基本修养，注意NPE产生的场景： 
1） 返回类型为包装数据类型，有可能是null，返回int值时注意判空。2） 数据库的查询结果可能为null。 3） 集合里的元素即使isNotEmpty，取出的数据元素也可能为null。 4） 远程调用返回对象，一律要求进行NPE判断。 5） 对于Session中获取的数据，建议NPE检查，避免空指针。 6） 级联调用obj.getA().getB().getC()；一连串调用，易产生NPE。
9. 【推荐】在代码中使用“抛异常”还是“返回错误码”，对于公司外的http/api开放接口必须使用“错误码”；而应用内部推荐异常抛出；跨应用间RPC调用优先考虑使用Result方式，封装isSuccess、“错误码”、“错误简短信息”。 说明：关于RPC方法返回方式使用Result方式的理由： 1）使用抛异常返回方式，调用方如果没有捕获到就会产生运行时错误。 2）如果不加栈信息，只是new自定义异常，加入自己的理解的error message，对于调用端解决问题的帮助不会太多。如果加了栈信息，在频繁调用出错的情况下，数据序列化和传输的性能损耗也是问题。
10. 【推荐】定义时区分unchecked / checked 异常，避免直接使用RuntimeException抛出，更不允许抛出Exception或者Throwable，应使用有业务含义的自定义异常。推荐业界已定义过的自定义异常，如：DaoException / ServiceException等。

### (二) 日志规约
---
1. 【强制】应用中不可直接使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架SLF4J中的API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。 import org.slf4j.Logger; import org.slf4j.LoggerFactory; private static final Logger logger = LoggerFactory.getLogger(Abc.class);
2. 【强制】对trace/debug/info级别的日志输出，必须使用条件输出形式或者使用占位符的方式。 说明：logger.debug("Processing trade with id: " + id + " symbol: " + symbol); 如果日志级别是warn，上述日志不会打印，但是会执行字符串拼接操作，如果symbol是对象，会执行toString()方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。 
正例：（条件）
```java
if (logger.isDebugEnabled()) { logger.debug("Processing trade with id: " + id + " symbol: " + symbol); } 
```
正例：（占位符） 
```java
logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol);
```

3. 【强制】避免重复打印日志，浪费磁盘空间，务必在log4j.xml中设置additivity=false。 正例：<logger name="com.taobao.ecrm.member.config" additivity="false">

4. 【强制】异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么往上抛。 正例：logger.error(各类参数或者对象toString + "_" + e.getMessage(), e);

## 三、MYSQL规约
### (一) 建表规约
---
1. 【强制】表达是与否概念的字段，必须使用is_xxx的方式命名，数据类型是unsigned tinyint（ 1表示是，0表示否），此规则同样适用于odps建表。 说明：任何字段如果为非负数，必须是unsigned。
2. 【强制】唯一索引名为uk_字段名；普通索引名则为idx_字段名。 说明：uk_ 即 unique key；idx_ 即index的简称。
3. 【强制】小数类型为decimal，禁止使用float和double。 说明：float和double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储。
4. 【强制】如果存储的字符串长度几乎相等，使用CHAR定长字符串类型。
5. 【强制】varchar是可变长字符串，不预先分配存储空间，长度不要超过5000，如果存储长度大于此值，定义字段类型为TEXT，独立出来一张表，用主键来对应，避免影响其它字段索引效率。
6. 【强制】表必备三字段：id, gmt_create, gmt_modified。 说明：其中id必为主键，类型为unsigned bigint、单表时自增、步长为1；分表时改为从TDDL Sequence取值，确保分表之间的全局唯一。gmt_create, gmt_modified的类型均为date_time类型。
7. 【推荐】字段允许适当冗余，以提高性能，但是必须考虑数据同步的情况。冗余字段应遵循： 1）不是频繁修改的字段。 2）不是varchar超长字段，更不能是text字段。 正例：各业务线经常冗余存储商品名称，避免查询时需要调用IC服务获取。
8. 【推荐】单表行数超过500万行或者单表容量超过2GB，才推荐进行分库分表。 说明：如果预计三年后的数据量根本达不到这个级别，请不要在创建表时就分库分表。 反例：某业务三年总数据量才2万行，却分成1024张表，问：你为什么这么设计？答：分1024张表，不是标配吗？

### (二) 索引规约
---
1. 【强制】业务上具有唯一特性的字段，即使是组合字段，也必须建成唯一索引。 说明：不要以为唯一索引影响了insert速度，这个速度损耗可以忽略，但提高查找速度是明显的；另外，即使在应用层做了非常完善的校验和控制，只要没有唯一索引，根据墨菲定律，必然有脏数据产生。
2. 【强制】超过三个表禁止join。需要join的字段，数据类型保持绝对一致；多表关联查询时，保证被关联的字段需要有索引。 说明：即使双表join也要注意表索引、SQL性能。
3. 【强制】在varchar字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度。
    说明：索引的长度与区分度是一对矛盾体，一般对字符串类型数据，长度为20的索引，区分度会高达90%以上，可以使用count(distinct left(列名, 索引长度))/count(*)的区分度来确定。
4. 【强制】页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。 说明：索引文件具有B-Tree的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索引。
5. 【推荐】如果有order by的场景，请注意利用索引的有序性。order by 最后的字段是组合索引的一部分，并且放在索引组合顺序的最后，避免出现file_sort的情况，影响查询性能。 正例：where a=? and b=? order by c; 索引：a_b_c 反例：索引中有范围查找，那么索引有序性无法利用，如：WHERE a>10 ORDER BY b; 索引a_b无法排序。
6. 【推荐】利用覆盖索引来进行查询操作，来避免回表操作。 说明：如果一本书需要知道第11章是什么标题，会翻开第11章对应的那一页吗？目录浏览一下就好，这个目录就是起到覆盖索引的作用。 正例：IDB能够建立索引的种类：主键索引、唯一索引、普通索引，而覆盖索引是一种查询的一种效果，用explain的结果，extra列会出现：using index.
7. 【推荐】利用延迟关联或者子查询优化超多分页场景。 说明：MySQL并不是跳过offset行，而是取offset+N行，然后返回放弃前offset行，返回N行，那当offset特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行SQL改写。 正例：先快速定位需要获取的id段，然后再关联： SELECT a.* FROM 表1 a, (select id from 表1 where 条件 LIMIT 100000,20 ) b where a.id=b.id
8. 【推荐】 SQL性能优化的目标：至少要达到 range 级别，要求是ref级别，如果可以是consts最好。 说明： 1）consts 单表中最多只有一个匹配行（主键或者唯一索引），在优化阶段即可读取到数据。 2）ref 指的是使用普通的索引。（normal index） 3）range 对索引进范围检索。
9. 【推荐】建组合索引的时候，区分度最高的在最左边。 正例：如果where a=? and b=? ，a列的几乎接近于唯一值，那么只需要单建idx_a索引即可。说明：存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如：where a>? and b=? 那么即使a的区分度更高，也必须把b放在索引的最前列。

### (三) SQL规约
---
1. 【强制】不要使用count(列名)或count(常量)来替代count(*)，count(*)就是SQL92定义的标准统计行数的语法，跟数据库无关，跟NULL和非NULL无关。说明：count(*)会统计值为NULL的行，而count(列名)不会统计此列为NULL值的行。
2. 【强制】count(distinct col) 计算该列除NULL之外的不重复数量。注意 count(distinct col1, col2) 如果其中一列全为NULL，那么即使另一列有不同的值，也返回为0。
3. 【强制】当某一列的值全是NULL时，count(col)的返回结果为0，但sum(col)的返回结果为NULL，因此使用sum()时需注意NPE问题。 正例：可以使用如下方式来避免sum的NPE问题：SELECT IF(ISNULL(SUM(g)),0,SUM(g)) FROM table;
4. 【强制】使用ISNULL()来判断是否为NULL值。注意：NULL与任何值的直接比较都为NULL。 说明： 1） NULL<>NULL的返回结果是NULL，不是false。 2） NULL=NULL的返回结果是NULL，不是true。 3） NULL<>1的返回结果是NULL，而不是true。
5. 【强制】在代码中写分页查询逻辑时，若count为0应直接返回，避免执行后面的分页语句。
6. 【强制】不得使用外键与级联，一切外键概念必须在应用层解决。 说明：（概念解释）学生表中的student_id是主键，那么成绩表中的student_id则为外键。如果更新学生表中的student_id，同时触发成绩表中的student_id更新，则为级联更新。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。
7. 【强制】禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。
8. 【强制】IDB数据订正时，删除和修改记录时，要先select，避免出现误删除，确认无误才能提交执行。
9. 【推荐】in操作能避免则避免，若实在避免不了，需要仔细评估in后边的集合元素数量，控制在1000个之内。
10. 【参考】TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少，但TRUNCATE无事务且不触发trigger，有可能造成事故，故不建议在开发代码中使用此语句。 说明：TRUNCATE TABLE 在功能上与不带 WHERE 子句的 DELETE 语句相同。

### (四) ORM规约
---
1. 【强制】在表查询中，一律不要使用 * 作为查询的字段列表，需要哪些字段必须明确写明。 说明：1）增加查询分析器解析成本。2）增减字段容易与resultMap配置不一致。
2. 【强制】POJO类的boolean属性不能加is，而数据库字段必须加is_，要求在resultMap中进行字段与属性之间的映射
3. 【强制】xml配置中参数注意使用：#{}，#param# 不要使用${} 此种方式容易出现SQL注入。
4. 【强制】iBATIS自带的queryForList(String statementName,int start,int size)不推荐使用。 说明：其实现方式是在数据库取到statementName对应的SQL语句的所有记录，再通过subList取start,size的子集合，线上因为这个原因曾经出现过OOM。
5. 正例：在sqlmap.xml中引入 #start#, #size# Map<String, Object> map = new HashMap<String, Object>(); map.put("start", start); map.put("size", size);
6. 【强制】不允许直接拿HashMap与HashTable作为查询结果集的输出。 反例：某同学为避免写一个<resultMap>，直接使用HashTable来接收数据库返回结果，结果出现日常是把bigint转成Long值，而线上由于数据库版本不一样，解析成BigInteger，导致线上问题。
7. 【强制】更新数据表记录时，必须同时更新记录对应的gmt_modified字段值为当前时间。
8. 【参考】@Transactional事务不要滥用。事务会影响数据库的QPS，另外使用事务的地方需要考虑各方面的回滚方案，包括缓存回滚、搜索引擎回滚、消息补偿、统计修正等。

##四、工程规约
### (一) 应用分层
---
1. 【参考】（分层异常处理规约）在DAO层，产生的异常类型有很多，无法用细粒度异常进行catch，使用catch(Exception e)方式，并throw new DaoException(e)，不需要打印日志，因为日志在Manager/Service层一定需要捕获并打到日志文件中去，如果同台服务器再打日志，浪费性能和存储。在Service层出现异常时，必须记录日志信息到磁盘，尽可能带上参数信息，相当于保护案发现场。如果Manager层与Service同机部署，日志方式与DAO层处理一致，如果是单独部署，则采用与Service一致的处理方式。Web层绝不应该继续往上抛异常，因为已经处于顶层，无继续处理异常的方式，如果意识到这个异常将导致页面无法正常渲染，那么就应该直接跳转到友好错误页面，尽量加上友好的错误提示信息。开放接口层要将异常处理成错误码和错误信息方式返回。
2. 【参考】分层领域模型规约： 
    - DO（Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。 
    - DTO（Data Transfer Object）：数据传输对象，Service和Manager向外传输的对象。  
    - BO（Business Object）：业务对象。可以由Service层输出的封装业务逻辑的对象。 
    - QUERY：数据查询对象，各层接收上层的查询请求。注：超过2个参数的查询封装，禁止使用Map类来传输。 
    - VO（View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。
### (二) 二方库规约
---
1. 【推荐】工具类二方库已经提供的，尽量不要在本应用中编程实现。  json操作： fastjson
 md5操作：commons-codec  工具集合：Guava包  数组操作：ArrayUtils（org.apache.commons.lang3.ArrayUtils）  集合操作：CollectionUtils(org.apache.commons.collections4.CollectionUtils)  除上面以外还有NumberUtils、DateFormatUtils、DateUtils等优先使用org.apache.commons.lang3这个包下的，不要使用org.apache.commons.lang包下面的。原因是commons.lang这个包是从JDK1.2开始支持的所以很多1.5/1.6的特性是不支持的，例如：泛型。
2. 【推荐】所有pom文件中的依赖声明放在<dependencies>语句块中，所有版本仲裁放在<dependencyManagement>语句块中。 说明：<dependencyManagement>里只是声明版本，并不实现引入，因此子项目需要显式的声明依赖，version和scope都读取自父pom。而<dependencies>所有声明在主pom的<dependencies >里的依赖都会自动引入，并默认被所有的子项目继承

### (三) 服务器规约
---
1. 【推荐】高并发服务器建议调小TCP协议的time_wait超时时间。 说明：操作系统默认240秒后，才会关闭处于time_wait状态的连接，在高并发访问下，服务器端会因为处于time_wait的连接数太多，可能无法建立新的连接，所以需要在服务器上调小此等待值。 正例：在linux服务器上请通过变更/etc/sysctl.conf文件去修改该缺省值（秒）： net.ipv4.tcp_fin_timeout = 30
2. 【推荐】调大服务器所支持的最大文件句柄数（File Descriptor，简写为fd）。 说明：主流操作系统的设计是将TCP/UDP连接采用与文件一样的方式去管理，即一个连接对应于一个fd。主流的linux服务器默认所支持最大fd数量为1024，当并发连接数很大时很容易因为fd不足而出现“open too many files”错误，导致新的连接无法建立。 建议将linux服务器所支持的最大句柄数调高数倍（与服务器的内存数量相关）。
3. 【推荐】给JVM设置-XX:+HeapDumpOnOutOfMemoryError参数，让JVM碰到OOM场景时输出dump信息。
4. 【参考】服务器内部重定向必须使用forward；外部重定向地址必须使用URL Broker生成，否则因线上采用HTTPS协议而导致浏览器提示“不安全”。此外，还会带来URL维护不一致的问题。

## 五、安全规约
---
1. 【强制】可被用户直接访问的功能必须进行权限控制校验。 说明：防止没有做权限控制就可随意访问、操作别人的数据，比如查看、修改别人的订单。
2. 【强制】用户敏感数据禁止直接展示，必须对展示数据脱敏。 说明：支付宝中查看个人手机号码会显示成:158****9119，隐藏中间4位，防止隐私泄露。
3. 【强制】用户输入的SQL参数严格使用参数绑定或者METADATA字段值限定，防止SQL注入，禁止字符串拼接SQL访问数据库。
4. 【强制】用户请求传入的任何参数必须做有效性验证。 说明：忽略参数校验可能导致：  page size过大导致内存溢出  恶意order by导致数据库慢查询  正则输入源串拒绝服务ReDOS  任意重定向  SQL注入  Shell注入  反序列化注入
5. 【强制】禁止向HTML页面输出未经安全过滤或未正确转义的用户数据。
6. 【强制】表单、AJAX提交必须执行CSRF安全过滤。 说明：CSRF(Cross-site request forgery)跨站请求伪造是一类常见编程漏洞。对于存在CSRF漏洞的应用/网站，攻击者可以事先构造好URL，只要受害者用户一访问，后台便在用户不知情情况下对数据库中用户参数进行相应修改。
7. 【强制】URL外部重定向传入的目标地址必须执行白名单过滤。 正例： 
```java
try { if (com.alibaba.fasttext.sec.url.CheckSafeUrl .getDefaultInstance().inWhiteList(targetUrl)){ response.sendRedirect(targetUrl); } } catch (IOException e) { logger.error("Check returnURL error! targetURL=" + targetURL, e); throw e;
```
8. 【强制】Web应用必须正确配置Robots文件，非SEO URL必须配置为禁止爬虫访问。 User-agent: * Disallow: /
9. 【强制】在使用平台资源，譬如短信、邮件、电话、下单、支付，必须实现正确的防重放限制，如数量限制、疲劳度控制、验证码校验，避免被滥刷、资损。 说明：如注册时发送验证码到手机，如果没有限制次数和频率，那么可以利用此功能骚扰到其它用户，并造成短信平台资源浪费。
10. 【推荐】发贴、评论、发送即时消息等用户生成内容的场景必须实现防刷、文本内容违禁词过滤等风控策略。

# 编程命名规范
1. 抽象类命名使用 Abstract 或 Base 开头；异常类命名使用 Exception 结尾；测试类命名以它要测试的类的名称开始，以 Test 结尾。
2.	中括号是数组类型的一部分，数组定义如下：String[] args;
    -  ~~反例~~ ：请勿使用 String args[]的方式来定义.
3.	包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用单数形式，但是类名如果有复数含义，类名可以使用复数形式。
    - 正例： 应用工具类包名为 com.alibaba.mpp.util、 类名为 MessageUtils
4.	杜绝完全不规范的缩写，避免望文不知义。
    - ~~反例~~：<某业务代码>AbstractClass“缩写”命名成 AbsClass；condition“缩写”命名成condi，此类随意缩写严重降低了代码的可阅读性。
5.	如果使用到了设计模式，建议在类名中体现出具体模式。
    - ==说明==：将设计模式体现在名字中，有利于阅读者快速理解架构设计思想。
    - 正例：

        ```
        public class OrderFactory;
        public class LoginProxy;
        public class ResourceObserver;
        ```

1.	接口类中的方法和属性不要加任何修饰符号（public 也不要加），保持代码的简洁性，并加上有效的 javadoc 注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定是与接口方法相关，并且是整个应用的基础常量。
    - 正例：接口方法签名：void f();接口基础常量表示：
    		
    ```
    String COMPANY = "alibaba";
    ```

	- ~~反例~~：接口方法定义：public abstract void f();
    - ==说明==：JDK8 中接口允许有默认实现，那么这个default方法，是对所有实现类都有价值的默认实现。
1.	枚举类名建议带上 Enum 后缀，枚举成员名称需要全大写，单词间用下划线隔开。
    - ==说明==：枚举其实就是特殊的常量类，且构造方法被默认强制是私有。
    - 正例：枚举名字：DealStatusEnum；成员名称：SUCCESS / UNKOWN_REASON。
# Service/DAO 层方法命名规约
- 获取单个对象的方法用 get 做前缀。 
- 获取多个对象的方法用 list 做前缀。 
- 获取统计值的方法用 count 做前缀。 
- 插入的方法用 save（推荐）或 insert 做前缀。 
- 删除的方法用 remove（推荐）或 delete 做前缀。 
# 领域模型命名规约
1. 数据对象：xxxDO，xxx 即为数据表名。 
1. 数据传输对象：xxxDTO，xxx 为业务领域相关的名称。 
    > 表现层与应用层之间是通过数据传输对象（DTO）进行交互的，数据传输对象是没有行为的POJO对象，它 的目的只是为了对领域对象进行数据封装，实现层与层之间的数据传递。为何不能直接将领域对象用于数据传递？因为领域对象更注重领域，而DTO更注重数据。不仅如此，由于“富领域模型”的特点，这样做会直接将领域对象的行为暴露给表现层。需要了解的是，数据传输对象DTO本身并不是业务对象。数据传输对象是根据UI的需求进行设计的，而不 是根据领域对象进行设计的。比如，Customer领域对象可能会包含一些诸如FirstName, LastName,Email,Address等信息。但如果UI上不打算显示Address的信息，那么CustomerDTO中也无需包含这个Address的数据。
1. 展示对象：xxxVO，xxx 一般为网页名称。


# 常量定义 
1. long 或者 Long 初始赋值时，必须使用大写的 L，不能是小写的 l，小写容易跟数字 1 混淆，造成误解。
1. 不允许出现任何魔法值（即未经定义的常量）直接出现在代码中。
    - ~~反例~~： String key="Id#taobao_"+tradeId；cache.put(key, value);
1. 常量的复用层次有五层：跨应用共享常量、应用内共享常量、子工程内共享常量、包内共享常量、类内共享常量。
    1. 跨应用共享常量：放置在二方库中，通常是 client.jar 中的 const 目录下。
    1. 应用内共享常量：放置在一方库的 modules 中的 const 目录下。
        >  ~~反例~~ ：易懂变量也要统一定义成应用内共享常量，两位攻城师在两个类中分别定义了表示“是”的变量：
        类 A 中：public static final String YES = "yes";
        类 B 中：public static final String YES = "y";
        A.YES.equals(B.YES)，预期是 true，但实际返回为 false，导致产生线上问题。

    1.  子工程内部共享常量：即在当前子工程的 const 目录下。
    1.  包内共享常量：即在当前包下单独的 const 目录下。
    1.  类内共享常量：直接在类内部 private static final 定义。
1. 如果变量值仅在一个范围内变化用 Enum 类。如果还带有名称之外的延伸属性，必须使用 Enum 类，下面正例中的数字就是延伸信息，表示星期几。
    - 正例：public Enum{ MONDAY(1), TUESDAY(2), WEDNESDAY(3), THURSDAY(4), FRIDAY(5),SATURDAY(6), SUNDAY(7);}
# OOP规约
1. Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用 equals。 
    - 正例： "test".equals(object); 
    - ~~反例~~：  object.equals("test"); 
    - ==说明==：推荐使用 java.util.Objects#equals （JDK7 引入的工具类）

1. 所有的相同类型的包装类对象之间值的比较，全部使用 equals 方法比较。 
    - ==说明==：对于 Integer var=?在-128 至 127 之间的赋值，Integer 对象是在 IntegerCache.cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用==进行判断，但是这个区间之 外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑，推荐使用 equals 方 法进行判断。

1. POJO 类必须写 toString 方法。使用工具类 source> generate toString 时，如果继 承了另一个 POJO 类，注意在前面加一下 super.toString。 
    - ==说明==：在方法执行抛出异常时，可以直接调用 POJO 的 toString()方法打印其属性值，便于排 查问题。

1. 使用索引访问用 String 的 split方法得到的数组时，需做最后一个分隔符后有无内 容的检查，否则会有抛 IndexOutOfBoundsException 的风险。 
    - ==说明==： 
    ```
    String str = "a,b,c,,";
    String[] array = str.split(",");
    System.out.println(array.length);//预期结果是大于3，实际输出是3
    ```


5. 关于基本数据类型与包装数据类型的使用标准如下： 
    - 所有的 POJO 类属性必须使用包装数据类型。 
    - RPC 方法的返回值和参数必须使用包装数据类型。 
    - 所有的局部变量推荐使用基本数据类型。 
        - ==说明==：POJO 类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何 NPE 问题，或者入库检查，都由使用者来保证。 
        - 正例：数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险。 
        - ~~反例~~：某业务的交易报表上显示成交总额涨跌情况，即正负 x%，x 为基本数据类型，调用的RPC服务，调用不成功时，返回的是默认值，页面显示：0%，这是不合理的，应该显示成中划 线-。所以包装数据类型的 null 值，能够表示额外的信息，如：远程调用失败，异常退出。

# 集合处理
1.  Map/Set 的 key 为自定义对象时，必须重写 hashCode 和 equals。 
    - 正例：String 重写了 hashCode 和 equals 方法，所以我们可以非常愉快地使用 String 对象作 为 key 来使用。
1. ArrayList 的 subList 结果不可强转成 ArrayList，否则会抛出 ClassCastException 异常：java.util.RandomAccessSubList cannot be cast to java.util.ArrayList ; 
    - ==说明==：subList 返回的是 ArrayList 的内部类 SubList，并不是 ArrayList ，而是 ArrayList 的一个视图，对于 SubList 子列表的所有操作最终会反映到原列表上。
    - 正例：有一个列表存在1000条记录，我们需要删除100-200位置处的数据：
        
    ```
    list1.subList(100, 200).clear();
    ```

1. 在 subList 场景中，高度注意对原集合元素个数的修改，会导致子列表的遍历、增加、 删除均产生 ConcurrentModificationException 异常。
1.  使用集合转数组的方法，必须使用集合的 toArray(T[] array)，传入的是类型完全一样的数组，大小就是 list.size()。 
    - ~~反例~~：直接使用 toArray 无参方法存在问题，此方法返回值只能是 Object[]类，若强转其它 类型数组将出现 ClassCastException 错误。
    - ==说明==：使用 toArray 带参方法，入参分配的数组空间不够大时，toArray 方法内部将重新分配内存空间，并返回新数组地址；如果数组元素大于实际所需，下标为[ list.size()]的数组元素将被置为null，其它数组元素保持原值，因此最好将方法入参数组大小定义与集合元素 个数一致。
    - 正例： 
    
        ```
        List list = new ArrayList(2);
        list.add("guan");
        list.add("bao");
        String[] array = new String[list.size()];
        array = list.toArray(array);
        ```


    
1. 使用工具类 Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常。 
    - ==说明==：asList 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。Arrays.asList 体现的是适配器模式，只是转换接口，后台的数据仍是数组。 
    
        ```
        String[] str = new String[] { "a", "b" };
        List list = Arrays.asList(str);
        list.add("c"); //运行时异常。
        str[0]= "gujin"; //list.get(0)也会随之修改。
        ```

1. 使用 entrySet 遍历 Map 类集合 KV，而不是 keySet 方式进行遍历。
    - ==说明==： keySet 其实是遍历了 2 次， 一次是转为 Iterator 对象， 另一次是从 hashMap 中取出 key所对应的 value。而 entrySet 只是遍历了一次就把 key 和 value 都放到了 entry 中，效率更高。如果是 JDK8，使用 Map.foreach 方法。
    - 正例：values()返回的是 V 值集合，是一个 list 集合对象；keySet()返回的是 K 值集合，是一个 Set 集合对象；entrySet()返回的是 K-V 值组合集合。
# 并发处理
1.  SimpleDateFormat 是线程不安全的类，一般不要定义为 static 变量，如果定义为 static，必须加锁，或者使用 DateUtils 工具类。 
    - 正例：注意线程安全，使用 DateUtils。亦推荐如下处理： 
    
        ```
        private static final ThreadLocal <DateFormat> df = new ThreadLocal<>() {
                @Override
                protected DateFormat initialValue() {
                    return new SimpleDateFormat("yyyy-MM-dd");
                }
            };
        ```

    - ==说明==：如果是 JDK8 的应用，可以使用 Instant 代替 Date，Localdatetime 代替 Calendar， Datetimeformatter 代替 Simpledateformatter，官方给出的解释：simple beautiful strong immutable thread-safe。

1. 线程池不允许使Executor去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 
    - ==说明==：Executors 各个方法的弊端：
        - newFixedThreadPool 和 newSingleThreadExecutor: 主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。 
        - newCachedThreadPool 和 newScheduledThreadPool: 主要问题是线程数最大数是 Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至 OOM。
1. 并发修改同一记录时，避免更新丢失，要么在应用层加锁，要么在缓存加锁，要么在数据库层使用乐观锁，使用version作为更新依据。
    - ==说明==：如果每次访问冲突概率小于 20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于 3 次。
1. 多线程并行处理定时任务时，Timer 运行多个 TimeTask 时，只要其中之一没有捕获抛出的异常， 其它任务便会自动终止运行， 使用 ScheduledExecutorService 则没有这个问题。
1. 通过双重检查锁（double-checked locking）（在并发场景）实现延迟初始化的优化问题隐患,推荐将目标属性声明为 volatile 型（比如反例中修改 helper 的属性声明为volatile；
    - ~~反例~~：
    
        ```
        class Foo {
                private Helper helper = null;
                public Helper getHelper() {
                    if (helper == null) synchronized(this) {
                        if (helper == null)
                            helper = new Helper();
                    }
                    return helper; }
                // other functions and members...
            }
        ```

1. ThreadLocal无法解决共享对象的更新问题， ThreadLocal对象建议==使用static修饰==。这个变量是针对一个线程内所有操作共有的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。
# 方法中需要进行参数校验的场景：
- 调用频次低的方法。 
- 执行时间开销很大的方法，参数校验时间几乎可以忽略不计，但如果因为参数错误导致 中间执行回退，或者错误，那得不偿失。 
- 需要极高稳定性和可用性的方法。 
- 对外提供的开放接口，不管是 RPC/API/HTTP 接口。
# 方法中不需要参数校验的场景：
- 极有可能被循环调用的方法，不建议对参数进行校验。但在方法说明里必须注明外部参 数检查。 
- 底层的方法调用频度都比较高，一般不校验。毕竟是像纯净水过滤的最后一道，参数错 误不太可能到底层才会暴露问题。一般 DAO 层与 Service 层都在同一个应用中，部署在同一台 服务器中，所以 DAO 的参数校验，可以省略。 
- 被声明成 private 只会被自己代码所调用的方法，如果能够确定调用方法的代码传入参 数已经做过检查或者肯定不会有问题，此时可以不校验参数。
# 其他
- 在使用正则表达式时，利用好其预编译功能，可以有效加快正则匹配速度。 
说明：不要在方法体内定义：Pattern pattern = Pattern.compile(规则);
- 避免用 Apache Beanutils 进行属性的 copy。 
说明：Apache BeanUtils 性能较差，可以使用其他方案比如 Spring BeanUtils, Cglib BeanCopier。
- 工具类二方库已经提供的，尽量不要在本应用中编程实现。 
    - json 操作： fastjson 
    - md5 操作：commons-codec 
    - 工具集合：Guava 包 
    - 数组操作：ArrayUtils（org.apache.commons.lang3.ArrayUtils） 
    - 集合操作：CollectionUtils 
    - 除上面以外还有 NumberUtils、DateFormatUtils、DateUtils 等优先使用 org.apache.commons.lang3 这个包下的，不要使用 org.apache.commons.lang 包下面 的。原因是 commons.lang 这个包是从 JDK1.2 开始支持的所以很多 1.5/1.6 的特性是不 支持的，例如：泛型。

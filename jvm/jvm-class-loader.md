### hotspot类信息内存结构

![][1]
如上图 class的字节码放在数据区，然后在Metaspace中创建对应的Class对象，这个是java反射的基础。

### 问题： java.lang.ClassNotFoundException 和 java.lang.NoClassDefFoundError的区别？ 

方法 loadClass()抛出的是 java.lang.ClassNotFoundException异常；方法 defineClass()抛出的是 java.lang.NoClassDefFoundError异常。

### 类加载器

![][2]


- BootstrapClassLoader
	- 加载路径通过 System.getProperty("sun.boot.class.path")获取, 包含 rt.jar
- ExtClassLoader
	- 路径包含 ext.jar 
- AppClassLoader
	- 加载的路径是通过 System.getProperty("java.class.path")得到的， 也可以使用 java -cp/-classpath 来追加AppClassLoader要加载的类路径。 

Launcher 类会调用 

```
# 设置APPClassLoader放在thread的上下文中
Thread.currentThread().setContextClassLoader(this.loader);


```


### 指定完类的路径，什么情况下会把class加载到metaspace?

主动使用class会触发类初始化， 类的初始化一定会触发类加载，类加载不一定触发初始化。

- getStatic 类的字段
- invokeStatic 方法
- 调用类的main方法
- new
- Class.forname
- 子类初始化，会触发父类的接在和初始化

```

public class MoheClassLoader extends ClassLoader {

    private String classLoaderName = null;
    private String loadPath = "";

    public MoheClassLoader(String classLoaderName, String loadPath) {
        //默认的父 类加载器 为 APPClassLoader
        super();
        this.classLoaderName = classLoaderName;
        this.loadPath = loadPath;
    }

    @Override
    public Class findClass(String name) {
        byte[] b = loadClassData(name);
        System.out.println("MoheClassLoader load class " + name);
        return defineClass(name, b, 0, b.length);
    }

    private byte[] loadClassData(String name) {
        // load the class data from the connection
        String abstractFilePath = loadPath + "/" + name.replace(".", "/") + ".class";
        try {
            FileInputStream fin = new FileInputStream(new File(abstractFilePath));
            byte classByte[] = new byte[fin.available()];
            fin.read(classByte);
            return classByte;
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}

```
我们自定义一个Person类 

```
public class Person {
    private String name;
    private int age;
    
    public Person toPerson(Object o){
        return (Person)o;
    }
    
    //getter setter toString
}
```

执行如下代码, 把编译好的Person.class 方到 resources/classloader/test/com/moheqionglin/classLoader/Person.class 里面。

```

public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
    	 //resources/classloader/test/com/moheqionglin/
        String path = Main.class.getClassLoader().getResource(".").getPath();
        path = path + "classloader/test/";
        System.out.println(path);
        MoheClassLoader loader = new MoheClassLoader("loader1", path);
        System.out.println(loader.getParent());
        Class<?> aClass = loader.loadClass("com.moheqionglin.classLoader.Person");
        System.out.println(aClass.getClassLoader());
    }
}

# 这个执行结果如下
loader parent is sun.misc.Launcher$AppClassLoader@135fbaa4
sun.misc.Launcher$AppClassLoader@135fbaa4

# 如果我们把 target中的Person.class删了以后，在执行
loader parent is sun.misc.Launcher$AppClassLoader@135fbaa4
MoheClassLoader load class com.moheqionglin.classLoader.Person
com.moheqionglin.classLoader.MoheClassLoader@4ac68d3e
```

### class loader的命名空间namespace
类加载器的namespace是通过，类加载器本身和所有父加载器所加载出来的 binary name(full class name)组成的

- 同一namespace，不允许出现两个完全一样的类。。
- 不同namespace，可以出现两个相同类，此时 两个Class对象互相不感知对方存在。也就是Class对象类型不一样

关于命名空间的重要说明 

- 1、子加载器所加载的类能够访问到父加载器所加载的类 
- 2、父加载器所加载的类无法访问到子类加载器所加载的类


```

private static void loadClassTestWithDifferentLoaderName() throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        String path = Main.class.getClassLoader().getResource(".").getPath();
        path = path + "classloader/test/";

        MoheClassLoader loader1 = new MoheClassLoader("", path);
        MoheClassLoader loader2 = new MoheClassLoader("", path);

        Class<?> aClass1 = loader1.loadClass("com.moheqionglin.classLoader.Person");
        Class<?> aClass2 = loader2.loadClass("com.moheqionglin.classLoader.Person");

        Object p1 = aClass1.newInstance();
        Object p2 = aClass2.newInstance();

        Method toPersonMethod = aClass1.getMethod("toPerson", Object.class);
        toPersonMethod.invoke(p1, p2);
    }
    
# 结果
Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.moheqionglin.classLoader.Main.loadClassTestWithDifferentLoaderName(Main.java:31)
	at com.moheqionglin.classLoader.Main.main(Main.java:14)
Caused by: java.lang.ClassCastException: com.moheqionglin.classLoader.Person cannot be cast to com.moheqionglin.classLoader.Person
	at com.moheqionglin.classLoader.Person.toPerson(Person.java:13)
	... 6 more    
```

### 线程上下文加载器

SPI的 ServiceLoader 打破双亲委派模型
关于双亲委派的的重要说明 

- 1、子加载器所加载的类能够访问到父加载器所加载的类 
- 2、父加载器所加载的类无法访问到子类加载器所加载的类
- 3、双亲委派模型加载的类，只会被加载一次。
- 4、双薪委派模型加的类，一旦加载，无法卸载。

因为双亲委派无法解决所有情况的类加载问题，比如jdbc中，DriverManager是在rt.jar中，由BootstrapClassLoader加载，但是具体实现比如 com.mysql.Driver是由 AppClassLoader中加载，BootstrapClassLoader无法找到com.mysql.Driver这个类，从因此无法加载。 因此为了解决这种僵化的类加载机制，引入线程上下文加载器，其中SPI就是他的一个具体实践的形式，来解 JDBC、JCE、JNDI、JAXP 和 JBI 等问题。
<br>
**使用线程上下文类加载器，可以在执行线程中抛弃双亲委派加载链模式，使用线程上下文里的类加载器加载类**
<br>
线程上下文从根本解决了一般应用不能违背双亲委派模式的问题。使java类加载体系显得更灵活。随着多核时代的来临，相信多线程开发将会越来越多地进入程序员的实际编码过程中。因此，在编写基础设施时， 通过使用线程上下文来加载类，应该是一个很好的选择。

![][3]

实现SPI步骤

-  声明一个Interface
-  在resrouces/META-INF/services/接口全类名


```

public interface SipInterface {
    void doSomething();
}

public class SipSub1 implements SipInterface{
    @Override
    public void doSomething() {
        System.out.println("SipSub1 class");
    }
}

public class SipSub2 implements SipInterface{
    @Override
    public void doSomething() {
        System.out.println("SipSub2 class");
    }
}

```

```
#在resources中创建如下文件
META-INF/services/com.moheqionglin.classLoader.spi.SipInterface

com.moheqionglin.classLoader.spi.SipSub1
com.moheqionglin.classLoader.spi.SipSub2

```

```

public class Main {
    public static void main(String[] args) {

        ServiceLoader<SipInterface> load = ServiceLoader.load(SipInterface.class);

        for(Iterator<SipInterface> iterator = load.iterator(); iterator.hasNext();){
            iterator.next().doSomething();
        }
    }
}

#输出 
SipSub1 class
SipSub2 class

```

源码

```

public static <S> ServiceLoader<S> load(Class<S> service) {
		 //默认是线程上下文loader
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,
                                            ClassLoader loader){
        return new ServiceLoader<>(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }
    
    public void reload() {
        providers.clear();
        lookupIterator = new LazyIterator(service, loader);
    }
    
    public Iterator<S> iterator() {
        return new Iterator<S>() {

            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }
    
    public S next() {
            if (acc == null) {
                return nextService();
            } else {
                PrivilegedAction<S> action = new PrivilegedAction<S>() {
                    public S run() { return nextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }
        
        private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {//这里可以看到，还是用Class.forname形式加载的，默认不初始化，loader是线程上下文loader
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                S p = service.cast(c.newInstance());
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }
    
```

### 开源框架的类加载器

下面是dubbo的类加载器顺序，默认是线程上下文类加载器，如果没有的话，是当前类加载器，最后才是AppClassLoader。

```
public static ClassLoader getClassLoader(Class<?> clazz) {
        ClassLoader cl = null;
        try {
            cl = Thread.currentThread().getContextClassLoader();
        } catch (Throwable ex) {
         // Cannot access thread context ClassLoader - falling back to system class loader...
        }
        if (cl == null) {
            // No thread context class loader -> use class loader of this class.
            cl = clazz.getClassLoader();
            if (cl == null) {
                // getClassLoader() returning null indicates the bootstrap ClassLoader
                try {
                    cl = ClassLoader.getSystemClassLoader();
                }
                catch (Throwable ex) {
                    // Cannot access system ClassLoader - oh well, maybe the caller can live with null...
                }
            }
        }

        return cl;
    }

```

问题：何时使用Thread.getContextClassLoader()?


这是一个很常见的问题，但答案却很难回答。这个问题通常在需要动态加载类和资源的系统编程时会遇到。总的说来动态加载资源时，往往需要从三种类加载器里选择：系统或说程序的类加载器、当前类加载器、以及当前线程的上下文类加载器。在程序中应该使用何种类加载器呢？


系统类加载器通常不会使用。此类加载器处理启动应用程序时classpath指定的类，可以通过ClassLoader.getSystemClassLoader()来获得。所有的ClassLoader.getSystemXXX()接口也是通过这个类加载器加载的。一般不要显式调用这些方法，应该让其他类加载器代理到系统类加载器上。由于系统类加载器是JVM最后创建的类加载器，这样代码只会适应于简单命令行启动的程序。一旦代码移植到EJB、Web应用或者Java Web Start应用程序中，程序肯定不能正确执行。


因此一般只有两种选择，当前类加载器和线程上下文类加载器。当前类加载器是指当前方法所在类的加载器。这个类加载器是运行时类解析使用的加载器，Class.forName(String)和Class.getResource(String)也使用该类加载器。代码中X.class的写法使用的类加载器也是这个类加载器。


线程上下文类加载器在Java 2(J2SE)时引入。每个线程都有一个关联的上下文类加载器。如果你使用new Thread()方式生成新的线程，新线程将继承其父线程的上下文类加载器。如果程序对线程上下文类加载器没有任何改动的话，程序中所有的线程将都使用系统类加载器作为上下文类加载器。Web应用和Java企业级应用中，应用服务器经常要使用复杂的类加载器结构来实现JNDI（Java命名和目录接口)、线程池、组件热部署等功能，因此理解这一点尤其重要。


为什么要引入线程的上下文类加载器？将它引入J2SE并不是纯粹的噱头，由于Sun没有提供充分的文档解释说明这一点，这使许多开发者很糊涂。实际上，上下文类加载器为同样在J2SE中引入的类加载代理机制提供了后门。通常JVM中的类加载器是按照层次结构组织的，目的是每个类加载器（除了启动整个JVM的原初类加载器）都有一个父类加载器。当类加载请求到来时，类加载器通常首先将请求代理给父类加载器。只有当父类加载器失败后，它才试图按照自己的算法查找并定义当前类。


有时这种模式并不能总是奏效。这通常发生在JVM核心代码必须动态加载由应用程序动态提供的资源时。拿JNDI为例，它的核心是由JRE核心类(rt.jar)实现的。但这些核心JNDI类必须能加载由第三方厂商提供的JNDI实现。这种情况下调用父类加载器（原初类加载器）来加载只有其子类加载器可见的类，这种代理机制就会失效。解决办法就是让核心JNDI类使用线程上下文类加载器，从而有效的打通类加载器层次结构，逆着代理机制的方向使用类加载器。



顺便提一下，XML解析API(JAXP)也是使用此种机制。当JAXP还是J2SE扩展时，XML解析器使用当前累加载器方法来加载解析器实现。但当JAXP成为J2SE核心代码后，类加载机制就换成了使用线程上下文加载器，这和JNDI的原因相似。


好了，现在我们明白了问题的关键：这两种选择不可能适应所有情况。一些人认为线程上下文类加载器应成为新的标准。但这在不同JVM线程共享数据来沟通时，就会使类加载器的结构乱七八糟。除非所有线程都使用同一个上下文类加载器。而且，使用当前类加载器已成为缺省规则，它们广泛应用在类声明、Class.forName等情景中。即使你想尽可能只使用上下文类加载器，总是有这样那样的代码不是你所能控制的。这些代码都使用代理到当前类加载器的模式。混杂使用代理模式是很危险的。


更为糟糕的是，某些应用服务器将当前类加载器和上下文类加器分别设置成不同的ClassLoader实例。虽然它们拥有相同的类路径，但是它们之间并不存在父子代理关系。想想这为什么可怕：记住加载并定义某个类的类加载器是虚拟机内部标识该类的组成部分，如果当前类加载器加载类X并接着执行它，如JNDI查找类型为Y的数据，上下文类加载器能够加载并定义Y，这个Y的定义和当前类加载器加载的相同名称的类就不是同一个，使用隐式类型转换就会造成异常。



这种混乱的状况还将在Java中存在很长时间。在J2SE中还包括以下的功能使用不同的类加载器：


l        JNDI使用线程上下文类加载器


l        Class.getResource()和Class.forName()使用当前类加载器


l        JAXP使用上下文类加载器


l        java.util.ResourceBundle使用调用者的当前类加载器


l        URL协议处理器使用java.protocol.handler.pkgs系统属性并只使用系统类加载器。


l        Java序列化API缺省使用调用者当前的类加载器


这些类加载器非常混乱，没有在J2SE文档中给以清晰明确的说明。


该如何选择类加载器？

如若代码是限于某些特定框架，这些框架有着特定加载规则，则不要做任何改动，让框架开发者来保证其工作（比如应用服务器提供商，尽管他们并不能总是做对）。如在Web应用和EJB中，要使用Class.gerResource来加载资源。在其他情况下，需要考虑使用下面的代码，这是作者本人在工作中发现的经验：


 


public abstract class ClassLoaderResolver


{


    /**


     * This method selects the best classloader instance to be used for


     * class/resource loading by whoever calls this method. The decision


     * typically involves choosing between the caller's current, thread context,


     * system, and other classloaders in the JVM and is made by the {@link IClassLoadStrategy}


     * instance established by the last call to {@link #setStrategy}.


     *


     * @return classloader to be used by the caller ['null' indicates the


     * primordial loader]  


     */


    public static synchronized ClassLoader getClassLoader ()


    {


        final Class caller = getCallerClass (0);


        final ClassLoadContext ctx = new ClassLoadContext (caller);



        return s_strategy.getClassLoader (ctx);


    }


    public static synchronized IClassLoadStrategy getStrategy ()


    {


        return s_strategy;


    }


    public static synchronized IClassLoadStrategy setStrategy (final IClassLoadStrategy strategy)


    {


        final IClassLoadStrategy old = s_strategy;


        s_strategy = strategy;


       


        return old;


    }


       


    /**


     * A helper class to get the call context. It subclasses SecurityManager


     * to make getClassContext() accessible. An instance of CallerResolver


     * only needs to be created, not installed as an actual security


     * manager.


     */


    private static final class CallerResolver extends SecurityManager


    {


        protected Class [] getClassContext ()


        {


            return super.getClassContext ();


        }


       


    } // End of nested class


   


   


    /*


     * Indexes into the current method call context with a given


     * offset.


     */


    private static Class getCallerClass (final int callerOffset)


    {       


        return CALLER_RESOLVER.getClassContext () [CALL_CONTEXT_OFFSET +


            callerOffset];


    }


   


    private static IClassLoadStrategy s_strategy; // initialized in <clinit>


   


    private static final int CALL_CONTEXT_OFFSET = 3; // may need to change if this class is redesigned


    private static final CallerResolver CALLER_RESOLVER; // set in <clinit>


   


    static


    {


        try


        {


            // This can fail if the current SecurityManager does not allow


            // RuntimePermission ("createSecurityManager"):


           


            CALLER_RESOLVER = new CallerResolver ();


        }


        catch (SecurityException se)


        {


            throw new RuntimeException ("ClassLoaderResolver: could not create CallerResolver: " + se);


        }


       


        s_strategy = new DefaultClassLoadStrategy ();


    }


} // End of class.


 


    可通过调用ClassLoaderResolver.getClassLoader()方法来获取类加载器对象，并使用其ClassLoader的接口来加载类和资源。此外还可使用下面的ResourceLoader接口来取代ClassLoader接口：


 


public abstract class ResourceLoader


{


    /**


     * @see java.lang.ClassLoader#loadClass(java.lang.String)


     */


    public static Class loadClass (final String name)


        throws ClassNotFoundException


    {


        final ClassLoader loader = ClassLoaderResolver.getClassLoader (1);


        return Class.forName (name, false, loader);


    }


    /**


     * @see java.lang.ClassLoader#getResource(java.lang.String)


     */   


    public static URL getResource (final String name)


    {


        final ClassLoader loader = ClassLoaderResolver.getClassLoader (1);


        if (loader != null)


            return loader.getResource (name);


        else


            return ClassLoader.getSystemResource (name);


    }


    ... more methods ...


} // End of class


    决定应该使用何种类加载器的接口是IClassLoaderStrategy：


 


public interface IClassLoadStrategy


{


    ClassLoader getClassLoader (ClassLoadContext ctx);


} // End of interface


    为了帮助IClassLoadStrategy做决定，给它传递了个ClassLoadContext对象作为参数：


 


public class ClassLoadContext


{


    public final Class getCallerClass ()


    {


        return m_caller;


    }


   


    ClassLoadContext (final Class caller)


    {


        m_caller = caller;


    }


   


    private final Class m_caller;


} // End of class


    ClassLoadContext.getCallerClass()返回的类在ClassLoaderResolver或ResourceLoader使用，这样做的目的是让其能找到调用类的类加载器（上下文加载器总是能通过Thread.currentThread().getContextClassLoader()来获得）。注意


 


调用类是静态获得的，因此这个接口不需现有业务方法增加额外的Class参数，而且也适合于静态方法和类初始化代码。具体使用时，可以往这个上下文对象中添加具体部署环境中所需的其他属性。


    上面代码看起来很像Strategy设计模式，其思想是将“总是使用上下文类加载器”或者“总是使用当前类加载器”的决策同具体实现逻辑分离开。往往设计之初是很难预测何种类加载策略是合适的，该设计能够让你可以后来修改类加载策略。


 


这儿有一个缺省实现，应该可以适应大部分工作场景：


 


public class DefaultClassLoadStrategy implements IClassLoadStrategy


{


    public ClassLoader getClassLoader (final ClassLoadContext ctx)


    {


        final ClassLoader callerLoader = ctx.getCallerClass ().getClassLoader ();


        final ClassLoader contextLoader = Thread.currentThread ().getContextClassLoader ();


       


        ClassLoader result;


       


        // If 'callerLoader' and 'contextLoader' are in a parent-child


        // relationship, always choose the child:


       


        if (isChild (contextLoader, callerLoader))


            result = callerLoader;


        else if (isChild (callerLoader, contextLoader))


            result = contextLoader;


        else


        {


            // This else branch could be merged into the previous one,


            // but I show it here to emphasize the ambiguous case:


            result = contextLoader;


        }


        final ClassLoader systemLoader = ClassLoader.getSystemClassLoader ();


       

        // Precaution for when deployed as a bootstrap or extension class:


        if (isChild (result, systemLoader))


            result = systemLoader;


       


        return result;


    }


   


    ... more methods ...


} // End of class


 


    上面代码的逻辑很简单：如调用类的当前类加载器和上下文类加载器是父子关系，则总是选择子类加载器。对子类加载器可见的资源通常是对父类可见资源的超集，因此如果每个开发者都遵循J2SE的代理规则，这样做大多数情况下是合适的。


    当前类加载器和上下文类加载器是兄弟关系时，决定使用哪一个是比较困难的。理想情况下，Java运行时不应产生这种模糊。但一旦发生，上面代码选择上下文类加载器。这是作者本人的实际经验，绝大多数情况下应该能正常工作。你可以修改这部分代码来适应具体需要。一般来说，上下文类加载器要比当前类加载器更适合于框架编程，而当前类加载器则更适合于业务逻辑编程。


    最后需要检查一下，以便保证所选类加载器不是系统类加载器的父亲，在开发标准扩展类库时这通常是个好习惯。


    注意作者故意没有检查要加载资源或类的名称。Java XML API成为J2SE核心的历程应该能让我们清楚过滤类名并不是好想法。作者也没有试图检查哪个类加载器加载首先成功，而是检查类加载器的父子关系，这是更好更有保证的方法。


## 附录 参考文档
[https://www.ibm.com/developerworks/cn/java/j-lo-classloader/][4]

[1]: ../images/jmv-optimize/jvm-class.jpg
[2]: ../images/jmv-optimize/jvm-classloader.jpg
[3]: ../images/jmv-optimize/jvm-sip-thread-context-loader.png
[4]: https://www.ibm.com/developerworks/cn/java/j-lo-classloader/
[5]: https://www.ibm.com/developerworks/cn/java/j-lo-hotswapcls/
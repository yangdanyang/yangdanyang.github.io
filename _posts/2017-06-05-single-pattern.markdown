## 单例模式

### 定义:
    确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例
    类，它提供全局访问的方法。单例模式是一种对象创建型模式。

### 特点:
    一:某个类只能有一个实例；
    二:它必须自行创建这个实例；
    三:它必须自行向整个系统提供这个实例。

### 角色:
    Singleton(单例): 在单例类的内部只生成一个实例，同时它提供一个静态的getInstance()
    工厂方法，让客户可以访问它的唯一实例；为了防止在外部对其实例化，将其构造函数设计为私
    有；在单例类内部定义了一个Singleton类型的静态对象，作为外部共享的唯一实例。

### 饿汉式单例类:
    在定义静态变量的时候实例化单例类，因此在类加载的时候就已经创建了单例对象。当类被加载
    时，静态变量instance会被初始化此时类的私有构造函数会被调用，单例类的唯一实例将被创
    建。
    代码示例:
    public class EagerSingleton {

    private static EagerSingleton instance = new EagerSingleton();

    private EagerSingleton(){}

    public static EagerSingleton getEagerSingleton(){
        return instance;
    }
}

### 懒汉式单例类:
    为了避免多个线程同时调用getInstance()方法，我们可以使用关键字synchronized,该懒汉
    式单例类在getInstance()方法前面增加了关键字synchronized进行线程锁，以处理多个线程
    同时访问的问题。
    例如:
    class LazySingleton { 
	    private static LazySingleton instance = null; 
	
	    private LazySingleton() { } 
	
	    synchronized public static LazySingleton getInstance() { 
	        if (instance == null) {
	            instance = new LazySingleton(); 
	        }
	        return instance; 
	    }
    }
    
    但是，上述代码虽然解决了线程安全问题，但是每次调用getInstance()时候
    都需要进行线程锁定判断，在多线程高并发访问环境中，将会导致系统性能大大降低。我们继续对
    懒汉式单例进行改进。事实上，我们无须对整个getInstance()方法进行锁定，只需要对其中的
    代码"instance = new LazySingleton();"进行锁定即可。
    代码如下：
    public static LazySingleton getInstance() {   
	    if (instance == null) {  
	        synchronized (LazySingleton.class) {  
	            instance = new LazySingleton();   
	        }  
	    }  
       return instance;   
    }  

    问题貌似得以解决，事实并非如此。如果使用以上代码来实现单例，还是会存在单例对象不唯一。
    原因如下:
    假如在某一瞬间线程A和线程B都在调用getInstance(）方法，此时instance对象为null值，
    均能通过instance == null的判断。由于实现了synchronized加锁机制，线程A进入
    synchronized锁定的代码中执行实例创建代码，线程B处于排队等待状态，必须等待线程A执行
    完毕后才可以进入synchronized锁定代码。但当A执行完毕后，线程B并不知道实例已经创建，
    将继续创建新的实例导致产生多个单例对象，违背了单例模式的设计思想，因此需要进一步进行
    改进，在synchronized中再进行一次（instance== null）判断，这种方法称为双重检查锁
    定。需要注意的是，如果使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量之前增加修
    饰符volatile，被volatile修饰的成员变量可以确保多个线程都能够正确处理，且该代码只能在
    JDK1.5及以上版本中才能正确执行
    代码如下:
    public class LazySingleton {
	    private volatile static LazySingleton instance = null;
	
	    private LazySingleton(){};
	
	    public static LazySingleton getInstance(){
	        //第一重判断
	        if(instance == null){
	            //锁定代码块
	            synchronized (LazySingleton.class){
	                //第二重判断
	                if(instance == null){
	                    instance = new LazySingleton();//创建单例实例
	                }
	            }
	        }
	
	        return instance;
	    }
	}    

#### 比较:
    饿汉式相对于饱汉式调用速度和反应时间角度有优势，但无论是否需要，都创建单例对象，资源利
    用效率角度来讲，不及懒汉式单例，而且在系统加载时由于需要创建饿汉式单例对象，加载时间可
    能会比较长。
    双重检查锁定等机制，将使懒汉式单例的系统性能受到一定影响。

### Initialization Demand Holder (IoDH)：
    由于静态单例对象没有作为Singleton的成员变量直接实例化，因此类加载时不会实例化
    Singleton,第一次调用getInstance()时将加载内部类HolderClass，在该内部类中定义
    了一个static类型的变量instance，此时会首先初始化这个成员变量，由Java虚拟机来保证
    其线程安全性，确保该成员变量只能初始化一次。由于getInstance(）方法没有任何线程锁定，
    因此其性能不会造成任何影响。通过使用IoDH,我们既可以实现延迟加载，又可以保证线程安全，
    不影响系统性能，不失为一种最好的Java语言单例模式实现方式（缺点是与编程语言本省的特性
    相关，很多面向对象语言不支持IoDH).
    代码如下:
    public class Singleton {
	    private Singleton(){}
	
	    private static class HolderClass{
	        private final static Singleton instance = new Singleton();
	    }
	
	    public static Singleton getInstance(){
	        return HolderClass.instance;
	    }
	}
    

### 优点:
    1)单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控
    制客户怎样以及何时访问它。
    2）由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的
    对象单例模式无疑可以提高系统的性能
    3）允许可变数目的实例。基于单例模式我们可以进行扩展，使用与单例控制相似的方法来获得
    指定个数的对象实例，既节省系统资源又解决了单例对象共享过多有损性能的问题。

### 缺点:
    1)由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
    2）单例类的职责过重，在一定程度上违背了单一职责原则，因为单例类既充当了工厂角色，提
    供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能
    融合到一起。
    3）很多语言的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不
    被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致
    共享的单例对象状态的丢失。

### 使用场景:
    1)系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或者资源管理器，或者
    需要考虑资源消耗太大而只允许创建一个对象。
    2）客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径
    访问该实例。
    
### 问题
#### 单例模式为什么要将成员变量定义为静态变量？
     回答:在单例模式中使用静态方法返回示例给外部对象，而静态方法不能访问非静态成员变量。
     原因如下:
         static成员是在JVM的CLASSLOADER加载类的时候初始化的，而非static的成员是在创
         建对象，即new 操作的时候才初始化的；类加载的时候初始化static的成员，此时
         static 已经分配内存空间，所以可以访问；非static的成员还没有通过new创建对象而
         进行初始化，所以必然不可以访问。
         简单点说：静态成员属于类,不需要生成对象就存在了.而非静态需要生成对象才产生，
         所以静态成员不能直接访问.
#### 如何对单例模式进行改造，使得系统中的某个类的对象可以存在有限多个，例如两例或三例？ 
    回答:假设最多有max_num种实例，如果调用次数超过，则默认为第max_num种
    代码如下:
    public class Test {  
    private static ArrayList<TestInstance> testInstance = new ArrayList<TestInstance>();  
    static int num = 0;  
    static int max_num = 3;  
    
    //最多有max_num种套餐，如果调用次数超过，则默认为最后一种  
    public static TestInstance getinstance(){        
	    synchronized (Test.class) {  
	            if(num < max_num){  
	                num++;  
	                testInstance.add(new TestInstance());  
	            }  
	        }  
	        return testInstance.get(num-1);  
	    }  
	      
	}  
  
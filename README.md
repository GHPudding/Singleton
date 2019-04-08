java单例设计模式详解（懒汉饿汉式）+深入分析为什么懒汉式是线程不安全的终极解决办法
单例设计模式就是：运用单例模式设计的这个类在每次实例化的时候只能产生一个对象。比如A类是利用单例设计模式设计的一个类，现在B，C两个类都需要使用到A类的实例怎么办？这个时候就要看谁先实例化的A类了，如果是B类第一次实例化的A类，那么A类就只能实例化这一次了，C类调用A类的时候检查到A类已经被实例化过了，那么他就只能使用已经实例化过的A类。是不是A,B,C绕来绕去把你们绕迷了。那么下面就进入代码实战。
还有一点需要注意，一个类如果只能实例化一次，那么它的构造函数一定是私有的。如果它的构造是public的话，那么需要使用到该类的所有类都能实例化这个类了，那就不行了。所以这个类必须是私有的。既然是私有的那就只能自己调用自己来实例化了也就顺理成章的想到通过一个方法return new A();返回一个对象。
下面这几行代码就是一个最简单的单例设计模式，下面我们就从这个最简单的开始一步一步深入讲解。
public class Singleton {
    private Singleton(){}
    public Singleton getInsatnce(){
        return new Singleton();
    }
}
以上代码虽然完成了我们上面的要求，但是getInstance()这个方法不能直接调用啊，如果不实例化Singletonle这个类的话。
那么自然就想到把这个类static化，使用static关键字就可以在不产生实例化对象的情况下去调用该方法。于是就变成：
public class Singleton {
    private Singleton(){}
    public static Singleton getInsatnce(){
        return new Singleton();
    }
}
但是这样的话有一个问题就是每次调用getInstance()方法的时候都会new一个Singleton对象，
所以为了只让它产生一个实例化对象，我们提前造一个对象，在该方法中返回就行了。
public class Singleton {
    private static Singleton singleton=new Singleton();//这里为什么要使用private 和static两个关键字修饰是有原因的，希望你们能弄清楚。
    private Singleton(){}
    public static Singleton getInsatnce(){
        return singleton;
    }
}
以上我们便是实现了著名的饿汉式单例，是不是看起来很饿，没人叫他他就先实例化好自己，整装待发时刻准备着去吃饭。
也正是有这一个特性，使用饿汉式有一点不好就是浪费资源，只要这个方法一加载立即就产生对象，产生对象那是要占用堆内存和栈内存的啊。
你说这是浪费资源吗。所以我们来看懒汉式。懒汉式单例与饿汉式刚好相反，别人不叫他他就不实例化自己刚好解决了上面出现的问题。
但是他也有缺点，下面分析缺点，这里先看代码：
public class Singleton {
    private static Singleton singleton; //-----------1
    private Singleton(){//;-----------2
    //;-----------
    }//;-----------4
    public static Singleton getInsatnce(){//;-----------5
        if (singleton == null){//;-----------6
        singleton = new Singleton();//;-----------7
    }//;-----------8
    return singleton;//;-----------9
    }
}
这就是大名鼎鼎的懒汉式单例设计模式，只有当你要使用这个类的时候才加载这个类，其他时候不加载。但是懒汉式设计虽然好用，但是它是线程不安全的。
什么是线程不安全？那就接着往下看把！
为了测试上面的类在被实例化的时候到底产生类几个对象，我们就在私有的构造方法中输出一行话，因为每产生一个新的类都要调用构造的，意思就是产生几个类构造中的这句话就被输出几次，下面我们来实现这个思路：
class Singleton {
    private static Singleton singleton; //-----------1
    private Singleton(){//;-----------2
        System.out.println("这是在构造方法中的一句话，用来验证产生了几个对象");//;-----------3
    }//;-----------4
    public static Singleton getInsatnce(){//;-----------5
        if (singleton == null){//;-----------6
            singleton = new Singleton();//;-----------7
        }//;-----------8
        return singleton;//;-----------9
    }
}
public class TestDemo{
    public static void main(String args[]) {
        new Thread(new Runnable() {
            public void run() {
                Singleton single=Singleton.getInsatnce();
            }
        },"线程A").start();
        new Thread(new Runnable() {
            public void run() {
                Singleton single=Singleton.getInsatnce();
            }
        },"线程B").start();
    }
}
输出结果：
这是在构造方法中的一句话，用来验证产生了几个对象
这是在构造方法中的一句话，用来验证产生了几个对象
或者输出：
这是在构造方法中的一句话，用来验证产生了几个对象（多运行几次就出现了）
纳尼？为什么会出现两种结果？不是说好的吗只会输出一句话的吗，为什么还会输出两句话？我也在程序中的第6句话判断了啊？
怎么还会输出两句话？别着急，容我慢慢分析。如果正常的话还谈什么线程不安全，输出错误才是正常的。
之所以出现错误是因为线程A和线程B的执行是随机的（随机就是不知道cpu什么是时候切换，反正1每个线程执行的总时间差不多是一样的），两个线程到底谁先执行，执行到那句代码停止让出资源让另外的一个线程执行，这也是随机的，具体的分配是靠进程去处理。所以，我们可以这样去分析：
假如当线程A先去执行，执行到第6句话的时候（第六句话执行完了），恰好停止了，这时候线程B就开始执行，线程B执行完第7句话才停止，释放资源让线程A去执行，但是A已经判断过了啊，已经执行完第六句话了啊，于是A继续执行第7句话，于是再次new了一个Singleton对象，因此，虽然已经判断了但是还是造了两个对象出来，到此分析完毕！
我们明白了问题的产生原理，那么怎么解决这个问题呢？那只有使用神奇的synchronized关键字了。那么问题出现了，我们应该怎么使用synchronized关键字化解这场危机呢？关于synchronized关键字有两种使用方法，一种是同步代码块（不知道这两种方法的可以自行百度），另一种是同步方法，至于使用哪一种，根据自己代码中的场景。下面我们使用同步方法把synchronized关键字加在getInstance()这个方法前面。代码如下：
class Singleton{
    private static Singleton singleton=null;
    private Singleton() {
        System.out.println("这是在singleton的构造方法中，主要是用来验证singleton类被实例化了几次");
    }
    public synchronized static Singleton getInstance () {
        if(singleton==null) {
            singleton=new Singleton();
        }
        return singleton;
    }
}
public class TestDemo{
    public static void main(String args[]) {
        new Thread(new Runnable() {
            public void run() {
                Singleton singleton=Singleton.getInstance();
            }
        },"线程A").start();
        new Thread(new Runnable() {
            public void run() {
                Singleton singleton=Singleton.getInstance();
            }
        },"线程B").start();
    }
}
看见没有，把synchronized关键字加在了getInstance()方法的前面，问题肯定会得以解决，请看下面输出：
这是在singleton的构造方法中，主要是用来验证Singleton类被实例化了几次
就算多运行几次还是只输出一个，意思是只实例化了这一次。但是虽然问题解决了。有一点不完美的地方就是把整个方法都加上了synchronized关键字，无疑会使得代码等待的时间变长，一次也就拖慢了代码的执行效率。那我们能不能不把整个方法加锁，只把可能出现问题的地方加一把锁可以吗？当然可以了，下面请看代码？
class Singleton{
    private static Singleton singleton=null;
    private Singleton() {
        System.out.println("这是在singleton的构造方法中，主要是用来验证Singleton类被实例化了几次");
    }
    public static Singleton getInstance () {
        if(singleton==null {//-----------------------1
            synchronized(Singleton.class) {//---------2
                if(singleton==null) {//---------------3
                    singleton=new Singleton();//----------4
                }
            }
        }
        return singleton;
    }
}
public class TestDemo{
    public static void main(String args[]) {
        new Thread(new Runnable() {
            public void run() {
                Singleton singleton=Singleton.getInstance();
            }
        },"线程A").start();
        new Thread(new Runnable() {
            public void run() {
                Singleton singleton=Singleton.getInstance();
            }
        },"线程B").start();
    }
}
输出：
这是在singleton的构造方法中，主要是用来验证Singleton类被实例化了几次
这次使用的是同步代码块的方式来解决的，至于同步代码块的方式怎么使用的，为什么要在synchronized()里面加上Apple.class，这些问题都是关于同步代码块怎么使用的，在这里不做详细解释。
但是到此你是不是已经觉得非常完美了，在这里我想遗憾的告诉你 NO！，在这里还有可能会出现错误，你可能会问为什呢？？？在这里就牵扯到JVM在编译时存在指令重排序的的优化机制了。

此时此刻你是不是已经绝望了这还有天理吗？？？别急，此时我告诉你还有解决方案！嘿嘿（猥琐而优雅的笑）。。。。
此时此刻牛逼plus的volatile关键字浮出水面。至于怎么使用volatile关键字，下面我们再来一一分解。不过，在此之前我要给大家讲上两个知识点：一个是什么是原子操作，一个是什么是指令重排。（如果你是大神，请自行略过）
首选来讲一下什么是原子操作：
简单的来说，原子操作（atomic）就是不可分割的操作，在计算机中，就是指不会因为线程调度被打断的操作。 
比如，简单的赋值就是一个原子操作
m=10;
例如m原先的值为10，那么对于这个操作，要么执行成功变成了10，要么执行失败变成了0，而不会出现诸如m=3这种中间状态，即使是在并发的线程中。 重点来啦！！然而声明赋值则不是原子操作，例如：
int m =10;
对于这个语句，至少有两个操作： 
1、声明一个变量m 
2、给m赋值为10 
这样就会有一个中间状态：变量m已经被申明了但是还没有赋值 
所以，这种状况在多线程中，由于线程执行的不确定性，如果两个线程都使用m，就可能倒是不稳定的结果出现。
二、指令重排
简单的来说：就是计算机为了提高执行效率，会做一些优化，在不影响最终结果的情况下，可能会对一些语句的执行顺序进行调整。例如;
int a;//语句1
a=8;//语句2
int b =9; //语句3
int c =a+b;//语句4
正常来说，对于顺序结构，执行的顺序是自上到下
但是，由于指令重排，因为不影响最终结果，所以执行的顺序很可能编程3124，或者1234
由于3，4没有原子性问题，语句3，4可能会被拆分成原子操作，再重排。
也就是说，对于非原子性操作，在不影响结果的情况下，其拆分成的原子操作可能会重排执行顺序

所以在上面的懒汉式代码中 singleton = new Singleton()这句，这并非是一个原子操作，事实上在JVM中这句话大概做了下面3件事情
1.给singleton分配内存
2.调用Singleton的构造函数来初始化成员变量
3.将singleton对象指向分配的内存空间（执行到这一步，singleton才是非null的了）
但是在JVM的及时编译器中，存在指令重拍的优化，也就是说，第二步和第三步的顺序是无法保证的
而导致程序出错
什么鬼？？讲人话！！！通俗一点说我们首先要理解new Apple()做了什么。new一个对象有几个步骤。1.看class对象是否加载，如果没有就先加载class对象，2.分配内存空间，初始化实例，3.调用构造函数，4.返回地址给引用。而cpu为了优化程序，可能会进行指令重排序，打乱这3，4这几个步骤，导致实例内存还没分配，就被使用了。
再用个线程A和线程B举例。线程A执行到new Singleton()，开始初始化实例对象，由于存在指令重排序，这次new操作，先把引用赋值了，还没有执行构造函数。这时时间片结束了，切换到线程B执行，线程B调用new Singleton()方法，发现引用不等于null，就直接返回引用地址了，然后线程B执行了一些操作，就可能导致线程B使用了还没有被初始化的变量。
所以我们拥有了一个最终版本！！！
class Singleton{
    private static volatile Singleton singleton=null;
    private Singleton() {
        System.out.println("这是在singleton的构造方法中，主要是用来验证Singleton类被实例化了几次");
    }
    public static Singleton getInstance () {
        if(singleton==null {//-----------------------1
            synchronized(Singleton.class) {//---------2
                if(singleton==null) {//---------------3
                    singleton=new Singleton();//----------4
                }
            }
        }
        return singleton;
    }
}
public class TestDemo{
    public static void main(String args[]) {
        new Thread(new Runnable() {
            public void run() {
                Singleton singleton=Singleton.getInstance();
            }
        },"线程A").start();
        new Thread(new Runnable() {
            public void run() {
                Singleton singleton=Singleton.getInstance();
            }
        },"线程B").start();
    }
}

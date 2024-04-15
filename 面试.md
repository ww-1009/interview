# 面向过程 VS 面向对象

## 面向过程

以<font color='red'>事件</font>为中心的编程思想

**优点：**

* 流程化使得编程任务明确，在开发之前基本考虑了实现方式和最终结果，具体步骤清楚，便于节点分析。
* 效率高，面向过程强调代码的短小精悍，善于结合数据结构来开发高效率的程序。

**缺点：**

* 需要深入的思考，耗费精力。
* 代码重用性低，扩展能力差，后期维护难度比较大。

## 面向对象

以<font color='red'>对象</font>为中心的编程思想，把要解决的问题分解成各个对象，建立对象的目的不是为了完成一个步骤，而是<font color='red'>为了描叙某个对象在整个解决问题的步骤中的属性和行为</font>。

**优点：**

* 结构清晰，程序是模块化和结构化；
* <font color='red'>易扩展，代码重用率高;可继承，可覆盖；易维护</font>
  
**缺点：**

* 开销大，当要修改对象内部时，对象的属性不允许外部直接存取；
* 性能低，由于面向更高的逻辑抽象层

# Java中实现接口与继承的区别

## 接口

接口一般是使用interface来定义的,分为接口的声明和接口体，其中接口体由常量定义和方法定义两部分组成

**基本格式**

```java
[修饰符] interface 接口名 [extends 父接口名列表]{
    [public] [static] [final] 常量;
    [public] [abstract] 方法;
}
```

**实现接口的代码**

```java
public class ButtonListener implements ActionListener {

}
```

**特点**

* <font color='red'>一个类能实现多个接口</font>
* 接口的数据成员默认为static和final
* 接口只提供一种形式，并<font color='red'>不提供实施的细节</font>
  
## 继承

它是一种子类继承父类的特征和行为，使得子类对象（实例）具有父亲的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

**基本格式**

```java
public class UNStudent extends Student{}

public class 子类(派生类) extends 父类(基类) implements 接口，接口，，{}
```

**特点**

* 子类拥有父类非private的属性、方法
* 子类可以拥有自己的属性和方法，即子类可以就自己的功能需求对父类进行扩展
* 子类可以用自己的方式实现父类的方法
* <font color='red'>一个子类只能继承一个父类</font>

**区别**

* 不同的关键字，即实现接口（implements），继承（extends）
* <font color='red'>只能是单继承，但是实现接口可以有多个</font>
* 在接口中只能定义全局变量和抽象方法，而在继承中可以定义属性方法，变量，常量的等等
* 当某个接口被实现的时候，<font color='red'>在类中一定要用接口中的抽象方法</font>，而继承中子类能随意调用父类的属性和方法

# [工厂模式](https://juejin.cn/post/6989591565295419400)

[23 种设计模式详解](https://blog.csdn.net/A1342772/article/details/91349142)

**模式设计意图**

工厂模式将复杂的对象创建工作隐藏起来，而仅仅暴露出一个接口供客户使用，具体的创建工作由工厂管理而对用户封装，将对象的创建和使用分离开来，降低耦合度，便于管理，能够很好的支持变化。

将对象的创建和使用分离开来，所有的创建工作由工厂管理，工厂可以管理类并适应相应业务变化，而无须对客户暴露。

## 简单工厂模式(静态工厂模式)

**设计**

定义一个创建对象的接口，创建一个子类通过传入的参数决定创建哪一种实例。

**适用场景**

* 工厂类负责生产的对象较少。
* 一个类不确定它所必须创建的对象的类的时候，具备不确定性。
* 客户知道需要传入工厂的参数而并不关心具体的类创建逻辑。

**代码实例**

```java
'''假设我们具有如下需求：一家游戏厂商开发了A、B、C三种游戏，某个测试者被要求试玩相应游戏。'''
//定义游戏接口
interface Game {
    public void play();
}

//A游戏
class GameA implements Game{
    public void play(){System.out.println("Playing GameA");};
}

//B游戏
class GameB implements Game{
    public void play(){System.out.println("Playing GameB");};
}

//C游戏
class GameC implements Game{
    public void play(){System.out.println("Playing GameC");};
}

class GameFactory {
    public Game createGame(char type) {
        switch (type) {
            case 'A':
                return new GameA();

            case 'B':
                return new GameB();

            case 'C':
                return new GameC();
        }
        return null;
    }
}

//测试者(客户)类
class Tester {
    private char type;
    public Tester(char type) {
        this.type = type;
    }
    public void testGame() {
        GameFactory gameFactory = new GameFactory();
        //通过简单工厂获取游戏实例
        Game game = gameFactory.createGame(type);
        //试玩游戏
        game.play();
    }

}

//代码测试
public class Test {
    public static void main(String[] args) {
        //要求测试者试玩游戏A
        Tester tester = new Tester('A');
        tester.testGame();
    }
}
```

**UML图**
![Lapland](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e4537bfec1945c095b243d7e7e7f87e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp "Lapland")

**总结**

优点：简单工厂模式结构简单，易与使用，当对象实例较少时可以考虑。

缺点：适用于较为简单的场景，当增加或减少实例时，我们必须对工厂进行较为大量的修改

## 工厂方法模式

**设计**

定义一个创建对象的接口，由子类决定实例化哪一个类，工厂方法将类的实例化推迟到子类实现。

**适用场景**

* 一个类不确定它所必须创建的对象的类的时候，具备不确定性。
  
* 你期望获得较高的扩展性。
  
* 当一个类希望由它的子类来指定它所创建的对象。
  
* 当类将创建对象的职责委托给多个帮忙子类的中的某一个，并且客户知道将要使用哪一个帮忙子类。

**代码实例**

假设我们同样具有需求：一家游戏厂商开发了A、B、C三种游戏，测试者被要求试玩相应游戏。

```java
'''假设我们具有如下需求：一家游戏厂商开发了A、B、C三种游戏，某个测试者被要求试玩相应游戏。'''
//定义游戏接口
interface Game {
    public void play();
}

//A游戏
class GameA implements Game{
    public void play(){System.out.println("Playing GameA");};
}

//B游戏
class GameB implements Game{
    public void play(){System.out.println("Playing GameB");};
}

//C游戏
class GameC implements Game{
    public void play(){System.out.println("Playing GameC");};
}

//定义工厂(父类)
interface GameFactory {
    Game createGame();
}

//帮忙子类，游戏A工厂
class GameAFactory implements GameFactory {
    @Override
    public Game createGame() {
        return new GameA();
    }
}

//帮忙子类，游戏B工厂
class GameBFactory implements GameFactory {
    @Override
    public Game createGame() {
        return new GameB();
    }
}

//帮忙子类，游戏C工厂
class GameCFactory implements GameFactory {
    @Override
    public Game createGame() {
        return new GameC();
    }
}

//测试者(客户)类
class Tester {
    private GameFactory gameFactory;

    public Tester(GameFactory gameFactory) {
        this.gameFactory = gameFactory;
    }

    public void testGame() {
        //通过工厂获取游戏实例
        Game game = gameFactory.createGame();
        //试玩游戏
        game.play();
    }

}

//代码测试
public class Test {
    public static void main(String[] args) {
        //要求测试者1试玩游戏A
        GameFactory gameFactory = new GameAFactory();
        Tester tester1 = new Tester(gameFactory);
        tester1.testGame();
        
         //要求测试者2也试玩游戏A
        Tester tester2 = new Tester(gameFactory);
        tester2.testGame();
        
        //... 测试者1000也试玩游戏A
    }
}
```

**UML图**

![Lapland](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/405e03870a5a4656926fafd03eb2626c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp "Lapland")

**总结**

优点：工厂方法模式具有高扩展性，如果后续想要增加类时，直接编写一个新的帮忙子类即可

缺点：当客户新建一个产品时，就不得不创建一个产品工厂，增加了代码量，而且对于父类接口难以进行修改

## 抽象方法模式

**设计**

提供一个接口以创建一系列相关或互相依赖的对象，将类的实例化延迟到子类。

**适用场景**

* 代码需要与多个不同系列的相关产品交互， 但是由于无法提前获取相关信息，或者出于对未来扩展性的考虑， 你不希望代码基于产品的具体类进行构建而仅希望显示创建它们的接口。
* 需要创建的对象是一系列相互关联或相互依赖的产品族。
* 当一系列中的产品被设计为一起使用时，在一个应用中仅使用同一个系列中的对象。

**代码实例**

```java
'''
假设具有需求如下：一家品牌鞋垫可以生产大号尺寸的鞋子和对应鞋垫和小号尺寸的鞋子和对应鞋垫，
现在有一个客户想要购买一双鞋子穿。
'''
//定义鞋子接口
interface Shoe {
    void printShone();
}

//定义鞋垫接口
interface Insole {
    void printInsole();
}

//具体产品，大号鞋子
class LargeShoes implements Shoe {
    @Override
    public void printShone() {
        System.out.println("大号鞋子");
    }
}

//具体产品，大号鞋垫
class LargeInsole implements Insole {
    @Override
    public void printInsole() {
        System.out.println("大号鞋垫");
    }
}

//具体产品，小号鞋子
class SmallShoes implements Shoe {
    @Override
    public void printShone() {
        System.out.println("小号鞋子");
    }
}

//具体产品，小号鞋垫
class SmallInsole implements Insole {
    @Override
    public void printInsole() {
        System.out.println("小号鞋垫");
    }
}

//定义完整鞋子工厂接口，一个完整的鞋子由鞋子和鞋垫组成
interface CompleteShoeFactory {
    Shoe createShoe();
    Insole createInsole();
}

//大型鞋子工厂，生产配套的大号鞋子和鞋垫
class CompleteLargeShoeFactory implements CompleteShoeFactory {
    @Override
    public Shoe createShoe() {
        return new LargeShoes();
    }

    @Override
    public Insole createInsole() {
        return new LargeInsole();
    }
}

//小号鞋子工厂，生产配套的小号鞋子和鞋垫
class CompleteSmallShoeFactory implements CompleteShoeFactory {
    @Override
    public Shoe createShoe() {
        return new SmallShoes();
    }

    @Override
    public Insole createInsole() {
        return new SmallInsole();
    }
}

//客户类，购买鞋子
class Customer {
    private CompleteShoeFactory factory;

    public Customer(CompleteShoeFactory factory) {
        this.factory = factory;
    }

    //购买完整的鞋
    public void buyCompleteShoe() {
        Shoe myShoe = factory.createShoe();
        myShoe.printShone();

        Insole myInsole = factory.createInsole();
        myInsole.printInsole();

        System.out.println("我已经买了配套产品，终于有鞋穿了！");
    }
}

//代码测试类
public class Test {
    public static void main(String[] args) {
        //购买大号鞋子
        //这里通常使用单例模式生成工厂
        CompleteShoeFactory factory = new CompleteLargeShoeFactory();
        Customer customer = new Customer(factory);
        customer.buyCompleteShoe();
    }
}
```

具体工厂类通常使用单例设计模式创建，由于一个具体工厂类在一个应用中仅出现一次，那么变更产品系列将十分简单，我们仅需要修改一行代码——创建工厂时的代码，这具有极大的灵活性。

**UML图**

![Lapland](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b8e5ebd328f403dbb84ab0a87c5abdc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp "Lapland")

**总结**

抽象工厂模式用于创建的对象是一系列相互关联或相互依赖的产品族，易于交换产品系列，能够较好的应对变化。因为一个对象的产品被设计成一起工作，因此也利于维护产品的一致性，如果采用简单工厂模式，那么得创建四个工厂——大鞋子工厂、大鞋垫工厂、小鞋子工厂和鞋垫工厂，引入的类也增多。

缺点：当工厂接口的功能越来越多时，它会变得越来越笨重，因为所有的子类都必须要去实现这里接口，这就要保证不同系列的产品种类都是一致的，此外，后续想要增加新种类或者删减种类，都不得不对所有子类做出更改。
---
{"dg-publish":true,"permalink":"/design-pattern/strategy-pattern/"}
---


在策略模式（Strategy Pattern）中一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

在策略模式定义了一系列算法或策略，并将每个算法封装在独立的类中，使得它们可以互相替换。通过使用策略模式，可以在运行时根据需要选择不同的算法，而不需要修改客户端代码。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。

## 介绍

**意图：** 定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。

**主要解决：** 在有多种算法相似的情况下，使用 if...else 所带来的复杂和难以维护。

**何时使用：** 一个系统有许多许多类，而区分它们的只是他们直接的行为。

**如何解决：** 将这些算法封装成一个一个的类，任意地替换。

**关键代码：** 实现同一个接口。

**应用实例：** 1、诸葛亮的锦囊妙计，每一个锦囊就是一个策略。 2、旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略。 3、JAVA AWT 中的 LayoutManager。

**优点：** 1、算法可以自由切换。 2、避免使用多重条件判断。 3、扩展性良好。

**缺点：** 1、策略类会增多。 2、所有策略类都需要对外暴露。

**使用场景：** 1、如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 2、一个系统需要动态地在几种算法中选择一种。 3、如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。

**注意事项：**如果一个系统的策略多于四个，就需要考虑使用混合模式，解决策略类膨胀的问题。

**策略模式包含以下几个核心角色：**

- 环境（Context）：维护一个对策略对象的引用，负责将客户端请求委派给具体的策略对象执行。环境类可以通过依赖注入、简单工厂等方式来获取具体策略对象。
- 抽象策略（Abstract Strategy）：定义了策略对象的公共接口或抽象类，规定了具体策略类必须实现的方法。
- 具体策略（Concrete Strategy）：实现了抽象策略定义的接口或抽象类，包含了具体的算法实现。

策略模式通过将算法与使用算法的代码解耦，提供了一种动态选择不同算法的方法。客户端代码不需要知道具体的算法细节，而是通过调用环境类来使用所选择的策略。

![](https://pic.imgdb.cn/item/658e4e81c458853aef12e6fc.jpg)
Strategy接口
```Java
public interface Strategy { 
	public int doOperation(int num1, int num2);
}
```

具体strategy实现
```Java
public class OperationAdd implements Strategy{  
    @Override  
    public int doOperation(int num1, int num2) {  
        return num1 + num2;  
    }  
}

public class OperationSubtract implements Strategy{  
    @Override  
    public int doOperation(int num1, int num2) {  
        return num1 - num2;  
    }  
}

public class OperationMultiply implements Strategy{  
    @Override  
    public int doOperation(int num1, int num2) {  
        return num1 * num2;  
    }  
}
```

创建context
```Java
public class Context {  
    private Strategy strategy;  
  
    public Context(Strategy strategy){  
        this.strategy = strategy;  
    }  
  
    public int executeStrategy(int num1, int num2){  
        return strategy.doOperation(num1, num2);  
    }  
}
```
结合Spring实现
```Java
public class Context{
	@Autowired
	private List<Strategy> strategyList;
	private Map<StrategyEnum,Strategy> strategyMap;

	@PostConstruct
	public void init(){
		//list->map
	}
	public int executeStrategy(StrategyEnum strategyEnum,int num1,int num2){
		Strategy strategy = strategyMap.get(strategyEnum);
		return strategy.doOperation(num1, num2)
	}
}
```

执行策略
```Java
public class StrategyPatternDemo {  
    public static void main(String[] args) {  
        Context context = new Context(new OperationAdd());  
        System.out.println("10 + 5 = " + context.executeStrategy(10, 5));  
  
        context = new Context(new OperationSubtract());  
        System.out.println("10 - 5 = " + context.executeStrategy(10, 5));  
  
        context = new Context(new OperationMultiply());  
        System.out.println("10 * 5 = " + context.executeStrategy(10, 5));  
    }  
}
```
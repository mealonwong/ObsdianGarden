---
{"dg-publish":true,"permalink":"/spring/spring/"}
---


## 什么是事务传播行为？

事务传播行为被用来描述由一个事务传播行为修饰的方法被嵌套进另一个方法时事务如何传播。
用伪代码说明：
```Java
public void methodA(){  
    methodB();  
    //doSomething  
 }  
  
 @Transaction(Propagation=XXX)  
 public void methodB(){  
    //doSomething  
 }
```
代码中`methodA()`方法嵌套调用了`methodB()`方法，`methodB()`的事务传播行为由`@Transaction(Propagation=XXX)`设置决定。这里需要注意的是`methodA()`并没有开启事务，某一个事务传播行为修饰的方法并不是必须要在开启事务的外围方法中调用。

## Spring中事务的七种传播行为

   | 事务传播行为类型 | 说明 |
   | ------ | ------ |
   | PROPAGATION_REQUIRED | 如果当前没有事务，就新建一个事务；如果当前存在事务，就加入到当前事务中。最常用|
   | PROPAGATION_SUPPORTS| 支持当前事务，如果当前没有事务，就以非事务形式运行
   | PROPAGATION_MANDATORY |  使用当前事务，如果当前不存在事务，则抛出异常
   | PROPAGATION_REQUIRES_NEW |  新建事务，如果当前存在事务，则将当前事务挂起
   | PROPAGATION_NOT_SUPPORTED |  以非事务方式运行，如果当前存在事务，则挂起当前事务
   | PROPAGATION_NEVER|  以非事务方式运行，如果当前存在事务，则抛出异常
   | PROPAGATION_NESTED|  如果当前存在事务，则在嵌套事务内执行。如果当前不存在事务，则执行与PROPAGATION_REQUIRED类似
   
   
注意点：
PROPAGATION_REQUIRED：
- **在外围方法未开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**
- **在外围方法开启事务的情况下`Propagation.REQUIRED`修饰的内部方法会加入到外围方法的事务中，所有`Propagation.REQUIRED`修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。**

PROPAGATION_REQUIRES_NEW：
- **在外围方法未开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。**
- **在外围方法开启事务的情况下`Propagation.REQUIRES_NEW`修饰的内部方法依然会单独开启独立事务，且与外部方法事务也独立，内部方法之间、内部方法和外部方法事务均相互独立，互不干扰。**

PROPAGATION_NESTED：
- **在外围方法未开启事务的情况下`Propagation.NESTED`和`Propagation.REQUIRED`作用相同，修饰的内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰。**
- **在外围方法开启事务的情况下`Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外围主事务回滚，子事务一定回滚，而内部子事务可以单独回滚而不影响外围主事务和其他子事务**


## 实战举例

假设我们有一个注册的方法，方法中调用添加积分的方法，如果我们希望添加积分不会影响注册流程（即添加积分执行失败回滚不能使注册方法也回滚），我们会这样写：

```Java
   @Service  
   public class UserServiceImpl implements UserService {  
  
        @Transactional  
        public void register(User user){  
  
            try {  
                membershipPointService.addPoint(Point point);  
            } catch (Exception e) {  
               //省略...  
            }  
            //省略...  
        }  
        //省略...  
   }
```

我们还规定注册失败要影响`addPoint()`方法（注册方法回滚添加积分方法也需要回滚），那么`addPoint()`方法就需要这样实现：

```Java
   @Service  
   public class MembershipPointServiceImpl implements MembershipPointService{  
  
        @Transactional(propagation = Propagation.NESTED)  
        public void addPoint(Point point){  
  
            try {  
                recordService.addRecord(Record record);  
            } catch (Exception e) {  
               //省略...  
            }  
            //省略...  
        }  
        //省略...  
   }
```

我们注意到了在`addPoint()`中还调用了`addRecord()`方法，这个方法用来记录日志。他的实现如下：

```Java
   public class RecordServiceImpl implements RecordService{  
  
        @Transactional(propagation = Propagation.NOT_SUPPORTED)  
        public void addRecord(Record record){  
  
  
            //省略...  
        }  
        //省略...  
   }
```

我们注意到`addRecord()`方法中`propagation = Propagation.NOT_SUPPORTED`，因为对于日志无所谓精确，可以多一条也可以少一条，所以`addRecord()`方法本身和外围`addPoint()`方法抛出异常都不会使`addRecord()`方法回滚，并且`addRecord()`方法抛出异常也不会影响外围`addPoint()`方法的执行。

通过这个例子相信大家对事务传播行为的使用有了更加直观的认识，通过各种属性的组合确实能让我们的业务实现更加灵活多样。


原文链接：
[Spring事务传播行为](https://mp.weixin.qq.com/s/IglQITCkmx7Lpz60QOW7HA)
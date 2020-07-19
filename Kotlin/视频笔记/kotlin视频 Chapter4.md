### 扩展方法
扩展函数允许我们在不修改已有类的前提下，给它增加新的方法。例如：
```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```
在例子中，类型`MutableList<Int>`被称为 **接收者类型**，也就是被扩展的类型作为扩展函数的前缀。 `this`对应的是这个类型所创建的**接收者对象**，当在函数中调用接收者对象的`property`时`this`可以省略。
现在就可以对任意的 `MutableList<Int>` 对象调用该函数：

```kotlin
val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // “swap()”内部的“this”会保存“list”的值
```
#### 扩展属性

与函数类似，Kotlin 支持扩展属性：
> property = backing field + setter + getter

```kotlin
class PoorGuy {
  var pocket: Double
}
//扩展属性
var PoorGuy.moneyLeft: Double 
	get() {
    return this.pocket //必须是公共字段
    //或者 return 0
    //扩展属性没有field字段，所以只能返回类里原有的属性或者固定值
  }
	set(value) {
    pocket = value //同样需要是公共字段
  }
```

注意：由于扩展没有实际的将成员插入类中，因此对扩展属性来说`field`是无效的。这就是为什么**扩展属性不能有初始化器**。他们的行为只能由显式提供的 getters/setters 定义。
例如 `val House.number = 1` 是错误的，因为扩展属性不能有初始化器

同样没有`backing field`的情况还出现在接口定义中：

```kotlin
interface Guy {
  var moneyLeft: Double
  	get() {  //Property in an interface cannot have a backing field
      return 0.0
    }
  	set(value) {
      
    }
}
```

接口中是没有状态的，只有行为。

#### 扩展方法的类型

```kotlin
fun String.times(count: Int): String {}
引用 String::times
类型 (String, Int) -> String

绑定了Receiver的：
引用 "*"::times
类型 (Int) -> String
```

### 空类型安全

```kotlin
var nonNull: String = "Hello"
//nonNull = null  不可以 不可空类型,不能赋值为null
val length = nonNull.length //一定不为空的
```

```kotlin
var nullable: String? = "Hello" //声明为可空类型
nullable = null
val length = nullable.length //不可以 可能会为空
val length = nullable!!.length //用!!强转类型为不可空类型
val length = nullable?.length//用?使用安全访问 length就是Int?可空类型
val length: Int = nullable?.length ?: 0 //用 ?:(elvis运算符) 如果前面的表达式为空，返回后面的值
```

#### 空类型的继承关系

String 是 String? 的一个子类
> 里氏替换原则：任何基类可以出现的地方，子类一定可以出现
#### 平台类型

平台类型存在于Java、JavaScript、Native中，但是没有加空类型判断，需要自己去控制空类型安全，所以在kotlin中使用Java代码时要注意空指针。

```Java
public class Person {
  public String getTitle() {
    return null;
  }
}
```

```kotlin
val person = Person()
val title = person.title //title的类型是String! 
```

String!是一种平台类型，不能确定是否为空的类型，需要自己确认。

### 智能类型转换

#### 用法

```java
public interface Kotliner{ }
public class Person implements Kotliner {
  public final String name;
  public final int age;
  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
```

```java
Kotliner kotliner = new Person("benny", 20);
if ( kotliner instanceof Person ) {
  System.out.println( ((Person)kotliner).name );
}
```

Java中如果判断了类型之后还需要强转类型来使用方法和属性。

```kotlin
val kotliner: Kotliner = Person("benny", 20)
if ( kotliner is Person ) { //Kotlin中is等同于Java中的instanceof
  //as--转换类型
  println( (kotliner as Person).name )//1
  println( kotliner.name )//2
}
```

Kotlin中注释2的书写也是可以的，因为Kotlin可以智能类型转换，编译器会自动转换类型。

#### 作用范围

```kotlin
var value: String? = null
value = "benny"             //value: String? 
if ( value != null ) {      
  println(value.length)     //value: String
}
                            //value: String?
```

#### 不支持智能转换的情况

```kotlin
var tag: String? = null
fun main() {
  if ( tag!=null ) {
    println(tag.length)     //不可以
  }
}
```

写在方法外的对象，任何方法都可以调用，可能会出现多线程操作这样的公共的对象，其他线程可能对它做修改，所以不支持智能转换。

#### 类型的安全转换

```kotlin
val kotliner: Kotliner = Person("benny", 20)
if ( kotliner is Person ) { 
  println( (kotliner as? Person)?.name )  
  // as? 安全转换，失败返回null
}
```

#### 建议
+ 尽可能使用`val`来声明不可变引用，让程序的含义更加清晰确定。
+ 尽可能减少函数对外部变量的访问，也为函数式编程提供基础。
+ 必要时创建局部变量指向外部变量，避免因它变化引起程序错误。


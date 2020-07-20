### 常量和变量

#### 变量

```java
int a = 3;
a = 2;
```

```kotlin
var a = 3;
a = 2;
```

#### 只读变量
Java: `final int b = 3;`
Kotlin: `val b = 3`

需要注意的是，两种方法在定义**局部变量**的时候都是作为常量的，但是当`val`用于在类中声明一个**属性**的时候就不能成为常量，因为可能返回的值不定，例如：

```kotlin
class X {
  val b: Int
  	get() {
      return (Math.random()*100).toInt()
    }
}
```

#### 常量值

Java: `static final int b = 3;`
Kotlin: `const val b = 3`  只能定义在全局范围；只能修饰基本类型；必须立即用字面量初始化

#### 常量引用

```kotlin
val person = Person(18, "Benny")  //堆上创建对象
person.age = 19 //对象改变但引用没变
```

#### 编译期和运行时常量

`const val b = 3`编译时即可确定常量的值，并用值替换掉用处。

`val c: Int` 运行时才能确定值，调用处通过引用获取值。

```kotlin
val c: Int
if (a == 3) 
	c = 4
else 
	c = 5
```

### 分支表达式

#### if./.else

在Java中if..else..是语句不是表达式，但是Kotlin中if..else..就是表达式
Java: `c = a==3 ? 4 : 5;`
Kotlin: `c = if(a==3) 4 else 5`

#### when...

相当于Java中的switch语句

```java
switch(a) {
  case 0: c=5; break;
  case 1: c=100; break;
  default: c=20;
}
```

```kotlin
when(a) {
  0 -> c=5
  1 -> c=100
  else -> c=20
}

//同样，when..也可以是表达式
c = when(a) {
  0 -> 5
  1 -> 100
  else -> 20
}
```
可以不填when后的括号内容，判断条件转移到分支进行判断
```kotlin
var x: Any = ...
when { //条件转移到分支
  x is String -> c=x.length  //x智能类型转换
  x == 1 -> c=100
  else -> c=20
}
//同样也可以写成表达式：
c = when {
  x is String -> x.length
  x == 1 -> 100
  else -> 20
}
```
when后面的括号里面还可以做赋值操作
```kotlin
c = when(val input = readLine()) {
  null -> 0
  else -> input.length //智能转换类型为不可空类型
}
```

#### try...catch

```java
try {
  c = a / b;
} catch (Exception e) {
  e.printStackTrace();
  c = 0;
}
```

```kotlin
try {
  c = a / b
} catch (e: Exception) {
  e.printStackTrace()
  c = 0
}
//同样，可以写成表达式格式
c = try {
  a / b
} catch (e: Exception) {
  e.printStackTrace()
  0
}
```

> Java两种异常类型: 受检异常(checked exception)和非受检异常(unchecked exception)
>    1.Error和RuntimeException及其子类都是非受检异常(unchecked exception);
>    2.其余的异常Exception都是受检异常(checked exception)。
>这两种异常在作用上没有差别,唯一差别在于在编译时编译器会检查受检异常,
>所以受检异常需要try catch捕获来避免编译错误,而非受检异常不需要!
>
>可见受检异常(Checked Exceptions)使用比较麻烦,争议非常大,可能会导致java API变得很复杂,
>程序跟异常检查代码混杂在一起,这仅仅是为了通过编译器的编译,
>许多人批评Java的受检异常,认为受检异常(Checked Exception)是软件工程中一次失败的试验!

Kotlin**没有受检异常**，但是不代表不需要处理异常

### 运算符与中缀表达式

#### 运算符
+ Kotlin支持运算符重载
+ 运算符的范围仅限[官方指定的符号](https://kotlinlang.org/docs/reference/operator-overloading.html)

##### == 与 equals

`"Hello" == "World"`   ->   `"Hello".equals("World")`

##### + 与 plus

`2+3`  ->  `2.plus(3)`

##### in 与 contains
```kotlin
val list = listOf(1,2,3,4)

2 in list -> list.contains(2)
```
##### [] 与 get
```kotlin
val map = mapOf(
	"Hello" to 2,
	"World" to 3
)

val value = map["Hello"]  ->  val value = map.get("Hello")
```

##### [] 与 set

```kotlin
val map = mapOf(
	"Hello" to 2,
	"World" to 3
)

map["World"] = 4  ->  map.set("World", 4)
```

##### > 与 compareTo

`2 > 3`  ->   `2.compareTo(3) > 0`

##### () 与 invoke

```kotlin
val func = fun() { println("Hello") }

func()  ->  func.invoke()
```

#### 重载运算符

```kotlin
class Complex(var real: Double, var image: Double) {
    override fun toString() = "$real + ${image}i"
}

operator fun Complex.plus(other: Complex): Complex {
    return Complex(this.real + other.real, this.image + other.image)
}

operator fun Complex.plus(other: Double): Complex {
    return Complex(this.real + other, this.image)
}

operator fun Complex.minus(other: Complex): Complex {
    return Complex(this.real - other.real, this.image - other.image)
}

operator fun Complex.plus(other: Int): Complex {
    return Complex(this.real + other, this.image)
}

operator fun Complex.get(index: Int):Double = when (index) {
    0 -> this.real
    1 -> this.image
    else -> throw IndexOutOfBoundsException()
}

fun main() {
    val c1 = Complex(3.0, 4.0)
    val c2 = Complex(2.0, 2.0)

    println(c1+2.0)
    println(c1+c2)
    println(c1+3)
    println(c1 - c2)

    println(c1[0])
    println(c1[1])
    println(c1[2])
}
```

#### 中缀表达式

```kotlin
2 to 3    ->    2.to(3)
infix fun<A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
//只有一个参数 有infix标识 可以写成中缀表达式
```

```kotlin
println("HelloWold" rotate 5)

infix fun String.rotate(count: Int): String {
  val index = count%length
  return this.substring(index) + this.substring(0, index)
}
```

### Lambda 表达式

#### 匿名函数

```kotlin
//一个普通函数
fun func() { println("Hello") }  //此时func是函数名
//写成匿名函数
fun() { println("Hello") }
//匿名函数的传递
val func = fun() { println("Hello") }  //此时的func是变量名
//使用
func()
//匿名函数的类型和普通函数相同
val func: () -> Unit = fun() { println("Hello") }
```

#### Lambda表达式的定义

```java
Runnable lambda = () -> { System.out.println("Hello"); }
//lambda 类型 实际上是一个只有一个抽象方法的接口
```

```kotlin
val lambda = { println("Hello") }
//没有参数的函数可以不用写()
val lambda: () -> Unit = { println("Hello") }
//lambda表达式的返回值是函数的最后一行
```

#### Lambda 表达式的参数省略形式

```kotlin
val f1: Function1<Int, Unit> = { p -> Int
		println(p)
}

//只有一个参数，简化
val f1: Function1<Int, Unit> = {
		println(it)
}
```
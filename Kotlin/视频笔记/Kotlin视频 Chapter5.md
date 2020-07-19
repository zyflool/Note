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




### 高阶函数

**参数类型**包含函数类型或**返回值类型**位函数类型的函数为**高阶函数**。

```kotlin
fun needsFunction(block:() -> Unit) {
  block()
}

fun returnFunction(): () -> Long { 
  return {System.currentTimeMillis()}
}
```

#### 常见的高阶函数

```kotlin
inline fun IntArray.forEach(action: (Int) -> Unit): Unit {
  for (element in this) 
  	action(element)
}

inline fun <R> IntArray.map(transform: (Int) -> R): List<R> {
  return mapTo(ArrayList<R>(size), transform)
}
```

#### 高阶函数的调用

```kotlin
intArray.forEach(::println)

intArray.forEach( {
	println("Hello $it")
})

//函数类型作为最后一个参数，可以移到括号外面
intArray.forEach() {
	println("Hello $it")
}

//只有一个Lambda表达式作为参数，可以省略小括号
intArray.forEach {
	println("Hello $it")
}
```

### 内联函数

```kotlin
val ints = intArrayOf(1,2,3,4)
ints.forEach {
  println("Hello $it")
}

//内联函数
inline fun IntArray.forEach(action: (Int) -> Unit): Unit {
  for (element in this) 
  	action(element)
}

//实际上的调用
val ints = intArrayOf(1,2,3,4)
for (element in ints) {
  println("Hello $element")
}
```

高阶函数与内联函数联合使用：

+ 函数本身被内联到调用处
+ 函数的函数参数被内联到调用处

#### 内联高阶函数的return

```kotlin
val ints = intArrayOf(1,2,3,4)
ints.forEach {
  if ( it == 3 ) return@forEach //跳出这一次的内联函数的调用
  println("Hello $it")
}
```

#### non-local return

```kotlin
inline fun nonLocalReturn(block: () -> Unit) {
  block()
}

nonLocalReturn {
  return   //从外部函数跳出
}
```


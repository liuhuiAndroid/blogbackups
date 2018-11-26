### 一、HelloWorld 编写
1. 在intellij上安装kotlin插件
2. 创建一个gradle工程并勾选上kotlin 
```
fun main(args: Array<String>) {
    println("Hello World!")
}
```
### 二、数据类型
1. Boolean类型
```
val aBoolean: Boolean = true
val anotherBoolean: Boolean = false
```
2. Number类型
```
val anInt: Int = 8
val anotherInt: Int = 0xFF
val moreInt: Int = 0b00000011
val maxInt: Int = Int.MAX_VALUE
val minInt: Int = Int.MIN_VALUE

val aLong: Long = 12368172397127391
val another: Long = 123
val maxLong: Long = Long.MAX_VALUE
val minLong: Long = Long.MIN_VALUE

val aFloat: Float = 2.0F
val anotherFloat: Float = 1E3f
val maxFloat: Float = Float.MAX_VALUE
val minFloat: Float = -Float.MAX_VALUE

val aDouble: Double = 3.0
val anotherDouble: Double = 3.1415926
val maxDouble: Double= Double.MAX_VALUE
val minDouble: Double= -Double.MAX_VALUE

val aShort: Short = 127
val maxShort: Short = Short.MAX_VALUE
val minShort: Short = Short.MIN_VALUE

val maxByte: Byte = Byte.MAX_VALUE
val minByte: Byte = Byte.MIN_VALUE
```
3. 拆箱装箱与Char数据类型
```
val aChar: Char = '0'
val bChar: Char = '中'
val cChar: Char = '\u000f'
```
4. 基础数据类型转换与字符串
```
open class Parent
class Child: Parent(){
    fun getName(): String {
        return "Child"
    }
}
fun main(args: Array<String>) {
    val parent: Parent = Parent()

    val child: Child? = parent as? Child
    println(child)

    val string: String? = "Hello"
    if(string != null)
        println(string.length)
}
```
```
val string: String = "HelloWorld"
val fromChars: String = String(charArrayOf('H', 'e','l','l','o','W','o','r','l','d'))

fun main(args: Array<String>) {
    println(string == fromChars)
    println(string === fromChars)

    val arg1: Int = 0
    val arg2: Int = 1
    println("$arg1 + $arg2 = ${arg1 + arg2}")

    //Hello "Trump"
    val sayHello : String = "Hello \"Trump\""
    println(sayHello)

    //$salary
    println("\$salary")

    val rawString: String = """
        \t
        \n
        \$$$ salary
    """
    println(rawString)
}
```
5. Kotlin中类和对象初始化
```
class girl(character: String, appearance: String, voice: String): man(character, appearance, voice)
class boy(character: String, appearance: String, voice: String): man(character, appearance, voice)

open class man(var character: String, var appearance: String, var voice: String){
    init {
        println("new 了一个${this.javaClass.simpleName}, ta性格:$character, 长相:$appearance, 声音:$voice")
    }
}

fun main(args: Array<String>) {
    val aGirl: girl = girl("温柔", "甜美", "动人")
    val aBoy: boy = boy("彪悍", "帅气", "浑厚")
    println(aGirl is man)
}
```
6. 空类型和智能类型转换
```
fun getName(): String? {
    return null
}

fun main(args: Array<String>) {
    val value: String? = "HelloWorld"
    println(value!!.length)

    val name: String = getName()?: return
    println(name.length)
}
```
7. 包
```
import net.println.kotlin.chapter2.test.Test as ts
```
8. 区间
```
val range: IntRange = 0..1024 // [0, 1024]
val range_exclusive: IntRange = 0 until 1024 // [0, 1024) = [0, 1023]
val emptyRange: IntRange = 0..-1

fun main(args: Array<String>) {
    println(emptyRange.isEmpty())
    println(range.contains(50))
    println(50 in range)

    for(i in range_exclusive){
        print("$i, ")
    }
}
```
9. 数组
```
val arrayOfInt: IntArray = intArrayOf(1,3,5,7)
val arrayOfChar: CharArray = charArrayOf('H', 'e','l','l','o','W','o','r','l','d')
val arrayOfString: Array<String> = arrayOf("我", "是", "码农")

fun main(args: Array<String>) {
    println(arrayOfChar.joinToString())
    println(arrayOfInt.slice(1..2))
}
```
10. 例子
```
val FINAL_HELLO_WORLD: String = "Hello"
var helloWorld: String = FINAL_HELLO_WORLD
var nullableHelloWorld: String? = helloWorld
val helloWorldArray: Array<Char> = arrayOf('H', 'e', 'l', 'l', 'o', 'W', 'o', 'r', 'l', 'd')
val helloWorldCharArray: CharArray = charArrayOf('H', 'e', 'l', 'l', 'o', 'W', 'o', 'r', 'l', 'd')
val helloWorldLength: Int = helloWorld.length
val helloWorldLengthLong: Long = helloWorldLength.toLong()
class HelloWorld

fun main(args: Array<String>) {
    println("final hello world: " + FINAL_HELLO_WORLD)
    println("assignable hello world: " + helloWorld)
    println("hello world from nullable type: " + nullableHelloWorld)
    println("hello world from Array: " + helloWorldArray.joinToString(""))
    println("hello world from CharArray: " + String(helloWorldCharArray))
    println("class name hello world: " + HelloWorld::class.java.simpleName)
    println("class name hello world: " + HelloWorld::class.java.name)
    println("part of the class name of HelloWorld: "
            + HelloWorld::class.java.simpleName.slice(0 until helloWorldLength)) // [0, 11)
    println("the length of hello world is : " + helloWorldLength)
    println("the length of hello world is (long): " + helloWorldLengthLong)
}
```
### 三、程序结构
1. 常量与变量
```
//如果在编译期间，属性值就能被确定, 该类属性值使用const 修饰符
const val FINAL_HELLO_WORLD: String = "HelloWorld"
var helloWorld: String = FINAL_HELLO_WORLD
val FINAL_HELLO_CHINA = "HelloChina"
```
2. 函数
```
fun main(args: Array<String>):Unit {
    checkArgs(args)
    val arg1 = args[0].toInt()
    val arg2 = args[1].toInt()
    println("$arg1 + $arg2 = ${sum2(arg1, arg2)}")
    println(int2Long(3))
    println(sum2(1, 3))
    println(sum2)
    println(int2Long)
}

fun checkArgs(args: Array<String>) {
    if (args.size != 2) {
        printUsage()
        System.exit(-1)
    }
}

val sum2 = fun (arg1: Int, arg2: Int): Int{
    return arg1 + arg2
}

val int2Long = fun(x: Int): Long {
    return x.toLong()
}

fun printUsage() {
    println("请传入两个整型参数，例如 1 2")
}
```
3. Lambda表达式
```
val sum = fun (arg1: Int, arg2: Int): Int{
    println("$arg1 + $arg2 = ${arg1 + arg2}")
    return arg1 + arg2
}

val sum = { arg1: Int, arg2: Int ->
    println("$arg1 + $arg2 = ${arg1 + arg2}")
    arg1 + arg2
}
```
4. 类成员
成员方法和成员变量
```
class X{
    var v = 0
        get() {
            println("get v")
            return -1
        }
        set(value) {
            println("set ${value}")
            field = value
        }
}
class A{
    var b = 0
    lateinit var c: String
    lateinit var d: X
    val e: X by lazy {
        println("init X")
        X()
    }
    var cc: String? = null
}
fun main(args: Array<String>) {
    println("start")
    val a = A()
    println("init a")
    println(a.b)
    println(a.e)
    a.d = X()
    println(a.d)
    println(a.cc?.length)
    println(a.c)
}
```
5. 基本运算符
```
class Complex(var real: Double, var imaginary: Double){
    operator fun plus(other: Complex): Complex{
        return Complex(real + other.real, imaginary + other.imaginary)
    }
    operator fun plus(other: Int): Complex{
        return Complex(real + other, imaginary)
    }
    operator fun plus(other: Any): Int{
        return real.toInt()
    }
    operator fun invoke(): Double{
        //Math.hypot() 函数返回它的所有参数的平方和的平方根
        return Math.hypot(real, imaginary)
    }
    override fun toString(): String {
        return "$real + ${imaginary}i"
    }
}
class Book{
    infix fun on(any: Any): Boolean{
        return true
    }
}
class Desk
fun main(args: Array<String>) {
    val complex1 = Complex(5.0, 6.0)
    val complex2 = Complex(1.0, 2.0)
    println(complex1)
    println(complex2)
    val plusComplex = complex1.plus(complex2)
    println(plusComplex)
    val invoke = plusComplex.invoke()
    println(invoke)
    if(Book() on Desk()){
        println("???")
    }
    if("-name" in args){
        println(args[args.indexOf("-name") + 1])
    }
}
```
6. 表达式
中缀表达式
```
```
分支表达式
when表达式
7. 循环语句
8. 异常捕获
9. 具名参数、变长参数、默认参数
10. 小案例：命令行计数

### 参考
[Kotlin 语言中文站](https://www.kotlincn.net/)



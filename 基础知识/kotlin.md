[toc]

## 零. 变量和表达式

 1. 导言部分

    Android官方支持语言。运行在jvm上但是可用脱离虚拟机层。可以编译为在win\mac\ios\android上运行的原生二进制代码，和java遗留代码无缝操作（保守的更新风格）。

    

 2. 变量声明和类型

    ***var指的是一般变量，val指的是只读变量***

    ```
    //编译时常量
    const val a = 5
    fun main(){
    	//类型推断功能可以省略String的类型定义 var b = "hello $a world"
    	var b: String = "hello $a world"
    	println(b)
    }
    ```

    八种内置类型（全部为引用类型，没有基本类型。编译时会转成基本类型但是这不是程序员考虑的）：String\Char\Boolean\Int\Double(注意所有数字类型都是有符号的)\List\Set\Map

    

 3. 查看kotlin字节码

    tools->kotlin->show kotlin bytecode

    

 4. 条件表达

    if/ if-else 同java

    range:

    ```
    //!in：not in ,左闭右开，所以下面这个不会打印
    if(a in 0..5){
    	println("a fits")
    }
    ```

    when（类似switch case，建议使用）：

    ```
    val level = when(a){
    	3-> "small"
    	4-> "middle"
    	5-> "big"
    	else -> {
    		println("exception out of cases")
    	}
    }
    ```

    

 5. string 模板不仅可以引用变量，还可以引用条件表达

    ```
    val flag = false
    var c ="the result is ${
    			if(flag) 
    				"good"
    			else 
    				"not good"
    		} for us"
    ```

    



## 一. 函数和闭包

1. 典型普通函数

   ```
   //当函数不需要返回值，类型省略声明为Unit
   private fun typicalFun(age:Int, flag:Boolean = false): String{
   	
   
   }
   fun main(){
   	println(typicalFun(5, true))
   	// when params are so many that you need naming:
   	println(typicalFun(age = 5, flag = true))
   }
   ```

   

2. TODO函数： 

   内置函数，返回Nothing，***Nothing类型函数会终止程序运行***

   

3. 反引号函数

   使用空格和特殊字符对于函数命名，但是需要反引号括起来

   也可以用于解决同java的保留关键字的冲突

   ```
   fun `1st special fun with keyword int *?!`(){}
   call: `1st special fun with keyword int *?!`()
   //java代码里面有一个方法叫做is(), 但是is在kotlin中是关键字, 怎么调用这个方法：
   JavaObj.`is`()
   ```

   

4. 匿名函数（lambda表达式）

   ```
   val paramInt = 5;
   val paramDuoble = 5;
   val anoFun: (Int, Double) -> String //这个类型定义在没有参数时可以省略掉，有参数时省略需要在后面函数体中定义
   	= { paramInt, paramDouble ->
               val returnStr = "return val for $paramInt"
               returnStr
   	} //匿名函数定义 ={参数->函数体}
   //当只有一个参数，用it定义
   
   fun main(){
   	//内置函数定义中匿名函数类型确定，直接传入函数定义对象。此句需要一个条件参数
   	val total = givenStr.count({letter ->
   			letter == 's'
   	})
   }
   ```

   ```
   var anotheranoFun = {
   	val Char = '2'
   	Char
   } //这是一个匿名函数，不是一个静态代码块
   ```

   

5. 函数的参数是另一个函数

   ```
   fun show(name:String, inner: (String, Int)->String){
   	//directly call lambda in outer fun：
   	println(inner("sample", 6))
   }
   
   //call outer fun directly：
   //lambda参数唯一时，可以省略圆括号；排在最后时可以把它拉到外面去[这种写法在框架写法中较常见，需要了解，容易误解成为定义函数]
   //普通函数引用的话就是两个冒号加函数名字，定义一样 ::FunName
   show("name") {
   	iStr:String, iInt:Int -> 
   		"$iStr $iInt"
   }
   ```

   

6. 内联函数

   预编译中进行复制替换，不需要分配lambda对象而是使用运行时调用的方式，节约内存

   使用inline关键字定义outerFun即可。适用于短小和没有递归的函数（同C++）

   

7. 函数的返回值是另一个函数

   ```
   fun returnFun(iStr:String): (Int) -> String {
       //匿名函数能修改并引用自己作用域之外的变量（即引用定义自身的函数（本例子中是returnFun）中的变量，称为闭包(闭包=lambda=匿名函数)）
   	return {
   		iInt:Int ->
   			"$iStr $iInt"
   	}
   }
   fun main(){
   	var innerFun = returnFun("iStrC")
   	println(innerFun(5))
   }
   ```

   <font color="red">***闭包的最大作用是脱离作用域的限制，消除重名的影响（因为kotlin没有包和类管理）***</font>

   参考java里面的匿名内部类，只定义一个接口，之后实现交给外面填起来。java模式化代码多一些

   

## 二. 安全、异常和字符串

 1. 空问题

    kotlin没有NPE，变量默认不能为空。空类型时需要使用安全调用操作（不推荐用java的if判断了，不符合rx之类的链式调用风格，带来阅读困难）：

    ```
    //为空时必须定义为可空类型
    val str:String? = readLine()
    //1. 安全调用操作符, 当为空时直接跳过调用（不是显示异常）
    res = str?.capitalize()
    
    //2. 使用let函数，使用于空类型分支或复杂操作，推荐
    res = str?.let{
    	if(it.isNotBlank())
    	    it.capitalize()
    	else
    		"default" 
    }
    
    //3. 非空调用操作符，当为空时会抛出异常（不是跳过调用）
    try{
    	res = str!!.capitalize()
    }catch(e: Exception){
    	println(e)
    }
    
    //4. 空合并操作符，为空时使用右值。初学不推荐使用（如java三元表达式一样不推荐）
    res = str ?: "butterfly"
    res = str?.let{ it.capitalize() } ?: "butterfly"
    //str为空时输出butterfly
    ```

    

 2. 异常

    ```
    //自定义异常
    class OwnException:IllegalArgumentException("自定义参数不当")
    //抛出
    number ?: throw OwnException
    
    //先决条件函数 kt标准库提供的内置异常函数 参考手册
    checkNotNull 要求参数为not null
    require 要求参数为true
    requreNotNull 要求参数为not null
    error 要求参数为not null
    assert 要求参数为true
    举例：checkNotNull(number, {"this is an IllegalStateException Message"})
    ```

    

 3. 字符串

    ```
    const val STR = "Jay's dog is stupid"
    fun main{
    	val index = STR.indexOf('\'')
    	//截”Jay“，等价写法 STR.substring(0 until index)
    	val res = STR.substring(0, index)
    	
    	//分割或解构
    	val data: List<String> = STR.split(" ")
    	val (names: String, pet: String, useless:String, adj:String) = 
    		STR.split(" ")
    		
    	//替换 注意闭包是第二个参数
    	val res = STR.replace(Regex([aeiou])){
    		when(it.value){
    			"a" -> "b"
    			"e" -> "f"
    			"i" -> "j"
    			else -> "j"
    		}
    	}
    	
    	//比较
    	if(STR == res){} //内容是否相同
    	if(STR === res){} //引用是否一致
    	
    	//遍历
    	STR.forEach{
    		print("$it *")
    	}
    }
    ```

    

 4. 数字类型的安全转换

    ```
    val int1 = "8".toInt() //num1是一个Int类型
    val cannotConvert = "8.8".toIntOrNull() //cannotConvert是一个Int?类型
    val int2 = 8.888.toInt() //int2 = 8
    val int3 = 8.888.roundToInt() //int3 = 9
    val str = "%.2f".format(8.888)
    ```

    

## 三. 标准库函数、集合

 1. apply (DSL-领域特定语言，暴露接收者的函数和特性，以便使用定义的lambda表达式读取和配置它)

    ```
    val file1  = File("E://1.txt")
    file1.setReadable(true)
    file1.setWritable(true)
    file1.setExecutable(true)
    //apply接受闭包, 用于配置（连续调用）对象的函数，返回配置完成的对象。以上片段等价为
    val file2 = File("E://1.txt").apply{
    	//相关作用域内针对接收者隐式调用
    	setReadable(true)
    	setWritable(true)
    	setExecutable(true)
    }
    
    ```

    

 2. let&also

    let使得变量作用于lambda表达式里面，让it关键字可以引用它。 **apply返回接受者对象，let返回lambda最后一行**。例子前面有，不再列了。

    also也是可以用it引用，但是**和let不同的它是返回接收者对象本身**，比较适合于基于原始对象的链式调用。

    

 3. run

    也是接受闭包配置调用对象，和apply的作用域行为一致（隐式调用，不是用it），但是返回lambda结果（就是最后一行）

    ```
    val res = File("E://1.txt").run{
    	//ret val
    	readText().contains("aim")
    }
    
    //run函数也可以执行函数引用, 需要链式调用结果的时候常用（一个函数就直接调用就好了）
    val res2 = File("E://1.txt").readText().
    	.run(::isLong)
    	.run(::show).
    	.run(::println)
    fun isLong(name: String) = name.length > 10
    fun show(isLong: Boolean): String{
    	return if(isLong){
    		"isLong"
    	}
    	else{
    	    "notLong"
    	}
    }
    ```

    

 4. with

    和run一样，不过是把接受对象放到里面当做参数...基本不用

    ```
    val res = with(File("E://1.txt")){
    	readText().contains("aim")
    }
    ```

    <font color = "red">**run和let是结果作为参数的链式调用，run用本身let用it;**</font>

    <font color = "red">**apply和also是对象本身不断调用的链式调用，apply用本身also用it**</font>

    

 5. takeIf&takeUnless

    直接在对象实例上调用的if语句判断是否继续执行，***传入的lambda表达式返回结果为true时返回接受的对象，否则返回空。***takeUnless完全反过来（false返回对象），辅助语义需求。两个take函数可以避免临时对象赋值判断的麻烦。

    ```
    File("E://1.TXT").takeIf{
    	it.exists() && it.canRead()
    }?.readText()
    ```

    

 6. List&Set

    ```
    val list:List<String> = listOf("1","2","3") //unchangable
    // use get or list[x] to obtain ele.
    print(list.getOrElse(3){"4"})
    print(list.getOrNull(3) ?: "4")
    
    //可以list.toMutableList或者toList()
    val list:MutableList<String> = mutableListOf("1", "2", "3")
    list.add("4") //等价于mutableList += "4"
    list.remove("4")
    list.removeIf{
    	it.contains("4")
    }
    
    //遍历和解构
    for(s in list){} 
    list.forEach{ it... }
    list.forEachIndexed{
    	index, item ->
    	...
    }
    val (str1:String, _:String, str3:String) = list
    
    //Set还是不允许有重复元素
    用setOf创建，elementAt获取，orElse或者orNull以及mutableSetOf和list一样，不列了。List可以转Set(去重)：list.toSet()。List去重之后还是list可以用list.distinct()
    ```

    

 7. Array&Map

    ```
    Array可以转成Java基本数据类型 Int Double Long Short Byte Float Boolean Array
    val intArray:IntArray = intArrayOf(1,2,3)
    val objArray:Array<File> = arrayOf(File(".."),File(".."))
    ```

    ```
    //定义中的to是一个函数而不是关键字，返回Pair类型 mapOf(Pair("jack",20)...)也是可以的
    val map = mapOf("jack" to 20, "jason" to 30, "jim" to 40)
    map["jack"] //map.getValue
    map.getOrElse("tom"){"jack"}; 
    map.getOrDefault("tom", 50)
    
    //遍历
    map.forEach{
    	it.key
    	it.value...//it是个Entry
    }
    //可变
    mmap = mutableMapOf... 
    mmap += "tom" to 50
    mmap.put("tom", 50) //注意getOrPut方法 其他看文档了
    ```

    

## 四. 类和对象引入

 1. 定义类和field

    ```
    public class Player{
    	var name = "jack" 
    	//如果需要自定义get set函数，使用field代表原属性值（看到不一定用到field）
    	var gsdefine = "t m"
    		get = field.capitalize()
    		set(value){
    			field = value.trim()
    		}
    }
    fun main(){
    	var p = Player()
    	p.name = "Jack" 
    	println(p.name) //定义属性后自动生成getset方法（因为不可空类型，保证不空）
    }
    
    ```

    

 2. 构造函数和初始化

    ```
    //Kt的构造函数直接写在类名后面，通常这种参数只用一次，所以推荐用下划线开头。可以定义默认值(跳过的情况下需要具名传参哦)
    public class Player(
    	_name:String,
    	_age:Int = 20,
    	var isNormal: Boolean//直接定义了属性，name和age的定义也可以写到这里面
    ){
    	var name = _name
    		get = field.capitalize()
    		//name搞成只读的了
    		private set(value){}
    	var age = _age
    	
    	//次构造函数，只需要传入部分参数, 返回this定义的本对象，可以初始化某些代码逻辑
    	constructor(name: String) : this(name, age = 0, isNormal = true){
    		this.name = name.toUpperCase()
    	}
    	
    	init{
    		//java的静态代码块在类加载时执行（即使没有创建对象），而kt的初始化代码块在创建对象完成前执行
    		require(age > 0){"age must be positive"}
    	}
    }
    
    fun main(){
    	var p = Player("jack", isNormal = false)
    	val q = Player("newborn")
    }
    ```

    初始化顺序：主构造函数->类级别属性赋值->init块->次构造函数。但是编译顺序是从上至下的...所以使用前还是要在前面定义，和java的属性顺序随意有区别

    使用lateinit var修饰属性，从而使得在对象创建时不需要初始化属性（***但是用之前也不会自动初始化而是需要手动检查，初始化还是要在某个地方自己手动做***）

    使用lazyinit方式的话是***先定义好初始化方式，使用时自动调用这个方式完成初始化***

    注意两者的区别，都是延迟初始化，但是后者显然需要动态配合，前者不涉及这个问题

    ```
    lateinit var eq...
    eq = "define somewhere in set fun" ...
    if(::eq.isInitailzed) {...
    
    var config by lazy{
    	println("loading...")
    	return "config string"
    }...
    //use it:（注意类创建的时候loading不会打印，而使用p.config的时候才会打印出loading）
    println(p.config)
    ```

    

 3. 继承，重载和类型转换

    ```
    //只有open修饰的类或者函数才能继承或重载
    open class Product(val name: String){
    	fun desc() = "Product: $name"
    	open fun load() = "Loading..."
    }
    class LuxuryProduct: Product("Luxury"){
    	override fun load() = "Loading Luxury Product..."
    }
    val p: Product = LuxuryProduct()
    //使用is进行类型检测
    println(p is LuxuryProduct) //false
    println(p is Product) //true
    //使用as进行类型转换，注意和java不同的是转换一次之后不用再转了，属于永久操作
    println((p as LuxuryProduct).desc())
    ```

    

 4. Any是所有类的父类，类似java里面的Object，里面也有equals, hashCode,toString默认实现（但是不在源码里面）

    

 5. 单例对象，匿名类对象，伴生对象

    ```
    //object是一个单例对象
    object ApplicationConfig{
    	init{
    		println("loading")
    	}
    	fun dosth(){
    		...
    	}
    }
    //伴生对象可以把某个对象的初始化同一个类实例捆绑在一起,一个类只能有一个半生对象
    //该对象只有在类初始化时或者load函数被直接调用时才会载入。且该对象不论类对象多少次，只有一个实例存在。可以视作局部单例。
    //伴生对象相当于static（Kt没有该关键字）
    open class ConfigMap{
    	companion object {
    		private const val PATH = ",,,"
    		fun load() = File(PATH).readBytes()
    	}
    
    }
    fun main(){
    	ApplicationConfig.dosth()
    	//匿名类对象
    	val p = object: Player(){
    		override fun load() = "Loading cheap prod"
    	}
    	//使用伴生对象，看作自己的方法或者属性即可
    	ConfigMap.load()
    }
    ```

    

## 五. 嵌套类，数据类，枚举类，密封类，接口

 1. 嵌套类

    ```
    class Player{
    	class Equipment(var name: String){
    		fun show() = println("equipment:$name")
    	}
    	fun battle(){...}
    }
    fun main(){
    	//直接实例化就行，和java有区别
    	Player2.Equipment("sharp knife")
    }
    ```

    

 2. 数据类

    没有函数内容。提供toString的个性化实现（不是打印引用地址了），重写了equals（==的方法）&hashcode

    主构造函数至少有一个参数，不能使用abstract open sealed enum

    ```
    data class Coordinate(var x: Int, var y: Int){
    	val isInBounds = x > 0 && y > 0
    	
    	//数据类的运算符重载
    	operator fun plus(other: Coordinate) = Coordinate(x + other.x, y + other.y)
    	//其他运算符包括： plusAssign(+=) equals(==)...见手册
    }
    //data class提供copy方法使得属性可以变化部分属性
    val c = Coordinate(2,3)
    val cc = c.copy(5)
    //data class提供解构语法(普通类可以使用operator fun component1() = x实现)
    val (x,y) = c
    //重载效果
    val ccc = c + cc
    ```

    

 3. 枚举类

    ```{
    //注意和java不一样需要class哦
    enum class Direction{
    	E,W,S,N
    }
    //可以添加构造函数和函数
    enum class Direction2(private val c:Coordinate){
    	//枚举时一组子类型的闭集，是一种简单的ADT（抽象数据类型）
    	E(Coordinate(1,0)),W(Coordinate(-1,0)),
    	S(Coordinate(0,-1)),N(Coordinate(0,1));
    	fun updateCoordinate(cc: Coordinate) = 
    		Coordinate(c.x+cc.x,c.y+cc.y)
    }
    fun main(){
    	println(Direction.W)
    	println(Direction2.W.updateCoordinate(Coordinate(10,20)))
    }
    ```

    

 4. 密封类

    密封类包括若干个子类，类似枚举类的升级版(理解密封类需要从枚举类开始)，用于内部细节不同的ADT定义：

    ```
    sealed class LicenseStatus{
    	object UnQualified : LicenseStatus(),
    	object Learning :  LicenseStatus()
    	class Qualified(val licenseId: String) :  LicenseStatus()
    }
    class Driver(var status: LicenseStatus){
    	fun checkL(): String{
    		//直接使用.进行引用和判断即可
    		return when(status){
    			is LicenseStatus.UnQualified -> "not complete"
    			is LicenseStatus.Learning -> "completing"
    			is LicenseStatus.Qualified -> "complete as 
    				${(this.status as LicenseStatus.Qualified).licenseId}"
    		}
    	}
    }
    fun main(){
    	val status = LicenseStatus.Qualified("0001")
    	println(Driver(status).checkL())
    }
    ```

    

 5. 接口和抽象类

    ```
    interface Movable{
    	var speed:Int
    	var wheels: Int
    	fun move(movable: Movable): String
    }
    //接口实现，注意变量和函数都要加override
    class Car(_name:String, override var wheels: Int = 4): Movable{
    	override var speed: Int
    	var ownAttr: String
    	override fun move(movable: Movable): String{
    		...
    	}
    }
    ```

    抽象类还是abstact修饰。和java差不多，比接口可以多做的还是可以定义默认的函数实现，也是不能通过abstract class多重继承（只能通过接口多重继承）。不写了

    

## 六. 泛型

1. 泛型类和泛型函数的使用

   ```
   class MagicBox<T>(item: T){
   	var available = false
   	private var subject: T = item
   	
   	fun fetch(): T?{
   		return subject.takeIf{ available }
   	}
   	//在fun以及函数名间可以定义新的泛型类型
   	fun <R> fetch(subModFun: (T) -> R): R?{
   		return subModFun(subject).takeIf{ available }
   	}
   }
   
   class Boy(val name: String, val age: Int)
   class Man(val name: String, val age: Int)
   
   fun main(){
   	val box1 = MagicBox(Boy("jack", 10))
   	box1.available = true
   	box1.fetch()?.run{
   		println("you find $name available")
   	}
   	
   	//将男孩变成男人 (只有一个参数时省略括号，只有一个参数时省略->前的内容参数用it代替)
   	val man: Man? = box1.fetch{
   		Man(it.name, it.age.plus(10))
   	}
   }
   ```

   

2. 可变参数

   ```
   class MagicBox<T: Human>(vararg item: T){ //泛型类型约束（相当于extends）
   	var available = false
   	//可变参数列表 可变参数不仅泛型可用，这里只是用泛型做例子。规范同java
   	private var subject: Array<out T> = item
   	
   	fun fetch(index: Int): T?{
   		return subject[index].takeIf{ available }
   	}
   	//在fun以及函数名间可以定义新的泛型类型
   	fun <R> fetch(index: Int, subModFun: (T) -> R): R?{
   		return subModFun(subject[index]).takeIf{ available }
   	}
   	//重载一个运算符[]（复习一下）
   	operator fun get(index: Int): T? = 
   		subject[index].takeIf{ available }
   }
   fun main(){
   	//使用可变参数，按照规则传入不定数量的参数即可
   	val box1 = MagicBox(Boy("jack", 10), Boy("jackn",20))
   	box1.available = true
   	box1.fetch(1)?.run{
   		println("you find $name available")
   	}
   	//实验重载的get运算符
   	println(box1[1].name)
   }
   ```

   

3. in和out

   out: 协变。将泛型类型作为函数的返回，生产指定泛型对象

   in: 逆变。将泛型类型作为函数的参数，消费指定泛型对象

   什么都不加的时候即做返回又做输出

   协变和逆变用于解决java中泛型类不能继承子类的问题，在***使用协变或者逆变关键字可用使得相关泛型类也可以和泛型类型的继承关系一致变化***

   <font color="red">即父类泛型对象可用赋值给子类泛型对象，用in</font>

   <font color="red">子类泛型对象可用赋值给父类泛型对象，用out</font>

   ```
   //case:
   interface Production<out T>{
   	fun product(): T
   }
   interface Consumer<in T>{
   	fun consume(item: T)
   }
   
   //define out or in, T's class is able to involve in T's extend system
   val production: Production<Food> = FastFoodStore() 
   	//this class Imples Production<FastFood>, FastFood extends Food
   val consumer: Consumer<Burger> = MordernPeople()
   	//this class Imples Consumer<FastFood>, Burger extends FastFood
   ```

   

4. **使用reified修饰使得可用对泛型参数类型进行检查**

   kotlin和java一样具有泛型擦除特质（T的类型在运行时不可知）

   ```
   class MB<T: Human>(){
   	//随机产生一个对象，不是指定类型对象时调用backup对象
   	inline fun <reified T> randomOrBackup(backup: () -> T): T{
   		val items: List<Boy> = listOf(Boy("jack",10), Man("jerry",30))
   		val random: Human = items.shuffled().first()
   		return if(random is T){
   			//此处由于擦除特质默认不能使用if(random is T), 在Java中使用random.javaClass.name反射解决。而kotlin这里使用reified和inline都是为了可用is T，其实逻辑就是把泛型替换掉了，从而保留了具体类型（inline的必要性），减少了使用反射的运行时成本
   			random
   		}else{
   			//不是T的情况，比如T=Boy时shuffled到Man
   			backup()
   		}
   	}
   }
   fun main(){
   	val box1: MB<Man> = MB()
   	val it1 = box1.randomOrBackup{
   		Man("jim", 20)
   	}
   	//it1的内容可能是jerry或者jim
   }
   ```

   

5. 类拓展[极其灵活的框架定义]

   ```
   //类.函数名, 用this代替类对象，用于难以修改类定义的情况
   fun String.addExt(amount: Int = 1) = this  + "!".repeat(amount)
   //可空类型拓展一般需要提供默认值。函数添加infix前缀可用省略点和括号调用，典型的比如map的xx to xx。这里不写了
   fun String?.addExt(amount: Int = 1, default: String = "") 
   	= (this ?: default) + "!".repeat(amount)
   fun Any.easyPrint() = println(this)
   
   //泛型拓展函数 可以定义针对泛型的拓展函数
   fun <R> R.standardPrint(): R{
   	println(this)
   	return this
   }
   //尤其注意泛型拓展函数
   //阅读let apply等的函数定义, 理解泛型拓展函数和前面提及的标准函数的实现
   public inline fun<T,R> T.let(block: (T)->R):R{...}
   （let可用在任意类型调用，返回lambda的执行结果，lambda中用it代替，run/apply等...这些规则的原因。）
   
   //拓展属性，不大常用
   val String.numVowels
   	get() = count{"aeiou".contains(it)} //String的count接受一个lambda，接受每个字符（it），统计lambda返回true的个数
   
   fun main(){
   	"".addExt(3)
   	"u15da".numVowels.easyPrint()
   	"abc".standardPrint().addExt(2).easyPrint()
   }
   ```

   


##  七. 函数式编程（高阶函数链式调用）

 1. map

    ***<font color="red">不可变数据的副本在链上的数据间传递</font>***

    ```
    //map遍历每个元素，返回包含已修改元素的集合，变量名自定义
    //常用于集合修改或者获取属性
    val alpbets = listOf("a","b","c")
    alpbets.map{alpbet -> alpbet.toUpperCase()}.map{uperalp -> " $uperalp "}
    ```

    

 2. flatMap

    ```
    //操作集合的集合，将多个集合的元素合并后返回包含所有元素的单一集合
    //用于集合展开、修改和属性获取
    val res = listOf(
    	listOf(1,2,3),
    	listOf(4,5,6)).flatMap{ it + 1 }
    ```

    

 3. filter

    ```
    //接受predicate函数，为true时添加受检元素，为false时移除受检元素
    //用于集合移除元素
    val res = listOf("jack", "jim", "tom").filter{
    	it.contains("j")
    }
    
    //拿出所有红色水果
    val item = listOf(
    	listOf("red apple","green apple","blue apple")
    	listOf("red pear","green pear")
    	listOf("yellow banana","grown banana")
    }
    val res2 = 
    	item.flatMap{it.filter{it.contains("red")||it.contains("green")}}
    	
    //找素数
    val primesInGivenNums = 
    	numbers.filter{
    		//条件是2-number集合中的元素被number取模，没有一个结果为0
    		(2 until number)
    			.map {number % it}
    			//none函数考察集合中的每个元素，仅在集合中所有元素不满足条件时返回true
    			//当然也可以在map里面用if 为0就返回false
    			.none{it == 0}
    	}
    ```

    

 4. zip

    ```
    val set1 = listOf(1,2,3)
    val set2 = listOf(4,5,6)
    //和filterMap不同的是他返回一个键值对集合，即1 to 4, 2 to 5, 3 to 6
    set1.zip(set2)
    ```

    

 5. folder

    接受一个初始累加器值，根据匿名函数的结果更新集合中的每个值。***相当于递归的map, 最后返回结果值***

    ```
    val res = listOf(1,2,3,4).fold(0){
    	accumulator, number ->
    		println("val is now $accumulate")
    		accumulator + (number * 3)
    }
    //res是3+2*3+3*3+4*3，它是一个Int类型
    ```

    

## 八 序列、互操作和注解

 1. 内置惰性集合类型称序列，不会索引内容或记录元素数目（可能无限多，因为数据源可能不停产生元素）

    ```
    //产生头1k个素数
    //普通方式 产生许多非必要判断，上界也可能难以判断准确
    val resList = (1..1000000).toList().filter{
    	(2 until number)
    		.map{number % it}
    		.none{it == 0}
    }.take(1000)
    
    //序列方式 使用generateSequence传入seed, 定义从种子开始的生成规则, 对每个元素仅在使用时判断（好像用个while循环加上MutableList也行，了解吧）
    val resList = generateSequence(2){
    	value -> value + 1
    }.filter{
    	...同上。
    }.take(1000) //这个filter和take显然和上一个不同, 参源码，他是返回序列的，惰性的。
    ```

    

 2. 同java的互操作和可空性

    ```
    ---Jhava.java
    public class Jhava{
    	public String attri = "myJhava"
    	@NotNull
    	public String greet(){
    		return "hello"
    	}
    	@Nullable
    	public String detemineLevel(){
    		return null
    	}
    }
    ---Hero.kt
    @file:JvmName("Hero") //见3
    class SpellBook{
    	companion object{
    		@JvmField
    		val MAX_SPELL_COUNT = 19
    		@JvmStaic
    		fun so(){...}
    	}
    
    	@JvmField
    	val spells = listOf("Magic Ms. L")
    }
    @JvmOverloads
    fun judge(lh:String = "berry", rh:String = "pang"){
    	println("$lh rh")
    }
    fun main(){
    	val adversary = Jhava()
    	adversary.greet()
    	//互操作的属性不需要调用get set
    	adversary.attri = "myjhava"
    	//使用单感叹号表示平台类型, 表示（基于平台的）可能为空
     	val level:String! = adversary.detemineLevel()
     	//调用方式同安全调用
     	level?.toLowerCase()
    }
    ```

    

 3. 在java中调用kotlin

    ```
    public static void main(){
    	//全局函数直接调用文件名.函数 HeroKt.main()
    	//或者使用文件开头的JvmName注解
    	Hero.main();
    	
    	//在java中访问kotlin属性，属性需要有@JvmField注解
    	SpellBook sb = new SpellBook();\
    	
    	//get set市kotlin自己生成的，java调用的时候还是要显式写出来
    	sb.getSpells();
    	
    	//加了JvmField注解的属性才可以不用调用get set
    	sb.spells = listOf(" ");
    	
    	//java里面调kotlin函数如果有默认参数，kotlin函数需要加@JvmOverloads注解
    	Hero.judge("perry")
    	
    	//调用伴生对象SpellBook.Companion.getMAX_SPELL_COUNT();
    	//伴生对象加入@JvmField注解时可用直接当作静态属性调用，加入@JvmStaic可当作静态函数调用
    	SpellBook.MAX_SPELL_COUNT;
    	SpellBook.sp();
    }
    ```

    

 4. Java和Kotlin异常检查的差异

    ```
    public void myExpFun throws IOException{
    	throw new IOException();
    }
    public void main(){
    	//java捕捉kt异常必须加throws注解
    	try{
    		KtExpFu()
    	}catch(IOException e){
    		...
    	}
    	
    }
    ---
    //kt捕捉java不需要搞。且kotlin和java不同，不会强迫处理该异常（try catch）
    myExpFun()
    @Throws(IOException::class)
    fun ktExpFu(){
    	throw IOException()
    }
    ```

    

 5. 函数类型互操作

    ```
    val translator = {
    	utterance: String ->
    		println(utterance.toLowerCase())
    }
    
    ---
    java中匿名函数的调用方式（FunctionN中的N代表实参的数目[0-22]）：
    Function1<String, Unit> translator = Hero.getTranslator()
    translator.invoke("truTh")
    ```

    

    


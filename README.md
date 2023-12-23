# Java8NewFeatures

- Java8中函数式接口和lambda表达式, 在自定义时 是不是这样的逻辑

1. 定义函数式接口, 标记@FunctionalInterface的interface, 里面只有一个方法
2. 在新的类中定义功能性方法, 将函数作为参数传入, 
3. 在想只用该功能的地方调用该功能性方法, 这里可以使用lambda表达式

- 是的，您的理解基本正确 

- 首先定义一个带有@FunctionalInterface标记的接口，该接口内部只有一个抽象方法。
  然后在新的类中定义一个功能性方法，该方法接受函数式接口作为参数。
  最后在需要使用该功能的地方，可以使用lambda表达式来实现该函数式接口的方法，并将其作为参数传入功能性方法中。

- 对于Stream里, 我们分析源码, 他也是根据这个
  Java中定义了四大核心函数式接口, Consumer, Supplier, Function, Predicate, 然后 我们Stream里也定义不同的功能性方法, 例如filter, map,等 在这些方法里传入了这其中某个函数式接口.
  然后我们在自己使用stream时, 就可以使用这些方法(map, filter等) 然后使用lambda表达式

  

- 是的，您的理解是正确的。在Java中，Stream类提供了一系列操作方法，比如filter、map等，这些方法接受函数式接口作为参数。通过使用lambda表达式，我们可以很方便地将这些函数式接口的方法传入到Stream的操作方法中，从而实现对流的各种操作。这种设计使得代码更为简洁和灵活。



# 1. Lambda 表达式

匿名内部类是一个**接口**, 并且其中**只有一个抽象方法**需要重写, 则可以进行Lambda表达式简化.

```java
new Thread(() -> {
	System.out.println("example");
}).start();)
```

只有函数式接口才能用这种写法. 一个Lambda表达式就是一个函数式接口. 

函数式接口就是只有一个抽象方法的接口.

```java
int i = calculateNum((int left, int right) -> {
	return left + right;
});

public static int caculateNum(IntBinaryOperator operator) {
	int a = 10;
	int b = 10;
	return operator.applyAsInt(a, b);
}
```



## Lambda省略规则

1. 参数类型可以省略
2. 方法体只有一句代码时大括号return和唯一一句代码的分号可以省略
3. 方法只有一个参数时小括号可以省略
4. 快捷键


# 2. Stream流

!要和I/O流区分开

Java8的Stream使用的是函数式编程模式, 它可以被用来对集合和数组进行链状流式的操作, 可以更方便的让我们对**集合或数组**操作.


## 流的常用操作

### 创建流

1. 单列集合:  `集合对象.stream()`

   ```java
   List<Author> authors = getAuthors();
   Stream<Author> stream = authors.stream();
   ```

2. 数组:`Arrays.stream(Array) or stream.of `

   ```java
   Integer[] arr = {1, 2, 3, 4, 5};
   Stream<Integer> stream = Arrays.stream(arr);
   Stream<Integer> stream2 = Stream.of(arr);
   ```

3. 双列集合: 转换成单列集合后再创建

   ```java
   Map<String, Integer> map = new HashMap<>();
   map.put(..);
   map.put(..);
   ..
   
   Stream<Map.Entry<String, Integer>> stream = map.entrySet().stream();
   
   ```


### 中间操作

#### Map

```java
stream()
	.map(function)

// 这个Function<A, B>内的泛型
A: 这个流的类型, B: 这个流通过这个函数想转换成的类型
authors.stream()
	.map(new Function<Author, String>(){
		@Override
		public String apply(Author author) {
			return author.getName();
		}
	})
	.forEach()...
//简化为lambda
authors.stream()
	.map(author -> author.getName())
	.forEach()..

```



### 终结操作

#### reduce归并

对流中的数据按照你指定的计算方式计算出一个结果(缩减)

reduce的作用是把stream中的元素给组合起来, 我们可以传入一个初始值, 它会按照我们的计算方式依次拿流中的元素和在初始化值的基础上进行计算, 计算结果再和后面的元素计算.

他内部的计算方式如下

```java
T result = identity;
for (T element : this stream) 
	result = accumlator.apply(result, element)
return result;
```

其中Identity 就是我们可以通过方法参数传入的初始值, accumlator的apply具体进行什么计算也是我们通过方法参数来确实的.

(Image 数组for 循环计算计算累加)


e.g. 使用reduce求所有Author年龄的和

分析stream, 最开始会有的一个问题就是, element和result是同一个类型, 所以, 在使用reduce前需要先使用map. mapReduce.

```java
private static void test() {
	List<Author> authors = getAuthors();
	Integer sum = authors.stream()
	.distinct()
	.map(author -> author.getAge())
	.reduce(0, (result, element) -> result + element);
}
```

##### reduce另一种重载形式: 一个参数

就是把流的第一个元素变为result

```java
boolean foundAny = false;
T result = null;
for (T element : this stream) {
	if (!foundAny) {
		foundAny = true;
		result = element;
	} else {
		result = accumlator.apply(result, element);
	}
}
return foundAny ? Optional.of(result) : Optional.empty();
```


### 注意事项

1. 惰性求值(如果没有终结操作,没有中间操作是不会得到执行的)
2. 流是一次性的(一旦一个流对象经过一个终结操作后, 这个流就不能再被使用)
3. 不会影响原数据 (我们在流中可以多数据做很多处理. 但是正常情况下是不会影响原来集合中的元素的, 这往往也是我们期望的)


## 3. Optional

### 1. 概述

我们在编写代码的时候出现的空指针异常需要进行非空判断, 这个可以避免

### 2. 使用 (如何创建? 如何消费?)

#### 1. 创建对象

Optional 就好像是包装类, 可以把我们的具体封装数据封装Optional对象内部. 然后我们去使用Optional中封装好的方法操作封装进去的数据就可以非常优雅的避免空指针异常.

我们一般使用Optional的静态方法ofNullable来把数据封装成一个Optional对象. 无论传入的参数是否为null都不会出现问题. 

```java
Author author = getAuthor();
Optional<Author> authorOptional = Optional.ofNullable(author);
```

有时觉得加一行代码来封装数据很麻烦, 可以改造下getAuthor方法, 让其的返回值就是封装好的Optional.

```java
public static Optional<Author> getAuthorOptional() {
	Author author = new Author(...);
	return Optional.ofNullable(author);
}
```

```java
Optional<Author> authorOptional = getAuthorOption();
authorOptional.ifPresent(author -> Symtem.out.println(author.getName())); // 进行消费
```

实际开发很多数据从数据库获取了, 很多现在也支持Optional.  可以直接把dao方法的返回值类型定义成Optional类型, MyBatis会自己把数据封装成Optional对象返回.

- 如果确实对象非空可以使用Optional 的静态方法of来📦

```java
Author author = new Author();
Optional<Author> authorOptional = Optional.of(author);
```

必须非空!

- 如果一个方法的返回值类型是Optional类型. 而如果我们经过判断发现某次计算得到的返回值为null, 这个时候就需要把null封装成Optional对象返回. 这时则可以使用Optional的静态方法Empty来进行封装.

```java
Optional.empty()
```

```java
getAuthor() {
Author author = ...
return author == null ? Optional.empty() : ... ;
}
```

### 安全的消费值

我们获取到一个Optional对象后肯定需要对其中的数据进行使用. 这时候我们可以使用其**ifPresent**方法来消费其中的值.

这个方法会判断其内封装的数据是否为空, 不空时才会执行具体的消费代码. 这样使用起来就更加安全了.

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
authorOptional.ifPresent(author -> System.out.println(author.getName()));
```

### 安全的获取值

1. 不推荐使用get()

#### 安全获取值

如果我们期望安全的获取值. 不推荐使用get, 而是使用Optional提供的以下方法

- orElseGet

获取数据并且设置数据为空时的默认值. 如果数据不为空就能获取到该数据. 如果为空则根据你传入的参数来创建对象作为默认值返回. 

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
Author author1 = authorOptional.orElseGet(() -> new Author());
```

- orElseThrow

获取数据, 如果数据不为空就能获取到该数据, 如果为空则根据你传入的参数来创建异常抛出.

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
try {
	Author author = authorOptional.orElseThrow((Supplier<Throwable>) () -> new RuntimeException("author为空"));
	System.out.println(author.getName());
} catch (Throwable throwable) {
	throwable.printStackTrace();
}
```

### 过滤

我们可以使用filter 方法对数据进行过滤, 如果原本是有数据的, 但不符合判断, 也会变成一个无数据的Optional对象.

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
authorOptional.filter(author -> author.getAge() > 100).ifPresent(author -> sout);
```

### 判断

我们可以使用isPresent方法进行是否存在数据的判断. 如果为空返回值为false, 如果不为空, 返回值为true. 但是这种方法并不能体现Optional的好处, 更推荐使用ifPresent方法.

### 数据转换

Optional 还提供了map可以让我们对数据进行转换, 并且转换得到的数据也还是被Optional包装好的, 保证了我们的使用安全. e.g.

```java
Optional<Author> authorOptional = Optional.ofNullable(getAuthor());
Optional<List<Book>> books = authorOptional.map(author -> author.getBooks());
books.ifPresent(new Consumer<List<Book>>() {
	@Override
	public void accept(List<Book> books) {
		books.forEach(book -> System.out.println(book.getName()));
	}
})
```

## 函数式接口

### 1. 概述

只有一个抽象方法的接口我们称之为函数式接口. 

JDK的函数式接口都加上了@FunctionalInterface注解进行标识.

### 2. 常见函数式接口

### Consumer消费接口

根据其中抽象方法的参数列表和返回值类型知道, 我们可以在方法中对传入的参数进行消费.

```java
@FunctionalInterface
public interface Consume<T> {
	void apply(T t);
}
```

### Function 计算转换接口

根据其中抽象方法的参数列表和返回值类型知道, 我们可以在方法中对传入的参数计算或转换, 把接口返回.

```java
@FunctionalInterface
public interface Function<T,R> {
	R apply(T t);
}
```

### Predicate 判断接口

根据其中抽象方法的参数列表和返回值类型知道, 我们可以在方法中对传入的参数条件判断, 返回判断结果.

```java
@FunctionalInterface
public interface Predicate<T t> {
	boolean test(T t);
}
```

### Supplier 生产型接口

根据其中抽象方法的参数列表和返回值类型知道, 我们可以在方法中创建对象, 把创建好的对象返回.

```java
@FunctionalInterface
public interface Supplier<T> {
	T get();
}
```

## 常用方法

- and
  我们在使用Predicate接口时候可能需要进行判断条件的拼接. 而and方法相当于是使用&&来拼接两个判断条件.
  e.g.打印作家中年龄大于17并且名字的长度大于1的作家.

  ```
  List<Author> authors = getAuthors();
  stream<Author> authorStream = authors.stream();
  authorStream.filter(new Predicate<Author>() {
  	@Override
  	public boolean test(Author author) {
  		return author.getAge() > 17;
  	}
  }.and(new Predicate<Author>() {
  	@Override
  	public boolean test (Author author) {
  		return author.getName().length() > 1;
  	}
  
  })).forEach(author -> ...)
  ```

    常用的情况: 自己定义方法, 调用时使用.

- or
  我们在使用Predicate接口时候可能需要进行判断条件的拼接. 而and方法相当于是使用||来拼接两个判断条件.

- negate

  - Predicate接口中的方法. 相当于在判断条件前面加了个!标识取反.


## 方法的引用

在使用lambda时, 如果只有一个方法的调用的话(包括构造方法), 我们可以用方法引用进一步简化代码.


### 推荐用法

在写完lambda方法发现方法体只有一行代码, 并且是方法的调用时使用快捷键尝试是否能转换成方法引用即可.


### 基本格式

```java
类名或者对象名:: 方法名
```

什么时候用类名什么时候用对象名?


### 语法详解

#### 1. 引用静态方法

其实就是引用类的静态方法

```java
类名::方法名
```

使用前提

如果我们在重写方法的时候, 方法体中只有一行代码, 并且这行代码是调用了某个类的静态方法, 并且我们要把重写的抽象方法中所有的参数都按顺序传入了这个静态方法中, 这个时候我们就可以引用类的静态方法.

#### 2. 引用对象的实例方法

```java
对象名:: 方法名
```

重写方法的时候, 方法体中**只有一行代码**, 并且这行代码是调用了某个对象的成员方法, 并且我们把要重写的抽象方法中所有的参数都按照顺序传入了这个成员方法中, 这个时候就可以引用对象的实例方法.


### 3. 引用类的实例方法

重写方法的时候, 方法体中**只有一行代码**, 并且这行代码是**调用了第一个参数的成员方法**, 并且我们把要重写的抽象方法中所有的参数都按照顺序传入了这个成员方法中, 这个时候就可以引用对象的实例方法.


### 4. 构造器引用

```java
类名::new
```

重写方法的时候, 方法体中**只有一行代码**, 并且这行代码是**调用了某个类的构造方法**, 并且我们把要重写的抽象方法中所有的参数都按照顺序传入了这个成员方法中, 这个时候就可以引用对象的实例方法.

```java
List<Author> authors = getAuthors();
authors.stream()
	.map(author -> author.getName())
	.map(name -> new StringBuilder(name))
	.map(sb -> sb.append("s").toString);
	.forEach(..)

List<Author> authors = getAuthors();
authors.stream()
	.map(author -> author.getName())
	.map(StringBuilder::new)
	.map(sb -> sb.append("s").toString);
	.forEach(..)
```



## 基本类型优化

**map()**方法中的第一个泛型不能改(第一个类型就是对象的元素,已经是定义好的,不能修改不然会报错),但是第二个泛型是可以修改的(可以修改为想要的数据类型),简单理解为就是把流当中的元素转换为另外一种元素类型然后再放到流中。

    **mapToInt() 高级用法.....还有mapTOLong()等等，针对基本数据类型进行优化。**


## 并行流


**当流中有大量元素,可以使用并行流提高操作效率。其实并行流就是把任务分配给多个线程去完成。如果使用Stream的话,只需要修改一个方法的调用就可以使用并行流来提高效率**

Stream.parallel()方法

parallelStream() 直接获取并行流

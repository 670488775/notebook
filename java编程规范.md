### 代码规范

1、java类的设计要符合高内聚低耦合

2、if/else写了很多层要减少层数

3、方法不应该超过50行，只写主干逻辑，处理细节另开方法去实现。

4、Cmetrics代码度量指标：

>* LCOM： 方法的内聚缺乏度。值越大，内聚越小。
>* FANIN： 类的扇入，扇入飙屎调用该模块上级模块的个数，扇入越大，表示该模块的复用越好。
>* FANOUT： 类的扇出，扇出表示该模块直接调用的下级模块的个数，扇出过大表示模块复杂度高，扇出过小也不好。

**设计要求一般是高内聚低耦合，即LCOM值要小，FANIN值要大，FANOUT值要合理。**

### 常见错误

**1、资源未及时释放。**

涉及需要释放的资源，如**IO、数据库操作**等操作需要显式调用关闭方法如close（）释放资源。

推荐用法：JDK1.7的自动资源管理特性try—withresource。

不需要手动关闭。优于try—catch—finnally，这样的代码更加简洁、清晰，产生的异常也更有价值。

```java
try (FileInputStream in = new FileInputStream(inputFile);
   	FileOutputStream out = new FileOutputStream(outputFile)){
    copy(in,out)
} catch (IOException e){
    LOGGER.error("copy file failed")
}
```

**2、返回零长度的数组/集合而非NULL**

防止别人在使用时忘了进行判空处理而发生NPE。

**3、switch未使用break**

如果不使用break要加上注释，防止其他人在看代码时以为漏写了

注释内容：**//fall through**

**4、集合迭代器**

> 不要在foreach循环里进行元素的remove/add操作，删除元素请使用removeIf方法或Iterator
>
> 普通for循环删除会发生漏删的情况，而foreach循环删除元素抛错：ConcurrentModificationException
>
> 普通for循环删除会发生漏删的情况：因为普通for循环是根据索引删除的，由于两个相同值在相邻的位置，当删除第一个值之后，集合发生改变要重新排序索引
>
> foreach循环删除元素抛出异常ConcurrentModificationException：集合迭代器内部的modCount与expectedModCount不一致，造成并发修改异常

```java
// 使用JAVA 8 Collection中的removeIf方法
list.removeIf(item -> "1".equals(item));

// 或者
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()){
    String item = iterator.next();
    if(isRemovable()){
        iterator.remove();
    }
}
```

map的遍历删除元素也可使用Iterator进行。

**迭代器遍历时，每一次调用 next() 函数，至多只能对容器修改一次**

如果在遍历过程中对map同时进行了put操作和remove操作，会抛出异常ConcurrentModificationException。

**5、浅拷贝和深拷贝**

浅clone和深clone都是clone，区别在于所clone对象内部的成员属性在clone时是否处理为引用，如果仍保留为引用，则成为浅clone，反之则称为深clone。浅clone方式得到clone对象即可，深clone方式在得到对象后，还需要对引用的成员属性进行“clone”处理，需要clone到底。

**6、集合转数组时参数使用错误**

将集合转为数组时使用Collection<T>.toArray(T[])方法，且参数是类型相同的零长度数组。

数组容量大小的影响：

- 等于0，动态创建与size相同的数组，性能最好。
- 大于0但小于size，重新创建大于等于size的数组，增加GC负担
- 等于size，在高并发情况下，数组创建完成之后，size正在变大的情况下，负面影响与上相同
- 大于size，浪费空间，且在size处插入null值，存在NPE隐患

案例：

```java
List<String> list = new Arraylist<>(DEFAULT_CAPACTITY);
List.add(getElm());
String[] array = list.toArray(new String[0]);
```

**7、过多不必要的嵌套，使用卫语句**

卫语句就是把复杂的条件表达式拆分成多个条件表达式，比如将多个if-else语句拆成多个if语句，可以让代码更简洁，可读。

### 安全编码

**1、未经检验的不可信数据可能会引起某种注入攻击，对系统造成严重影响。**

案例：

- 拼接URL（URL越权）
- 拼接SQL（SQL注入）
- 打印日志（敏感信息、非法字符）
- 文件下载（文件路径越权，消耗过多资源）

**2、数值运算溢出问题**

使用int型进行数据运算时有可能会造成数据溢出，建议使用JAVA8新特性Math.*Exact()

**java.lang.Math.addExact() 返回参数的总和。如果结果溢出int或long，则将引发异常。**

示例：

```java
public static int mulAccum(int oldAcc,int newVal,int scale){
    return Math.addExact(oldAcc,Math.multiplyExact(newVal,scale));
}
```

**3、数值运算精度问题**

java简单类型无法对浮点数进行精确计算，浮点数不精确的根本原因在于尾部部分的位数是固定的，一旦需要表示的数字的精度高于浮点数精度，必然会产生 误差。所以，数值运算类问题，如金额计算，使用BigDecimal防止精度丢失。

**4、防止通过异常泄露敏感信息**

如果对所有异常都直接抛出，有时会把一些敏感信息泄露出去。

![常见异常](C:\Users\67048\Desktop\常见异常.jpg)
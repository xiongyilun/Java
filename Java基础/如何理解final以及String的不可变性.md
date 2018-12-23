### 不可变对象
大家都听说过String是不可变的，为什么呢。很多人摆出了下面的源码：

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```

你看类名用final修饰了，表示String类不能被继承，底层的char[]数组也被final修饰了，所以不可被修改

被final修饰的数组，真的不可变吗？

```
final int[] array = {1,2,3,4,5};
array[1] = 11;
array = new int[]{6, 7, 8, 9, 10};//Cannot assign a value to final variable 'array'
```
我们发现其实final修饰变量array，是指array指向的地址不能改变。也就是不能指向一个新的对象，但是数组对象的内容本身是可以被修改的。所以按道理来说String的值是可变的呀。那为什么不可变呢？

---


### String的不可变性

其实很简单，就像你自己写一个VO类一样。你对外只提供Get()方法，不提供Set()方法，那其他类也无法修改你的值，对吧。

所以String不可变实现的原理不在于final，而是Sun的工程师没有提供给你Set方法，所有的如subString()等方法都是新建了一个字符串对象然后返回的，而不是在字符串上进行修改。


---
### 值传递还是引用传递

这里也就可以理解了什么String作为参数时，是值传递还是引用传递了。其实你说值传递也好，引用传递也好，都可以。像下面这种例子

```
public class Test1 {
    public static void main(String[] args) {
        String temp = "temp";
        changeStr(temp);
        System.out.println(temp);//输出temp

    }

    static void changeStr(String s) {
        s += "temp";
    }
}
```

如果有人问你，为什么这个temp值没有被改变，你别开口就是什么值传递，引用传递，String的不变性等等。你就说，Sun的工程师，没有提供给你在原字符串上修改的方法。都是new一个新的String对象，然后指向新的对象就好了。所以原字符串没变。

---
### String不可变的好处
安全，这个理由够嘛。比如在HashSet中，元素是不允许重复的，但是如果该元素是可变的话，那么就会破坏这种特性。
```
import java.util.HashSet;
import java.util.Objects;

public class Test1 {
    public static void main(String[] args) {
        HashSet<Student> set = new HashSet<Student>();

        Student zhangsan = new Student();
        zhangsan.setName("张三");
        zhangsan.setAge(18);

        Student lisi = new Student();
        lisi.setName("李四");
        lisi.setAge(20);

        set.add(zhangsan);
        set.add(lisi);
        System.out.println("=======修改前");
        for (Student student : set) {
            System.out.print(student.toString());
            System.out.println();
        }

        zhangsan.setName("李四");
        zhangsan.setAge(20);
        System.out.println("=======修改后");
        for (Student student : set) {
            System.out.print(student.toString());
            System.out.println();
        }

    }
}

class Student {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student)) return false;
        Student student = (Student) o;
        return getAge() == student.getAge() &&
                Objects.equals(getName(), student.getName());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getName(), getAge());
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```
看一看输出:
```
=======修改前
Student{name='张三', age=18}
Student{name='李四', age=20}
=======修改后
Student{name='李四', age=20}
Student{name='李四', age=20}
```
由于Student的可变性，导致了HashSet中存在了两个相同的元素。所以千万不要使用可变类型做HashMap和HashSet的键值。

一般在日常使用中都是用String做键，正是因为String具有不可变性。


虽然s1已经被修改成了"123zxc"，但是这并不是在s1指向的String对象上修改的，而是新建了一个new String("123zxc")对象，然后用s1指向该对象，并没有对HashSet里的原String对象产生影响，仍是123

---
### 如何修改String对象
如何证明String可以被修改呢？其实用反射可以实现。

```
String temp = "temp";
Field field = temp.getClass().getDeclaredField("value");
field.setAccessible(true);
field.set(temp,new char[]{'p'});
System.out.println(temp);//输出p
```



# Java专项练习错题集

---

### 继承与实现

```java
package com.sunline.java;
public class A implements B extends C {
	public static void main(String args[]) {
		System.out.println("hello sunshine!");
	}
}
```

错误原因：先继承后实现，故第2行编译错误。

---

### 关于Java的类加载器：

`bootstrap classloader` －引导（也称为原始）类加载器，它负责加载Java的核心类。 

`extension classloader` －扩展类加载器，它负责加载JRE的扩展目录（JAVA_HOME/jre/lib/ext或者由java.ext.dirs系统属性指定的）中JAR的类包。 

`system classloader` －系统（也称为应用）类加载器，它负责将系统类路径 java -classpath或-Djava.class.path变量所指的目录下 的类库加载到内存中。

`app classloader` -系统（也称为应用程序）类加载器，负责加载应用程序classpath目录下的所有jar和class文件。派生于classloader。

---

### sleep()与wait()

> sleep()就是让线程占着CPU去睡觉！

1. sleep()来自Thread类，wait()来自Object类。
2. sleep()没有释放锁，wait()释放了锁，使得敏感词线程可以使用同步控制块或者方法。
3. wait(), notify()和notifyAll()只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用。
4. sleep()必须捕获异常，而wait，notify和notifyAll不需要捕获异常。 

---

### 枚举类enum

```java
enum AccountType {
    SAVING, FIXED, CURRENT;
    private AccountType() {
        System.out.println(“It is a account type”);
    }
}
class EnumOne {
    public static void main(String[]args) {
        System.out.println(AccountType.FIXED);
    }
}
```

枚举类在实现时，会转化为一个继承了java.lang.Enum类的实体类，原先的枚举类型变成对应的实体类型，所以`AccountType`被转化成`Class AccountType`，并且生成新的构造函数。

```java
private AccountType(String s, int i) {
    super(s,i); 
    System.out.println(“It is a account type”); 
}
```

而枚举类中的具体的枚举类型被转化成：

```java
public static final AccountType SAVINGl;
public static final AccountType FIXED;
public static final AccountType CURRENT;
```

并添加一段静态初始化代码：

```java
static {
	SAVING = new AccountType("SAVING", 0);
	...
	CURRENT = new AccountType("CURRENT", 0);
	
	$VALUES = new AccountType[] {
		SAVING, FIXED, CURRENT
	}
}
```

因此上述程序输出：

	It is a account type
	It is a account type
	It is a account type
	FIXED
	
---

### Java数组

java中的数组在声明时不能赋予长度，只能在实例化时指定长度

```java
String a[];
String a[] = new String[50];
char c[];
String d[50];
char e[50];
```

因此，1, 2, 3行能执行，5, 6编译错误。

---

### 类方法和实例方法

#### 类方法

指的是用static关键字修饰的方法，即静态方法，没有this指针。

静态方法可以直接引用该类的静态成员变量和函数，但不允许直接引用非静态成员变量。

#### 实例方法

即成员方法


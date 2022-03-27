#### 封装
#### 继承
1. 继承的使用案例
	- 继承是特殊到一般的关系，子类到父类的关系
	- 继承作用：提高代码复用，相同代码可以定义在父类中
	- 子类更强大，不仅得到类父类的功能，还有自己的功能
	```
	//父类（姓名、年龄；通用行为：吃饭）
	public class People {
		private String name;
		private int age;

		//通用方法
		public void eat(){
			System.out.println(name + "在吃饭~");
		}

		//无参构造方法
		public People() {
		}

		//有参构造方法
		public People(String name, int age) {
			this.name = name;
			this.age = age;
		}

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
	}

	```
	```
	public class Teacher extends People {
		//老师特有的方法
		public void teach() {
			System.out.println(getName() + "老师打人~");
		}
	}
	```
	```
	public class TestMain {
		public static void main(String[] args) {
			Teacher linyuan = new Teacher();
			linyuan.setName("林源");
			linyuan.setAge(22);
			System.out.println(linyuan.getName());
			System.out.println(linyuan.getAge());
			linyuan.eat();
			linyuan.teach();
		}
	}
	```
	```
	林源
	22
	林源在吃饭~
	林源老师打人~
	```
1. 子类不能继承父类的内容
	- 子类不能继承父类的构造器，子类有自己的构造器
	- 子类是否可以继承父类的私有成员（私有成员变量、私有成员方法）？
		- 可以继承，只是不能直接访问
		- 可暴力访问！
		```
		class Animal {
			public String name;
			public void run() {}
		}

		```
		```
		class Cat extends Animal {}
		```
		```
		public class TestMain {
			public static void main(String[] args) {
			Cat c = new Cat();
			System.out.println(c.name);
			c.run();
			}
		}
		```
	- 子类是否可以继承父类的静态成员？
		- 不能继承父类的静态成员
		- 父类的静态成员只有一份，子类只是可以访问，共享访问，共享并非继承
		```
		class Animal {
			public static String name;
			public static void run(){}
		}
		
		```
		```
		class Cat extends Animal {}
		```
		```
		public class TestMain {
			public static void main(String[] args) {
			Cat c = new Cat();
			System.out.println(c.name);
			c.run();
			}
		}
		```
1. 继承后，成员变量的访问特点（就近原则）
	- this代表了当前对象的引用
	- super代表了父类对象的引用
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.show();
		}
	}


	class Animal {
		public String name = "动物名";
	}

	class Cat extends Animal {
		public String name = "猫名";

		public void show() {
			String name = "xiaohui";
			System.out.println(name);
			System.out.println(this.name);
			System.out.println(super.name);
		}
	}
	```
1. 继承后，成员方法的访问特点（就近原则）
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.run();
		}
	}

	class Animal {
		public void run() {
			System.out.println("动物可以跑~");
		}
	}

	class Cat extends Animal {
		public void run() {
			System.out.println("猫类可以跑~");
		}
	}
	```
1. 继承后，方法的重写
	- 子类继承了父类的方法，但是无法满足自身的需求，子类重写父类的方法，子类的这个方法就进行了重写
	- Java建议在重写的方法上加@Override注解
	- 申明不变，重新实现！
	- 特点：
		- 名称和形参列表需要和父类保持一致
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.run();
		}
	}

	class Animal {
		public void run() {
			System.out.println("动物可以跑~");
		}
	}

	class Cat extends Animal {
		//进行了方法的重写
		@Override
		public void run() {
			System.out.println("猫类可以跑~");
		}
	}
	```
1. super调用父类被重写的方法
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.run();
        cat.go();
		}
	}

	class Animal {
		public void run() {
			System.out.println("动物可以跑~");
		}
	}

	class Cat extends Animal {
		//进行了方法的重写
		@Override
		public void run() {
			System.out.println("猫类可以跑~");
		}

		//通过方法中转，访问父类方法
		public void go() {
			super.run();
		}
	}
	```
1. 静态方法和私有方法是否可以被重写？
	- 都不可以
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.run();
        cat.go();
		}
	}

	class Animal {
		private void run() {
			System.out.println("动物可以跑~");
		}

		public static void go() {
			System.out.println("动物可以走~");
		}
	}

	class Cat extends Animal {
		//进行了方法的重写
		@Override
		private void run() {
			System.out.println("猫类可以跑~");
		}

		@Override
		public static void go() {
			System.out.println("猫类可以走~");
		}
	}
	```
1. 继承后，构造器的特点
	- 子类的全部构造器（无参或者有参）的第一行默认一定先访问父类的无参构造器，再执行子类自己的构造器
	- 子类构造器的第一行默认有一个super()调用父类的无参数构造器，写不写都存在
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat();
        Cat xiaohui = new Cat("小灰");
		}
	}

	class Animal {
		public Animal() {
			System.out.println("父类Animal的无参构造器");
		}

	}

	class Cat extends Animal {
		public Cat() {
			super();//默认存在的，写不写都存在，它根据参数去匹配父类的构造器
			System.out.println("子类Cat的无参构造器");
		}

		public Cat(String name) {
			System.out.println("子类Cat的有参构造器");
		}
	}
	```
2. super调用父类构造器
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat cat = new Cat("小灰", 3, '公');
        cat.sleep();
		}
	}

	class Cat extends Animal {
		public Cat(String name, int age, char sex) {
			super(name, age, sex);//根据参数匹配父类构造器
		}

		public void sleep() {
			System.out.println(getAge() + "岁大的" + getName() + getSex() + "猫在碎觉~");
		}
	}

	class Animal {
		private String name;
		private int age;
		private char sex;

		public Animal() {
		}

		public Animal(String name, int age, char sex) {
			this.name = name;
			this.age = age;
			this.sex = sex;
		}

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

		public char getSex() {
			return sex;
		}

		public void setSex(char sex) {
			this.sex = sex;
		}
	}
	```
1. this和super关键字使用总结
	- this代表了当前对象的引用
		- this.子类成员变量
		- this.子类成员方法
		- this(...)可以根据参数匹配本类构造器
	- super代表了父类对象的引用
		- super.父类成员变量
		- super.父类成员方法
		- super(...)可以根据参数匹配访问父类的构造器
	- this(...)和super(...)必须放在构造器的第一行，否则报错
	- this(...)和super(...)不能同时出现在构造器中
	```
	public class TestDemo {
    public static void main(String[] args) {
        Student linyuan = new Student("林源", 22, '男');
        System.out.println(linyuan.getName());
        System.out.println(linyuan.getAge());
        System.out.println(linyuan.getSex());
		}
	}

	class Student {
		private String name;
		private int age;
		private char sex;

		public Student() {
		}

		public Student(String name, int age) {
			//借助兄弟构造器的功能，只是一个中转功能
			this(name, age, '男');
		}

		public Student(String name, int age, char sex) {
			this.name = name;
			this.age = age;
			this.sex = sex;
		}

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

		public char getSex() {
			return sex;
		}

		public void setSex(char sex) {
			this.sex = sex;
		}
	}
	```
1. 继承的特点
	- 单继承：一个类只能直接继承一个一个直接父类
	- 多层继承：一个类可以间接继承多个类
	- 一个类要么默认继承了object类，要么间接继承了object类，object是java中的超类
#### 多态
#### 抽象类
1. 概念
	- 什么是抽象类
		- 父类知道子类一定会完成某个功能，但每个子类完成情况不一样；子类也只会用自己重写的功能，那么父类就可以把该功能定义成抽象方法
	- 什么是抽象方法
		 - 没有方法体，只有方法名称，必须用abstract修饰
		 - 拥有抽象方法的类必须定义成抽象类
	```
	public class TestDemo {
    public static void main(String[] args) {
        Cat c = new Cat();
        c.run();
		}
	}

	abstract class Animal {
		public abstract void run();
	}

	class Cat extends Animal {
		@Override
		public void run() {
			System.out.println("猫跑~");
		}
	}

	class Dog extends Animal {
		@Override
		public void run() {
			System.out.println("狗跑~");
		}
	}
	```
1. 抽象类的特征
	- 有得有失
		- 抽象类得到了拥有抽象方法的能力
		- 抽象类失去了创建对象的能力
	- 抽象类作为类，一定有构造器，而且抽象类必须有构造器，是为了提供给子类创建对象调用父类构造器使用的
	- 抽象类虽然有构造器，但是不能创建对象，假如能创建对象，调用抽象方法，因抽象方法没有方法体，所以不能执行！
	- 除此之外，类有的成分，它都有
1. 抽象类存在的意义
	- 抽象类存在是为了被子类继承，否则毫无意义
	- 抽象类体现的是模版思想，部分实现，部分抽象
#### 接口
1. 接口概述
	- 什么是接口
		- 接口是更彻底的抽象，在JDK1.8之前，接口中只能是抽象方法和常量
		- 接口体现的是规范思想，实现接口的子类必须重写接口的全部抽象方法
		```
		public interface TestInterface {
		//接口中抽象方法可以省略public abstract
		//抽象方法
		public abstract void run();

		//常量
		//变量值只有一个，而且在程序运行过程中不可更改
		//常量修饰符：public static final，缺一不可
		//常量变量名建议全部大写，多个单词用_连接
		public static final String REAL_NAME = "林源";
		}
		```
1. 接口基本实现
	- 类与接口是实现关系，接口是被类实现的，关键字是implements
	- 一个类实现接口必须重写完接口中的全部抽象方法，否则这个类要定义为抽象类
	- 格式
		```
		修饰符 class implements 接口1,接口2...{}
		```
		```
		public interface TestInterface {
		public static void main(String[] args) {
			RuningMan rm = new RuningMan();
			rm.run();
			rm.jump();
			}
		}

		//实现类实现接口
		class RuningMan implements SportsMan {
			@Override
			public void run() {
				System.out.println("运动员跑~");
			}

			@Override
			public void jump() {
				System.out.println("运动员跳~");
			}
		}

		//运动员接口：规范
		interface SportsMan {
			void run();
			void jump();
		}
		```
1. 接口的多实现
	```
	public interface TestInterface {
		public static void main(String[] args) {
			RuningMan rm = new RuningMan();
			rm.run();
			rm.jump();
			rm.rule();
		}
	}

	//实现类实现接口
	class RuningMan implements SportsMan, Law {
		@Override
		public void run() {
			System.out.println("运动员跑~");
		}

		@Override
		public void jump() {
			System.out.println("运动员跳~");
		}

		@Override
		public void rule() {
			System.out.println("运动员守法~");
		}
	}

	//运动员接口：规范
	interface SportsMan {
		void run();
		void jump();
	}

	interface Law {
		void rule();
	}
	```
1. 接口与接口的多继承
	```
	public interface TestInterface {
		public static void main(String[] args) {
			RuningMan rm = new RuningMan();
			rm.run();
			rm.jump();
			rm.rule();
			rm.go();
		}
	}

	//实现类实现接口
	class RuningMan implements SportsMan {
		@Override
		public void run() {
			System.out.println("运动员跑~");
		}

		@Override
		public void jump() {
			System.out.println("运动员跳~");
		}

		@Override
		public void rule() {
			System.out.println("运动员守法~");
		}

		@Override
		public void go() {
			System.out.println("运动员走~");
		}
	}

	//运动员接口：规范
	interface SportsMan extends Law, Go {
		void run();
		void jump();
	}

	interface Law {
		void rule();
	}

	interface Go {
		void go();
	}
	```
#### 代码块
1. 1
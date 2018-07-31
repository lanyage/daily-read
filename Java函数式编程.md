# Java函数式编程

java的函数时编程和Lisp这样的函数式编程语言还是有差别的。最主要的原因就是java的是强类型。 要想实现类似函数式编程特性，java使用了匿名内部类。

常用的匿名内部类有Consumer，Predict等等

首先看看Consumer:

	public class StreamDemo {
	
	    public static void service(Consumer<Cons> consumer) {
	        Cons cons = new Cons();     //why on earth there is a newed Cons? because
	        cons.beforeService();
	        try {
	            consumer.accept(cons);
	        } finally {
	            cons.afterService();
	        }
	
	    }
	}
	
	class Cons {
	    public void beforeService() {
	        System.out.println("before service." + hashCode());
	    }
	
	    public void service() {
	        System.out.println("cons is doing some business things." + hashCode());
	    }
	
	    public void afterService() {
	        System.out.println("after service." + hashCode());
	    }
	}
	
	class Test {
	    public static void main(String[] args) {
	        StreamDemo.service(new Consumer<Cons>() {       //实际上Java的函数式编程是匿名内部类+类型推断,实际上匿名类已经持有了对象的引用了，这个对象可以是非常复杂的对象。
	            @Override
	            public void accept(Cons cons) {
	                cons.service();
	            }
	        });
	
	        StreamDemo.service(cons -> cons.service()); //函数式编程真的好爽啊-Consumer
	    }
	}
	
这样就极大的简化了代码,不需要关注一些复杂繁琐的操作,只关注业务就好了。

再比如说Predict:

	public static void main(String[] args) {
	        List<Integer> integerList = new ArrayList<>();
	        integerList.add(1);
	        integerList.add(2);
	        integerList.add(3);
	        integerList.add(4);
	        integerList.add(5);
	        integerList.add(6);
	        integerList.stream().filter(e -> e < 6).forEach(e -> System.out.println(e));    // predict一般用来满足条件
	    }



当然现在还支持并行计算，将数据映射到多核上去进行计算，但是这样不一定总能提高性能，也要看数据是否容易切分。


	
	
	
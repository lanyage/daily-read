# ThreadLocal应用场景

当你的应用对于在方法的最深处需要调用一个变量，又不希望通过函数的参数来传递该变量。

如：

	public void static func1(int content){
		a.func2(content) {
			b.func3(content) {
				c.sout(content);
			}
		}
	}	


则可以通过这样来实现，通过使用线程私有变量。

	public class ThreadLocalDemo {
	    private static ThreadLocal<String> tl = new ThreadLocal<>();
	
	    public static void setContent(String content) {
	        tl.set(content);
	    }
	
	    public static String getContent() {
	        return tl.get();
	    }
	}
	
	class ThreadlocalTest {
	    public static void main(String[] args) {
	        for (int i = 0; i < 2; i++) {
	            int finalI = i;
	            new Thread(() -> {
	                ThreadLocalDemo.setContent("str-" + finalI);
	                System.out.println(Thread.currentThread().getName() + " : " + finalI);
	            }).start();
	        }
	    }
	}	
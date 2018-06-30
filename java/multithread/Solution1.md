#### 现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行？

```
public class Solution1 {

  public static void main(String[] args) throws InterruptedException {
    final Random RND = new Random();
    Thread t1 = new Thread(new Task("task1"));
    Thread t2 = new Thread(new Task("task2"));
    Thread t3 = new Thread(new Task("task3"));
//    t1.start();
//    t2.start();
//    t3.start();

    t1.start();
    t1.join();
    t2.start();
    t2.join();
    t3.start();

  }

  private static class Task implements Runnable{
    private String name = "";
    public Task(String name){
      this.name = name;
    }

    @Override
    public void run() {
      try {
        Thread.sleep(new Random().nextInt(1000));
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(this.name);
    }
  }

}
```

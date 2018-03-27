---
title: threadlocal
date: 2018-03-27 15:44:32
tags: java线程
---

### ThreadLocal
```
public class RawRunnableTest {

    @Test
    public void testRawInheritableThreadLocal() throws InterruptedException {
        final ThreadLocal<Span> threadLocal = new ThreadLocal<>();
        threadLocal.set(new Span("xiexiexie"));
        //输出 xiexiexie
        Object o = threadLocal.get();
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(threadLocal.get() + "thead-" + Thread.currentThread());
                threadLocal.set(new Span("zhangzhangzhang1"));
                System.out.println(threadLocal.get() + "thead-" + Thread.currentThread());
            }
        };

        Runnable runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println(threadLocal.get() + "thead-" + Thread.currentThread());
                threadLocal.set(new Span("zhangzhangzhang2"));
                System.out.println(threadLocal.get() + "thead-" + Thread.currentThread());
            }
        };

        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.submit(runnable);
        TimeUnit.SECONDS.sleep(1);
        executorService.submit(runnable2);
        TimeUnit.SECONDS.sleep(1);
        Span span = threadLocal.get();
        System.out.println(span + "thead-" + Thread.currentThread());
    }

    class Span {
        public String name;
        public int age;

        public Span(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Span{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```
输出结果:
```
nullthead-Thread[pool-1-thread-1,5,main]
Span{name='zhangzhangzhang1', age=0}thead-Thread[pool-1-thread-1,5,main]
nullthead-Thread[pool-1-thread-2,5,main]
Span{name='zhangzhangzhang2', age=0}thead-Thread[pool-1-thread-2,5,main]
Span{name='xiexiexie', age=0}thead-Thread[main,5,main]
```
在子线程中获取到的内容为null


### InheritableThreadLocal 

```
public class RawRunnableTest {

    @Test
    public void testRawInheritableThreadLocal() throws InterruptedException {
        final InheritableThreadLocal<Span> inheritableThreadLocal = new InheritableThreadLocal<>();
        inheritableThreadLocal.set(new Span("xiexiexie"));
        //输出 xiexiexie
        Object o = inheritableThreadLocal.get();
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
                inheritableThreadLocal.set(new Span("zhangzhangzhang1"));
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
            }
        };

        Runnable runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
                inheritableThreadLocal.set(new Span("zhangzhangzhang2"));
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
            }
        });

        ExecutorService executorService = Executors.newFixedThreadPool(1);
        executorService.submit(runnable);
        TimeUnit.SECONDS.sleep(1);
        executorService.submit(runnable2);
        TimeUnit.SECONDS.sleep(1);
        Span span = inheritableThreadLocal.get();
        System.out.println(span + "thead-" + Thread.currentThread());
    }

    class Span {
        public String name;
        public int age;

        public Span(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Span{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```
输出：
```
Span{name='xiexiexie', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='zhangzhangzhang1', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='zhangzhangzhang1', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='zhangzhangzhang2', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='xiexiexie', age=0}thead-Thread[main,5,main]
```
由于线程重用，Runnable获取到的内容是脏的（第3行输出）,实际上应该是获取提交到线程池时的内容

### TransmittableThreadLocal

```
public class RawRunnableTest {

    @Test
    public void testRawInheritableThreadLocal() throws InterruptedException {
        final InheritableThreadLocal<Span> inheritableThreadLocal = new TransmittableThreadLocal<>();
        inheritableThreadLocal.set(new Span("xiexiexie"));
        //输出 xiexiexie
        Object o = inheritableThreadLocal.get();
        Runnable runnable = TtlRunnable.get(new Runnable() {
            @Override
            public void run() {
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
                inheritableThreadLocal.set(new Span("zhangzhangzhang1"));
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
            }
        });

        Runnable runnable2 = TtlRunnable.get(new Runnable() {
            @Override
            public void run() {
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
                inheritableThreadLocal.set(new Span("zhangzhangzhang2"));
                System.out.println(inheritableThreadLocal.get() + "thead-" + Thread.currentThread());
            }
        });

        ExecutorService executorService = Executors.newFixedThreadPool(1);
        executorService.submit(runnable);
        TimeUnit.SECONDS.sleep(1);
        executorService.submit(runnable2);
        TimeUnit.SECONDS.sleep(1);
        Span span = inheritableThreadLocal.get();
        System.out.println(span + "thead-" + Thread.currentThread());
    }

    class Span {
        public String name;
        public int age;

        public Span(String name) {
            this.name = name;
        }

        @Override
        public String toString() {
            return "Span{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```

输出结果：
```
Span{name='xiexiexie', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='zhangzhangzhang1', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='xiexiexie', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='zhangzhangzhang2', age=0}thead-Thread[pool-1-thread-1,5,main]
Span{name='xiexiexie', age=0}thead-Thread[main,5,main]
```
当然也可以包装Callable和Executors，具体见https://github.com/alibaba/transmittable-thread-local







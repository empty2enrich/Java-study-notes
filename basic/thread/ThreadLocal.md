# ThradLocal

 + [为什么要使用 ThreadLocal ](#1)
 + [ThreadLocal 原理](#2)
 + [ThreadLocal 疑问](#3)

## <p id=1>为什么要使用 ThreadLocal </p>

ThreadLocal 在每个线程中为变量创建一个副本,每个线程访问自己的内部副本变量。

## <p id=2>ThreadLocal 原理</p>

Thread 实例化对象会捕捉 ThreadLocal 变量,将找到的都拷贝到 thread.threadlocals(type:ThreadLocalMap) 中,在使用的时候获取 threadlocal 对象失败就将心得 threadlocal 变量添加到 thread.threadlocals 中,(*** 注：** 在将 threadlocal 副本放到 threadlocals 中时,默认初始化变量值 为 `null`,若要修改,需要重写 initialValue()),每个线程只访问 threadlocals 中的 threadlocal 变量,互不影响,一些主要方法的源码:

```java

    /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }

    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }

    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to 
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     * @param map the map to store.
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

    /**
     * Returns the current thread's "initial value" for this
     * thread-local variable.  This method will be invoked the first
     * time a thread accesses the variable with the {@link #get}
     * method, unless the thread previously invoked the {@link #set}
     * method, in which case the <tt>initialValue</tt> method will not
     * be invoked for the thread.  Normally, this method is invoked at
     * most once per thread, but it may be invoked again in case of
     * subsequent invocations of {@link #remove} followed by {@link #get}.
     *
     * <p>This implementation simply returns <tt>null</tt>; if the
     * programmer desires thread-local variables to have an initial
     * value other than <tt>null</tt>, <tt>ThreadLocal</tt> must be
     * subclassed, and this method overridden.  Typically, an
     * anonymous inner class will be used.
     *
     * @return the initial value for this thread-local
     */
    protected T initialValue() {
        return null;
    }



```

重写 threadlocal 的 initialValue() :

```java
    ThreadLocal<Long> longLocal = new ThreadLocal<Long>(){
        protected Long initialValue() {
            return Thread.currentThread().getId();
        };
    };

```

## <p id=3>ThreadLocal 疑问</p>

 + ThreadLocal 有什么用,本来是为了了减少每次使用都 new 一个对象,但使用 ThreadLocal 重写 initialValue() 应该是深拷贝了一个新对象出来,但这和直接 new 似乎差不多？

# Android Test Hello World

这篇文章将简单介绍Android怎么写单元测试

## 配置

一般情况下，我们新建一个module之后，Android Studio会自动帮我们建立简单的单元测试环境。在gradle文件里面的dependencies会加上：

```groovy
testImplementation 'junit:junit:4.12'
androidTestImplementation 'com.android.support.test:runner:1.0.1'
androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
```

在文件目录里面会加上test和androidTest目录，分别对应Java单元测试和Android单元测试，里面会有两个简单的sample。

---

## JUnit Test

Java Unit Test里面不能包含Android相关的代码，Context和Log等不能出现，最好是通过debug看代码执行，如果需要打出log，可以使用`System.out.print()` 代替。我们新建一个类MyJavaClass：

```java
public class MyJavaClass {
    public static final int CODE_CACHED = 1;
    public static final int CODE_OK = 0;
    public static final int CODE_UNKNOW = -1;

    private int errorCode;
    private String msg;

    public MyJavaClass(int errorCode, String msg) {
        this.errorCode = errorCode;
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }

    public boolean isSuccessful() {
        return errorCode >= CODE_OK;
    }

    public static boolean isMsgEmpty(String msg) {
        return msg == null || msg.equals("");
    }
}
```
我们测试其中的两个函数，`isSuccessful()` 和`isMsgEmpty()` 。

在test目录的对应包名下面生成一个MyJavaClassTest类（将光标移动到MyJavaClass类名上，Alt+Enter代码提示可以自动创建文件）：

```java
public class MyJavaClassTest {
    private MyJavaClass instance;

    @Before
    public void init() {
        instance = new MyJavaClass(MyJavaClass.CODE_OK, "OK");
    }

    @Test
    public void testSuccess() {
        System.out.println("hello world");
        assertTrue(instance.isSuccessful());
    }

    @Test
    public void testMsgEmpty() {
        assertFalse(MyJavaClass.isMsgEmpty(instance.getMsg()));
    }

    @After
    public void tearDown() {
        instance = null;
    }
}
```

可以以类为单位执行，也可以执行某个单元测试。在编辑器左边有运行的按钮，或者右键方法和类名，也可以将光标移至要运行的单元测试内部用快捷键`Ctrl+Shift+F10`运行，`Ctrl+Shift+F9`调试。

#### annotation(注解)

代码里面以后三种注解，就是字面含义。

- `@Before`：在单元测试之前执行的准备工作。如果是运行一个测试类，每个单元测试方法执行之前都会走一遍所有的`@Before`注解的函数。
- `@Test`：单元测试。
- `@After`：在单元测试之后执行的恢复工作。和`@Before`相同，每个单元测试之后都会执行一次。

在单元测试里面，只能保证这三个注解之间的执行顺序`@Before`->`@Test`->`@After`，当有同一个注解的时候，**不是按照函数定义顺序执行**，可以看看这个例子[UnitOrderTest](https://github.com/iamjinge/AndroidTestingSample/blob/master/helloworld/src/test/java/com/jinge/helloworld/UnitOrderTest.java)。根据文档描述，在JDK7以及一些早期版本，单元测试的顺序是不确定的，甚至可能每次运行的顺序都不一样，但是从JUnit 4.11开始使用`MethodSorters#DEFAULT`作为默认单元测试默认执行顺序，从源码分析，首先根据方法名的hash code排序，如果hash code相同就根据`MethodSorters#NAME_ASCENDING`排序。一般情况下，好的单元测试代码是**不应该依赖于执行顺序**。

#### assert(断言)

每个单元测试都有要验证的结果，这个验证过程就是由`assert`完成，JUnit包里面提供了很多`assert*`方法

| 方法                                       |                                     |
| :--------------------------------------- | ----------------------------------- |
| assertTrue(boolean condition)            | 判断condition是否为True                  |
| assertEquals(T expected, T actual)       | 判断expected和actual是否相等，并不是判断是否为同一个对象 |
| assertThat(T actual, Matcher<? super T> matcher) | 判断actual是否满足matcher条件               |

---

## Android Unit Test

我们用默认的MainActivity做测试。简单的修改布局文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.jinge.helloworld.MainActivity">

    <TextView
        android:id="@+id/text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_world"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```

我们在androidTest目录的对应包名下创建MainActivityTest，针对TextView测试：

```java
public class MainActivityTest {
    @Rule
    public ActivityTestRule<MainActivity> activityRule =
            new ActivityTestRule<>(MainActivity.class);
    private TextView textView;

    @Before
    public void init() {
        textView = activityRule.getActivity().findViewById(R.id.text);
        textView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                textView.setVisibility(View.INVISIBLE);
            }
        });
    }

    @Test
    public void testContent() {
        onView(withId(R.id.text))
                .check(matches(withText(R.string.hello_world)));
    }

    @Test
    public void testShow() throws Throwable {
        onView(equalTo((View) textView))
                .perform(click())
                .check(matches(not(isDisplayed())));
    }

    @Test
    public void testUiThread() throws Throwable {
        System.out.println("waiting");
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("end UI Thread");
            }
        });
        System.out.println("end");
    }
}
```

Android Test是运行在手机上的，log和System.out.print()输出在logcat里面。JUnit Test是运行在电脑上的，用System.out.print()输出在Run Tab`(Alt+4)`里面。

一般来说，为了做UI测试，需要一个Activity环境，`ActivityTestRule<T>`就是用来初始化这个Activity环境，并且测试开始的时候直接启动指定的Activity，如果指定的Activity启动依赖于Intent，可以使用`ActivityTestRule<T>.launchActivity(Intent startIntent)`启动。在`@Before`里面做一些预备工作。

UI测试一般有三个步骤：**找到View -> 操作View -> 验证View**。有些时候可以不需要操作View，或者说是通过其他的方式改变View，所以操作View可能不会出现。比如我们第一个单元测试`testContent()`就没有操作View。

Espresso库里面带了很多与View相关的方法：

`onView(Matcher<View> viewMatcher) `：用于**找到View**，里面带的`Matcher`称为`ViewMatcher`，[ViewMatchers](https://developer.android.com/reference/android/support/test/espresso/matcher/ViewMatchers.html)里面提供了通用的`Matcher`，用的最多的[`withId(int id)`](https://developer.android.com/reference/android/support/test/espresso/matcher/ViewMatchers.html#withId(int))通过ID找到View。

`check(ViewAssertion viewAssert)`：用于**验证View**，`viewAssert`大多数时候使用`matches(Matcher<? super View> viewMatcher)`就足够满足我们测试需求了。

`perform(ViewAction... viewActions)`：用于**操作View**，`click()`对找到的View触发点击。

需要注意的是**单元测试运行的线程不是主线程**，如果要操作UI，可以使用`UiThreadStatement.runOnUiThread(Runnable)`。这个方法会等到Runnable运行之后再执行后续的测试代码。比如第三个单元测试`testUiThread()`输出结果为：

> waiting
> end UI Thread
> end

如果用自己写的Handler来处理，需要注意保证执行顺序。或者是在单元测试中需要等到某个线程执行完毕，才能执行后续操作的时候，可以使用类似下面的方式：

```java
final Object mutex = new Object();
Handler mainHandler = new Handler(Looper.getMainLooper());
System.out.println("waiting");
mainHandler.post(new Runnable() {
    @Override
    public void run() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        synchronized (mutex) {
            mutex.notify();
        }
        System.out.println("end UI Thread");
    }
});
synchronized (mutex) {
    mutex.wait();
}
System.out.println("end");
```

在测试线程`wait()`，然后在需要等待的线程`notify()`。这里的输出结果和上一个里面一样。

在某些时候，如果直接执行Android Test，Android Studio会误识别为JUnit Test，可以在类名之前加上`@RunWith(AndroidJUnit4.class)`。

---

代码地址：[hello-world](https://github.com/iamjinge/AndroidTestingSample/tree/master/helloworld)

这里只是对Android单元测试进行了初步探索，后面进一步研究之后会针对性的展开。
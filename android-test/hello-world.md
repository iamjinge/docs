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
}
```

一般来说，为了做UI测试，需要一个Activity环境，`ActivityTestRule<T>`就是用来初始化这个Activity环境，并且测试开始的时候直接启动指定的Activity，如果指定的Activity启动依赖于Intent，可以使用`ActivityTestRule<T>.launchActivity(Intent startIntent)`启动。在`@Before`里面做一些预备工作。


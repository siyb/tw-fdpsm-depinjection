% Dependency Injection and View Binding
% Patrick Sturm
% 11.04.2018

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-fdpsm-1-depinjection](https://github.com/siyb/tw-fdpsm-1-depinjection)

# Agenda

## Agenda

* Recap: annotations
* ButterKnife
* Android Data Binding Library

# Annotation Recap

## Recap: Annotations - 1 - Example: Declaration

```java
// accessiblity of annotation
@Retention(RetentionPolicy.RUNTIME)
// this annotation is only applicable to...
@Target(ElementType.FIELD)
// the actual annotation definition
public interface @MyAnnotation {
  // annotation parameter definition
  int someParameter() default 0;
}
```
## Recap Annotations - 2 - Example Explained

* Retention (can be left out):
    * SOURCE: annotation only available in source code (e.g.: compile time checks)
    * CLASS: default, available in class file and source code (e.g.: compile time checks in external lib)
    * RUNTIME: available at runtime, in class file and source code (everything that requires runtime checks)
* Target
    * Defines what kind of construct can be annotated using this annotation (can be left out)
    * Can be a number of things including: FIELD, METHOD and TYPE

## Recap Annotations - 3 - Example: Usage

```java
@MyAnnotation(1)
private Object testField;
```

## Recap Annotations - 4 - Example: Accessing Runtime Annotations

```java
MyAnnotation a = (MyAnnotation) myObject
  .getClass()
  .getDeclaredField("testField")
  .getAnnotation(MyAnnotation.class);		   
// getting our value!
a.testField();
```

# ButterKnife

## Butter Knife - 1 - Intro

* Butter Knife could be described as a micro framework
* It does two things (and does them well):
    * Resource Injection
    * Listener Bindings
* Uses code generation, older versions rely on reflections
* If you want something more sophisticated and less focused, Dagger 2 might be more to your liking

## Butter Knife - 2 - Supported Annotations

* Butter Knife supports the following Annotations:
    * @BindView(s): allows view bindings
    * @Bind[Array,Bitmap,Drawable,Bool,Color,Dimen,Int,String,Float,Font,Anim]: binds resource type
    * @On[Click,Touch,CheckChange,EditorAction,FocusChange,ItemClick,
        ItemLongClick,ItemSelected,LongClick,TextChanged]: mark method as listener for specified action

## Butter Knife - 3 - Mechanism

* ButterKnife generates glue code for you.
* In order to make Butter Knife aware of the annotations, we need to manually bind Butter Knife to the component in question, likewise, we need to unbind Butter Knife when we are done (Fragments)
    * By calling `bind`, you are delegating the binding process to the code generated by ButterKnife
* Use: ButterKnife.bind(targetClass), ButterKnife.unbind(targetClass), Butter Knife also supports providing Activities, Dialogs and Views as secondary parameters, which is important if we want to bind views of Fragments!
* Please note that fields, which we want to inject using Butter Knife must not be `private`

## Butter Knife - 4 - Mechanism cont.

Taken From: [http://jakewharton.github.io/butterknife/](http://jakewharton.github.io/butterknife/)

```java
public void bind(ExampleActivity activity) {
  activity.subtitle = 
    (android.widget.TextView) activity
        .findViewById(2130968578);
  activity.footer = 
    (android.widget.TextView) activity
        .findViewById(2130968579);
  activity.title = 
    (android.widget.TextView) activity
        .findViewById(2130968577);
}
```

# [ButterKnife - CODE EXAMPLE!](https://github.com/SphericalElephant/android-example-butterknife)

# Databinding Library

## Data Binding Library - 1 - Intro

* Part of the support library.
* Eliminates view and event listener binding boilerplate code by generating the glue code for you - similar to what ButterKnife does.
* Supports two different compilers, V2 is not backwards compatible - only an issue with precompiled libraries.

## Data Binding Library - 2 - Build File / Gradle Properties

Enable Data Binding in build file:

```groovy
    android {
       ...
        dataBinding {
            enabled = true
        }
    }
```
Use V2 compiler, gradle.properties:
```
android.databinding.enableV2=true
```

## Data Binding Library - 3 - First Layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android=
    "http://schemas.android.com/apk/res/android">
    <data>

        <variable
            name="myclassalias"
            type="com.test.MyClass"/>
    </data>
    <TextView
       android:id="@+id/myTextView"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:text="@{myclassalias.field}" />
</layout>
```

## Data Binding Library - 4 - Corresponding Activity

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   ActivityMainBinding binding = DataBindingUtil
       .setContentView(this, R.layout.activity_main);
   // while setting an id is not required, doing so
   // allows access to the View via the binding.
   binding.myTextView.getId(); 
   MyClass myClass = new MyClass("some data");
   binding.setMyClass(myClass);
}
```

## Data Binding Library - 5 - Explanation - 1

* Layout must be encapsulated using the "layout" element.
* Data binding must be configured in "data -> variable" element.
    * Compiletime type safety
    * Careful: multi-layout variable declarations (e.g. layout-port and layout-land) must be unique, otherwise they cause errors on code generation.
    * ```context``` variable is created for you.
* Expressions are defined using @{expression}.
    * Expressions automatically checks for null pointer access and prevents crashes by using default values instead.

## Data Binding Library - 6 - Explanation - 2

* The Data Binding Library will generate a ${layout}Binding class for you, that you must use to create the binding.
    * Getters and setters are generated for each of the variables that you defined.
    * You can use the "class" attribute of data to override the default generated name of the binding class.
* Properties that you are referring to must either be publicly accessible via an accessor method or must be declared public.
    * ```public String foo()``` or ```public String foo``` would work.
    * Make sure to provide correct access methods, e.g. if you want to bind an float to a TextView, you need to provide the appropriate getter.
* Bind the data directly after View inflation to prevent problematic changes of the layout.

## Data Binding Library - 7 - Advanced Expressions - 1

* The "data -> import" element may be used to import Java classes...
    * Features alias support to circumvent naming conflicts.
    * Allows accessing static fields and methods.
* Similary, ```<include />``` can be configured to pass on bindings, using the ```bind``` attribute.

## Data Binding Library - 8 - Advanced Expressions - 2 - Example

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <import type="com.test.Info">
        <variable
            name="myclassalias"
            type="com.test.MyClass"/>
    </data>
    <TextView
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:text=
           "@{info.MORE_INFO ? 
                myclassalias.moreInfoField : 
                myclassalias.lessInfoField}" />
</layout>
```

## Data Binding Library - 9 - Advanced Expressions - 3 - Operators and Advanced References

* The Data Binding Library supports various Java operators
    * Arithmetic operators, String concatination, Logical, Binary, Tenary and Unary operators, Shifting operators and Comparisons operators.
    * Support for ?? - x ?? y -> x != null ? x : y
* It also supports
    * Casting, calling methods, accessings fields (including arrays) and the use of various literals
* It does not support access to: this, super, new, generics
* You can also reference other resources within your expressions:
    * android:text="@{user.isAdmin() ? @string/such_admin_wow : @string/puny_user}"
* CAREFUL: Make sure not to violate SOC (separation of concerns) by introducing overly complex expressions!

## Data Binding Library - 10 - Advanced Expressions - 3 - Collection Access and Listener Bindings 

* Using ```@{myCollection["item"]}``` you can access individual items of a collection, indecies are also supported.
* The Data Binding Library also supports method references and listener bindings


## Data Binding Library - 11 - Method References Example - 1

```java
public class MyClickListener {
    // careful: method signature!
    public void onClick(View v) {
        // code here
    }
}
```

## Data Binding Library - 12 - Method References Example - 2

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="listener"
            type="com.test.MyClickListener"/>
    </data>
    <Button
       android:onClick="@{listener::onClick}" />
</layout>
```

## Data Binding Library - 13 - Explanation

* Uses "Java Style" method references
* Make sure that the signature of the callback matches the indended callback of the actual listener interface that you are binding.

## Data Binding Library - 14 - Listener Example - 1

```java
public class MyListener {
    // careful: method signature!
    public void onClick(MyClass c) {
        // code here
    }
}
```

## Data Binding Library - 15 - Listener Example - 2

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="listener"
            type="com.test.MyListener"/>
        <variable 
            name="myClass"
            type="com.test.MyClass">
    </data>
    <Button
       android:onClick="@{() -> listener.onClick(myClass)}" />
</layout>
```

## Data Binding Library - 16 - Explanation

* Uses "Java Style" lambda expressions
    * You can either choose to supply or ommit all parameters of the lambda expression
    * Make sure that your handler returns the data expected by the original listener!
* You may choose to implement your listener interface as you wish (e.g. multiple parameters)

## Data Binding Library - 17 - Two Way Binding

* This is where things get really interesting.
* We have only looked at regular POJO binding so far:
    * As soon as the binding has been established, successive changes to the POJO will refresh the UI.
    * Two Way binding facilitates real time refreshes of UI based on the state of the POJO.
* You need to implement the ```Observable``` interface or extend the ```BaseObservable``` convenience implementation to make your bindings two way.

## Data Binding Library - 18 - Example

```java
public MyClass extends BaseObservable {
    private String myString;

    @Bindable public String getMyString() {
        return myString;
    }
    public String setMyString(String myString) {
        this.myString = myString;
         notifyPropertyChanged(BR.myString);
    }
}
```

## Data Binding Library - 19 - Explanation

* All bindable access methods must be declared @Bindable
    * Careful: only methods that adhere to the JavaBeans spec are supported!
* The Databinding Library generates the BR class, containing references of bindable properties
* Using ```notifyPropertyChanged``` the specified property is marked as dirty and thus, can be refreshed in the UI.

## Data Binding Library - 20 - Observable Types

* The Data Binding Library also supports observable types.
    * ObservableField<T> - generic, observes the field's value only, does not detect state changes in the field itself!
    * Observable[Byte,Char,Short,Int,Long,Float,Double,Fields,
        Map,ArrayMap,ArrayList]
* No manual wiring necessary, but access is tedious and the use of specialized fields might seem to intrusive to some.

## Data Binding Library - 21 - Criticism

* Possible issues when writing unit tests due increased complexibility
* Cluttered (view)model classes
* Slowing down of build process

## Data Binding Library - 22 - Advanced Features

* Please check out: ViewStubs, Converters, Custom Setters and Attribute Setters at home ;)

# [Data Binding Library - Code Example](https://github.com/SphericalElephant/android-example-databinding)

# Any Questions?
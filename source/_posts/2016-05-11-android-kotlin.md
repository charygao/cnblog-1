title: Android, Kotlin and other JVM languages
date: 2016/05/01 12:00
authorId: JMT
tags: [kotlin, android, jvm languages]
---

Since the dawn on Android development, there have always been efforts to use some of the alternative JVM languages instead of plain Java. But given the constraints of the mobile world, most of them faced issues that prevented them from receiving large adoption. [**Kotlin**](https://kotlinlang.org/), a new language developed by JetBrains with Android in mind, aims to change this.

<!-- more -->

## Android specifics

Although the language for writing apps is Java, Android has always been a bit different than a typical server-side Java development environment. The virtual machine (originally Dalvik; ART since KitKat) is not a regular JVM. It doesn't support the *InvokeDynamic* instruction, so dynamic languages need to resort to reflection, which is still quite slow even nowadays (although the situation is not as bad as it used to be at the time of single-core phones with 512 MB of RAM). Runtime code generation is a problem as well, because the byte code has to be written to and read from the file system which is again very slow (it can delay the app starts by several seconds).

Because of this, the obvious Java alternatives such as **Groovy** or JRuby have always struggled to be of practical use. However, Groovy's [2.4 release](http://groovy-lang.org/releasenotes/groovy-2.4.html) finally brings official Android support, such as a Gradle plugin and special *grooid* jar variants optimized for mobile usage. In order to avoid the above-mentioned performance issues, it's still recommended to use static compilation wherever possible and limit the bytecode size using ProGuard, but Groovy's expressiveness can still significantly reduce the typical Android boilerplate.

## Java 8

Language-wise, Android is lagging behind official Java as well. The syntax-sugar features of Java 7 (such as diamond operator or switch on strings) have been available for quite some time, but any API-based language features are still dependent on the platform version supported by the given app. The *try-with-resources* block for example requires API level 19 (KitKat), but the various Jelly Bean versions still occupy about 20% of the market.

Even if we ignore the APIs that make no sense on Android such as Swing or CORBA, it is still not fully Java 7 compliant at the moment: The ForkJoin framework has been implemented in API 21, but the NIO.2 is still missing. The reason for this is that Google based the Android SDK on *Apache Harmony* - an alternative, but currently dead JRE implementation, so they have been forced to reimplement most of the the new language and library features from scratch.

But the good news is that Google and Oracle recently announced that they finally agreed on adopting OpenJDK for the Android SDK and the first results can already be seen in the [Android N preview](http://developer.android.com/preview/j8-jack.html). Thanks to the new Jack compiler, lambda expressions and method references can finally be used! They are compiled into traditional anonymous classes, so it's possible to backport them even to older platform versions in the same manner as other syntactic sugar. Default and static interface methods are supported as well, and currently, at least the whole Stream API is implemented and we can possibly expect most or all of the Java 7/8 APIs to be included in the final N release. However, it will still take a couple of years before the app developers can require N as the minimum SDK level. And in the end, even Java 8 is still only Java and we can do better than that, can't we?

## The Babylon of languages

We've already covered Groovy, so what other options are there? A popular Java alternative in the server world is **Scala**, which blends OOP with functional programming into a very powerful, yet type-safe language. Being statically-typed, it doesn't suffer as much perfomance-wise (although it relies on higher-order constructs much more than Java) and the size of the standard library (which includes a massive collection framework) isn't much of a problem anymore. The real issue can come from the complexity of using an academia-based language that supports every feature ever created. Interoperability with Java is mostly one-way and the tooling support (e.g. Gradle integration) isn't first class as well. But still, the boilerplate reduction with the flexibility and power of the language (on par with Groovy) can probably outweigh this.

Another JVM language whose popularity is on the rise, is **Clojure**. Like Groovy, it has to pay the price for dynamism, but the optimizing compiler named *Skummet* aims to address this. Clojure seems a bit like a world of its own, not only syntactically, but also from the tooling point of view, so in the end, the discussion is not as much about Android developers using Clojure, but Clojure developers writing apps for Android.

A direct competition to Kotlin is **Ceylon**, also a recent statically-typed language developed by Gavin King (the author of Hibernate) at Red Hat. One of its main features is a strong type system without the complexity of Scala. Similarly to Kotlin, it supports nullable types such as `String?`, whose value can either be a String or null (as opposed to `String` which is compile-time checked not to contain null). However, in case of Ceylon, this is just an alias of `String | Null`, which is a union type. We can know them from Java's multicatch blocks, but here, they are a first-class language feature. One of the issues with Ceylon is the tooling support: targeting Android is not its priority and the official Gradle plugin is currently only in version `0.0.2`.

## Fun with Kotlin

So how does **Kotlin** stand in all this? According to the official website, it's a *Statically typed programming language for the JVM, Android and the browser* and also *100% interoperable with Java*.  When looking into the notes for the recent [1.0 release](https://blog.jetbrains.com/kotlin/2016/02/kotlin-1-0-released-pragmatic-language-for-jvm-and-android/), it's clear that the main idea behind the its design is **pragmatism**: Rather than offering lots of fancy and magical language features or a massive standard library, it just tackles the most painful issues with Java, such as null handling, lack of properties and the inability to add methods to 3rd party classes, while maintaining bidirectional compatibility with existing Java code and providing great tooling support as well. In the end, the language just feels like "a better Java".

One of the main selling points of Kotlin are *(not)nullable types* (but without the union types as in Ceylon), where nullness of values is explicit and null-checking is enforced by the compiler, so that the dreaded `NullPointerException` is nearly impossible to be thrown at runtime when developing in pure Kotlin. However, because we would like to be able to reuse existing libraries and frameworks written in Java (which don't have any nullness information on them), we would be forced to treat all of their APIs as nullable, even though many methods' contracts guarantee to never return null. Because of that, JetBrains have made the *pragmatic* choice to relax the restriction a bit and to allow assigning values coming from Java into not-nullable types. However, the compiler places an assertion at the place of assignment which stops the null from propagating further.

Kotlin also has language support for object *properties* and it's possible not only to use such syntax when accessing Java-based getters and setters, but even using them from the Java side is not a problem, because they get compiled as a pair of `get`/`set` methods.

Another feature implemented with Java interoperability in mind are *extension methods*. They are an easy way to add methods to existing classes that we can't modify or extend (such as 3rd party libraries). They are actually compiled into static methods where the first parameter is the type being extended, so from Java, they actually look like ordinary static utility methods. In order to do so, we define a package-level function whose name is prefixed by the name of the extended type:

```kotlin
// StringExtensions.kt

@file:JvmName("StringUtils")
package my.extensions

fun String.reverse() = StringBuilder(this).reverse().toString()
```

Calling that method from Kotlin is then as simple as:

```kotlin
import my.extensions.reverse // importing a function, not a class

"foo".reverse()
```

In Java, we import the generated class that the package function was compiled into. The default name is derived from the file name, so it would be `StringExtensionsKt` in this case, but we can use the above annotation to specify a more friendly looking name:

```java
import my.extensions.StringUtils;

StringUtils.reverse("foo");
```

Given that Kotlin is developed by the authors of IntelliJ IDEA and its derivates, great tooling support is no surprise. Once we download the official plugin into our Android Studio or IntelliJ, enabling Kotlin support in the project is as simple as clicking the *Configure Kotlin in Project* command in the Tools menu. This will add the Gradle plugin and the required dependencies into the build file. After that, we can start writing Kotlin classes either in the `src/main/kotlin` directory or we can put them directly into `src/main/java` along with the existing Java classes. But that's not all, the IDE even provides a convenient action that automatically converts existing Java classes into Kotlin! However, as we'll see in the following exercise, such code sometimes requires a bit of manual fine-tuning to get it working.

{% img center-block /attachments/2016-05-11-android-kotlin/convert-to-kotlin.png %}

# Code example

We'll illustrate the process of converting Java to Kotlin on a very simplistic [TODO app](https://github.com/natix643/kotlin-todo). It only contains a single listview of todo items, each with a text and a checkbox to mark it's completion. Pressing the plus button opens a dialog with text input for adding a new todo. The items can be deleted either individually or it's possible to delete all completed todos using the overflow menu.

{% img center-block /attachments/2016-05-11-android-kotlin/todo-app.png 400 %}

We'll take off from the `master` branch, which contains only Java, and gradually convert all classes into Kotlin while trying not to break any functionality. The `kotlin` branch already contains a working result.
 
## Todo class

We'll start simply:

```java
// Todo.java

public class Todo {

    private final String text;
    private boolean completed;

    public Todo(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    public boolean isCompleted() {
        return completed;
    }

    public void setCompleted(boolean completed) {
        this.completed = completed;
    }
}
```

The model for a todo item is implemented using a very simple value class with only 2 properties. But do we really need so much code for that? Let's try out the conversion magic:

```kotlin
// Todo.kt

class Todo(val text: String) {
    var isCompleted: Boolean = false
}
```

If you've ever written any Scala, this code probably looks more than familiar. Kotlin has removed all the boilerplate, so properties are declared only once, either in the class body, or (if initialized by the constructor) directly in the class header. Classes are also public by default. An interesting detail is that properties don't have default values (such as `null`, `false` or zero), so we must manually initialize the value of `isCompleted`.

The cool thing is that all the other Java classes remain working without any problems, the getters, setters and the constructor are still there in the bytecode, we just don't need to write them explicitly. This means that we can just hit **Run** and the app still works.

## TodoAdapter class

```java
// TodoAdapter.java

public class TodoAdapter extends BaseAdapter {

    private final LayoutInflater inflater;

    private List<Todo> items = new ArrayList<>();

    public TodoAdapter(Context context) {
        this.inflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return items.size();
    }

    @Override
    public Todo getItem(int position) {
        return items.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    public List<Todo> getItems() {
        return items;
    }

    public void add(Todo todo) {
        items.add(todo);
        notifyDataSetChanged();
    }

    public void remove(int position) {
        items.remove(position);
        notifyDataSetChanged();
    }

    public void removeAll(Collection<Todo> todos) {
        items.removeAll(todos);
        notifyDataSetChanged();
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        final Todo todo = getItem(position);

        View layout = inflater.inflate(R.layout.item, parent, false);

        final TextView textView = (TextView) layout.findViewById(R.id.todoText);
        textView.setText(todo.getText());

        CheckBox checkBox = (CheckBox) layout.findViewById(R.id.todoCompleted);
        checkBox.setChecked(todo.isCompleted());
        checkBox.setOnCheckedChangeListener(new OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
                todo.setCompleted(isChecked);

                int flags = textView.getPaintFlags();
                if (isChecked) {
                    textView.setPaintFlags(flags | Paint.STRIKE_THRU_TEXT_FLAG);
                } else {
                    textView.setPaintFlags(flags ^ Paint.STRIKE_THRU_TEXT_FLAG);
                }
            }
        });

        return layout;
    }

}
```

This is a classic Android list adapter. It wraps a list of todos and implements  the `getView()` method which creates a layout for each of the items. For the sake of simplicity of the demo, this app doesn't persist its data anyhow, so the adapter is actually the only holder of the data model and all creating and deleting of todos is simply done through the adapter's methods that just modify the underlying list. The translation gives us this:

```kotlin
// TodoAdapter.kt

class TodoAdapter(context: Context) : BaseAdapter() {

    private val inflater: LayoutInflater

    private val items = ArrayList<Todo>()

    init {
        this.inflater = LayoutInflater.from(context)
    }

    override fun getCount(): Int {
        return items.size
    }

    override fun getItem(position: Int): Todo {
        return items[position]
    }

    override fun getItemId(position: Int): Long {
        return position.toLong()
    }

    fun getItems(): List<Todo> {
        return items
    }

    fun add(todo: Todo) {
        items.add(todo)
        notifyDataSetChanged()
    }

    fun remove(position: Int) {
        items.removeAt(position)
        notifyDataSetChanged()
    }

    fun removeAll(todos: Collection<Todo>) {
        items.removeAll(todos)
        notifyDataSetChanged()
    }

    override fun getView(position: Int, convertView: View, parent: ViewGroup): View {
        val todo = getItem(position)

        val layout = inflater.inflate(R.layout.item, parent, false)

        val textView = layout.findViewById(R.id.todoText) as TextView
        textView.text = todo.text

        val checkBox = layout.findViewById(R.id.todoCompleted) as CheckBox
        checkBox.isChecked = todo.isCompleted
        checkBox.setOnCheckedChangeListener { buttonView, isChecked ->
            todo.isCompleted = isChecked

            val flags = textView.paintFlags
            if (isChecked) {
                textView.paintFlags = flags or Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                textView.paintFlags = flags xor Paint.STRIKE_THRU_TEXT_FLAG
            }
        }

        return layout
    }

}
```

Except for the missing semicolons and a nice lambda expression in the place of `OnCheckedChangeListener`, not much has actually changed, but there are a few places that we can improve by hand.

We can initialize the inflater inline instead of the `init` block:

```kotlin
private val inflater = LayoutInflater.from(context)
```

The `items` private field with a getter can be replaced with a public property. We could omit the type declaration, but that would make its type `ArrayList`, so if we want to declare `java.util.List` as usual, we need to use its kotlin counterpart `kotlin.collections.MutableList` (`kotlin.collections.List` is an immutable interface for contrast).

```kotlin
val items : MutableList<Todo> = ArrayList()
```

And all the single line method bodies can be replaced with expressions. No return type declaration is needed in such case. Note that list supports the array operator and that there is no implicit numeric conversion from `Int` to `Long` which makes Kotlin's type system stricter than in most strongly typed languages.

```kotlin
 override fun getCount() = items.size
 override fun getItem(position: Int) = items[position]
 override fun getItemId(position: Int) = position.toLong()
```

However, if we run the app now and try to add an item, it crashes with this exception:

```
java.lang.IllegalArgumentException: Parameter specified as non-null is null: method kotlin.jvm.internal.Intrinsics.checkParameterIsNotNull, parameter convertView
    at cz.natix.todo.TodoAdapter.getView(TodoAdapter.kt:0)
```

Unlike the above auto-translated code, which could be cosmetically improved, but otherwise worked correctly, null handling is currently something that needs manual verification and correction. The tool generates the `getView()` method header with all types not-nullable, however the value of `convertView` parameter can be null, so we need to add a question mark to its type:

```kotlin
override fun getView(position: Int, convertView: View?, parent: ViewGroup): View
```

## TextInputDialog class

```java
// TextInputDialog.java

public class TextInputDialog extends DialogFragment {

    public interface Callback {
        void onTextInput(String text);
    }

    private Callback callback;

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        callback = (Callback) activity;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        View layout = LayoutInflater.from(getActivity()).inflate(R.layout.dialog, null);
        final EditText editText = (EditText) layout.findViewById(R.id.editText);

        return new AlertDialog.Builder(getActivity())
                .setTitle(R.string.dialog_title)
                .setView(layout)
                .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        callback.onTextInput(editText.getText().toString());
                    }
                })
                .setNegativeButton(android.R.string.cancel, null)
                .create();
    }
}
```

The dialog with a text field for adding a new item is implemented using a dialog fragment, which uses the verbose and cumbersome (yet standard) pattern of defining a callback interface that the creating activity must implement. In the `onAttach()` method, we cast the activity to the callback and in the OK button listener, we pass the contents of the textfield it. Sadly, Kotlin itself can't save us from this boilerplate.

```kotlin
// TextInputDialog.kt

class TextInputDialog : DialogFragment() {

    interface Callback {
        fun onTextInput(text: String)
    }

    private var callback: Callback? = null

    override fun onAttach(activity: Activity) {
        super.onAttach(activity)
        callback = activity as Callback
    }

    override fun onCreateDialog(savedInstanceState: Bundle): Dialog {
        val layout = LayoutInflater.from(activity).inflate(R.layout.dialog, null)
        val editText = layout.findViewById(R.id.editText) as EditText

        return AlertDialog.Builder(activity)
                .setTitle(R.string.dialog_title)
                .setView(layout)
                .setPositiveButton(android.R.string.ok) { dialog, which ->
                    callback!!.onTextInput(editText.text.toString())
                }
                .setNegativeButton(android.R.string.cancel, null)
                .create()
    }
}
```

After the previous lesson, we won't be much surprised by the fact that the `savedInstanceState` parameter needs to be redeclared as nullable. However, there's also the opposite case: because the  `callback` property is initialized in a method and not the in constructor, the translator declared it as nullable. This means that in order to access it, we must either:

 1. check it for null
 2. bypass null safety using the `!!` operator as in the code above

Because the `onAttach()` method is guaranteed to be called by the framework before anything else and in a sense serves as a constructor, we can safely assume that `callback` is actually not nullable, so option 1 isn't suitable. However, using visually frightening things such as `!!` is not good for your code review, so what can we do?

There is actually a third option: changing the `callback` property to not-nullable isn't possible on its own (it must be initialized inline), but we can declare it as _late init_:

```kotlin
private lateinit var callback: Callback
```

This allows us to have not-nullable properties that are safely initialized by other means than the constructor, such as framework callbacks or dependency injection. You might object that this is like throwing the null-safety away and no better than sticking to Java and its NPEs, but there is actually a subtle difference:

 - Even after a nullable property has a value, it's still possible to reassign null to it again at any point in the program. Once a late init property has been initialized, it can never become "un-initialized" again.
 - When a late init property is accessed before its initialization, an exception with a very specific message is thrown instead of a plain NullPointerException.

In the end, this feature is not intended for bypassing the type system, but for situations where the logic of the program safely guarantees that the initialization of the properties always happens before accessing them, but the compiler just cannot prove it.

### MainActivity class

```java
// MainActivity.java

public class MainActivity extends AppCompatActivity implements TextInputDialog.Callback {

    private TodoAdapter adapter;

    private ListView listView;
    private TextView emptyListText;
    private FloatingActionButton floatingButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        listView = (ListView) findViewById(android.R.id.list);
        emptyListText = (TextView) findViewById(android.R.id.empty);

        floatingButton = (FloatingActionButton) findViewById(R.id.floatingButton);
        floatingButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                new TextInputDialog().show(getFragmentManager(), "");
            }
        });

        adapter = new TodoAdapter(this);

        listView.setAdapter(adapter);
        listView.setEmptyView(emptyListText);
        registerForContextMenu(listView);
    }

    @Override
    public void onTextInput(String text) {
        adapter.add(new Todo(text));
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        if (item.getItemId() == R.id.menu_delete_completed) {
            Collection<Todo> completedTodos = Collections2.filter(adapter.getItems(), isCompleted);
            adapter.removeAll(completedTodos);
            return true;
        } else {
            return false;
        }
    }

    @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenuInfo menuInfo) {
        getMenuInflater().inflate(R.menu.context, menu);
    }

    @Override
    public boolean onContextItemSelected(MenuItem item) {
        if (item.getItemId() == R.id.menu_delete) {
            AdapterContextMenuInfo info = (AdapterContextMenuInfo) item.getMenuInfo();
            adapter.remove(info.position);
            return true;
        } else {
            return false;
        }
    }

    private static final Predicate<Todo> isCompleted = new Predicate<Todo>() {
        @Override
        public boolean apply(Todo todo) {
            return todo.isCompleted();
        }
    };

}
```

And last but not least, we have an activity that wires all the previous code together: initializes the listview, binds it with the adapter and adds creating of the dialog to the listener of the floating button. Also the dialog's callback is implemented by the `onTextInput()` method.

Apart from that, the activity also registers the list items for a context menu with a command for individual deletion. And finally, the options menu provides a command to delete all completed todos at once. Because we're stuck with Java ~ 7 at moment, we use Guava's `Predicate` and `filter()` to simplify selection of the adapter's items.

```kotlin
// MainActivity.kt

class MainActivity : AppCompatActivity(), TextInputDialog.Callback {

    private var adapter: TodoAdapter? = null

    private var listView: ListView? = null
    private var emptyListText: TextView? = null
    private var floatingButton: FloatingActionButton? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main)

        listView = findViewById(android.R.id.list) as ListView?
        emptyListText = findViewById(android.R.id.empty) as TextView?

        floatingButton = findViewById(R.id.floatingButton) as FloatingActionButton?
        floatingButton!!.setOnClickListener { TextInputDialog().show(fragmentManager, "") }

        adapter = TodoAdapter(this)

        listView!!.adapter = adapter
        listView!!.emptyView = emptyListText
        registerForContextMenu(listView)
    }

    override fun onTextInput(text: String) {
        adapter!!.add(Todo(text))
    }

    override fun onCreateOptionsMenu(menu: Menu): Boolean {
        menuInflater.inflate(R.menu.main, menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.menu_delete_completed) {
            val completedTodos = Collections2.filter(adapter!!.items, isCompleted)
            adapter!!.removeAll(completedTodos)
            return true
        } else {
            return false
        }
    }

    override fun onCreateContextMenu(menu: ContextMenu, v: View, menuInfo: ContextMenuInfo) {
        menuInflater.inflate(R.menu.context, menu)
    }

    override fun onContextItemSelected(item: MenuItem): Boolean {
        if (item.itemId == R.id.menu_delete) {
            val info = item.menuInfo as AdapterContextMenuInfo
            adapter!!.remove(info.position)
            return true
        } else {
            return false
        }
    }

    companion object {

        private val isCompleted = Predicate<cz.natix.todo.Todo> { todo -> todo!!.isCompleted }
    }

}
```

This time, the translated code runs without any exceptions (the `savedInstanceState` parameter is correctly defined as `Bundle?`), but again, we can improve it a bit. Because all the properties are initialized in the `onCreate()` method, we could change them to late init as in the previous class, but there is also another possibility: because the properties don't actually depend on the method parameter (only on the state of the activity), it's possible to initialize them inline, but lazily. After then, we can remove all of the ugly exclamation marks from the calling code.

```kotlin
private val adapter by lazy { TodoAdapter(this) }
private val listView by lazy { findViewById(android.R.id.list) as ListView }
private val emptyListText by lazy { findViewById(android.R.id.empty) as TextView }
private val floatingButton by lazy { findViewById(R.id.floatingButton) as FloatingActionButton }
```

An interesting detail here is that the `isCompleted` constant is placed in a _companion object_. This is another similarity with Scala: instead of declaring methods and fields as static, Kotlin places them as members of singleton classes (denoted by the `object` keyword instead of `class`). In case a class contains both instance and static methods, the static ones are placed into a so called companion singleton with the same name as the class. However in this case, we don't really need that, we can place the property directly into the activity class. Well, actually we don't need even that. Because Kotlin's standard library defines a lot of extension methods to Java types, we can call `filter()` directly on the list. The single argument lambda can use  the `it` keyword known from Groovy.

```kotlin
val completedTodos = adapter.items.filter { it.isCompleted }
```

The last things to improve are the callback methods for context and options menu. If-else blocks are treated like expressions (there is no need for ternary operators), so we can simplify the context menu handler to this:

```kotlin
override fun onContextItemSelected(item: MenuItem) =
        if (item.itemId == R.id.menu_delete) {
            val info = item.menuInfo as AdapterContextMenuInfo
            adapter.remove(info.position)
            true
        } else false

```

We could do the same with handler of the options menu, but for the sake of exercise, we'll use a `when` block. It's a more powerful version of the classic `switch` that is based on _pattern matching_ known from functional programming. The branches can match much more complex expressions than just primitive values, enums and strings. For example, there can be multiple values in one branch condition, therefore there is no need for `break` at the end. Each branch and the whole block in the end also serve as an expression.

```kotlin
override fun onOptionsItemSelected(item: MenuItem) = when (item.itemId) {
    R.id.menu_delete_completed -> {
        val completedTodos = adapter.items.filter { it.isCompleted }
        adapter.removeAll(completedTodos)
        true
    }
    else -> false
}
```

# Conclusion

We've seen that there is quite a little cost when adopting Kotlin even for existing Android projects and the tooling support and interoperability with Java don't fail the expectations. The only issue is the automatic conversion feature that shouldn't be used without manual review of the generated code, but it can still make the transition from Java a bit easier. In the end, the resulting Kotlin code doesn't seem too different from the original Java one, but given the bigger power of the language, we can expect new libraries to appear in the future that will reduce the Android boilerplate and make the life of developers easier.

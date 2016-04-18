# Android Training

跟着google developers官网学习android


### Adding the App Bar

how to use the v7 appcompat support library's Toolbar widget as an app bar.

**what is Toolbar (v7 appcompat support library's Toolbar) ?**  
`public class Toolbar extends ViewGroup`

**why use Toolbar  ?**  
using the appcompat Toolbar makes it easy to set up an app bar that works on the widest range of devices, and also gives you room to customize your app bar later on as your app develops.


##### Setting Up the App Bar 设置一个简单AppBar

1.Add the the v7 appcompat support library to your project, as described in Support Library Setup.  
加依赖。（添加 v7 appcompat support library）

```
dependencies {
  compile 'com.android.support:appcompat-v7:23.2.1'
}
```

2.Make sure the activity extends AppCompatActivity:  
改继承：（使用AppCompatActivity替换Activity）
```
public class MyActivity extends AppCompatActivity {
  // ...
}
```

3.In the app manifest, set the `<application>` element to use one of appcompat's NoActionBar themes. Using one of these themes prevents the app from using the native ActionBar class to provide the app bar. For example:  
添样式：（默认样式已经设置了ActionBar）

```xml
<application
    android:theme="@style/Theme.AppCompat.Light.NoActionBar"/>
```

4.Add a Toolbar to the activity's layout.  
使用Toolbar。

```xml
<android.support.v7.widget.Toolbar
   android:id="@+id/my_toolbar"
   android:layout_width="match_parent"
   android:layout_height="?attr/actionBarSize"
   android:background="?attr/colorPrimary"
   android:elevation="4dp"
   android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
   app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```

（1）`android:elevation="4dp"`  
The Material Design specification recommends that app bars have an elevation of 4 dp.  
（2）`android:layout_height="?attr/actionBarSize"`  
Toolbar的layout_height属性，要用“?attr/actionBarSize”而不是“?android:attr/actionBarSize”，替换后可解决NavigationIcon不垂直居中的问题原因是系统的actionBarSize比AppCompat中的要小。使用“?android:attr/actionBarSize”调用了较小的那个。  
（3）`app:popupTheme="@style/ThemeOverlay.AppCompat.Light"`  
有时候我们有需求 :

ActionBar文字是白的，ActionBar Overflow弹出的是白底黑字

让ActionBar文字是白的，那么对应的theme肯定是Dark。
可是让ActionBar弹出的是白底黑字，那么需要Light主题。
这时候`popupTheme`就派上用场了。

5.In the activity's onCreate() method, call the activity's setSupportActionBar() method, and pass the activity's toolbar. This method sets the toolbar as the app bar for the activity.  
设ActionBar（将Toolbar设置为默认的ActionBar）。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_my);
    Toolbar myToolbar = (Toolbar) findViewById(R.id.my_toolbar);
    setSupportActionBar(myToolbar);
    }
```

到此已经实现了一个基本的 作为ActionBar的 Toolbar

##### Adding and Handling Actions 添加和处理操作（按钮）

**1.Add Action Buttons**
All action buttons and other items available in the action overflow are defined in an XML menu resource. To add actions to the action bar, create a new XML file in your project's res/menu/ directory.
所有的操作按钮和在下拉列表中的选项都定义在菜单资源文件中，次文件保存在res/menu文件夹中。

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >

    <!-- "Mark Favorite", should appear as action button if possible -->
    <item
        android:id="@+id/action_favorite"
        android:icon="@drawable/ic_favorite_black_48dp"
        android:title="@string/action_favorite"
        app:showAsAction="ifRoom"/>

    <!-- Settings, should always be in the overflow -->
    <item android:id="@+id/action_settings"
          android:title="@string/action_settings"
          app:showAsAction="never"/>

</menu>
```

（1）`app:showAsAction="ifRoom"`  
the action is displayed as a button if there is room in the app bar for it.如果ActionBar还有空间就显示为一个按钮。
（2）`app:showAsAction="never"`  
the action is always listed in the overflow menu, not displayed in the app bar.一定存在下拉菜单中。
（3）`app:showAsAction="always"`  
一定显示为按钮。

**2.Where add menu in code**
```java
@Override public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(your_Menu_resource_Id,menu);
    return true;
}
```

**3.Respond to Actions**
```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.action_settings:
            // User chose the "Settings" item, show the app settings UI...
            return true;

        case R.id.action_favorite:
            // User chose the "Favorite" action, mark the current item
            // as a favorite...
            return true;

        default:
            // If we got here, the user's action was not recognized.
            // Invoke the superclass to handle it.
            return super.onOptionsItemSelected(item);

    }
}
```

##### Adding an Up Action 添加返回上一页面（Activity）功能

**1.Declare a Parent Activity 声明父activity**

通过添加
`<meta-data
    android:name="android.support.PARENT_ACTIVITY"
    android:value="com.example.myfirstapp.MainActivity" />`
为Activity指明父Activity，

```xml
<application ... >
    ...

    <!-- The main/home activity (it has no parent activity) -->

    <activity
        android:name="com.example.myfirstapp.MainActivity" ...>
        ...
    </activity>

    <!-- A child of the main activity -->
    <activity
        android:name="com.example.myfirstapp.MyChildActivity"
        android:label="@string/title_activity_child"
        android:parentActivityName="com.example.myfirstapp.MainActivity" >

        <!-- Parent activity meta-data to support 4.0 and lower -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
```

然后在代码中：`getSupportActionBar().setDisplayHomeAsUpEnabled(true)`

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_my_child);

    // my_child_toolbar is defined in the layout file
    Toolbar myChildToolbar =
        (Toolbar) findViewById(R.id.my_child_toolbar);
    setSupportActionBar(myChildToolbar);

    // Get a support ActionBar corresponding to this toolbar
    ActionBar ab = getSupportActionBar();

    // Enable the Up button
    ab.setDisplayHomeAsUpEnabled(true);
}
```

**2.比较简单的方法：直接在代码中finish()**

```java
protected void onCreate(Bundle savedInstanceState) {
  ...
  getSupportActionBar().setDisplayHomeAsUpEnabled(true);
  ...
}
@Override
  public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
      case android.R.id.home:
        finish();
        return true;
      default:
        return super.onOptionsItemSelected(item);
    }
}
```

##### Add an Action View 使用Action View

To add an action view, create an <item> element in the toolbar's menu resource, as Add Action Buttons describes. Add one of the following two attributes to the <item> element:  

* `actionViewClass`: The class of a widget that implements the action.  
* `actionLayout`: A layout resource describing the action's components.  

Set the  `showAsAction` attribute to either "ifRoom|collapseActionView" or "never|collapseActionView". The collapseActionView flag indicates how to display the widget when the user is not interacting with it: If the widget is on the app bar, the app should display the widget as an icon. If the widget is in the overflow menu, the app should display the widget as a menu item. When the user interacts with the action view, it expands to fill the app bar.

For example, the following code adds a SearchView widget to the app bar:   
```xml
<item android:id="@+id/action_search"
     android:title="@string/action_search"
     android:icon="@drawable/ic_search"
     app:showAsAction="ifRoom|collapseActionView"
     app:actionViewClass="android.support.v7.widget.SearchView" />
```
If the user is not interacting with the widget, the app displays the widget as the icon specified by android:icon. (If there is not enough room in the app bar, the app adds the action to the overflow menu.) When the user taps the icon or menu item, the widget expands to fill the toolbar, allowing the user to interact with it.

If you need to configure the action, do so in your activity's onCreateOptionsMenu() callback. You can get the action view's object reference by calling the static getActionView() method. For example, the following code gets the object reference for the SearchView widget defined in the previous code example:  
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main_activity_actions, menu);

    MenuItem searchItem = menu.findItem(R.id.action_search);
    SearchView searchView =
            (SearchView) MenuItemCompat.getActionView(searchItem);

    // Configure the search info and add any event listeners...

    return super.onCreateOptionsMenu(menu);
}
```

##### Responding to action view expansion

f the action's <item> element has a `collapseActionView `flag, the app displays the action view as an icon until the user interacts with the action view. When the user clicks on the icon, the built-in handler for `onOptionsItemSelected() `expands the action view. If your activity subclass overrides the `onOptionsItemSelected()` method, your override method must call `super.onOptionsItemSelected()` so the superclass can expand the action view.

If you want to do something when the action is expanded or collapsed, you can define a class that implements `MenuItem.OnActionExpandListener`, and pass a member of that class to setOnActionExpandListener(). For example, you might want to update the activity based on whether an action view is expanded or collapsed. The following snippet shows how to define and pass a listener:  
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.options, menu);
    // ...

    // Define the listener
    OnActionExpandListener expandListener = new OnActionExpandListener() {
        @Override
        public boolean onMenuItemActionCollapse(MenuItem item) {
            // Do something when action item collapses
            return true;  // Return true to collapse action view
        }

        @Override
        public boolean onMenuItemActionExpand(MenuItem item) {
            // Do something when expanded
            return true;  // Return true to expand action view
        }
    };

    // Get the MenuItem for the action item
    MenuItem actionMenuItem = menu.findItem(R.id.myActionItem);

    // Assign the listener to that action item
    MenuItemCompat.setOnActionExpandListener(actionMenuItem, expandListener);

    // Any other things you have to do when creating the options menu…

    return true;
}
```

##### Add an Action Provider
To declare an action provider, create an <item> element in the toolbar's menu resource, as described in Add Action Buttons. Add an actionProviderClass attribute, and set it to the fully qualified class name for the action provider class.  

For example, the following code declares a ShareActionProvider, which is a widget defined in the support library that allows your app to share data with other apps:
```xml
<item android:id="@+id/action_share"
    android:title="@string/share"
    app:showAsAction="ifRoom"
    app:actionProviderClass="android.support.v7.widget.ShareActionProvider"/>
```
In this case, it is not necessary to declare an icon for the widget, since ShareActionProvider provides its own graphics. If you are using a custom action, declare an icon.
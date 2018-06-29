![alt text](https://i.ytimg.com/vi/hc6pJUEgzj8/maxresdefault.jpg)

## Agenda
1. Enabling DataBinding.
2. Basic Example.
3. My DataBinding classes are not generated?
4. DataBinding in layouts.
5. Binding Click Listeners / Event Handling.
6. Updating UI using Observables.
7. Updating UI using ObservableFields.
8. Loading Images From URL (Glide or Picasso).

# DataBinding Basics

## Enabling DataBinding

To get started with DataBinding, you need to enable this feature in your android project first. Open the build.gradle located under app and enable dataBinding under android module. Once enabled, Sync the project and you are good to go.

```
app/build.gradle
android {
    dataBinding {
        enabled = true
    }

    compileSdkVersion 27

    defaultConfig {
        applicationId "com.knrmalhotra.databindingdevfest"
        minSdkVersion 16
        // ..
    }
}
```

## Basic Example

Let’s say we want to display user information from a **User POJO** class. We generally display the info in a **TextView** using **setText()** method. But instead of manually calling setText for each user property, DataBinding allows us to bind the values automatically.

The below POJO class creates an User object with name and email.

```
public class User {
    String name;
    String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

To enable DataBinding in a layout, the root element should start with <layout> tag. Along with it, <data> and <variable> tags are used.

Below is the structure of data-binding layout.

```
<layout ...>

    <data>

        <variable
            name="..."
            type="..." />
    </data>

    <LinearLayout ...>
       <!-- YOUR LAYOUT HERE -->
    </LinearLayout>
</layout>
```

- The layout should have <layout> as root element. Inside <layout> the usual code of layout will be placed.
- A <data> tag follows the <layout>. All the binding variables and methods should go inside <data> tag.
- Inside <data> tags, a variable will be declared using <variable> tag. The variable tag takes two attributes `name` and `type`. name attribute will be alias name and type should be of object model class. In our case, path to User class.
- To bind a value, @ annotation should be used. In the below layout, user name and email are bound to TextView using @{user.name} and @{user.email}

```
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="com.knrmalhotra.databinding.User" />
    </data>

    <LinearLayout xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="@dimen/fab_margin"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        tools:context=".MainActivity"
        tools:showIn="@layout/activity_main">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.email}" />

    </LinearLayout>
</layout>
```

- Once data binding is integrated in layout file, goto **Build -> Clean Project** and **Build -> Rebuild Project**. This will generate necessary binding classes.
- The generated binding classes follows the naming convention considering the layout file name in which binding is enabled. For the layout **activity_main.xml**, the generated binding class will be **ActivityMainBinding** (Binding suffix will be added at the end).
- To bind the data in UI, you need to inflate the binding layout first using the generated binding classes. Below, **ActivityMainBinding** inflates the layout first and **binding.setUser()** binds the User object to layout.

You can notice here, we haven’t used findViewById() anywhere.

```
MainActivity.java

import android.databinding.DataBindingUtil;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;

import info.androidhive.databinding.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    private User user;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // setContentView(R.layout.activity_main);

        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        user = new User();
        user.setName("Kunal Malhotra");
        user.setEmail("kunal.malhotra1@ibm.com");

        binding.setUser(user);
    }
}
```
If you run the app, you can see the user details displayed in TextViews.

## My DataBinding classes are not generated?

Current version of Android Studio fails to generate the binding classes most of the times. Usually this can be resolved by **Cleaning & Rebuilding** the project. If the problem still persists, goto **File ⇒ Invalidate Caches & Restart**. This should possibly solve the problem if your layout files are not having any errors.

## DataBinding in <include> layouts

As we can use, we haven’t included **CoordinatorLayout**, **AppBarLayout** and other elements in the above example. Usually, we separate the main layout and content in two different layouts i.e **activity_main.xml** and **content_main.xml**. The **content_main** will be included in main layout using <include> tag. Now we’ll see how to enable data binding when we have include layouts.

Below, we have activity_main.xml with CoordinatorLayout, AppBarLayout and FAB.

- The <layout> tag is used in activity_main.xml layout to enable data binding. As well, the <data> and <variable> tags are used to bind the User object.
- To pass the user to included content_main layout, bind:user=”@{user}” is used. Without this, the user object won’t be accessible in content_main layout.

```
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:bind="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="com.knrmalhotra.databinding.User" />
    </data>

    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:theme="@style/AppTheme.AppBarOverlay">

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="?attr/colorPrimary"
                app:popupTheme="@style/AppTheme.PopupOverlay" />

        </android.support.design.widget.AppBarLayout>

        <include
            android:id="@+id/content"
            layout="@layout/content_main"
            bind:user="@{user}" />

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="@dimen/fab_margin"
            app:srcCompat="@android:drawable/ic_dialog_email" />

    </android.support.design.widget.CoordinatorLayout>
</layout>
```
- The content_main.xml again includes <layout> tag to enable the data binding. The <layout>, <data> and <variable> tags are necessary in both parent and included layouts.
- android:text=”@{user.name}” and android:text=”@{user.email}” attributes are used to display the data in TextViews.

```
content_main.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="com.knrmalhotra.databinding.User" />
    </data>

    <LinearLayout xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="@dimen/fab_margin"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        tools:context=".MainActivity"
        tools:showIn="@layout/activity_main">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.email}" />

    </LinearLayout>
</layout>
```

```
MainActivity.java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        setSupportActionBar(binding.toolbar);

        User user = new User();
        user.setName("Kunal Malhotra");
        user.setEmail("kunal.malhotra1@ibm.com");

        binding.setUser(user);
    }
}
```

Now, if you run the app you can see the data displayed in included layout.

## Binding Click Listeners / Event Handling

Not just the data, we can also bind the click and other events on UI elements. To bind a click event, you need to create a class with necessary callback methods.

Below we have a class that handles the FAB click event.

```
public class MyClickHandlers {

        public void onFabClicked(View view) {
            Toast.makeText(getApplicationContext(), "FAB clicked!", Toast.LENGTH_SHORT).show();
        }
}
```

To bind the event, we again use the same <variable> tag with the path to handler class. Below android:onClick=”@{handlers::onFabClicked}” binds the FAB click to onFabClicked() method.

```
<layout xmlns:bind="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="handlers"
            type="com.knrmalhotra.databinding.MainActivity.MyClickHandlers" />
    </data>

    <android.support.design.widget.CoordinatorLayout ...>

        <android.support.design.widget.FloatingActionButton
            ...
            android:onClick="@{handlers::onFabClicked}" />

    </android.support.design.widget.CoordinatorLayout>
</layout>
```

- To assign long press event, the method should return boolean type instead of void. public boolean onButtonLongPressed() handles the view long press.
- You can also pass params while binding. public void onButtonClickWithParam(View view, User user) receives the user object bind from UI layout. In the layout, the parameter can be passed using android:onClick=”@{(v) -> handlers.onButtonClickWithParam(v, user)}”
- To bind the events, binding.setHandlers(handlers) is called from the activity.

Below example shows different button click events.

```
MainActivity.java
package com.knrmalhotra.databinding;

import android.content.Context;
import android.databinding.DataBindingUtil;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Toast;

import com.knrmalhotra.databinding.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    private User user;
    private MyClickHandlers handlers;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        setSupportActionBar(binding.toolbar);

        user = new User();
        user.setName("Ravi Tamada");
        user.setEmail("ravi@androidhive.info");

        binding.setUser(user);

        handlers = new MyClickHandlers(this);
        binding.content.setHandlers(handlers);
    }

    public class MyClickHandlers {

        Context context;

        public MyClickHandlers(Context context) {
            this.context = context;
        }

        public void onFabClicked(View view) {
            Toast.makeText(getApplicationContext(), "FAB clicked!", Toast.LENGTH_SHORT).show();
        }

        public void onButtonClick(View view) {
            Toast.makeText(getApplicationContext(), "Button clicked!", Toast.LENGTH_SHORT).show();
        }

        public void onButtonClickWithParam(View view, User user) {
            Toast.makeText(getApplicationContext(), "Button clicked! Name: " + user.name, Toast.LENGTH_SHORT).show();
        }

        public boolean onButtonLongPressed(View view) {
            Toast.makeText(getApplicationContext(), "Button long pressed!", Toast.LENGTH_SHORT).show();
            return false;
        }
    }
}
```

```
activity_main.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:bind="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="info.androidhive.databinding.User" />

        <variable
            name="handlers"
            type="info.androidhive.databinding.MainActivity.MyClickHandlers" />
    </data>

    <android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
        ...>

        <android.support.design.widget.AppBarLayout
            ...>

        </android.support.design.widget.AppBarLayout>

        <include
            android:id="@+id/content"
            layout="@layout/content_main"
            bind:user="@{user}" />

        <android.support.design.widget.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="@dimen/fab_margin"
            android:onClick="@{handlers::onFabClicked}"
            app:srcCompat="@android:drawable/ic_dialog_email" />

    </android.support.design.widget.CoordinatorLayout>
</layout>
```

```
content_main.xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="info.androidhive.databinding.User" />

        <variable
            name="handlers"
            type="info.androidhive.databinding.MainActivity.MyClickHandlers" />
    </data>

    <LinearLayout xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="@dimen/fab_margin"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        tools:context=".MainActivity"
        tools:showIn="@layout/activity_main">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.email}" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{handlers::onButtonClick}"
            android:text="CLICK" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{(v) -> handlers.onButtonClickWithParam(v, user)}"
            android:text="CLICK WITH PARAM" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="LONG PRESS"
            android:onLongClick="@{handlers::onButtonLongPressed}" />

    </LinearLayout>
</layout>
```

## Updating UI using Observables

Observables provides way to automatically sync the UI with data without explicitly calling setter methods. The UI will be updated when the value of a property changes in an object. To make the object observable, extend the class from BaseObservable.

- To make a property observable, use @Bindable annotation on getter method.
- Call notifyPropertyChanged(BR.property) in setter method to update the UI whenever the data is changed.
- The BR class will be generated automatically when data binding is enabled.

Below is the modified User class the extends BaseObservable. You can notice here notifyPropertyChanged is called after assigning new values.

```
package com.knrmalhotra.com.databinding;

import android.databinding.BaseObservable;
import android.databinding.Bindable;

public class User extends BaseObservable {
    String name;
    String email;

    @Bindable
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        notifyPropertyChanged(BR.name);
    }

    @Bindable
    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
        notifyPropertyChanged(BR.email);
    }
}
```
To test this, I am changing user data on FAB click. You can see the UI updated on FAB click right away.

```
public class MyClickHandlers {

    Context context;

    public MyClickHandlers(Context context) {
        this.context = context;
    }

    public void onFabClicked(View view) {
        user.setName("Ravi");
        user.setEmail("ravi8x@gmail.com");
    }
}
```

## Updating UI using ObservableFields

If your object class has fewer properties to be updated or if you don’t want to observe every field in the object, you can use ObservableFields to update the UI. You can declare the variable as ObservableField and when the new data is set, the UI will be updated.

The same User class can be modified as below using ObservableFields.

```
package info.androidhive.databinding;

import android.databinding.ObservableField;

public class User {
    public static ObservableField<String> name = new ObservableField<>();
    public static ObservableField<String> email = new ObservableField<>();

    public ObservableField<String> getName() {
        return name;
    }

    public ObservableField<String> getEmail() {
        return email;
    }
}
```

To update the values, you need to assign new value to property directly instead of using setter method.

```
public class MyClickHandlers {

    Context context;

    public MyClickHandlers(Context context) {
        this.context = context;
    }

    public void onFabClicked(View view) {
        user.name.set("Ravi");
        user.email.set("ravi8x@gmail.com");
    }
}
```

Loading Images From URL (Glide or Picasso)

You can also bind an ImageView to an URL to load the image. To bind an URL to ImageView, you can use @BindingAdapter annotation to object property.

Below, profileImage variable is bound to android:profileImage attribute. The image will be loaded using Glide or Picasso image library.

```
package info.androidhive.databinding;

import android.databinding.BaseObservable;
import android.databinding.Bindable;
import android.databinding.BindingAdapter;
import android.widget.ImageView;

import com.bumptech.glide.Glide;
import com.bumptech.glide.request.RequestOptions;

public class User {
    //..
    String profileImage;

    public String getProfileImage() {
        return profileImage;
    }

    public void setProfileImage(String profileImage) {
        this.profileImage = profileImage;
    }

    @BindingAdapter({"android:profileImage"})
    public static void loadImage(ImageView view, String imageUrl) {
        Glide.with(view.getContext())
                .load(imageUrl)
                .into(view);

        // If you consider Picasso, follow the below
        // Picasso.with(view.getContext()).load(imageUrl).placeholder(R.drawable.placeholder).into(view);
    }
}
```

To load the image into ImageView, add the android:profileImage=”@{user.profileImage}” attribute.

```
<ImageView
     android:layout_width="100dp"
     android:layout_height="100dp"
     android:layout_marginTop="@dimen/fab_margin"
     android:profileImage="@{user.profileImage}" />
```

Make sure you have added INTERNET permission in your manifest file.

```
<uses-permission android:name="android.permission.INTERNET" />
```






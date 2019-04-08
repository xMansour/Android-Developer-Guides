# Android Developer Guides  


## Fragments  
A `Fragment` represents a behavior or a portion of user interface in a `FragmentActivity`.  

You can combine multiple fragments in a single activity to build a multi-pane UI and reuse a fragment in multiple activities.  

A fragment must always be hosted in an activity and the fragment's lifecycle is directly affected by the host activity's lifecycle.  

When you add a fragment as a part of your activity layout, it lives in a ViewGroup inside the activity's view hierarchy and the fragment defines its own view layout. ou can insert a fragment into your activity layout by declaring the fragment in the activity's layout file, as a `<fragment>` element, or from your application code by adding it to an existing ViewGroup.  

By dividing the layout of an activity into fragments, you become able to modify the activity's appearance at runtime and preserve those changes in a back stack that's managed by the activity.  

You should design each fragment as a modular and reusable activity component.  


![alt text](https://github.com/xMansour/Android-Developer-Guides/Images/Fragments/fragment_lifecycle.png "Fragment Life Cycle")  

To create a fragment, you must create a subclass of `Fragment`. The `Fragment` class has code that looks a lot like an `Activity`. It contains callback methods similar to an activity, such as `onCreate()`, `onStart()`, `onPause()`, and `onStop()`.  

`onCreate()`  
The system calls this when creating the fragment. Within your implementation, you should initialize essential components of the fragment that you want to retain when the fragment is paused or stopped, then resumed.  

`onCreateView()`  
The system calls this when it's time for the fragment to draw its user interface for the first time. To draw a UI for your fragment, you must return a `View` from this method that is the root of your fragment's layout. You can return null if the fragment does not provide a UI.  

`onPause()`  
The system calls this method as the first indication that the user is leaving the fragment  

To provide a layout for a fragment, you must implement the `onCreateView()` callback method, which the Android system calls when it's time for the fragment to draw its layout. Your implementation of this method must return a `View` that is the root of your fragment's layout.  

To return a layout from `onCreateView()`, you can inflate it from a layout resource defined in XML. To help you do so, `onCreateView()` provides a `LayoutInflater` object.  


```java  
public static class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}
```  
The `container` parameter passed to `onCreateView()` is the parent `ViewGroup` (from the activity's layout) in which your fragment layout is inserted.  

The `savedInstanceState` parameter is a `Bundle` that provides data about the previous instance of the fragment, if the fragment is being resumed.  

`inflate()` takes three arguments:  
1. The resource ID of the layout you want to inflate.  
2. The `ViewGroup` to be the parent of the inflated layout. Passing the `container` is important in order for the system to apply layout parameters to the root view of the inflated layout, specified by the parent view in which it's going.  
3. A boolean indicating whether the inflated layout should be attached to the `ViewGroup` (the second parameter) during inflation. In this case, this is false because the system is already inserting the inflated layout into the `container`—passing true would create a redundant view group in the final layout.  


**Adding a fragment to an activity**  
1. Declare the fragment inside the activity's layout file.  
    ```
    <LinearLayout>
        ...
        <fragment
            android:name="com.example.news.ArticleListFragment"
            android:id="@+id/list"
            ....
        />
    </LinearLayout>  
    ```  

    The `android:name` attribute in the `<fragment>` specifies the Fragment class to instantiate in the layout.  

    When the system creates this activity layout, it instantiates each fragment specified in the layout and calls the `onCreateView()` method for each one, to retrieve each fragment's layout. The system inserts the View returned by the fragment directly in place of the `<fragment>` element.  
2. Programmatically add the fragment to an existing ViewGroup.  
    At any time while your activity is running, you can add fragments to your activity layout. You simply need to specify a `ViewGroup` in which to place the fragment.  

    To make fragment transactions in your activity (such as `add`, `remove`, or `replace` a fragment), you must use APIs from `FragmentTransaction`. You can get an instance of `FragmentTransaction` from your `FragmentActivity` like this:  
    ```java
    FragmentManager fragmentManager = getSupportFragmentManager();
    FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
    ```  

    You can then add a fragment using the `add()` method, specifying the fragment to add and the view in which to insert it. For example:  
    ```java
    ExampleFragment fragment = new ExampleFragment();
    fragmentTransaction.add(R.id.fragment_container, fragment);
    fragmentTransaction.commit();
    ```  

    The first argument passed to `add()` is the `ViewGroup` in which the fragment should be placed, specified by resource ID, and the second parameter is the fragment to add.  

    Once you've made your changes with `FragmentTransaction`, you must call `commit()` for the changes to take effect.  

**Managing Fragments**  
To manage the fragments in your activity, you need to use `FragmentManager`. To get it, call `getSupportFragmentManager()` from your activity.  

Some things that you can do with FragmentManager include:  
1. Get fragments that exist in the activity, with `findFragmentById()` (for fragments that provide a UI in the activity layout) or `findFragmentByTag()` (for fragments that do or don't provide a UI).  
2. Pop fragments off the back stack, with `popBackStack()`.  
3. Register a listener for changes to the back stack, with `addOnBackStackChangedListener()`.  



Before you call `commit()`, however, you might want to call `addToBackStack()`, in order to add the transaction to a back stack of fragment transactions. This back stack is managed by the activity and allows the user to return to the previous fragment state, by pressing the Back button.  
```java
ExampleFragment exampleFragment = new ExampleFragment();
FragmentTransaction fragmentTransaction = getSupportFragmentManager().beginTransaction();

// Replace whatever is in the fragment_container view with this fragment,
// and add the transaction to the back stack
fragmentTransaction.replace(R.id.fragment_container, exampleFragment);
fragmentTransaction.addToBackStack(null);

fragmentTransaction.commit();
```  
By calling `addToBackStack()`, the replace transaction is saved to the back stack so the user can reverse the transaction and bring back the previous fragment by pressing the Back button.  

`FragmentActivity` then automatically retrieve fragments from the back stack via `onBackPressed()`.  

The order in which you add changes to a FragmentTransaction doesn't matter.  

**Communicating with the Activity**  
The fragment can access the `FragmentActivity` instance with `getActivity()` and easily perform tasks such as find a view in the activity layout:  
```java
View listView = getActivity().findViewById(R.id.list);
```  

Likewise, your activity can call methods in the fragment by acquiring a reference to the Fragment from `FragmentManager`, using `findFragmentById()` or `findFragmentByTag()`. For example:  
```java
ExampleFragment exampleFragment = (ExampleFragment) getSupportFragmentmanager().findFragmentById(R.id.example_fragment);
```  

**Creating event callbacks to the activity**  
You might need a fragment to share events or data with the activity and/or the other fragments hosted by the activity.  

To share data, create a shared `ViewModel`. If you need to propagate events that cannot be handled with a `ViewModel`, you can instead define a callback interface inside the fragment and require that the host activity implement it. When the activity receives a callback through the interface, it can share the information with other fragments in the layout as necessary.  

For example, if a news application has two fragments in an activity—one to show a list of articles (fragment A) and another to display an article (fragment B)—then fragment A must tell the activity when a list item is selected so that it can tell fragment B to display the article. In this case, the `OnArticleSelectedListener` interface is declared inside fragment A:  

```java
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}
```  

Then the activity that hosts the fragment implements the `OnArticleSelectedListener` interface and overrides `onArticleSelected()` to notify fragment B of the event from fragment A. To ensure that the host activity implements this interface, fragment A's onAttach() callback method (which the system calls when adding the fragment to the activity) instantiates an instance of `OnArticleSelectedListener` by casting the Activity that is passed into onAttach():  

```java
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener listener;
    ...
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        try {
            listener = (OnArticleSelectedListener) context;
        } catch (ClassCastException e) {
            throw new ClassCastException(context.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...
}
```  

If the activity hasn't implemented the interface, then the fragment throws a `ClassCastException`. On success, the mListener member holds a reference to activity's implementation of `OnArticleSelectedListener`, so that fragment A can share events with the activity by calling methods defined by the `OnArticleSelectedListener` interface.  

```java
public static class FragmentA extends ListFragment {
    OnArticleSelectedListener listener;
    ...
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Append the clicked item's row ID with the content provider Uri
        Uri noteUri = ContentUris.withAppendedId(ArticleColumns.CONTENT_URI, id);
        // Send the event and Uri to the host activity
        listener.onArticleSelected(noteUri);
    }
    ...
}
```  
**IMPORTANT**
**Adding items to the App Bar**  
Your fragments can contribute menu items to the activity's Options Menu (and, consequently, the app bar) by implementing `onCreateOptionsMenu()`. In order for this method to receive calls, however, you must call `setHasOptionsMenu()` during `onCreate()`, to indicate that the fragment would like to add items to the Options Menu. Otherwise, the fragment doesn't receive a call to `onCreateOptionsMenu()`.  

Any items that you then add to the Options Menu from the fragment are **appended** to the existing menu items. The fragment also receives callbacks to `onOptionsItemSelected()` when a menu item is selected.  

You can also register a view in your fragment layout to provide a context menu by calling `registerForContextMenu()`. When the user opens the context menu, the fragment receives a call to `onCreateContextMenu()`. When the user selects an item, the fragment receives a call to `onContextItemSelected()`.  

*Note: Although your fragment receives an on-item-selected callback for each menu item it adds, the activity is first to receive the respective callback when the user selects a menu item. If the activity's implementation of the on-item-selected callback does not handle the selected item, then the event is passed to the fragment's callback. This is true for the Options Menu and context menus.*  

*The most significant difference in lifecycle between an activity and a fragment is how one is stored in its respective back stack. An activity is placed into a back stack of activities that's managed by the system when it's stopped, by default (so that the user can navigate back to it with the Back button. However, a fragment is placed into a back stack managed by the host activity only when you explicitly request that the instance be saved by calling `addToBackStack()` during a transaction that removes the fragment.*  

*Caution: If you need a Context object within your Fragment, you can call `getContext()`. However, be careful to call `getContext()` only when the fragment is attached to an activity. When the fragment isn't attached yet, or was detached during the end of its lifecycle, `getContext()` returns null.*  

**onAttach()**  
Called when the fragment has been associated with the activity (the Activity is passed in here).  
**onCreateView()**  
Called to create the view hierarchy associated with the fragment.  
**onActivityCreated()**  
Called when the activity's onCreate() method has returned.
**onDestroyView()**  
Called when the view hierarchy associated with the fragment is being removed.  
**onDetach()**  
Called when the fragment is being disassociated from the activity.  
**Resumed**  
The fragment is visible in the running activity.  
**Paused**  
Another activity is in the foreground and has focus, but the activity in which this fragment lives is still visible (the foreground activity is partially transparent or doesn't cover the entire screen).  
**Stopped**  
The fragment isn't visible. Either the host activity has been stopped or the fragment has been removed from the activity but added to the back stack. A stopped fragment is still alive (all state and member information is retained by the system). However, it is no longer visible to the user and is killed if the activity is killed.  

When the activity has received its `onCreate()` callback, a fragment in the activity receives no more than the `onActivityCreated()` callback.  

Once the activity reaches the resumed state, you can freely add and remove fragments to the activity.  


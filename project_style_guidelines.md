# Android Project Guidelines
---------------------------

The aim of this document is to define project guidelines. These should be followed throughout the Android project in order to help us to keep our code base clean and consistent. This guide is based on guidelines from buffer.com team.

The guide will grow on a weekly basis to ease the process of understanding and following it.

## 1. Project Guidelines

### 1.1 Project Structure

When contributing work, the project should maintain the following structure:



	src/androidTest
	src/test
	src/main


**androidTest** - Directory containing functional tests    
**test** - Directory containing unit tests  
**commonTest** - Directory containing shared test code for AndroidTest & Test  
**main** - Directory containing application code

The structure of the project should remain as defined above whenever you are modifying or adding new features.

Using this structure allows us to keep the application code seperated from any test-related code. The CommonTest directory allows us to share classes between the functional and unit tests, such as mock model creation and dagger test configuration classes.


### 1.2 File Naming

#### 1.2.1 Class Files

Any classes that you define should be named using UpperCamelCase, for example:

	ActivityAndroid, NetworkHelper, FragmentUser, PerActivity


Any classes extending an Android framework component should **always** end with the component name. For example:

	UserFragment, SignUpActivity, RateAppDialog, PushNotificationServer, NumberView

We use UpperCamelCase as this helps to seperate the words used to create the name, making it easier to read. Naming classes to end with the framework component makes it super clear as to what the class is used for. For example, if you're looking to make changes to the RegistrationDialog then this naming convention makes it really easy to locate that class.

#### 1.2.1 Resource Files

When naming resource files you should be sure to name them using lowercase letters and underscores instead of spaces, for example:

	activity_main, fragment_user, item_post

This convention again makes it really easy to locate the specific layout file that you're looking for. Within android studio, the layout package is sorted in alphabetical order meaning that activity, fragment and other layout types becomes grouped - so you know where to begin looking for a file. Other than this, begining the file name with the component name makes it clear what component/class the layout file is being used for.


#### 1.2.2.1 Drawable Files

Drawable resource files should be named using the **ic_** prefix along with the size and color of the asset. For example, white accept icon sized at 24dp would be named:

	ic_accept_24dp_white

And a black cancel icon sized at 48dp would be named:

	ic_cancel_48dp_black

We use this naming convention so that a drawable file is recognisable by its name. If the colour and size are not stated in the name, then the developer needs to open the drawable file to find out this information. This saves us a little bit of time :)

Other drawable files should be named using the corresponding prefix, for example:

| Type       | Prefix    | Example                |
|------------|-----------|------------------------|
| Selector   | selector_ | selector_button_cancel |
| Background | bg_       | bg_rounded_button      |
| Circle     | circle_   | circle_white           |
| Progress   | progress_ | progress_circle_purple |
| Divider    | divider_  | divider_grey           |

This convention again helps to group similar items within Android Studio. It also makes it clear as to what the item is used for. For example, naming a resource button_cancel could mean anything! Is this a selector resource or a rounded button background? Correct naming helps to clear any ambiguity that may arise.

When creating selector state resources, they should be named using the corresponding suffix:

| State    | Suffix    | Example             |
|----------|-----------|---------------------|
| Normal   | _normal   | btn_accept_normal   |
| Pressed  | _pressed  | btn_accept_pressed  |
| Focused  | _focused  | btn_accept_focused  |
| Disabled | _disabled | btn_accept_disabled |
| Selected | _selected | btn_accept_selected |

Using clear prefixes such as the above helps to make it absolutely obvious as to what a selector state resource is used for. Prefixing resources with the colour or any other identifier again requires the developer to open the selector file to be educated in what the different selector state resources are.

#### 1.2.2.2 Layout Files

When naming layout files, they should be named starting with the name of the Android Component that they have been created for. For example:

| Component        | Class Name      | Layout Name       |
|------------------|-----------------|-------------------|
| Activity         | MainActivity    | activity_main     |
| Fragment         | MainFragment    | fragment_main     |
| Dialog           | RateDialog      | dialog_rate       |
| Widget           | UserProfileView | view_user_profile |
| AdapterView Item | N/A             | item_follower     |

**Note:** If you create a layout using the merge tag then the layout_ prefix should be used.

Not only does this approach makes it easy to find files in the directory hierarchy, but it really helps when needing to identify what corresponding class a layout file belongs to.


#### 1.2.2.3 Menu Files

Menu files do not need to be prefixed with the menu_ prefix. This is because they are already in the menu package in the resources directory, so it is not a requirement.

#### 1.2.2.4 Values Files

All resource file names should be plural, for example:

	attrs.xml, strings.xml, styles.xml, colors.xml, dimens.xml



## 2. Code Guidelines

### 2.1 Java Language Rules

#### 2.1.1 Never ignore exceptions

Avoid not handling exceptions in the correct manner. For example:

	public void setUserId(String id) {
    	try {
        	mUserId = Integer.parseInt(id);
    	} catch (NumberFormatException e) { }
	}

This gives no information to both the developer and the user, making it harder to debug and could also leave the user confused if something goes wrong. When catching an exception, we should also always log the error to the console for debugging purposes and if necessary alert the user of the issue. For example:


	public void setCount(String count) {
    	try {
        	count = Integer.parseInt(id);
    	} catch (NumberFormatException e) {
    		count = 0;
        	Log.e(TAG, "There was an error parsing the count " + e);
        	DialogFactory.showErrorMessage(R.string.error_message_parsing_count);
    	}
	}

Here we handle the error appropriately by:

- Showing a message to the user notifying them that there has been an error
- Setting a default value for the variable if possible
- Throw an appropriate exception


#### 2.1.2 Never catch generic exceptions


Catching exceptions generally should not be done:


	public void openCustomTab(Context context, Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (Exception e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	}
	}

Why?

*Do not do this. In almost all cases it is inappropriate to catch generic Exception or Throwable (preferably not Throwable because it includes Error exceptions). It is very dangerous because it means that Exceptions you never expected (including RuntimeExceptions like ClassCastException) get caught in application-level error handling. It obscures the failure handling properties of your code, meaning if someone adds a new type of Exception in the code you're calling, the compiler won't help you realize you need to handle the error differently. In most cases you shouldn't be handling different types of exception the same way.* - taken from the Android Code Style Guidelines

Instead, catch the expected exception and handle it accordingly:

	public void openCustomTab(Context context, Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	}
	}


#### 2.1.3 Grouping exceptions

Where exceptions execute the same code, they should be grouped in-order to increase readability and avoid code duplication. For example, where you may do this:

	public void openCustomTab(Context context, @Nullable Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	} catch (NullPointerException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	} catch (SomeOtherException e) {
    		// Show some dialog
        }
	}

You could do this:

	public void openCustomTab(Context context, @Nullable Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e | NullPointerException e) {
        	Log.e(TAG, "There was an error opening the custom tab " + e);
    	} catch (SomeOtherException e) {
    		// Show some dialog
        }
	}


#### 2.1.4 Using try-catch over throw exception

Using try-catch statements improves the readability of the code where the exception is taking place. This is because the error is handled where it occurs, making it easier to both debug or make a change to how the error is handled.


#### 2.1.5 Never use Finalizers

*There are no guarantees as to when a finalizer will be called, or even that it will be called at all. In most cases, you can do what you need from a finalizer with good exception handling. If you absolutely need it, define a close() method (or the like) and document exactly when that method needs to be called. See InputStreamfor an example. In this case it is appropriate but not required to print a short log message from the finalizer, as long as it is not expected to flood the logs.* - taken from the Android code style guidelines



#### 2.1.6 Fully qualify imports

When declaring imports, use the full package declaration. For example:

Don’t do this:


    import android.support.v7.widget.*;

Instead, do this 😃


    import android.support.v7.widget.RecyclerView;


#### 2.1.7 Don't keep unused imports

Sometimes removing code from a class can mean that some imports are no longer needed. If this is the case then the corresponding imports should be removed alongside the code.


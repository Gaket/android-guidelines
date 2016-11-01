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
        	Timber.e(e, "There was an error parsing the count ");
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
        	Timber.e(e, "There was an error opening the custom tab ");
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
        	Timber.e(e, "There was an error opening the custom tab ");
    	}
	}


#### 2.1.3 Grouping exceptions

Where exceptions execute the same code, they should be grouped in-order to increase readability and avoid code duplication. For example, where you may do this:

	public void openCustomTab(Context context, @Nullable Uri uri) {
    	Intent intent = buildIntent(context, uri);
    	try {
        	context.startActivity(intent);
    	} catch (ActivityNotFoundException e) {
        	Timber.e(e, "There was an error opening the custom tab ");
    	} catch (NullPointerException e) {
        	Timber.e(e, "There was an error opening the custom tab ");
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
        	Timber.e(e, "There was an error opening the custom tab ");
    	} catch (SomeOtherException e) {
    		// Show some dialog
        }
	}
	
#### 2.1.4 String concatenation in exceptions call

Don't use string concatenation in log calls:

     String msg = "pay more attention to this guides"
     Timber.e(e, "Some message: " + msg + ". Check an e-mail with details.");


Timber handles string formatting automatically:

     String msg = "pay more attention to this guides"
     Timber.e(e, "Some message: %s. Check an e-mail with details.", msg);
     
#### 2.1.5 Using try-catch over throw exception

Using try-catch statements improves the readability of the code where the exception is taking place. This is because the error is handled where it occurs, making it easier to both debug or make a change to how the error is handled.

#### 2.1.6 Fully qualify imports

When declaring imports, use the full package declaration. For example:

Don’t do this:


    import android.support.v7.widget.*;

Instead, do this 😃


    import android.support.v7.widget.RecyclerView;


#### 2.1.7 Don't keep unused imports

Sometimes removing code from a class can mean that some imports are no longer needed. If this is the case then the corresponding imports should be removed alongside the code.

#### 2.1.8 Always add default clause

Usually, if there is switch-case and no one of existing cases was triggered, it means that something went wrong. Always add logging to default clause to be able to notice such cases.

### 2.2 Java Style Rules

#### 2.2.1 Field definition and naming

All fields should be declared at the top of the file, following these rules:


- Private, non-static field names should start with m. This is right:

    mUserSignedIn, mUserNameText, mAcceptButton

Not this:

    userSignedIn, userNameText, acceptButton


- Private, static field names should start with an s. This is right:

	sSomeStaticField, sUserNameText

Not this:

	someStaticField, userNameText


- All other fields also start with a lower case letter.


    int numOfChildren;
    String username;


- Static final fields (known as constants) are ALL_CAPS_WITH_UNDERSCORES.


    private static final int PAGE_COUNT = 0;

Field names that do not reveal intention should not be used. For example,

    int e; //number of elements in the list

why not just give the field a meaningful name in the first place, rather than leaving a comment!

    int numberOfElements;

That's much better!


#### 2.2.1.2 View Field Naming

When naming fields that reference views, the name of the view should be the last word in the name. For example:

| View           | Name              |
|----------------|-------------------|
| TextView       | usernameView      |
| Button         | acceptLoginView   |
| ImageView      | profileAvatarView |
| RelativeLayout | profileLayout     |

We name views in this way so that we can easily identify what the field corresponds to. For example, having a field named **user** is extremely ambiguous - giving it the name usernameView, userAvatarView or userProfieLayout helps to make it clear  exactly what view the field corresponds with.

Previously, the names for views often ended in the view type (e.g acceptLoginButton) but quite often views change and it's easy to forgot to go back to java classes and update variable names.

#### 2.2.2 Avoid naming with container types

Leading on from the above, we should also avoid the use of container type names when creating variables for collections. For example, say we have an arraylist containing a list of userIds:

Do:

    List<String> userIds = new ArrayList<>();

Don't:

    List<String> userIdList = new ArrayList<>();

If and when container names change in the future, the naming of these can often get forgotten about - and just like view naming, it's not entirely necessary. Correct naming of the container itself should provide enough information for what it is.


#### 2.2.3 Avoid similar naming

Naming variables, method and / or classes with similar names can make it confusing for other developers reading over your code. For example:

	hasUserSelectedSingleProfilePreviously

	hasUserSelectedSignedProfilePreviously

Distinguishing the difference between these at a first glance can be hard to understand what is what. Naming these in a clearer way can make it easier for developers to navigate the fields in your code.

#### 2.2.4 Number series naming

When Android Studio auto-generates code for us, it's easy to leave things as they are - even when it generate horribly named parameters! For example, this isn't very nice:

	public void doSomething(String s1, String s2, String s3)

It's hard to understand what these parameters do without reading the code. Instead:

	public void doSomething(String userName, String userEmail, String userId)

That makes it much easier to understand! Now we'll be able to read the code following the parameter with a much clearer understanding 🙂

#### 2.2.5 Pronouncable names

When naming fields, methods and classes they should:

- Be readable: Efficient naming means we'll be able to look at the name and understand it instantly, reducing cognitive load on trying to decipher what the name means.

- Be speakable: Names that are speakable avoids awkward conversations where you're trying to pronounce a badly named variable name.

- Be searchable: Nothing is worse than trying to search for a method or variable in a class to realise it's been spelt wrong or badly named. If we're trying to find a method that searches for a user, then searching for 'search' should bring up a result for that method.


#### 2.2.6 Treat acronyms as words

Any acronyms for class names, variable names etc should be treated as words - this applies for any capitalisation used for any of the letters. For example:

| Do              | Don't           |
|-----------------|-----------------|
| setUserId       | setUserID       |
| String uri      | String URI      |
| int id          | int ID          |
| parseHtml       | parseHTML       |
| generateXmlFile | generateXMLFile |


#### 2.2.7 Avoid justifying variable declarations

Any declaration of variables should not use any special form of alignment, for example:

This is fine:

    private int userId = 8;
    private int count = 0;
    private String username = "hitherejoe";

Avoid doing this:

    private String username = "hitherejoe";
    private int userId      = 8;
    private int count       = 0;

This creates a stream of whitespace which is known to make text difficult to read for certain learning difficulties.

#### 2.2.8 Use spaces for indentation


For blocks, 4 space indentation should be used:


    if (userSignedIn) {
        count = 1;
    }

Whereas for line wraps, 8 spaces should be used:


    String userAboutText =
            "This is some text about the user and it is pretty long, can you see!"


### 2.2.9 If-Statements

#### 2.2.9.1 Use standard brace style

Braces should always be used on the same line as the code before them. For example, avoid doing this:


    class SomeClass
    {
    	private void someFunction()
    	{
        	if (isSomething)
        	{

        	}
        	else if (!isSomethingElse)
        	{

        	}
        	else
        	{

        	}
    	}
	}

And instead, do this:


	class SomeClass {
    	private void someFunction() {
        	if (isSomething) {

        	} else if (!isSomethingElse) {

        	} else {

        	}
    	}
	}

Not only is the extra line for the space not really necessary, but it makes blocks easier to follow when reading the code.

#### 2.2.9.2 Inline if-clauses

Sometimes it makes sense to use a single line for if statements. For example:

    if (user == null) return false;

However, it only works for simple operations. Something like this would be better suited with braces:


    if (user == null) throw new IllegalArgumentExeption("Oops, user object is required.");

#### 2.2.9.3 Nested if-conditions

Where possible, if-conditions should be combined to avoid over-complicated nesting. For example:

Do:


    if (userSignedIn && userId != null) {

    }

Try to avoid:


    if (userSignedIn) {
        if (userId != null) {

        }
    }

This makes statements easier to read and removes the unnecessary extra lines from the nested clauses.

#### 2.2.9.4 Ternary Operators

Where appropriate, ternary operators can be used to simplify operations.

For example, this is easy to read:


    userStatusImage = signedIn ? R.drawable.ic_tick : R.drawable.ic_cross;

and takes up far fewer lines of code than this:


    if (signedIn) {
        userStatusImage = R.drawable.ic_tick;
    } else {
        userStatusImage = R.drawable.ic_cross;
    }

**Note:** There are some times when ternary operators should not be used. If the if-clause logic is complex or a large number of characters then a standard brace style should be used.

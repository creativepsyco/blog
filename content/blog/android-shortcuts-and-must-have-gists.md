+++
title = "android shortcuts and must have gists"
date = "2013-06-24"
slug = "2013/06/24/android-shortcuts-and-must-have-gists"
Categories = []
+++

###Making use of shared preferences for storing information
```java

    public boolean getStoredLoginState() {
        prefs = getSharedPreferences(PREF_NAME, MODE_PRIVATE);
        boolean state = prefs.getBoolean(B_LOGGED_IN_STATE, false);
        return state;
    }

    public void setLoggedInState(boolean isLoggedIn) {
        prefs = getSharedPreferences(PREF_NAME, MODE_PRIVATE);
        SharedPreferences.Editor editor = prefs.edit();

        editor.putBoolean(B_LOGGED_IN_STATE, isLoggedIn);
        editor.commit();
    }
```

###Making use of the redirection trick

Let's say you want to make sure the user is logged in before giving him access to a particular activity view. What you can do is to check the status of the login token through a shared preference and send back an intent with the intent that the current view was invoked with via an extra Intent parameter

This is the code for the invoker:

```java
  public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.detail_view);

        if (getStoredLoginState() == true) {
            Intent intent = this.getIntent();
            Log.i(TAG, "DetailView onCreate recd position of item: " + intent.getIntExtra(ITEM_ID, 10));

            int pos = intent.getIntExtra(ITEM_ID, 10);

            // Set the text
            TextView textView = (TextView) findViewById(R.id.textView1);

            textView.setText(" " + pos);
        } else {
            // not logged in

            Intent intent = this.getIntent();
            Intent newIntent = new Intent(this, LoginActivity.class);
            newIntent.putExtra(REDIRECT_INTENT, intent);
//            newIntent.setAction(Intent.ACTION_MAIN);
            newIntent.setFlags(Intent.FLAG_ACTIVITY_NO_HISTORY);

            startActivity(newIntent);
        }
    }
```

And this is the login view code. Note that in the invoker's code we use the `Intent.FLAG_ACTIVITY_NO_HISTORY` to make sure that the call does not stay in the stack since login is something we need to ensure out of navigation:

```java
@Override
    public void onCreate(Bundle savedInstanceState) {
        Log.i(TAG, "LoginActivity onCreate() hit");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        Log.i(TAG, "Login State: " + getStoredLoginState());

        // Check for a redirect
        Intent intent = getIntent();
        redirect_intent = intent.getParcelableExtra(REDIRECT_INTENT);

        if (redirect_intent != null) {
            Log.i(TAG, "LoginActivity : this is a redirect intent " + redirect_intent);
        }
    }

    public void onBtnSubmitClicked(View view) {
        TextView username = (TextView) findViewById(R.id.editText);
        TextView password = (TextView) findViewById(R.id.editText1);

        if (isValidLogin(username.getText().toString(), password.getText().toString())) {
            isLoggedIn = true;
            Log.i(TAG, "LoginActivity onBtnSubmitClicked() Username Password match");
        } else {
            isLoggedIn = false;
            new AlertDialog.Builder(this)
                    .setTitle("Wrong username/password")
                    .setMessage("You have entered wrong username/password")
                    .setPositiveButton("Ok", new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int which) {
                            // continue with delete
                        }
                    })
                    .show();
        }

        setLoggedInState();

        if (isLoggedIn) {
            Log.i(TAG, "LoginActivity onBtnSubmitClicked starting List View Activity");
            if (redirect_intent != null) {
                // this will make the activity pop up back from the Task stack
                redirect_intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
                startActivity(redirect_intent);
            } else {
                Intent intent = new Intent(this, ListActivity.class);
                startActivity(intent);
            }
        }
    }
```

More information available at the [Android Documentation][android]

Cheers

[android]: http://developer.android.com/guide/components/tasks-and-back-stack.html

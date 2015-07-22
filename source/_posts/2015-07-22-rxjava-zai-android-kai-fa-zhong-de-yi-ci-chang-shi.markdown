---
layout: post
title: "RxJava 在 Android 开发中的一次尝试"
date: 2015-07-22 10:01:35 +0800
comments: true
categories: 
---
接触 RxJava 有一段时间了，写的都是一些小打小闹的 Sample 程序，不如就在实际项目中一试伸手。

许多 App 都有注册功能，当没有输入或者输入不符合业务逻辑时，会有错误提示。好点的用户体验应该是，只有当所有输入符合业务逻辑时，才会让注册按钮变成可点击状态。用传统方法，实现起来费时费力、容易出错、难以维护，让我们开启响应式模式，用 RxJava 来实现。


```java
public class MainActivity extends Activity {

    @InjectView(R.id.user_name)
    EditText userName;
    @InjectView(R.id.password)
    EditText password;
    @InjectView(R.id.confirmedPassword)
    EditText confirmedPassword;
    @InjectView(R.id.submit_button)
    Button submitButton;

    private String userNameString;
    private String passwordString;
    private String confirmedPasswordString;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);

        addTextChangedListenners();
    }

    private void addTextChangedListenners() {

        userName.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
                userNameString = charSequence.toString();
                verifyInput();
            }

            @Override
            public void afterTextChanged(Editable editable) {

            }
        });

        password.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
                passwordString = charSequence.toString();
                verifyInput();

            }

            @Override
            public void afterTextChanged(Editable editable) {

            }
        });

        confirmedPassword.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

            }

            @Override
            public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
                confirmedPasswordString = charSequence.toString();
                verifyInput();
            }

            @Override
            public void afterTextChanged(Editable editable) {

            }
        });
    }

    private void verifyInput() {

        boolean isValidUserName = userNameString != null && userNameString.length() > 2;
        boolean isValidPassword = passwordString != null && passwordString.length() > 3;
        boolean passwordsEqualed = confirmedPasswordString != null && confirmedPasswordString.equals(passwordString);
        boolean allowSubmit = isValidUserName && isValidPassword && passwordsEqualed;

        Log.i("MainActivity", "verify: " + isValidUserName + " " + isValidPassword + " " + passwordsEqualed);

        submitButton.setClickable(allowSubmit);
    }

}
```

用 RxJava 来写 App，代码更加清晰，大大降低了耦合度，让上帝的归上帝，凯撒的归凯撒，感觉就是一个字酸爽
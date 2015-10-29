---
layout: post
title: "RxJava 在 Android 开发中的初次尝试（上）"
date: 2015-07-22 10:01:35 +0800
comments: true
categories: 
---
接触 RxJava 有一段时间了，写的都是一些小打小闹的 Sample 程序，不如就在实际项目中一试伸手。

许多 App 都有注册功能，当没有输入或者输入不符合业务逻辑时，会有错误提示。好点的用户体验应该是，只有当所有输入符合业务逻辑时，才会让注册按钮变成可点击状态，代码如下：

<!--more-->

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
                validateUserInput();
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
                validateUserInput();

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
                validateUserInput();
            }

            @Override
            public void afterTextChanged(Editable editable) {

            }
        });
    }
    private void validateUserInput() {
    
        boolean isUserNameValid = userNameString != null && userNameString.trim().length() > 2;
        boolean isPasswordValid = passwordString != null && passwordString.trim().length() > 3;
        boolean isPasswordEqual = confirmedPasswordSting != null && confirmedPasswordSting.trim().equals(passwordString);
        boolean enable = isUserNameValid && isPasswordValid && isPasswordEqual;
        submitButton.setEnabled(enable);
    }

}
```

用传统方法，实现起来费时费力、容易出错、难以维护，让我们开启响应式模式，用 RxJava 来实现，不过先上个流程图先。

![flow](/images/flow.jpg)

如图所示，在流中需要传递一个 model，为简化流程，用户名只规定长度大于2，密码长度大于3 

```
public class InputValidation {
    private String userName;
    private String password;
    private String confirmedPassword;

    public InputValidation(String userName, String password, String confirmedPassword) {
        this.userName = userName;
        this.password = password;
        this.confirmedPassword = confirmedPassword;
    }

    public boolean allowSubmit() {
        boolean isUserNameValid = userName.trim().length() > 2;
        boolean isPasswordValid = password.trim().length() > 3;
        boolean isPasswordEqual = confirmedPassword.trim().equals(password);
        boolean allowSubmit = isUserNameValid && isPasswordValid && isPasswordEqual;
        return allowSubmit;
    }

    public static String stringOfTextChangEvent(OnTextChangeEvent onTextChangeEvent) {
        return onTextChangeEvent.text().toString();
    }
}
```


创建并转成字符串流，WidgetObservable.text 是 RxAndroid 提供的获取 EditText 输入文本的 API，map中传入的是转换方法的引用，这也是 Java8 才支持的。

```
    private void initRx() {
        Observable<String> userNameObservable = WidgetObservable.text(userName)
                .map(InputValidation::stringOfTextChangEvent);
        Observable<String> passwordObservable = WidgetObservable.text(password)
                .map(InputValidation::stringOfTextChangEvent);
        Observable<String> confirmedPasswordObservable = WidgetObservable.text(confirmedPassword)
                .map(InputValidation::stringOfTextChangEvent);
	......
```

将三个输入流合并成一个，并根据验证结果设置提交按钮的状态。

```
......
        Observable<InputValidation> inputValidationObservable = Observable.combineLatest(
                userNameObservable,
                passwordObservable,
                confirmedPasswordObservable,
                InputValidation::new
        ).map(inputValidation -> {
            submitButton.setEnabled(inputValidation.allowSubmit());
            return inputValidation;
        });
......
```
combineLatest 的最后一个参数是个合并方法，我传入了个 Java8 支持的构造器引用。

```
......
        ViewObservable.clicks(submitButton).zipWith(
                inputValidationObservable,
                (onClickEvent, inputValidation) -> inputValidation
        ).subscribe();
......
```
最终结果：


![submit](/images/submit.gif)

在上面的代码中，起初调用的是 combineLatest，导制每次输入时，都会触发最终的 Observable，最后改用 zip 来合并，因为 zip 只有新的流出现，才会被触发，而 combineLatest 只要其中任意一个流有新数据，都会被触发，看图比较直观。

zip

![zip](/images/zip.png)

combineLatest

![combineLatest](/images/combineLatest.png)


用 RxJava 来写 App，代码更加清晰，大大降低了耦合度，使分离的逻辑形成链，让上帝的归上帝，凯撒的归凯撒，感觉就是一个字~~酸爽！
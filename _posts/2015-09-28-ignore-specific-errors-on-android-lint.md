---
layout: post
date: '2015-09-28'
title: "Ignore specific errors on Android Lint"
summary: "Android Lint is a powerful tool for Android developers, but sometimes we want to ignore specific errors on Android Lint. There is a lot of ways to do that."
categories:  ["Android"]
tags:  ["lint", "gradle"]
featured: false
---

Android Lint is a powerful tool for Android developers, but sometimes we want 
to ignore specific errors on Android Lint. There is a lot of ways to do that.

> The Android [lint tool](http://developer.android.com/tools/help/lint.html) 
> is a static code analysis tool that checks your 
> Android project source files for potential bugs and optimization 
> improvements for correctness, security, performance, usability, 
> accessibility, and internationalization.

## Running

### Command line

You can do this directly from the command line:

```gradle
gradle lint
```
### Android Studio

To run lint in Android Studio go to Analyze > Inspect Code

## Configuring

Lint has a lot of option you can use in your `build.gradle`

```gradle
lintOptions {
    // put your options here
}
```

See the [complete list of lintOptions](https://developer.android.com/reference/tools/gradle-api/4.1/com/android/build/api/dsl/LintOptions)

To see the complete list of issues and categories id's run:

```shell
lint --list
```

## Ignoring erros

Lint is part of Gradle build process, by default if it fails 
your build will stop and you will get a message like this:

```text
FAILURE: Build failed with an exception.
* What went wrong:
Execution failed for task ':app:lint'.
> Lint found errors in the project; aborting build.
```

### Ignoring all errors

In 99% of the cases, people will start to ignore lint instead 
of fixing the problems, adding this on `build.gradle` app

```gradle
lintOptions {
    abortOnError false
}
```
But in my opinion, it is the wrong thing to do. If lint is 
telling that you have a problem, the best thing to do is try to 
fix it. Lint is a tool to make your app and the UX better.

### Ignore specific errors on Android Lint

Sometimes you really need to ignore some lint errors. For 
example, when you are using `setJavaScriptEnabled(true);` 
in your WebView

In this case, you should disable **only** the specific ids 
instead of disabling the whole lint.

```gradle
lintOptions {
    disable 'SetJavaScriptEnabled'
}
```

You also can also ignore it directly in your code if you prefer:

```kotlin
@SuppressLint "SetJavaScriptEnabled")
```

Or in your XML layout file:


```xml
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="SomeLintIssueIdHere" >
```

If you prefer you can move all your issues rules to 
a lint.xml file in the root directory of your project.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<lint>
    <issue id="SetJavaScriptEnabled" severity="ignore" />
</lint>
```


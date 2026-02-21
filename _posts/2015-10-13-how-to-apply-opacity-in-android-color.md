---
layout: post
date: '2015-10-13'
title: "How to apply opacity in Android color"
summary: "Discover the hidden truth of applying opacity in Android color with these simple steps."
categories:  ["Android"]
tags:  ["argb", "color", "hex", "opacity", "rgb", "rgba"]
featured: false
---

Today I was chatting with my friend 
[Andres Galante](http://blog.andresgalante.com/) about colors in Android. 
He was about to send some colors for a developer team's use in an Android app
and asked me: Can you use rgba for the greys or they must be hex? 
How to apply opacity in Android color

The short answer is:

Yes We Can

You can use the [Android Color util method argb](https://developer.android.com/reference/android/graphics/Color.html#argb(int, int, int, int)):

```android
Color.argb(alpha, r, g, b);
```

That is not common. Android developers usually set colors as a resource
in XML file and unfortunately XML resource not accept RGB only HEX

**/res/values/color.xml**
```xml
<color name="textColor">#FFFFFF</color>
```

He told me:

> The problem is Material Design has specific opacity rules for text, icons,
> and dividers. How we apply opacity in Android color using HEX?

It's not a problem, Android uses 
[Hex ARGB values](http://developer.android.com/guide/topics/resources/more-resources.html#Color), 
which are formatted as #AARRGGBB

The first pair of letters (AA), represent the Alpha. You must convert your 
decimal opacity values to a Hexadecimal value. Here are the steps:

1. Take your opacity as a decimal value and multiply it by 255. So, 
if you have a block that is 50% opaque the decimal value would be 
.5. For example: .5 x 255 = 127.5
2. The fraction won't convert to hex, so you must round your number 
up or down to the nearest whole number. For example 127.5 rounds up 
to 128; 55.25 rounds down to 55.
3. Enter your decimal value in a decimal to hexadecimal converters, like 
this decimal-to-hex-converter, and convert your values
4. If you only get back a single value, prefix it with a zero. For 
example, if you're trying to get 5% opacity and your going through this 
process you'll end up with the hex value of D. Add a zero in front 
of it so it appears as 0D.

Source: [Alpha Hex Value Process](http://stackoverflow.com/questions/5445085/understanding-colors-in-android-6-characters/11019879#11019879)

**/res/values/color.xml**

```xml
<!-- Black text color with 87% of opacity -->
<color name="textColor">#DE000000</color>
```

... and if you are lazy (like me), [I did the math for you](https://gist.github.com/danielpassos/fda022f6db5f6defbc52)

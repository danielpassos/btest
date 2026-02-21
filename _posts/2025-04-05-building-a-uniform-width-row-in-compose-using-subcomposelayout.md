---
layout: post
date: '2025-04-05'
title: "Building a Uniform Width Row in Compose Using SubcomposeLayout"
summary: "Learn how to add SEO and Open Graph metadata to your Hugo site for better sharing and indexing."
categories:  ["Compose"]
tags:  ["layout", "subcomposelayout"]
featured: false
---

In UI design, uniformity often helps reduce visual noise. One simple 
example: when showing a horizontal list of items (cards, tiles, buttons)
you may want **all of them to have the same width**, even though their 
content differs. More specifically, you might want 
**each item to be as wide as the largest one**.

It sounds simple, but if you've tried this in Compose, you've probably 
hit a wall.

In this article, we'll look at **why this is a tricky problem**, 
how Compose's layout system works in this case, and build a reusable
`UniformWidthRow` that solves it from the ground up.

## The Problem

Let's say you've built a 
[`TilesCircle`](https://gist.github.com/danielpassos/9cfb63b0c432301f47d9e68c2bb9c896/b5e267ef3b5ecf72145018ed09e71fe03d680c00#file-tilescircle-kt) 
and are using it on your screen like this:

```kotlin
@Composable
fun App() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        LazyRow(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            item {
                TilesCircle(
                    icon = Icons.Default.AccountCircle,
                    text = "Account\nCircle"
                )
            }

            item {
                TilesCircle(
                    icon = Icons.Default.Call,
                    text = "Call"
                )
            }

            item {
                TilesCircle(
                    icon = Icons.Default.ShoppingCart,
                    text = "Shopping\nCart"
                )
            }

            item {
                TilesCircle(
                    icon = Icons.Default.Settings,
                    text = "Settings"
                )
            }

            item {
                TilesCircle(
                    icon = Icons.Default.FavoriteBorder,
                    text = "Favorite\nBorder"
                )
            }
        }
    }
}
```

But when you run the app, you notice that **the spacing between tiles looks
off**. Because each tile has different text lengths (and therefore different 
widths), their boundaries don't align, and the spacing between them looks 
visually uneven.

|                                                      Distance                                                       |                                                         Colored                                                         |
|:-------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------:|
| ![App with Tiles](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-tiles1-720w.jpg) | ![Tiles with colors](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-colors1-720w.jpg) |

We want all tiles to **have the same width**, ideally the width of the 
largest one because it helps create a more visually harmonious layout. 
When elements are aligned and consistently sized, users can scan content 
more easily, and the interface feels more structured and intentional. 
This small detail improves readability and the overall sense of polish 
in the design.

The first idea most Compose devs try is:

```kotlin
LazyRow {
    items(items) {
        TilesCircle(icon = it.icon, text = it.text)
    }
}
```

But Compose's layout system doesn't work the way you might hope.

Here's the core problem:

### Layout is a one-pass operation

[Compose measures](https://developer.android.com/develop/ui/compose/phases#phase2-layout) 
each item in isolation. It doesn't know what the other items will be —and it
doesn't allow you to go back and change a previous item after seeing the next 
one. There is no "look ahead" or "measure everything first" phase in a 
`Row` or `LazyRow`.

This means:

- Each item chooses its own width
- You can't dynamically make them all match the widest one
- You can't "ask" what the largest item is before laying them out

`LazyRow` optimizes for performance by **only composing and measuring visible 
items**. It doesn't know how wide the others are unless you scroll to them. 
This means it's impossible to scan all items to find the widest one in a 
`LazyRow`.

## The Solution: SubcomposeLayout

To make this work, we need to:

1. Measure **all** items once, independently, to determine the maximum width.
2. Then, remeasure all items again, but this time constrain them to that 
max width.
3. Finally, place them in a scrollable horizontal layout with spacing and 
padding.

This is only possible with `SubcomposeLayout`, because unlike standard layouts 
in Compose, it allows you to compose and measure child elements in multiple 
phases. This means we can first measure all the items to determine the maximum 
width, and then recompose each one with that width constraint applied. It's 
the only way in Compose to measure all children before deciding how to layout
them, which is essential when one item's layout depends on the size of others.

## Building a Uniform Width Row

Instead of building a lazy structure, we'll keep things simple and accept a list of composables.

```kotlin
@Composable
fun UniformWidthRow(
    items: List<@Composable () -> Unit>
) {
    val scrollState = rememberScrollState()

    SubcomposeLayout(
        modifier = Modifier.horizontalScroll(scrollState)
    ) { constraints ->
        layout(0, 0) {}
    }
}
```

This gives us full control and avoids LazyRow's limitations.

### Step 1: Subcompose and Measure All Items

We start by invoking each item using `subcompose`, giving each 
item a unique key (usually based on its index). This allows us 
to trigger Compose to create the contents for each item manually:

```kotlin
val measurablesFirstPass = items.mapIndexed { index, content ->
    subcompose("measure-$index", content).first()
}
```

Then we measure each of those with the original constraints:

```kotlin
val placeablesFirstPass = measurables.map { it.measure(constraints) }
```

Now we can calculate the maxWidth of all items:

```kotlin
val maxWidth = placeablesFirstPass.maxOfOrNull { it.width } ?: 0
```

### Step 2: Remeasure With Fixed Width

We now remeasure each item again, but this time force them all to use 
`maxWidth` by copying the constraints and setting both `minWidth` and 
`maxWidth` to that value:

```kotlin
val finalMeasurables = items.mapIndexed { index, content ->
    subcompose("recomposed-$index", content).first()
}
val finalPlaceables = finalMeasurables.map {
    it.measure(Constraints.fixedWidth(maxWidth))
}
```
### Step 3: Compute Size and Layout

We calculate the total width of the row by summing all the item widths, 
spacing, and padding:

```kotlin
val componentWidth = placeables.sumOf { it.width }
```

We also get the max height for vertical sizing:

```kotlin
val componentHeight = placeables.maxOfOrNull { it.height } ?: 0
```

And finally, we place each item left to right with spacing:

```kotlin 
layout(width = componentWidth, height = componentHeight) {
    var x = 0
    finalPlaceables.forEach { placeable ->
        placeable.placeRelative(x, 0)
        x += placeable.width
    }
}
```

⚠️ [Full version on Gist](https://gist.github.com/danielpassos/9cfb63b0c432301f47d9e68c2bb9c896/63926bfd3940966824aab119762c95e0e5dd5f89#file-uniformwidthrow-kt)

|                                                      Distance                                                       |                                                         Colored                                                         |
|:-------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------:|
| ![App with Tiles](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-tiles2-720w.jpg) | ![Tiles with colors](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-colors2-720w.jpg) |

## Adding Content Padding

To make our `UniformWidthRow` behave more like `LazyRow`, we can support `contentPadding`. This allows developers to define spacing at the start and end of the scrollable row — a common design requirement when aligning lists with other screen elements.

Let’s update our UniformWidthRow function signature to include it:

```kotlin
@Composable
fun UniformWidthRow(
    contentPadding: PaddingValues = PaddingValues(0.dp),
    items: List<@Composable () -> Unit>
)
```

We’ll need to do a few things:

### 1. Convert padding to pixels

Because layout is done in pixels (not `Dp`), we convert `PaddingValues` into pixel values using `density`:

```kotlin
val startPaddingPx = contentPadding
    .calculateStartPadding(layoutDirection)
    .roundToPx()
val endPaddingPx = contentPadding
    .calculateEndPadding(layoutDirection)
    .roundToPx()
```

### 2. Update total width

When calculating the total width of the layout, we need to include the start and end padding to the size of the component:

```kotlin
val componentWidth = startPaddingPx + finalPlaceables.sumOf { it.width } + endPaddingPx
```

### 3. Update item placement

Before placing the first item, we offset the initial `x` position by the start padding:

```kotlin
var x = startPaddingPx
finalPlaceables.forEach { placeable ->
    placeable.placeRelative(x, 0)
    x += placeable.width
}
```
⚠️ [Full version on Gist](https://gist.github.com/danielpassos/9cfb63b0c432301f47d9e68c2bb9c896/ad730024ff5c217c30bd4ab8c9acabfcbb550642#file-uniformwidthrow-kt)

And that’s it! With these small changes, your UniformWidthRow now has contentPadding, just like LazyRow. This makes it easier to drop into existing designs and match standard Compose behavior.

|                                                      Distance                                                       |                                                         Colored                                                         |
|:-------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------:|
| ![App with Tiles](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-tiles3-720w.jpg) | ![Tiles with colors](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-colors3-720w.jpg) |

## Supporting Horizontal Spacing

Just like `LazyRow`, it’s useful to control the spacing between items. For example, `LazyRow` lets you pass an Arrangement.spacedBy(16.dp) to easily insert spacing between list elements.

You might be tempted to support a full Arrangement.Horizontal API, but it comes with complexity. Arrangements like spacedBy, Center, or End control both spacing and alignment, which would require us to reimplement all that behavior, including calculating how to distribute items across the total layout width. That’s unnecessary here, because all items already have the same fixed width and we only want to add fixed space between items, like Arrangement.spacedBy

So instead of supporting a full `Arrangement.Horizontal`, we introduce a simpler `horizontalSpacing: Dp` parameter that gives us exactly what we need, predictable spacing between tiles, without the boilerplate.

Let’s update our UniformWidthRow function signature again to include it:

```kotlin
@Composable
fun UniformWidthRow(
    contentPadding: PaddingValues = PaddingValues(0.dp),
    horizontalSpacing: Dp = 0.dp,
    items: List<@Composable () -> Unit>
)
```

Then, inside the SubcomposeLayout, we apply the arrangement logic when placing the items.

### 1. Convert to Pixels

Compose layouts work in pixels, not `Dp`, so we convert `horizontalSpacing` using:

```kotlin
val spacingPx = horizontalSpacing.roundToPx()
```

### 2. Adjust the Total Width Calculation

When calculating how wide the layout should be, we now need to account for the spacing between items. If we have N items, there are N - 1 gaps between them.

```kotlin
val totalSpacingWidth = spacingPx * (finalPlaceables.size - 1)
val totalItemsWidth = finalPlaceables.sumOf { it.width }
val componentWidth = totalItemWidth + totalSpacingWidth + startPaddingPx + endPaddingPx
```

### 3. Update Placement Logic

```kotlin
layout(width = componentWidth, height = componentHeight) {
    var x = startPaddingPx
    finalPlaceables.forEachIndexed { index, placeable ->
        placeable.placeRelative(x, 0)
        x += placeable.width
        if (index < finalPlaceables.lastIndex) {
            x += spacingPx
        }
    }
}
```

⚠️ [Full version on Gist](https://gist.github.com/danielpassos/9cfb63b0c432301f47d9e68c2bb9c896/6cb15aa609ef170a12a8c98c7a00e6db7a85c066#file-uniformwidthrow-kt)

|                                                      Distance                                                       |                                                         Colored                                                         |
|:-------------------------------------------------------------------------------------------------------------------:|:-----------------------------------------------------------------------------------------------------------------------:|
| ![App with Tiles](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-tiles4-720w.jpg) | ![Tiles with colors](/images/posts/building-a-uniform-width-row-in-compose-using-subcomposelayout/app-colors4-720w.jpg) |

## Conclusion

Creating uniform width items in a horizontal row improves visual alignment and UI polish — especially when content sizes vary. However, Jetpack Compose’s regular `Layout` system, including `Row` and `LazyRow`, doesn’t support this use case because layout in Compose is a single-pass operation. Each item is measured independently, with no way to consider the size of its siblings.

That’s why `SubcomposeLayout` is essential here. It allows us to measure all items first, determine the widest one, and then remeasure everything to align to that width.

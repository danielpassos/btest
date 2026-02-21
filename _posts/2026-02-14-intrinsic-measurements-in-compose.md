---
layout: post
date: '2026-02-14'
cover: "/images/posts/intrinsic-measurements-in-compose/cover"
title: "Intrinsic Measurements in Compose"
summary: "When two siblings need to share height, Compose won’t solve it for you unless you understand intrinsics. In this deep dive, we explore how intrinsic measurement works and why it’s the right tool for cross-column alignment."
categories:  ["Compose"]
tags:  ["intrinsic", "measurements"]
featured: true
---

Some layout problems look trivial until you try to implement them correctly within Compose’s measurement model. This was one of those problems.

We were building a card component with a very common structure:

On the left side, dynamic textual content:

- A title (pharmacy name).
- Address and neighborhood (which may wrap).
- City and state (which may wrap).
- A chip at the bottom (“View map”).

On the right side:

- A small tag (“20M”) aligned with the title.
- A phone icon button aligned with the chip.

// TODO Image

Visually, this is simple. Architecturally, it forces us to confront how Compose measures children.

The height of the left column is _unknown_. Text may wrap to one, two, or three lines. Localization may increase its height. The right column, however, contains very little content, and yet it must stretch to exactly match the left column's height so that its icon button can sit at the bottom.

That requirement exposes something fundamental:

Siblings cannot coordinate their size directly. Only the parent can coordinate them. Unless the parent explicitly participates in that coordination, children will not "see" each other's dimensions during measurement.

## The Natural First Implementation

The first version of the component was idiomatic and structurally clean:

```kotlin
@Composable
fun PharmacyCard(
    pharmacy: Pharmacy,
    onViewMapClick: () -> Unit,
    onCallClick: () -> Unit
) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        shape = RoundedCornerShape(16.dp),
        border = BorderStroke(1.dp, MaterialTheme.colorScheme.outline)
    ) {
        Row {
            Column(modifier = Modifier.weight(1f)) {

                Text(
                    text = pharmacy.name,
                    style = MaterialTheme.typography.titleMedium
                )

                Spacer(Modifier.height(8.dp))

                Text(
                    text = "${pharmacy.address}, ${pharmacy.neighborhood}",
                    style = MaterialTheme.typography.bodySmall
                )

                Text(
                    text = "${pharmacy.city} - ${pharmacy.uf}",
                    style = MaterialTheme.typography.bodySmall
                )

                Spacer(Modifier.height(24.dp))

                Tag(                    
                    label = "View map"
                    onClick = onViewMapClick,
                )
            }

            Column(horizontalAlignment = Alignment.End) {
                Chip(
                    label = "20M"
                )
                
                Spacer(Modifier.weight(1f))

                IconButton(onClick = onCallClick) {
                    Icon(Icons.Default.Phone, contentDescription = null)
                }
            }
        }
    }
}
```

The right column uses a weighted spacer to push the icon button to the bottom. In isolation, that logic is correct.

// TODO image

Yet the icon does not align with the chip. The spacer does not expand.

To understand why, we must examine how measurement actually works in Compose.

## How Measurement Actually Works in Compose

Compose's layout model is deterministic and constraint-driven. Every layout pass follows the same contract:

- A parent receives constraints from its own parent.
- The parent measures each child with constraints it chooses.
- Each child returns a size within those constraints.
- The parent decides its own size based on its children.
- The parent positions its children in the layout phase.

The important detail is this:

> Measurement flows downward. Size flows upward.

> Constraints go from parent to child. Measured size goes from child to parent.

There is no sideways communication between siblings.

In our case, the `Row` receives constraints from its parent. In a typical list item, width is bounded, but height is effectively unbounded (the parent allows the `Row` to choose its own height).

When the Row measures its children:

1. It measures the left column.
2. It measures the right column.

It chooses its own height as the maximum of the two measured heights.

This sounds like it should work — but the timing is critical. The `Row` cannot determine its final height until after both children have already been measured.

Which means:

The right column was measured under an effectively unbounded height constraint, and when a column receives an unbounded height constraint, its weighted spacer has nothing to expand into.

The spacer only consumes extra space when the column itself is given a bounded height.

Without a bounded height, the column wraps its content.

By the time the Row decides that its final height should match the taller left column, it is too late. The right column has already completed measurement under the wrong constraints.

There is no automatic second chance. This is not a limitation. It is the contract.

## Why Intrinsic Measurement Exists

Intrinsic measurement exists for cases where a parent must know a child's preferred size **before** it can decide constraints for the final measure pass.

Instead of immediately measuring children normally, the parent can ask:

"If I constrained you in one dimension, what size would you want in the other?"

Compose exposes this through intrinsic queries:

- minIntrinsicWidth
- maxIntrinsicWidth
- minIntrinsicHeight
- maxIntrinsicHeight

These are not regular measure passes. They are pre-measurement calculations that allow a parent to compute a size decision before performing the actual measurement.

When we apply:

```kotlin
Row(
    modifier = Modifier.height(IntrinsicSize.Max)
)
```

We are telling Compose:

Before performing the normal measurement pass, compute the maximum intrinsic height of this Row and use that as the Row's fixed height.

This fundamentally changes the measurement flow.

Instead of:

Measure children → decide parent height.

We now do:

Ask children their intrinsic height → decide parent height → measure children with that fixed height.

That inversion is the entire solution.

## What IntrinsicSize.Max means for a row.

For a `Row`, `IntrinsicSize.Max` works like this:

The `Row` queries each child for its intrinsic height given the available width.

It takes the maximum intrinsic height among them.

It sets its own height to that value.

It performs the normal measurement pass with a bounded height equal to that value.

Now the right column is no longer measured under an unbounded height.

It receives a concrete height constraint equal to the tallest child’s intrinsic height.

Suddenly, the weighted spacer has space to expand.

The icon button is pushed to the bottom.

The layout behaves exactly as intended.

The tag remains aligned at the top because both columns start at the same vertical origin.

No hardcoded height. No coupling between columns. No manual synchronization.

Just a different measurement strategy.

## What actually changed architecturally

The only thing that changed was when the parent committed to its height.

Previously, height was decided after the children were measured.

Now, height is decided before children measure.

That shift allows the parent to enforce shared vertical constraints across siblings.

This is not a workaround. It is the correct application of intrinsic measurement as designed.

We are not bending the layout system. We are using one of its explicit coordination mechanisms.

## The final version

With intrinsic sizing applied, the component becomes structurally sound:

```kotlin
Row(
    modifier = Modifier.height(IntrinsicSize.Max)
) {
    Column(modifier = Modifier.weight(1f)) {
        // left content
    }

    Column(horizontalAlignment = Alignment.End) {
        // right content
    }
}
```

// TODO Image

## The mental model

The lesson here is not about pharmacy cards.

It is about internalizing the Compose layout contract deeply enough that these issues become predictable.

Compose does not allow siblings to coordinate size directly. If sibling coordination is required, the parent must participate.

Intrinsic measurement is the mechanism that allows a parent to base its constraints on children's preferred sizes before committing to the final measurement.

Once you understand that measurement is constraint-driven and unidirectional, you stop fighting these layouts, you start designing them, and that is the difference between knowing Compose and mastering it.

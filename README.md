# Flutter_doc_CokBK_Efct_Create_a_scrolling_parallax_effect
 https://docs.flutter.dev/cookbook/effects/parallax-scrolling#interactive-example
Create a scrolling parallax effect
==================================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Effects](https://docs.flutter.dev/cookbook/effects)
3.  [Create a scrolling parallax effect](https://docs.flutter.dev/cookbook/effects/parallax-scrolling)

When you scroll a list of cards (containing images, for example) in an app, you might notice that those images appear to scroll more slowly than the rest of the screen. It almost looks as if the cards in the list are in the foreground, but the images themselves sit far off in the distant background. This effect is known as parallax.

In this recipe, you create the parallax effect by building a list of cards (with rounded corners containing some text). Each card also contains an image. As the cards slide up the screen, the images within each card slide down.

The following animation shows the app's behavior:

![Parallax scrolling](https://docs.flutter.dev/assets/images/docs/cookbook/effects/ParallaxScrolling.gif)

[](https://docs.flutter.dev/cookbook/effects/parallax-scrolling#create-a-list-to-hold-the-parallax-items)Create a list to hold the parallax items
-------------------------------------------------------------------------------------------------------------------------------------------------

To display a list of parallax scrolling images, you must first display a list.

Create a new stateless widget called `ParallaxRecipe`. Within `ParallaxRecipe`, build a widget tree with a `SingleChildScrollView` and a `Column`, which forms a list.

content_copy

```
class ParallaxRecipe extends StatelessWidget {
  const ParallaxRecipe({super.key});

  @override
  Widget build(BuildContext context) {
    return const SingleChildScrollView(
      child: Column(
        children: [],
      ),
    );
  }
}
```

[](https://docs.flutter.dev/cookbook/effects/parallax-scrolling#display-items-with-text-and-a-static-image)Display items with text and a static image
-----------------------------------------------------------------------------------------------------------------------------------------------------

Each list item displays a rounded-rectangle background image, exemplifying one of seven locations in the world. Stacked on top of that background image is the name of the location and its country, positioned in the lower left. Between the background image and the text is a dark gradient, which improves the legibility of the text against the background.

Implement a stateless widget called `LocationListItem` that consists of the previously mentioned visuals. For now, use a static `Image` widget for the background. Later, you'll replace that widget with a parallax version.

content_copy

```
@immutable
class LocationListItem extends StatelessWidget {
  const LocationListItem({
    super.key,
    required this.imageUrl,
    required this.name,
    required this.country,
  });

  final String imageUrl;
  final String name;
  final String country;

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
      child: AspectRatio(
        aspectRatio: 16 / 9,
        child: ClipRRect(
          borderRadius: BorderRadius.circular(16),
          child: Stack(
            children: [
              _buildParallaxBackground(context),
              _buildGradient(),
              _buildTitleAndSubtitle(),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildParallaxBackground(BuildContext context) {
    return Positioned.fill(
      child: Image.network(
        imageUrl,
        fit: BoxFit.cover,
      ),
    );
  }

  Widget _buildGradient() {
    return Positioned.fill(
      child: DecoratedBox(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [Colors.transparent, Colors.black.withOpacity(0.7)],
            begin: Alignment.topCenter,
            end: Alignment.bottomCenter,
            stops: const [0.6, 0.95],
          ),
        ),
      ),
    );
  }

  Widget _buildTitleAndSubtitle() {
    return Positioned(
      left: 20,
      bottom: 20,
      child: Column(
        mainAxisSize: MainAxisSize.min,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(
            name,
            style: const TextStyle(
              color: Colors.white,
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          Text(
            country,
            style: const TextStyle(
              color: Colors.white,
              fontSize: 14,
            ),
          ),
        ],
      ),
    );
  }
}
```

Next, add the list items to your `ParallaxRecipe` widget.

content_copy

```
class ParallaxRecipe extends StatelessWidget {
  const ParallaxRecipe({super.key});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      child: Column(
        children: [
          for (final location in locations)
            LocationListItem(
              imageUrl: location.imageUrl,
              name: location.name,
              country: location.place,
            ),
        ],
      ),
    );
  }
}
```

You now have a typical, scrollable list of cards that displays seven unique locations in the world. In the next step, you add a parallax effect to the background image.

[](https://docs.flutter.dev/cookbook/effects/parallax-scrolling#implement-the-parallax-effect)Implement the parallax effect
---------------------------------------------------------------------------------------------------------------------------

A parallax scrolling effect is achieved by slightly pushing the background image in the opposite direction of the rest of the list. As the list items slide up the screen, each background image slides slightly downward. Conversely, as the list items slide down the screen, each background image slides slightly upward. Visually, this results in parallax.

The parallax effect depends on the list item's current position within its ancestor `Scrollable`. As the list item's scroll position changes, the position of the list item's background image must also change. This is an interesting problem to solve. The position of a list item within the `Scrollable` isn't available until Flutter's layout phase is complete. This means that the position of the background image must be determined in the paint phase, which comes after the layout phase. Fortunately, Flutter provides a widget called `Flow`, which is specifically designed to give you control over the transform of a child widget immediately before the widget is painted. In other words, you can intercept the painting phase and take control to reposition your child widgets however you want.

info Note: To learn more, watch this short Widget of the Week video on the Flow widget:

info Note: In cases where you need control over what a child paints, rather than where a child is painted, consider using a [`CustomPaint`](https://api.flutter.dev/flutter/widgets/CustomPaint-class.html) widget.

In cases where you need control over the layout, painting, and hit testing, consider defining a custom [`RenderBox`](https://api.flutter.dev/flutter/rendering/RenderBox-class.html).

Wrap your background `Image` widget with a [`Flow`](https://api.flutter.dev/flutter/widgets/Flow-class.html) widget.

content_copy

```
Widget _buildParallaxBackground(BuildContext context) {
  return Flow(
    children: [
      Image.network(
        imageUrl,
        fit: BoxFit.cover,
      ),
    ],
  );
}
```

Introduce a new `FlowDelegate` called `ParallaxFlowDelegate`.

content_copy

```
Widget _buildParallaxBackground(BuildContext context) {
  return Flow(
    delegate: ParallaxFlowDelegate(),
    children: [
      Image.network(
        imageUrl,
        fit: BoxFit.cover,
      ),
    ],
  );
}
```

content_copy

```
class ParallaxFlowDelegate extends FlowDelegate {
  ParallaxFlowDelegate();

  @override
  BoxConstraints getConstraintsForChild(int i, BoxConstraints constraints) {
    // TODO: We'll add more to this later.
  }

  @override
  void paintChildren(FlowPaintingContext context) {
    // TODO: We'll add more to this later.
  }

  @override
  bool shouldRepaint(covariant FlowDelegate oldDelegate) {
    // TODO: We'll add more to this later.
    return true;
  }
}
```

A `FlowDelegate` controls how its children are sized and where those children are painted. In this case, your `Flow` widget has only one child: the background image. That image must be exactly as wide as the `Flow` widget.

Return tight width constraints for your background image child.

content_copy

```
@override
BoxConstraints getConstraintsForChild(int i, BoxConstraints constraints) {
  return BoxConstraints.tightFor(
    width: constraints.maxWidth,
  );
}
```

Your background images are now sized appropriately. But, you still need to calculate the vertical position of each background image based on its scroll position, and then paint it.

There are three critical pieces of information that you need to compute the desired position of a background image:

-   The bounds of the ancestor `Scrollable`
-   The bounds of the individual list item
-   The size of the image after it's scaled down to fit in the list item

To look up the bounds of the `Scrollable`, you pass a `ScrollableState` into your `FlowDelegate`.

To look up the bounds of your individual list item, you pass your list item's `BuildContext` into your `FlowDelegate`.

To look up the final size of your background image, you assign a `GlobalKey` to your `Image` widget, and then you pass that `GlobalKey` into your `FlowDelegate`.

Make this information available to `ParallaxFlowDelegate`.

content_copy

```
@immutable
class LocationListItem extends StatelessWidget {
  final GlobalKey _backgroundImageKey = GlobalKey();

  Widget _buildParallaxBackground(BuildContext context) {
    return Flow(
      delegate: ParallaxFlowDelegate(
        scrollable: Scrollable.of(context),
        listItemContext: context,
        backgroundImageKey: _backgroundImageKey,
      ),
      children: [
        Image.network(
          imageUrl,
          key: _backgroundImageKey,
          fit: BoxFit.cover,
        ),
      ],
    );
  }
}
```

content_copy

```
class ParallaxFlowDelegate extends FlowDelegate {
  ParallaxFlowDelegate({
    required this.scrollable,
    required this.listItemContext,
    required this.backgroundImageKey,
  });

  final ScrollableState scrollable;
  final BuildContext listItemContext;
  final GlobalKey backgroundImageKey;
}
```

Having all the information needed to implement parallax scrolling, implement the `shouldRepaint()` method.

content_copy

```
@override
bool shouldRepaint(ParallaxFlowDelegate oldDelegate) {
  return scrollable != oldDelegate.scrollable ||
      listItemContext != oldDelegate.listItemContext ||
      backgroundImageKey != oldDelegate.backgroundImageKey;
}
```

Now, implement the layout calculations for the parallax effect.

First, calculate the pixel position of a list item within its ancestor `Scrollable`.

content_copy

```
@override
void paintChildren(FlowPaintingContext context) {
  // Calculate the position of this list item within the viewport.
  final scrollableBox = scrollable.context.findRenderObject() as RenderBox;
  final listItemBox = listItemContext.findRenderObject() as RenderBox;
  final listItemOffset = listItemBox.localToGlobal(
      listItemBox.size.centerLeft(Offset.zero),
      ancestor: scrollableBox);
}
```

Use the pixel position of the list item to calculate its percentage from the top of the `Scrollable`. A list item at the top of the scrollable area should produce 0%, and a list item at the bottom of the scrollable area should produce 100%.

content_copy

```
@override
void paintChildren(FlowPaintingContext context) {
  // Calculate the position of this list item within the viewport.
  final scrollableBox = scrollable.context.findRenderObject() as RenderBox;
  final listItemBox = listItemContext.findRenderObject() as RenderBox;
  final listItemOffset = listItemBox.localToGlobal(
      listItemBox.size.centerLeft(Offset.zero),
      ancestor: scrollableBox);
}

  // Determine the percent position of this list item within the
  // scrollable area.
  final viewportDimension = scrollable.position.viewportDimension;
  final scrollFraction =
      (listItemOffset.dy / viewportDimension).clamp(0.0, 1.0);
}
```

Use the scroll percentage to calculate an `Alignment`. At 0%, you want `Alignment(0.0, -1.0)`, and at 100%, you want `Alignment(0.0, 1.0)`. These coordinates correspond to top and bottom alignment, respectively.

content_copy

```
@override
void paintChildren(FlowPaintingContext context) {
  // Calculate the position of this list item within the viewport.
  final scrollableBox = scrollable.context.findRenderObject() as RenderBox;
  final listItemBox = listItemContext.findRenderObject() as RenderBox;
  final listItemOffset = listItemBox.localToGlobal(
      listItemBox.size.centerLeft(Offset.zero),
      ancestor: scrollableBox);
}

  // Determine the percent position of this list item within the
  // scrollable area.
  final viewportDimension = scrollable.position.viewportDimension;
  final scrollFraction =
      (listItemOffset.dy / viewportDimension).clamp(0.0, 1.0);
}
  // Calculate the vertical alignment of the background
  // based on the scroll percent.
  final verticalAlignment = Alignment(0.0, scrollFraction * 2 - 1);
}
```

Use `verticalAlignment`, along with the size of the list item and the size of the background image, to produce a `Rect` that determines where the background image should be positioned.

content_copy

```
@override
void paintChildren(FlowPaintingContext context) {
  // Calculate the position of this list item within the viewport.
  final scrollableBox = scrollable.context.findRenderObject() as RenderBox;
  final listItemBox = listItemContext.findRenderObject() as RenderBox;
  final listItemOffset = listItemBox.localToGlobal(
      listItemBox.size.centerLeft(Offset.zero),
      ancestor: scrollableBox);
}

  // Determine the percent position of this list item within the
  // scrollable area.
  final viewportDimension = scrollable.position.viewportDimension;
  final scrollFraction =
      (listItemOffset.dy / viewportDimension).clamp(0.0, 1.0);
}
  // Calculate the vertical alignment of the background
  // based on the scroll percent.
  final verticalAlignment = Alignment(0.0, scrollFraction * 2 - 1);
}
  // Convert the background alignment into a pixel offset for
  // painting purposes.
  final backgroundSize =
      (backgroundImageKey.currentContext!.findRenderObject() as RenderBox)
          .size;
  final listItemSize = context.size;
  final childRect =
      verticalAlignment.inscribe(backgroundSize, Offset.zero & listItemSize);
}
```

Using `childRect`, paint the background image with the desired translation transformation. It's this transformation over time that gives you the parallax effect.

content_copy

```
@override
void paintChildren(FlowPaintingContext context) {
  // Calculate the position of this list item within the viewport.
  final scrollableBox = scrollable.context.findRenderObject() as RenderBox;
  final listItemBox = listItemContext.findRenderObject() as RenderBox;
  final listItemOffset = listItemBox.localToGlobal(
      listItemBox.size.centerLeft(Offset.zero),
      ancestor: scrollableBox);
}

  // Determine the percent position of this list item within the
  // scrollable area.
  final viewportDimension = scrollable.position.viewportDimension;
  final scrollFraction =
      (listItemOffset.dy / viewportDimension).clamp(0.0, 1.0);
}
  // Calculate the vertical alignment of the background
  // based on the scroll percent.
  final verticalAlignment = Alignment(0.0, scrollFraction * 2 - 1);
}
  // Convert the background alignment into a pixel offset for
  // painting purposes.
  final backgroundSize =
      (backgroundImageKey.currentContext!.findRenderObject() as RenderBox)
          .size;
  final listItemSize = context.size;
  final childRect =
      verticalAlignment.inscribe(backgroundSize, Offset.zero & listItemSize);
}
  // Paint the background.
  context.paintChild(
    0,
    transform:
        Transform.translate(offset: Offset(0.0, childRect.top)).transform,
  );
}
```

You need one final detail to achieve the parallax effect. The `ParallaxFlowDelegate` repaints when the inputs change, but the `ParallaxFlowDelegate` doesn't repaint every time the scroll position changes.

Pass the `ScrollableState`'s `ScrollPosition` to the `FlowDelegate` superclass so that the `FlowDelegate` repaints every time the `ScrollPosition` changes.

content_copy

```
class ParallaxFlowDelegate extends FlowDelegate {
  ParallaxFlowDelegate({
    required this.scrollable,
    required this.listItemContext,
    required this.backgroundImageKey,
  }) : super(repaint: scrollable.position);
}
```

Congratulations! You now have a list of cards with parallax, scrolling background images.

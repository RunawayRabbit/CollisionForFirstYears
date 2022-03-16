# Euan's Way-Too-Long Collision Detection Thing

Hello everyone, welcome to my TEDTalk. This was supposed to be a quick explainer on basic collision detection of AABBs and Circles, but it kind of ballooned out into a mini-course. I hope it's useful cuz it took way longer than I expected.

## AABB Tests

### Simplify First: One Dimension

Conceptually, an AABB is a set of `n` ranges where `n` is the number of dimensions you're dealing with. To make that a bit more concrete, let's start with one dimension. Consider a line segment.

![](img/min_max.png)

A "Line" would be the white-ish line in this drawing, and it goes to infinity in both directions. A "Ray" can be defined as "Half A Line". That is, it has a beginning, but it has no end. It starts somewhere and goes to infinity. A "Line Segment" has a beginning *and* an end. There's some terminology for ya!
 
Now let's consider a second line segment:

![](img/two_line_segments.png)

Now ask yourself the question, **"do these lines overlap?"** The answer is clear from the picture, but if you only had the `min`/`max` values for both lines, could you work it out?

If you were to think about this for a while, you might try to consider all of the cases for how they can overlap, but I'll do you a favour and tell you: it's easier to think of the ways that they can **not** overlap. Why? Because there's only a couple of 'em.

![](img/two_lines_beside_eachother.png)

Either the red line segment is completely to the left of the blue one, or the red line is completely to the right of the blue one. If neither of those is true, then the lines don't overlap.

Another way of stating this is that

> if the left-most (ie: the minimum) point on the red line is to the right (greater than) of the right-most (maximum) of the blue line, no overlap exists. Or vica verca.

That becomes a simple test to structure!

```!(red.max < blue.min || blue.max < red.min)```

or rephrased to avoid the !..

```(red.max > blue.min && blue.max > red.min)```

### Boxes

So that's cool. But those are lines, lines aren't boxes, fuck off with your lines and min/maxes Euan we're doing boxes today and I don't see any boxes.

Don't worry, we're half way there! we have a 1D test, we just need to extend it into 2D. Consider a box.

![](img/a_box.png)

Now let's draw some axes on it.

![](img/a_box_with_axes.png)

so we already know how to test overlaps on lines. And the X axis is a line, right? so..

![](img/aabb_x_axis.png)

If we introduce a second box, we can now do the exact same test to see if they overlap in X.

![](img/aabb_2.png)

if they don't overlap in 1D, then they cannot overlap in 2D. Look at this picture and try to imagine a place where you can put the red box where it overlaps with purple, without the min/max ranges overlapping in the X axis. Can't do it! 

That's actually a general truth for all convex shapes, which is cool. But what about if they do overlap in X?

![](img/aabb_3.png)

**They still might not overlap.** The way we test to see if they do, as you might've guessed.. is to do the exact same thing we did before. We just do it on the other axis.

### Getting To The Point 

Using our overlapping test idea from before, we can extend to two dimensions.

```
(red.max.x > purple.min.x && purple.max.x > red.min.x)
(red.max.y > purple.min.y && purple.max.y > red.min.y)
```

But there's one question left I guess, which is how do we **combine** these? Is it overlapping if either of the two 1D tests are true? or both of them?

The answer is, it has to be both. Because as we just saw, you can quite happily overlap in one dimension and not the other, and still not be overlapping as a 2D shape. **Every dimension has to overlap for the shapes to actually overlap.**

So our final test is:

```
bool
AABBOverlapsAABB(const AABB& a, const AABB& b)
{
	return
	(a.max.x > b.min.x && b.max.x > a.min.x) &&
	(a.max.y > b.min.y && b.max.y > a.min.y);
}

```

And there's the 2D case. The 3D case would be the exact same thing, you'd just add a .z test in there, in the exact same fashion.

As a sidenote, this is the code for

```
struct AABB
{
	Vector2 min, max;
}
```

So that's AABB tests in a nutshell. 

If your AABBs are stored differently, this might be more what you're looking for. Hopefully you can see how they are equivalent!

```
struct AABB
{
	Vector2 pos, dim;
}

bool
AABBOverlapsAABB(const AABB& a, const AABB& b)
{
	Vector2 aMax = a.pos + a.dim;
	Vector2 bMax = b.pos + b.dim;

	return
	(aMax.x > b.pos.x && bMax.x > a.pos.x) &&
	(aMax.y > b.pos.y && bMax.y > a.pos.y);
}
```


## Circles vs AABBs

The next thing you'd probably want is a circle vs AABB test, because balls are not, and this is true.. boxes. The most convenient form for a circle to take when it comes to collision detection is something like this.

```
struct circle
{
  Vector2 center;
  float radius;
}
```

To find whether a circle overlaps with something, you can find the distance between the circle's center and the object you're testing against, and compare that to the circle's radius. If that distance is smaller than the radius, then it's overlapping!

![](img/circle_1.png)

"That's all well and good Euan but how the fuck am I supposed to know where the closest point is?" Great question, and there's two answers, the good one and the easy one. Let's stick to the easy one. Again, let's make life easy by going down to 1D. Imagine a line and a range once again. What is the "closest point" to the red dot on the blue line?

![](img/circle_2.png)

If you think about this problem for a bit, you'll see that there's basically 3 possible solutions. In the picture I showed, the closest point on the line is actually the line's **minimum**. If the red dot were on the other side (to the right of the line) then it would be the **maximum**. And if the dot was somewhere on the line then the **position of the dot itself** would be the closest point.

That's also pretty easy to phrase in code.

```
if(redDot < blueLine.min) return blueLine.min;
if(redDot > blueLine.max) return blueLine.max;
else return redDot;
```

The smart people among you might have noticed that this can also be phrased as:

```
max(blueLine.min, min(blueLine.max, redDot);
```

or even more succinctly, if you have a library that does these things..
```
// Clamp(x, min, max)
Clamp(redDot, blueLine.min, blueLine.max);
```

And once again, this scales to 2 dimensions very easily.

![](img/circle_3.png)

In this example, the `center` of the circle is greater than `max.y` in Y, so we take `max.y`, and less than `min.x` in X, so we take `min.x`.

In code:

```
Vector2
ClosestPointInAABBToPoint(const AABB& aabb, const Vector2& point)
{
	Vector2 result;

	result.x = Clamp(point.x, aabb.min.x, aabb.max.x);
	result.y = Clamp(point.y, aabb.min.y, aabb.max.y);

	return result;
}
```

So now all that's left to do is measure the distance between our circle's center and it's closest point in the AABB. If the point is closer to the circle's center than the radius, then that point is inside the circle, and we have an overlap.

```
bool
CircleOverlapsAABB(const Circle& circle, const AABB& aabb)
{
	Vector2 closestPoint = ClosestPointInAABBToPoint(aabb, circle.center);
	Vector2 difference = closestPoint - circle.center;

	if(difference.magnitude() < circle.radius) return true;
	else return false;
}
```

The magnitude of the vector, as you'll remember from Freya's lectures, just the dot product of a vector and itself. And the Dot product, as you'll also remember, is just the sum of all of the components, multiplied together.

```
float
Vector2::dot(const Vector2& a, const Vector2& b)
{
	return a.x * b.x + a.y * b.y;
}

float
Vector2::sqMagnitude()
{
	return dot(&this, &this);
}

float
Vector2::magnitude()
{
	return sqrt(this.sqMagnitude);
}


```

If you're fancy, though, you'll probably know that for this kind of `>` or `<` comparison, you don't actually *need* the magnitude of the vector. You can skip the square root and just do this instead: 

```
bool
CircleOverlapsAABB(const Circle& circle, const AABB& aabb)
{
	Vector2 closestPoint = ClosestPointInAABBToPoint(aabb, circle.center);
	Vector2 difference = closestPoint - circle.center;

	return 
	(difference.sqMagnitude() < circle.radius * circle.radius);
}
```
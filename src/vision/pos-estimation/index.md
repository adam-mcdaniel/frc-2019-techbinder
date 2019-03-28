# pos-estimation

![Pos Estimation](../../position-estimation-large.gif)

Position Estimation is an incredibly important part of our robot's code. It allows us to know our exact position in space relative to a vision target on the field.

![Pos Estimation 1](../../solve-pnp-1.gif)


> Using an OpenCV feature called Solve Perspective-n-Point, we can take the corners of an object on an image, compare them to their absolute positions relative to each other, get the perspective using this comparison, use the perspective to determine the X, Y, Z translation vector, and the euler angles of the target.

![SolvePNP](../../solvepnp.png)

To interact with SolvePNP's horribly unreadable and mind boggling code, we built a wrapper of our own called `pos_tools`.

Pos-Tools takes an object targetted using small-vision, and returns the corners of the object in counter-clockwise order, starting from the top right.

![Corners](../../corners.jpg)

> Then, we can use a PositionEstimator object to find the world coordinates of the targetted object using the corners of the object found using the method above, and the coordinates of the corners of the targetted object relative to each other.

Lastly, the PositionEstimator gives us the values given by SolvePNP. With these values, we can do some transformations on the image, and show the axes of the target's world coordinates like so.

![Pos Estimation 2](../../solve-pnp-2.gif)

With these data points, we can start to do path planning.

# auto-bot


![Auto Bot](../auto-bot-large.gif)

### Making modular and extendable autonomous code

Auto-bot is a library that allows us to structure our autonomous code in lists of segments that are easily interchangable, expandable, and manipulatable. Not at all unlike lego blocks.

> In auto-bot, a sequence of tasks is called a Path. You do not have to create your own types of Paths, but you can. Instead, what you should do is create your own Segment types, like so.

```python
class Start(Segment):
    def update(self, robot):
        if robot.get_angle() == 0:
            self.finish()
        elif robot.get_angle() < 0:
            robot.arcade_drive(0.1, 0)
        elif robot.get_angle() > 0:
            robot.arcade_drive(-0.1, 0)
```

This path Segment attempts to rotate the bot left or right until the robot's gyro reads zero degrees.

To implement this in our robot, all we need to do is this.


```python
@run
class MyRobot(TimedRobot):
    def robotInit(self):
        self.path = None

    def autonomousInit(self):
        self.path = Path(self, Start())

    def autonomousPeriodic(self):
        if not self.path.is_done():
            self.path.update(self)

    def get_angle(self): return 0       # implement me
    def arcade_drive(self, z, x):       # implement me
    def vision_target_is_in_range(self) # implement me
```


> We can also build more path Segments, and piece those together as needed.

Here are a few more sample Segments we can utilize

```python
class Intermediate(Segment):
    def update(self, robot):
        if not robot.vision_target_is_in_range():
            robot.arcade_drive(0.1, 1)
        else:
            self.finish()


class VisionApproach(Segment):
    def config(self, robot):
        print("Vision approach segment starting")
        robot.brake()

    def update(self, robot):
        if robot.get_angle() == 0:
            self.finish()
        elif robot.get_angle() < 0:
            robot.arcade_drive(0.1, 0)
        elif robot.get_angle() > 0:
            robot.arcade_drive(-0.1, 0)
```

We can add these Segments to our path by modifying the instantiation of the Path object.

```python
    def autonomousInit(self):
        self.path = Path(self,
            Start(),
            Intermediate(),
            VisionApproach()
        ) 
        # First, run the start segment, 
        # then the intermediate segment,
        # then the VisionApproach segment
```

We can also mix and match them, or call multiple instances of each path Segment.

```python
    def autonomousInit(self):
        self.path = Path(self,
            Start(),
            Start(),
            Start(),
            Start(),
            Intermediate(),
            Start(),
            Intermediate(),
            VisionApproach()
        ) 
        # First, run the start segment 4 times, 
        # then the intermediate segment,
        # then the start segment again,
        # then the intermediate segment again,
        # then the VisionApproach segment
```

This allows us to queue tasks for our robot to follow in real time. In addition, it let's us define very readable and customizable autonomous code.
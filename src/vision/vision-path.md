# Posisition estimation and Pathfinder

![Pos Estimation and Pathfinder](../../pos-estimation-large.gif)

> Using the data points given to us through position estimation, we can get the angle 1, angle 2, and the distance to the target.

![Pos Estimation Data Points](../../angles.png)

With the data in the diagram, we can immediately plug those values into pathfinder using our `auto-bot` | `remote-path` Trajectory object.

```python
def sign(n): return -1 if n < 0 else 1

class VisionTrajectory(Trajectory):
    def config(self, robot):
        print("Starting vision trajectory")
        current_angle = radians(robot.chassis.gyro.getAngle())

        distance_to_target = None
        angle1 = None
        angle2 = None

        # target_info[0] = timestamp of image in seconds.
        # target_info[1] = success (1.0) OR failure (0.0)
        # target_info[2] = mode (1=driver, 2=rrtarget)
        # target_info[3] = distance to target (inches)
        # target_info[4] = angle1 to target (radians) -- angle displacement of robot to target
        # target_info[5] = angle2 of target (radians) -- angle displacement of target to robot
        target_info = robot.dashboard.getNumberArray("vision/target_info", None)
        if target_info:
            distance_to_target = target_info[3]
            angle1 = target_info[4]
            angle2 = target_info[5]
        
            d = (distance_to_target+9)/12
            pathfinder_angle = radians(90) - abs(angle2)


            pathfinder_y = d * sign(angle1) * cos(pathfinder_angle)
            pathfinder_x = d * sin(pathfinder_angle)

            # print(pathfinder_y, pathfinder_x)

            final_angle = sign(angle1) * (radians(90) - abs(pathfinder_angle))
            
            super().__init__([
                (0, 0, 0),
                (pathfinder_x, -pathfinder_y, final_angle - current_angle),
            ])

            super().config(robot)
            # self.finish()

        else:
            print("Couldnt read target info")
            self.finish()
```

> And because the `Trajectory` object is a path segment that utilizes `remote_path`, we can create and start this path all under 0.1 seconds!

To implement this and automate our hatch and cargo deployment, we would simply insert this under a button press in our robot's teleop period method.

```python
    def teleopPeriodic(self:)
        if self.joystick.get_auto_dock():
            self.path.update(self)

        elif self.joystick.get_reset_auto_dock():
            self.path = Path(self, VisionTrajectory())
```

And our auto-alignment and auto-docking code is finished! Our robot now lines up with the vision targets as well as vision can accomplish.

> However, there are some setbacks. If SolvePNP returns bad data, pathfinder will receive bad data. This generally means that your robot will ram the rocket or cargo ship at full speed.

For this reason, it's not completely safe to use yet. Nevertheless, it has still helped us deploy several hatches and cargo very quickly.
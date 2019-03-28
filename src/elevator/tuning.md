# PID Controller

![PID Ex 1](../pid_graph.gif)

Our elevator is powered by a PID controller.
> A PID controller is a software tool that attempts to hone in on a given setpoint using a closed feedback loop.

This allows us to control our elevator very effectively without even needing a state machine.

Here's the code responsible for the majority of our robot's elevator's behavior.

```python
    def zero(self):
        self.set_commanded_position(0)
    
    def low(self):
        # go 9 inches up
        self.set_commanded_position(9/12 * 3.14)
    
    def mid(self):
        # go 35.5 inches up
        self.set_commanded_position(35.5/12 * 3.14)

    def high(self):
        # go 59.5 inches up
        self.set_commanded_position(59.5/12 * 3.14)

    def cargo_ship(self):
        # go 1.8687 feet up
        self.set_commanded_position(1.8687 * 3.14)
```

Now that we've abstracted the positions of our elevator, we can use the abstracted Joystick wrapper that we wrote to interact with it using extremely human readable and easily debuggable code.

```python
    # in teleop periodic
    
    # update the elevator's state
    self.elevator.update()


    # Command an elevator position
    if self.joystick.get_elevator_low():
        self.elevator.low()

    elif self.joystick.get_elevator_mid():
        self.elevator.mid()

    elif self.joystick.get_elevator_high():
        self.elevator.high()

    elif self.joystick.get_elevator_cargo_ship():
        self.elevator.cargo_ship()
```

Using just this code, our elevator is extremely accurate and resistant to any outside forces. It behaves very similar to this example of a PID in use.

![PID Ex 2](../pid_ball.gif)

Now of course, major setup is required for a PID closed loop controller.

Here is some of the code responsible for the initialization of the elevator.


```python
class Elevator:
    '''
    Object that controls the Elevator on the robot
    '''
    PIDIDX = 0

    # This controls the angle at which the elevator
    # decides the robot is tipping.
    TippingBound = 11

    PigeonCanID = 19
    OUTPUT_SHAFT_DIAMETER = 1.76 / 12  # in feet
    PID_SPEED = 0.68

    UP_SPEED = 0.30
    DOWN_SPEED = -0.18
    SLOW_DOWN_SPEED = -0.05
    HOLD_SPEED = 0.1

    KP = ntproperty("/SmartDashboard/KP", 0, writeDefault=True)
    KI = 0
    KD = ntproperty("/SmartDashboard/KD", 0, writeDefault=True)

    def __init__(self, master_id, slave_ids):
        '''
        Insantiate Elevator object on our bot.

        It takes the ids of the motors controlling the elevator as
        a variable list of arguments.

        Instantiate it like this: `Elevator(1, 2, 3, 4)`, assuming
        motors with ids 1, 2, 3, and 4 are
        driving your elevator.
        '''
        # The pigeon is a small gyro/accelerometer with 9 degrees
        # of freedom on the bot.

        # When it boots up, we want it to zero itself. 
        # However, there is no reset or calibrate
        # function on the pigeon. Instead, we take the initial values 
        # that the gyro reads, and offset
        # the gyro values from then on by those initial values.

        # Effectively, we zero the pigeon.
        self.commanded_position = 0
        self.PitchOffset = 0
        self.RollOffset = 0
        
        try:
            self.pigeon = Pigeon(self.PigeonCanID)
        except:
            print("===| ERROR |===> Couldn't find pigeon")

        self.PitchOffset = self.get_pitch()
        self.RollOffset = self.get_roll()

        # Here we create an instance of the Talon with the master id.
        # All of the other motor ids given will folow the master motor.
        self.master_talon = Talon(master_id)
        self.talons = [self.master_talon]
        for motor_id in slave_ids:
            m = Talon(motor_id)
            m.follow(self.master_talon)
            m.setInverted(True)
            self.talons.append(m)

        # Then we select the feedback sensor on the master talon.
        # We're using a CTRE Mag encoder plugged into the master talon,
        # so we'll set it to follow a QuadEncoder.
        self.master_talon.configSelectedFeedbackSensor(
            self.master_talon.FeedbackDevice.QuadEncoder, self.PIDIDX, 10
        )
        self.master_talon.setSelectedSensorPosition(0, self.PIDIDX, 10)

        # Next, we make sure the talon and the sensor aren't inverted.
        # This can be configured as needed.
        self.master_talon.setInverted(True)
        self.master_talon.setSensorPhase(False)

        # Then we enable voltage compensation on the master 
        # talon to keep the motor outputs
        # consistent regardless of our battery's charge.
        self.master_talon.enableVoltageCompensation(True)
        self.master_talon.configVoltageCompSaturation(12, 0)
        self.master_talon.configClosedLoopPeakOutput(1, self.PID_SPEED, 0)
        # Set PID for elevator
        self.set_state(True)
        # Lastly, we configure the PID values for going up/down the elevator
        self.set_p_up(0.065)
        self.set_i_up(0)
        self.set_d_up(1.000)
```


Although this can seem like a giant wall of code, it's very easy to sift through. Most of it, you never need to touch again until a major change to the elevator mechanism is made, in which case, it will need to be retuned.
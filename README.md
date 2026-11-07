# MotorTuner

`MotorTuner.java` is a helper file you can copy into an FRC robot project to tune feedback-controller gains and feedforward constants using DogLog tunables.

It supports:

- **CTRE TalonFX** PID gains, with an option to tune feedforward gains
- **REV SparkMax** PID and feedforward gains
- **WPILib controllers** (`PIDController` and `ProfiledPIDController`)
- **WPILib feedforward models** (`SimpleMotorFeedforward`, `ElevatorFeedforward`, and `ArmFeedforward`)

## Getting started

1. Add the required libraries to your robot project: DogLog, WPILib, CTRE Phoenix 6, and REVLib. Use versions compatible with your project.
2. Copy `MotorTuner.java` into your project’s Java source tree. The file uses the `frc.robot` package; adjust its package declaration if your project uses a different package.
3. Call `MotorTuner.requireTestMode()` during robot initialization to make DogLog tunables available only while the robot is in test mode.
4. Register the controllers and feedforwards you want to tune, as shown below.
5. Connect to your robot and use DogLog’s tunable values to adjust the gains.

## Usage

```java
public Robot() {
    MotorTuner.requireTestMode();

    MotorTuner.TalonMotor.tunablePID(
        "Intake",
        intakeMotor,
        new Slot0Configs()
    );

    MotorTuner.TalonMotor.tunablePIDFeedforward(
        "Arm",
        armMotor,
        new Slot0Configs()
    );

    MotorTuner.SparkMaxMotor.tunablePIDF("Spark Max", sparkMaxMotor);

    MotorTuner.NativeController.tunablePID("Position PID", pidController);
    MotorTuner.NativeController.tunablePID("Profiled PID", profiledPIDController);

    MotorTuner.NativeController.tunableMotorFeedforward(
        "Drive Feedforward",
        () -> motorFeedforward,
        updated -> motorFeedforward = updated
    );
}
```

The feedforward helper methods take a `Supplier` and a `Consumer`: the supplier returns the current feedforward object, and the consumer stores the replacement object when a value changes. The variable holding the feedforward must be reassignable—not `final`—while tuning.

## Available helpers

| Helper | Tunables |
| --- | --- |
| `TalonMotor.tunablePID(...)` | TalonFX `kP`, `kI`, `kD` |
| `TalonMotor.tunablePIDFeedforward(...)` | TalonFX PID gains plus `kS`, `kV`, `kA`, `kG` |
| `SparkMaxMotor.tunablePIDF(...)` | SparkMax `kP`, `kI`, `kD`, `kV` |
| `NativeController.tunablePID(...)` with `PIDController` | PID gains and error tolerance |
| `NativeController.tunablePID(...)` with `ProfiledPIDController` | PID gains, position tolerance, maximum velocity, and maximum acceleration |
| `NativeController.tunableMotorFeedforward(...)` | `SimpleMotorFeedforward` `kS`, `kV`, `kA` |
| `NativeController.tunableElevatorFeedforward(...)` | `ElevatorFeedforward` `kS`, `kG`, `kV`, `kA` |
| `NativeController.tunableArmFeedforward(...)` | `ArmFeedforward` `kS`, `kG`, `kV`, `kA` |

For TalonFX, the PID and feedforward values are applied through its configurator. For SparkMax, updates are configured with `PersistMode.kNoPersistParameters`, so changes are not persisted across power cycles.

## Example project

A complete example project is available at [`Examples/2026/MotorTunerExample`](https://github.com/NateCamChiao/MotorTuner/tree/main/Examples/2026/MotorTunerExample).

## Tuning notes

- Tune on a safely supported robot mechanism, with the area clear and an emergency stop available.
- Start with conservative values and change one parameter at a time.
- Use `requireTestMode()` if you want DogLog tunables restricted to test mode.
- WPILib feedforward objects are replaced when their values change. After tuning, consider setting the feedforward variable to `final` and removing or disabling the tuning calls before competition.
- Check your library versions and motor-controller configuration against your robot project before using the example code.

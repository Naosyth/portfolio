---
title: "Flight Assist"
date: 2017-11-29T12:00:00-07:00
draft: false
---

[FlightAssist on Github](https://github.com/Naosyth/FlightAssist)

## Introduction

FlightAssist is a collection of programs I wrote using the in-game programming feature of a game called [Space Engineers](https://store.steampowered.com/app/244850/Space_Engineers/). These in-game programs solve a number of problems I and many other players have faced when designing and operating vehicles. The first and most complex program, later known as the vector module, started just as a fun way to apply linear algebra (my favorite genre of math) in a tangible way.

### Vector Module

This module set out to solve one big problem that bothered me about spaceship design in Space Engineers - pretty much any stable and controllable ship required significant thrust in six directions. This meant that all of your vehicles you built were covered in thrusters, and looked stupid. You could make much cooler looking ships that only had one or two main thrust vectors, but without thrust in all directions, coming to a complete stop in space, where there is no gravity or air resistance to slow you down, was almost entirely impossiblem unless you had computer-like precision. Enter Flight Assist's Vector Module, which allows pilots to put the ship in an auto-pilot sort of mode where the ship automatically aligns itself along its retrograde vector, firing its main thrusters to come to a complete stop, similar to how Space X lands their boosters.

<figure>
  <video controls width="750px">
    <source src="https://i.imgur.com/8aezRTb.mp4" type="video/mp4" />
  </video>
  <figcaption>
    <h4>Vector Module in action</h4>
    <p>The Rocinante from The Expanse performing the famous retro-burn maneuver</p>
  </figcaption>
</figure>

### Hover Module

The hover module's goal was to make flying ships in gravity easier, by automatically aligning ships with the planet's gravity vector, and maneuvering the ship to automatically manage lateral velocity. This is similar to the Vector Module, but it was more complex since maneuvers have to be performed with respect to gravity, to avoid crashing the vehicle straight in to the ground.

<figure>
  <video controls width="750px">
    <source src="https://i.imgur.com/mEKyfAJ.mp4" type="video/mp4" />
  </video>
  <figcaption>
    <h4>Hover Module in action</h4>
    <p>This drone only has a single thrust vector (up). Flying it under normal circumstances would be impossible, because you would never be able to bring it to a complete stop.</p>
  </figcaption>
</figure>

## Technology

FlightAssist was developed using the Space Engineers in-game programming interface, which uses a slightly restricted C# environment. Two major consideration with Space Engineers programs is that you are restricted to a single file, with a length limit of 100,000 characters. To overcome these problems, I initially wrote a small batch script to combine multiple files in to a single string I could copy/paste in game, optionally with obfuscation to reduce file size by renaming variables and classes to smaller names. I later switched to using a Visual Studio plugin called [MDK](https://github.com/malware-dev/MDK-SE), which took care of this for me.

## Math

### Gyroscope Control

In Space Engineers, ships are rotated using gyroscopes. A single ship may have any number of gyroscopes, each of which may be oriented differently with respect to the ship's cockpit forward vector. This means that in order to rotate the ship using multiple gyros, we must do a little extra math per gyroscope to figure out what pitch, roll, and yaw values to set on each gyro individually. FlightAssist has a GyroController in `GyroController.cs` which performs these calculations for us.

We begin by iterating over each gyroscope, and finding that gyro's orientation
```
  Matrix localOrientation;
  g.Orientation.GetMatrix(out localOrientation);
```

Next, we transform our reference and target vectors to local vectors relative to this gyro
```
  var localReference = Vector3D.Transform(reference, MatrixD.Transpose(localOrientation));
  var localTarget = Vector3D.Transform(target, MatrixD.Transpose(g.WorldMatrix.GetOrientation()));
```

Now that we have our local reference and target vectors, we can get our axis of rotation, determine which direction to rotate, and scale the rotation axis based on some in-game constants
```
  var axis = Vector3D.Cross(localReference, localTarget);
  angle = axis.Length();
  angle = Math.Atan2(angle, Math.Sqrt(Math.Max(0.0, 1.0 - angle * angle)));
  if (Vector3D.Dot(localReference, localTarget) < 0)
      angle = Math.PI;
  axis.Normalize();
  axis *= Math.Max(minGyroRpmScale, g.GetMaximum<float>("Roll") * (angle / Math.PI) * gyroVelocityScale);
```

### Hovering with respect to gravity

To hover with our main thrust vector parallel to the gravity vector, we must first find our ship's pitch and roll. To do so, we can can take the arcosine of a dot product with a ship vector and the gravity vector
```
  Vector3D gravity = -Vector3D.Normalize(cockpit.GetNaturalGravity());
  pitch = Helpers.NotNan(Math.Acos(Vector3D.Dot(cockpit.WorldMatrix.Forward, gravity)) * Helpers.radToDeg);
  roll = Helpers.NotNan(Math.Acos(Vector3D.Dot(cockpit.WorldMatrix.Right, gravity)) * Helpers.radToDeg);
```

Now that we have our pitch and roll, we can tell our gyro controller to align our ship with the gravity vector. This can be done by creating pitch and roll quaternions, multiplying them together, transforming them with our down vector (the bottom of our ship will face the gravity source), and using that as the reference in our gyro controller.
```
  cockpit.Orientation.GetMatrix(out cockpitOrientation);
  var quatPitch = Quaternion.CreateFromAxisAngle(cockpitOrientation.Left, (float)(desiredPitch * Helpers.degToRad));
  var quatRoll = Quaternion.CreateFromAxisAngle(cockpitOrientation.Backward, (float)(desiredRoll * Helpers.degToRad));
  var reference = Vector3D.Transform(cockpitOrientation.Down, quatPitch * quatRoll);
  gyroController.SetTargetOrientation(reference, cockpit.GetNaturalGravity());
```

You'll notice that we have `desiredPitch` and `desiredRoll`. By setting desired pitch and roll values, we can adjust our alignment with respect to gravity. If we base these desired values on our velocity in reference to our ship's forward and down vectors, we can effectively control our ship's target velocity. If we want to come to a stop, we angle our main thrust vector to fire against our velocity vector. If we want to gain or maintain speed, we can slightly align our thrust vector with our movement vector.

## Modular Architecture

Space Engineers programs do not have any pre-defined structure, so I had to come up with a program architecture myself that was flexible, would allow for future expansion, and was easy to work with. I decided to structure the whole program as a series of modules. These modules all implemented an abstract base Module class:
```
  public abstract class Module
  {
      protected ModuleAction action;
      protected Dictionary<string, ModuleAction> actions = new Dictionary<string, ModuleAction>();
      private string printBuffer;

      public virtual void Tick() { printBuffer = ""; }

      public virtual void ProcessCommand(string[] args) {
          SetAction(Helpers.GetCommandFromArgs(args), args.Skip(2).ToArray());
      }

      protected void AddAction(string name, Action<string[]> initialize, Action execute)
      {
          actions.Add(name, new ModuleAction(name, initialize, execute));
      }

      protected bool SetAction(string actionName)
      {
          return SetAction(actionName, null);
      }

      protected bool SetAction(string actionName, string[] args)
      {
          if (!actions.Keys.Contains<string>(actionName))
              return false;

          if (actions.TryGetValue(actionName, out action))
          {
              action.initialize?.Invoke(args);
              OnSetAction();
              return true;
          }
          return false;
      }

      protected virtual void OnSetAction() { }

      public string GetPrintString() { return printBuffer; }
      protected void PrintLine(string line) { printBuffer += line + "\n"; }
  }
```

Modules operate on a `tick` system, where each time the programmable block was run, all modules would tick and perform some calculation. Modules can define actions, which are use-controlled events that perform certain tasks. They may also print information to an on-vehicle screen, for informational or debugging purposes.

As an example of a module's actions, we can examine the Vector Module's constructor, which registers four different actions:
```
  AddAction("disabled", (args) => { gyroController.SetGyroOverride(false); }, null);
  AddAction("brake", (args) => {
      startSpeed = cockpit.GetShipSpeed();
      cockpit.DampenersOverride = false;
  }, SpaceBrake);
  AddAction("prograde", null, () => { TargetOrientation(-Vector3D.Normalize(cockpit.GetShipVelocities().LinearVelocity)); });
  AddAction("retrograde", null, () => { TargetOrientation(Vector3D.Normalize(cockpit.GetShipVelocities().LinearVelocity)); });
```

This code is fairly self-explanatory. The disabled action turns this module off. The brake module runs an initial block of code when the action gets triggered, which notes the starting speed and disabled intertial dampeners (we wane them off until we're oriented correctly to come to a stop). Then, it states that a function called `SpaceBrake` should be run every module tick. The prograde and retrograde actions are both similar to each other - they either align the ship's forward vector with the ship's velocity vector or negative velocity vector.

This architecture allows me to compartmentalize FlightAssist's features, but does not restrict what I am able to do.

## Additional Media

<figure>
  <video controls width="750px">
    <source src="https://i.imgur.com/sPIw6K8.mp4" type="video/mp4" />
  </video>
  <figcaption>
    <h4>Artificial Horizon</h4>
    <p>As mentioned earlier, modules can optionally output to a screen. Here, we see the hover module's artificial horizon.</p>
  </figcaption>
</figure>

{{< figure src="https://i.imgur.com/lOMT7YW.png" title="FlightAssist on the Steam Workshop" caption="FlightAssist is currently used by about 6600 people">}}

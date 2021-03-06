# MYO Overview

Spatial data informs the application about the orientation and movement of the user's arm. The Myo SDK provides two kinds of spatial data:

- An orientation represents which way the Myo armband is pointed. In the SDK this orientation is provided as a quaternion that can be converted to other representations, like a rotation matrix or Euler angles.

- An acceleration vector represents the acceleration the Myo armband is undergoing at any given time. The SDK provides this as a three-dimensional vector.

Gestural data tells the application what the user is doing with their hands. The Myo SDK provides gestural data in the form of one of several preset poses, which represent a particular configuration of the user's hand. For example, one pose represents the hand making a fist, while another represents the hand being at rest with an open palm.

The Myo armband provides information about which arm it is being worn on and which way it is oriented - with the positive x axis facing either the wrist or elbow. This is determined by a Sync Gesture performed upon putting it on. The Myo armband similarly detects when it has been removed from the arm.

# MYO Project

- bin/ contains binary executables included in the SDK
- docs/ contains this documentation
- myo.framework includes the headers and libraries needed to use the SDK
- projects/ contains programs using the SDK


# MYO SDK

At the core of the Myo SDK is a library, libmyo. This library allows applications to interact with the Myo armband. All functionality in libmyo is exposed through a plain C API.

Typically, applications do not interact with the libmyo C API directly. Instead, they use a language binding corresponding to the programming language used by the application. For example, the C++ bindings in Myo SDK provide convenient access to libmyo through a familiar C++ interface.

- The entry point into the SDK is the hub. A hub simply provides a way to connect to one or more Myo armbands.

- The SDK provides data to the application in the form of events. There are three categories of events: spatial events (corresponding to spatial data from a Myo armband), gestural events (corresponding to gestural data a Myo armband) and auxiliary events.

- The C++ bindings are provided as a set of header files in the include/ directory of the SDK. The only header you need to include to use the bindings is <myo/myo.hpp>.

# Start a new XCode myo cocoa application

- Drag myo.framework into the Frameworks group in your project.
- In Build Settings, add @loader_path/../Frameworks to "Runpath Search Paths".
- Add a "Copy Files" Build Phase. Select "Frameworks" as the destination, and add myo.framework to the list.

# MYO Scripting in LUA

While the Myo SDK provides powerful and complete facilities for writing applications that make use of the Myo armband's capabilities, some tasks can be more easily accomplished through scripting.

Myo Connect meets this need by running connectors that can handle Myo events and use them to control applications. Myo Scripts are connectors written in Lua, which is a simple scripting language that is often used for application scripting.

Myo Scripts are managed via the Application Manager, accessible by clicking the Application Manager entry in the Myo Connect menu. This window allows the user to add, reload and remove scripts, as well as adjust the order in which they are given access to applications.

Myo Connect interacts with scripts through two primary mechanisms: Callback functions and API functions:

- Callback functions are implemented by scripts to handle specific events or request information. When a designated event occurs, Myo Connect calls into the script, in some cases providing additional information. This includes changes in the active application, changes in the Myo armband's active pose, and so on.

- API functions provide scripts with additional functionality to access the armband and system information. This includes angular values for the armband's orientation, vibrating the armband, the current time, and outputting to the debug window. It also includes commands for manipulating the system by emitting keyboard events, which can be used to control applications.

There are a few global variables that are used to communicate between scripts and Myo Connect. Myo Connect provides the platform variable to indicate which platform it is running on.

# MYO script tutorial

- scriptId = 'com.thalmic.examples.myfirstscript': Reverse domain name, needs to be unique for a script to be released
- scriptTitle = 'Name': This actually shows up in the application manager
- scriptDetailsUrl = '': url for details of the script

Now, there are a few different predefined “callbacks” you can implement in Myo Script. These are special functions that will get called for you in response to certain events. In Myo scripts there is no “main” method or anything like that. Technically any code written outside of a function will be executed when the script is first loaded by the Myo Script Manager (that’s how scriptId is getting set), but all of our actual work will be done from these callbacks.

- To define a function in Lua, you just write function <function name>(<argument 1>, <argument 2>, <… etc>) on a new line and end the function with end on it’s own line

- onForegroundWindowChange has two arguments, app and title. Since this is a callback defined by Myo Script, we can’t change this at all.
    - function onForegroundWindowChange(app, title)

      end
    - The job of onForegroundWindow is to let you determine if your script should be active or not. The idea is that each script is targeted at a certain app (or set of apps), so onForegroundWindow fires every time a new app is in the foreground. app is the bundle identifier on Mac OS X or the name of the .exe file on Windows and title is the actual title of the foreground window.
    - If you detect an app your script supports, return true.
    - Following script will just print the app's name and title.
    - ```
      scriptId = 'com.thalmic.examples.myfirstscript'
      scriptTitle = "My First Script"
      scriptDetailsUrl = ""
      function onForegroundWindowChange(app, title)
          myo.debug("onForegroundWindowChange: " .. app .. ", " .. title)
          return true
      end
      ```


Myo scripts have a global myo object defined. This is where all the myo-specific functions live. One of these is debug, which accepts a string to output to a special console window that will pop up as soon as you start writing to it, but ONLY if you have “Developer Mode” enabled in Myo Connect’s Preferences.

Make sure you re-add it every time you change (and save) something in your script

With Myo armband you can currently do five hand poses: Wave out, wave in, fist, fingers spread and double tap.

- To actually detect a pose, we’ll use the second of our predefined callbacks: onPoseEdge.
  - function onPoseEdge(pose, edge)

    end
  - edge will be set to on when a pose starts, and off when the user releases it. This will let you do things like start listening for changes in yaw to do something when the user makes a fist and stopping once they let go.
  - Currently, pose can have one of the following values:
    waveIn, waveOut, fist, doubleTap, fingersSpread, rest and unknown
  - Following script prints pose and edge.
  - ```
    function onPoseEdge(pose, edge)
      myo.debug("onPoseEdge: " .. pose .. ": " .. edge)
    end
    ```
  - If statements in LUA:
  - ```
    if  then
        -- do something
    elseif  then
        -- do something if something else is true
    else
        -- do something else
    end
    ```

- Example of onPoseEdge():
  - ```
    function onPoseEdge(pose, edge)
      myo.debug("onPoseEdge: " .. pose .. ": " .. edge)
      if (edge == "on") then
        if (pose == "waveOut") then
          onWaveOut()
        elseif (pose == "waveIn") then
          onWaveIn()
        elseif (pose == "fist") then
          onFist()
        elseif (pose == "fingersSpread") then
          onFingersSpread()
        end
      end
    end
    ```
  - ```
    function onWaveOut()
      myo.debug("Next")
      myo.keyboard("tab", "press")
    end
    function onWaveIn()
      myo.debug("Previous")
      myo.keyboard("tab","press","shift")
    end
    function onFist()
      myo.debug("Enter")
      myo.keyboard("return","press")
    end
    function onFingersSpread()
      myo.debug("Escape")
      myo.keyboard("escape", "press")
    end
    ```

- myo.getArm() returns left, right or unknown
- Write a helper function that switches waveOut for waveIn and waveIn for waveOut if the user is wearing their Myo on their left arm, and call it before your logic handling which pose the user just did
- helper function:
- ```
  function conditionallySwapWave(pose)
    if myo.getArm() == "left" then
      if pose == "waveIn" then
          pose = "waveOut"
      elseif pose == "waveOut" then
          pose = "waveIn"
      end
    end
    return pose
  end
  ```
- New onPoseEdge():
- ```
  function onPoseEdge(pose, edge)
    myo.debug("onPoseEdge: " .. pose .. ": " .. edge)
    pose = conditionallySwapWave(pose)
    if (edge == "on") then
      if (pose == "waveOut") then
        onWaveOut()
      elseif (pose == "waveIn") then
        onWaveIn()
      elseif (pose == "fist") then
        onFist()
      elseif (pose == "fingersSpread") then
        onFingersSpread()
      end
    end
  end
  ```
-  A useful technique is to give yourself a little vibration feedback whenever a gesture is detected. The best way to do that is with myo.notifyUserAction().
- You can make the Myo armband vibrate with the myo.vibrate(vibrationType) function, where vibrationType can be one of short, medium or long.

- By default, until a user does the unlock gesture with an application running that can accept Myo input, none of the other poses are detected or let through. This is the standard unlock policy.  If that flow doesn’t work for your script (for example, if you are controlling a game where the Myo should be active at all times), you can disable this behaviour by setting the lock policy:
    - myo.setLockingPolicy("none")
    - You can turn it back on with: myo.setLockingPolicy("standard")

- You may want to let user hold a single gesture (like to adjust the volume) or enter a series of commands. This is pretty straightforward to accomplish with the myo.unlock(unlockType) function. This will let you programatically unlock (or keep unlocked) a Myo armband. You can send either timed or hold. timed will reset the clock and keep the armband unlocked for a few more seconds, and hold will keep it unlocked until you manually lock it (or set it back to timed and wait).

- onPoseEdge gives you either on or off, while myo.keyboard() needs either down or up.

# Final script:

```lua
function onPoseEdge(pose, edge)
  myo.debug("onPoseEdge: " .. pose .. ": " .. edge)

  pose = conditionallySwapWave(pose)

  local keyEdge = edge == "off" and "up" or "down"

  if (pose == "waveOut") then
      onWaveOut(keyEdge)
  elseif (pose == "waveIn") then
      onWaveIn(keyEdge)
  elseif (pose == "fist") then
      onFist(keyEdge)
  elseif (pose == "fingersSpread") then
      onFingersSpread(keyEdge)
  end
end

function onWaveOut(keyEdge)
  myo.debug("Next")
  --myo.vibrate("short")
  myo.keyboard("tab", keyEdge)
end

function onWaveIn(keyEdge)
  myo.debug("Previous")
  --myo.vibrate("short")
  --myo.vibrate("short")
  myo.keyboard("tab",keyEdge,"shift")
end

function onFist(keyEdge)
  myo.debug("Enter")
  --myo.vibrate("medium")
  myo.keyboard("return",keyEdge)
end

function onFingersSpread(keyEdge)
  myo.debug("Escape")
  --myo.vibrate("long")
  myo.keyboard("escape", keyEdge)
end
```






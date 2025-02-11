## Event Types
* Heck
  - [`AnimateTrack`](#animatetrack)
  - [`AssignPathAnimation`](#assignpathanimation)
* Noodle Extensions
  - [`AssignTrackParent`](#assigntrackparent)
  - [`AssignPlayerToTrack`](#assignplayertotrack)
* Chroma
  - [`AnimateComponent`](#animatecomponent)

# Tracks
Tracks are a powerful tool integrated by Heck that allow you to group objects together and control them.

`track` is a string property in the `customData` of the object you want to give said track to. It can be placed on any object in the `obstacles`, `colorNotes`, `bombNotes`, `sliders`, or `burstSliders` arrays.

This example will add the note to the `ExampleTrack` track. These tracks are a way of identifying which objects that the custom events should affect.
```js
"colorNotes": [
  {
    "t": 8.0,
    "x": 2,
    "y": 0,
    "c": 1,
    "d": 1,
    "customData": {
      "track": "ExampleTrack"
    }
  }
]
```

# Point Definitions
Point definitions are used to describe what happens over the course of an animation, they are used **slightly differently for different properties.** They consist of a collection of points over time.

Here is an example of one being defined to animate [`offsetPosition`](AnimationProperties/#offsetPosition): (See: [AnimateTrack](#AnimateTrack))
```js
{
  "b": 3.0,
  "t": "AnimateTrack",
  "d": {
    "track": "ZigZagTrack",
    "duration": 1,
    "offsetPosition": [
      [0, 0, 0, 0],
      [1, 0, 0, 0.25],
      [-1, 0, 0, 0.75],
      [0, 0, 0, 1]
    ]
  }
}
```
A point definition usually follows the pattern of `[data, time, "optional easing", "optional spline"]`, 
- Data can be multiple points of data, this part of a point varies per property,
- Time is a float from 0 - 1, points must be ordered by their time values
- "optional easing" is an optional field, with any easing from easings.net (with the addition of `easeLinear` and `easeStep`). This is the easing that will be used in the interpolation from the last point to the one with the easing defined on it.
- "optional spline" is an optional field, with any spline implemented, currently only `"splineCatmullRom"`. It acts like easings, affecting the movement from the last point to the one with the spline on it. Currently only positions and rotations support splines.
```js
// Example of splines and easings being used
"offsetPosition": [
  [0, 0, 0, 0],
  [1, 5, 0, 0.5, "easeInOutSine"],
  [6, 0, 0, 0.75, "splineCatmullRom"],
  [5, -2, -1, 1, "easeOutCubic", "splineCatmullRom"]
]
```
Point definitions can also be defined inside the `pointDefinitions` field of your `customData`, any point definition defined here can be called via their property name when one would fit.
```js
{
  "version": "3.0.0",
  "customData": {
    "pointDefinitions": {
      "ZigZagPosition": [
        [0, 0, 0, 0],
        [1, 0, 0, 0.25],
        [-1, 0, 0, 0.75],
        [0, 0, 0, 1]
      ]
    },
    "customEvents": [
      {
        "b": 3.0,
        "t": "AnimateTrack",
        "d": {
          "track": "ZigZagTrack",
          "duration": 1,
          "offsetPosition": "ZigZagPosition"
        }
      }
    ]
  },
  "events": [],
  "notes": [],
  "obstacles": []
}
```
When a point definition is used, input time values outside of the defined points' times will automatically clamp to the first and last values respectively.
```js
// The scale of this note will be 2x up until 0.5, after which it will begin scaling down to 1x until 0.8, after which the note will remain at 1x
{
  "t": 2,
  "x": 1,
  "y": 0,
  "c": 1,
  "d": 0,
  "customData": {
    "animation": {
      "scale": [
        [2, 2, 2, 0.5],
        [1, 1, 1, 0.8]
      ]
    }
  }
}
```
If you only require one element in your point definition, instead of a list of points, your point definition can just be your point.

```js
// These are equivalent
"offsetPosition": [[245, 23, 54, 0]]
"offsetPosition": [245, 23, 54]
```

# Individual Path Animation
`animation` is an object that can be put in the `customData` of any object.

This will instantly apply a path animation to the object. See [`AssignPathAnimation`](#AssignPathAnimation). Will overwrite any path animation assigned through [`AssignPathAnimation`](#AssignPathAnimation)
```js
// Example
{
  "b": 90,
  "x": 1,
  "y": 0,
  "c": 0,
  "d": 1,
  "customData": {
    "animation": {
      "offsetPosition": [
        [0, 40, 0, 0],
        [0, 0, 0, 0.2]
      ]
    }
  }
}
```

# Events

## AnimateTrack
```js
{
  "b": float, // Time in beats.
  "t": "AnimateTrack",
  "d": {
    "track": string, // The track you want to animate.
    "duration": float, // The length of the event in beats (defaults to 0).
    "easing": string, // An easing for the animation to follow (defaults to easeLinear).
    "property": point definition, // The property you want to animate.
    "repeat": int // How many times to repeat this event (defaults to 0)
  }
}
```
Animate track will animate the properties of everything on the track individually at the same time. The animation will go over the point definition over the course of `duration`. 

Attempting to animate a property which is already being animated will stop the overwritten `AnimateTrack`. 

However, multiple `AnimateTrack` events may animate separate properties at the same time, i.e. one `AnimateTrack` could animate position over 10 beats while another `AnimateTrack` animates rotation over 5 beats.

Although not recommended, properties can be set to `null` to "erase" a track's property (This obviously cannot have a duration). This will return that property to as if it was never set at all. This is highly not recommended because it cannot update active objects and can instead be done by setting the property to a default point definition. i.e. `[[0,0,0,0]]`

### Track Properties
- [`offsetPosition`](AnimationProperties/#offsetPosition)
- [`offsetWorldRotation`](AnimationProperties/#offsetWorldRotation)
- [`localRotation`](AnimationProperties/#localRotation)
- [`scale`](AnimationProperties/#scale)
- [`dissolve`](AnimationProperties/#dissolve)
- [`dissolveArrow`](AnimationProperties/#dissolveArrow)
- [`color`](AnimationProperties/#color) (Chroma)
- [`interactable`](AnimationProperties/#interactable)
- [`time`](AnimationProperties/#time)

```js
// Example
// All the objects on ZigZagTrack will be offset 1 unit to the right, then offset 1 units to the left, and then finally centered.
{
  "b": 3.0,
  "t": "AnimateTrack",
  "d": {
    "track": "ZigZagTrack",
    "duration": 10,
    "offsetPosition": [
      [0, 0, 0, 0],
      [1, 0, 0, 0.25],
      [-1, 0, 0, 0.75],
      [0, 0, 0, 1]
    ]
  }
}
```

## AssignPathAnimation
```js
{
  "b": float, // Time in beats.
  "t": "AssignPathAnimation",
  "d": {
    "track": string, // The track you want to animate.
    "duration": float, // How long it takes to assign this path animation (defaults to 0).
    "easing": string, // An easing for moving to the path animation (defaults to easeLinear).
    "property": point definition // The property you want to assign the path to.
  }
}
```
`AssignPathAnimation` will assign a "path animation" to the notes. 

In this case, the time value of the point definition is the point each object on the track is at in its individual life span. 

Meaning a point with time `0` would be right when the object finishes jumping in, a point with time `0.5` would be when the object reaches the player, at `0.75`, walls and notes will begin their despawn animation and start flying away very quickly, and `1` being right when the object despawns.

![thanksbullet](media/pathanimationpoints.png)

**Note:** Objects CANNOT be animated while they are in their jumping animation. During that time, they will instead strictly use the first point in the point definition.

Although not recommended, path properties can be set to `null` to "erase" a track's path property. This will return that path property to as if it was never set at all. It is highly not recommended because although usually you can interpolate from one path animation to another using `duration`, you cannot interpolate from `null`.

### Path Properties

- [`offsetPosition`](AnimationProperties/#offsetPosition)
- [`offsetWorldRotation`](AnimationProperties/#offsetWorldRotation)
- [`localRotation`](AnimationProperties/#localRotation)
- [`scale`](AnimationProperties/#scale)
- [`dissolve`](AnimationProperties/#dissolve)
- [`dissolveArrow`](AnimationProperties/#dissolveArrow)
- [`color`](AnimationProperties/#color) (Chroma)
- [`interactable`](AnimationProperties/#interactable)
- [`definitePosition`](AnimationProperties/#definitePosition)

```js
// Example
// During their jump animation, the objects will be 40 units high. Once their jump animation is complete, the object will then start descending.
{
  "b": 3.0,
  "t": "AssignPathAnimation",
  "d": {
    "track": "DropNotes",
    "offsetPosition": [
      [0, 40, 0, 0],
      [0, 0, 0, 0.2],
    ]
  }
}
```

## AssignTrackParent
```js
{
  "b": float, // Time in beats.
  "t": "AssignTrackParent",
  "d": {
    "childrenTracks": [string], // Array of tracks to parent to _parentTrack.
    "parentTrack": string // The track you want to animate.
    "worldPositionStays": bool // Defaults to false if not set. See https://docs.unity3d.com/ScriptReference/Transform.SetParent.html
  }
}
```
`AssignTrackParent` will create an new GameObject with a [TransformController](Environment/#transformcontroller) and parent any number of children tracks to it.

## AssignPlayerToTrack
```js
{
  "b": float, // Time in beats.
  "t": "AssignPlayerToTrack",
  "d": {
    "track": string // The track you wish to assign the player to.
    "target": string // (optional) The specific player object you wish to target.
  }
}
```
`AssignPlayerToTrack` will assign the player a [TransformController](Environment/#transformcontroller).
Available targets are `Root`, `Head`, `LeftHand`, and `RightHand`.

### IT IS HIGHLY RECOMMENDED TO HAVE A TRACK DEDICATED TO THE PLAYER, AND NOT USE EASINGS IN MOVEMENT.
This is VR, non-linear movement or any form of rotation can easily cause severe motion sickness.
To clarify, it is very easy to make people motion sick with player tracks, please use them carefully and sparingly.

## AnimateComponent
```js
{
  "b": float, // Time in beats.
  "t": "AnimateComponent",
  "d": {
    "track": string // The track you want to animate.
    "duration": float, // The length of the event in beats (defaults to 0).
    "easing": string, // An easing for the animation to follow (defaults to easeLinear).
    "component name": { // name of component
      "field name": point definition // name of field on component
    }
  }
}
```
`AnimateComponent` allows for animating [components](Environment/#Components). Animating fog [demo](https://streamable.com/d1ztwq).

```js
// Example
// Lights start off extremely bright and then quickly dim.
{
  "b": 15,
  "t": "AnimateComponent",
  "d": {
    "track": "lights",
    "duration": 5,
    "TubeBloomPrePassLight":
      {
          "colorAlphaMultiplier": [[10, 0], [0, 1, "easeInExpo"]],
          "bloomFogIntensityMultiplier": [[10, 0], [0, 1, "easeInExpo"]]
      }
  }
},
```
## Introduction

The core source codes of AB3DMT are below. The entire AB3DMT will be released in the future. 

NetLogo library sources include: 
- [3D model scanning](ab3dmt-lib-cube.nls)
- [3D model loading](ab3dmt-lib-file.nls)

Sample 3D point cloud file can be found: [Arm 3D model](processed_models/arm.csv). 

## Initialization

```
extensions[
  csv
]

turtles-own[object-type status]

patches-own []

globals[
  model-file
  current
  min_x max_x
  min_y max_y
  min_z max_z
  cur_y
  delta_color
  total-points
  radius
]

```

## 3D model loading

```
to load-model-human
  set scale 10
   load-model-auto "processed_models/Male.csv" 30 -22 22 -66 17 -5 9

end


to load-model-head
  set scale 10
  set size-scale 1.5
  load-model-auto "processed_models/head.csv" 10 -4 4 0 19 -3 8

end

to load-model-hand
  set scale 0.3
  set size-scale 1.5
  load-model-auto "processed_models/hand.csv" 10 -43 27 -34 47 -1 20

end

to load-model-arm
  set scale 0.3
  set size-scale 1.5
  load-model-auto "processed_models/arm2.csv" 2 -18 55 -24 24 -53 109

end

to load-model-skeleton
  set scale 0.05
  set size-scale 1.5
  load-model-auto "processed_models/human_skeleton.csv" 50 -11 15 -22 22 -60 64

end

to load-model-leg
  set scale 0.8
  set size-scale 1.5
  load-model-auto "processed_models/leg.csv" 1 -9 12 0 118 -17 18

end

```

## Agent visiting algorithms

Four algorithms types: 

- Random neighbor agent expanding
- Random agent visting
- Random neighbor agent visiting
- Go From Bottom

```

to init-go
  reset-ticks
  clear-all-plots

  ; get a random turtle
  set current one-of turtles  with [object-type = "human"]
  ; color trasition
  set cur_y -60
  set delta_color 0
  ; color settings
  ask turtles with [object-type = "human"] [
    set color blue
  ]
  ; DLA
  set radius 0
  ask turtles with [object-type = "particle"] [die]

  ask turtles with [object-type = "box"][die]

  clear-links

  clear-patches

end

to go

  ;;ask n-of 100 turtles [
   ;; set color red
  ;; ]
  ask current[
    let all-near-turtles []
    ask neighbors6 [
      ask turtles-here with [object-type = "human"][
      set all-near-turtles lput self all-near-turtles
      set color pink
        if form-edges? [
        form-random-edge turtles-here
        ]
    ]
    ]

    ;ask patch-at dx dy dz[
    ;  ask turtles-here[
     ;   set color yellow
    ;  ]
    ;]

    ask turtles at-points [[0 0 0]][
     set color red - 1
    ]

    set current one-of all-near-turtles
    set color red
  ]


  tick
end

to go-by-random
;  ask turtles with [object-type = "human" and color = pink]
 ; [
;    set color blue
 ; ]

  ask one-of turtles with [ object-type = "human"][

    ask neighbors [
      ask turtles-here with [object-type = "human"][

        set color pink
      ]
    ]

  ]

  tick
end

to go-by-neighbor
;  ask turtles with [object-type = "human" and color = pink]
 ; [
 ;   set color blue

 ; ]

 ; clear-links

  ask current[

    ask neighbors [
      ask turtles-here with [object-type = "human"][
        if form-edges? [
        form-random-edge turtles-here
        ]
        set color pink
      ]
    ]

      set current one-of neighbors

  ]

  tick
end

to go-from-bottom
  ; color transition from bottom to top in gradient change
  ask turtles  with [object-type = "human"] [
    if ycor >= cur_y and ycor < cur_y + 1 [
     set color scale-color red cur_y world-min-y world-max-y
    ]
  ]

  set cur_y cur_y + 1
  set delta_color delta_color + 0.2

  if cur_y > int ( world-max-y) [
   stop
  ]

  tick

end

```

## Improved algorithms of infrared image rendering

```

to load-infrared-common [path]
  let file path
  file-open file
  while [ not file-at-end? ] [
    let row csv:from-row file-read-line

    let x_min item 0 row
    if not (x_min = "x_min") [
      let x_max item 1 row
       let y_min item 2 row
       let y_max item 3 row
       let r item 4 row
       let g item 5 row
       let b item 6 row

      ask turtles with [object-type = "human"][
         if xcor >= x_min and xcor < x_max [
            if ycor >= y_min and ycor < y_max [
            if not (r < 50 and g < 50 and b < 50) [
              set color (list r g b)
            ]
          ]

        ]

      ]

    ]

  ]
   file-close

end

to improve-infrared-model
   if (count turtles with [object-type = "human" and color = blue]) = 0[
     stop
    ]

  ask turtles with [object-type = "human"]
  [

    if color = blue [
      let neighbor one-of neighbors
      let turtles1 [turtles-here] of neighbor
      let turtle1 one-of turtles1
      if turtle1 != nobody [
        let mycolor [color] of turtle1
        set color mycolor
      ]
    ]

  ]
end

to improve-infrared-model2
   ask turtles with [object-type = "human"]
  [


      let neighbor one-of neighbors
      let turtles1 [turtles-here] of neighbor
      let turtle1 one-of turtles1
      if turtle1 != nobody [
        let mycolor [color] of turtle1
        set color mycolor
      ]

  ]

end
```
# Skew load factor 
hier komt de skew load factor


## Goal
According to the DNV standard, rigging equipment has a tolerance of 0.25%, what this means is that the sling/grommet may be 0.25% shorter or 0.25% longer than the original length. This affects all types of lifts, as the load can end up hanging askew. But on a four-point lift, this can affect the force distribution of the slings. According to the DNV standard, the factor for each sling is equal to 1.25, in reality this factor depends on several values. Such as the in-service length of the slings, the elasticity modulus, the diameter and the coordinates of both the lift points and the hook. Therefore, it is necessary to see how different lengths affect the forces in the slings, with an iterative process the skew load factor can be determined in more detail.


## Flowchart
![This is a image](/Images/Flowchart_skl.jpg)

With the help of the method make_skl_data the data is stored in the following structure, see code snippet below: 

```
Data_skl={"Lift_points": [nx[float(point_x),float(point_y),float(point_z)]],
              "ESlings": [float(Esling1),float(Esling2),float(Esling3],float(Esling4)],
              "Hook_point": [float(Hook_x),float(Hook_y),float(Hook_z],
              "DSlings": [float(Dsling1),float(Dsling2),float(Dsling3],float(Dsling4)],
              "LSlings": [float(Lsling1),float(Lsling2),float(Lsling3],float(Lsling4)]}
```
1. Determine the stifness constant of the slings 
2. Determine the length of the slings: 
    The length of slings depends on the in use length of the slings, and if the slings are longer than measure lenght because of the tolerance or are shorter than the measured length. In total there are 16 differen combinations possible for the slings lenghts, the code loops through all of them.
3. Determine start hook point, because of the different length of the slings, the start of the hook point is different from the original. See next chapter for a detailed explanation of the start of the hook point.
4. Determine number of slack slings, this is based on distance between the lift points and the the length of the slings with tolerance. 
5. Determine Fx, Fy and Fz forces in hook, Fx and Fy are equal to zero. The Fz force depens on which iteration the "Loop skew load is"
6. When there are 2 slings tight, this means that the hook is on the diagonal line of those two points. So the forces in the slings/ displacement of the hook can be determined using the 2d stifness method. When there are 3/4 slings tight, the force in the slings/ displacement of the hook can be solved using the 3d stifness method.
7. Determine the new position, based on the displacment of the hook that was calculated in the last step
8. Determine the total force in slings/ hook, based on the force in slings that was calculated in the last step
9. Checking if "Loop skew" has done all its steps, if this is the case the total force in z-direction is equal to the total force of the lifting object. If it is not the case, steps 3,4,5,6,7 and 8 are done again, unit all steps have been completed.
10. Store all the results in a diactionary
11. Checking if "Loop options short/long slings" had done all its 16 different combinations, if this is not the case the steps 3 t/m 10 will be done again.
12. Determine the max skew load factor of each sling. Also determine what the displacement of the hook/force in slings was with the max skew load factor and store these in a CSV file

## Determine start hook point 
Because of the different lenghts, the start of the hook point is different than the orignal hook point.  See below the flowchart, there has beem assumption made that the slings can not have any strain( no forces), when the forces in the hook is not equal to zero. This is the case when there is one rope tight, and the rope is not direct under the hook. The second case is when there are two tight and the hook lays not on the diagonal of the two liftings points.
![This is a image](/Images/Flowchart_start_hook_point.jpg)

1. Determine the z-positon of the hook, this point is the first point when a cable is tight, it is determined with the following code, see code snippet below: 
```
    z_hook = []
    for i in range(4):
        # Determine distance between x and y points
        x_dist = Lifting_points[i][0]-Hook_point[0]
        y_dist = Lifting_points[i][1]-Hook_point[1]
        # Z distance between lift point and hook point:
        z_dist = side_3d_line(Lslings[i], x_dist, y_dist)
        # Z_hook height:
        Z_hook_height = z_dist+Lifting_points[i][2]

        z_hook.append(Z_hook_height)
    Hook_point[2] = min(z_hook)+0.000001

```
2. Determine the number of slack slings
3. If there are 3 slack slings, the hook will move to the the lifting point of the tight sling, until the hook is right above the lifting point. 
    - First determine the x an y differnce between hook point and hook point(when hook is above lift point)
    - Move the hook in very small steps to the end hook point
    - Determine z-position hook( so that the length of the slings is constant)
    - Determine number of slack slings
    - when ther is no change in the number of slack slings, move do the loop again, otherwise go futher in the flowchart.
4. When there are two slings tight, the hook will move to the diagonal line of those two lifting points:
    - First determine the length of the radius line, which the hook will follow, so that both the slings will have a constant length
    - Determine the x,y,z coardinates of the intersection point with the radius line and the diagonal line 
    - Determine the hook with the vertical and the radius line. 
    - Change the hook of the radius line with the vertical in small steps until the radius line is perpendicular to the diagonal line.
    - Check the number of slack slings
    - If there are number of slack slings is not changed and the radius line is perpendicular to the diagonal line, the hook lays on the diagonal line and it is two point lift
    - If there are one or two slack pulled tight it is a 3/ 4 points lift

    -


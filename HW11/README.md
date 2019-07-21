## Homework 11 - Using GAZEBO to generate a sig legged robot with a claw

### Troubleshooting
The model was created on a Linux virtual machine inside my local computer using VirtualBox. This way I avoided using credits on the IBM Cloud, which I'm saving for the final project. I also installed *ROS* for later playthrough of the mechanics of the simulation of the robot, as most of the documentation out there is based more on *ROS* than *Gazebo*.

### Contents
* **Folder HW11**: Final robot - Combination of the hexapod base with a modified grippler. 
  - model.config has the configuration of the robot
  - model.sdf has the distribution of the robot: the hexapod base, the modified grippler and the joit between them. 
  
* **hexapod_base**: Based on the code on [gazebosim.org](http://answers.gazebosim.org/question/12044/hexapod-simulation-not-working/)
  - It has 6 legs attached to a rectangular box. Each leg has three joints
  
* **simple_gripper**: Based on the simple gripper use in the [tutorials](http://gazebosim.org/tutorials/?tut=simple_gripper)
  - It was modified to be a claw, based on an antish design. The main changes were in the scale of the z-dimension of each box, and the riser was modified to just a joint between the base and the claw.
  
### Final Design

![hexapod_robor](https://github.com/penpen86/W251/blob/master/HW11/Final%20Robot.JPG)


---
layout: project
title: Twig Walking Animation
repository: https://github.com/guiklink/Unity3d_Twig
date: August 31, 2015
image: https://github.com/guiklink/portfolio/blob/gh-pages/public/images/twig/Twig_logo.png?raw=true
---

<article></article><br/>

<iframe src="https://player.vimeo.com/video/137813432" width="758" height="426" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/137813432">Simple walking GAIT</a> from <a href="https://vimeo.com/user43396191">Guilherme Klink</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

# Introduction
Aiming to top their sales and reach more players, video game producers found out that the secret to make a best selling game is it to make it realistic.  

However, what is realism in something fictional such a game?    

The answer is simple, in games things need to behave, look and move accordingly to what would be expected in the "world" where the game takes place. In order to understand the purpose of Twig, let's talk about how things move accordingly. Animators design various sets of animations for every specific movement they expect the character to perform from running, jumping and fighting to even plucking a flower from the ground. A character will have different animations to sit in different chairs, just because the chair's size changes. At this point you probably already guessed, generating this huge animation library is very expensive and time taking, not to mention prohibitive for universities and independent developers.  

The idea behind TWIG is to create a simplified movement [gait](https://en.wikipedia.org/wiki/Gait), simple enough that it would not be too hard to implement nor require heavy processing power, therefore not compromising other aspects of the game (e.g. AI and rendering), but still able to adapt to small peculiarities and with a lower cost to produce. Twig was developed by Professor Ian Douglas Horswill from Northwestern University.  

My project consisted of creating a Twig walking GAIT for a biped humanoid. As shown in the video, the Twig GAIT implemented, although simple, would do a very good job as the GAIT of robot (I couldn't stop thinking about C3PO), a cartoon character or maybe a kid.

# Unity3D
My implementation of Twig used Unity3D. [Unity3D](http://unity3d.com/5?gclid=CNngyI300scCFYI7gQodzBYMlQ) is a common simulation and graphics rendering engine used in generating virtual environments. Some benefits of Unity3D are as follows:

1. It can be used in both Windows and MAC
2. Produce games for all the most popular platforms and cellphones
3. It is available for free personal use
4. It supports C#, making most of developer tools from Microsoft available
5. It has a big and active community

## Getting a Ragdoll
A Twig kinematic character is composed by rigid rods (black lines) connected to each other by nodes (red dots) as shown in the figure bellow.

![Twig_Character](https://github.com/guiklink/portfolio/blob/gh-pages/public/images/twig/Twig%20doll.png?raw=true)

The Unity3D analogy for that would be the 3D ragdoll, a  character represented by rigid bodies fully affected by physics and connected by configurable joints. In principle, any [3D humanoid rigged character](http://docs.unity3d.com/Manual/Preparingacharacterfromscratch.html) can be transformed in a ragdoll through the [Ragdoll Wizard](http://docs.unity3d.com/Manual/wizard-RagdollWizard.html). 
**PS:** It is recommended that the rigged body is saved in a "T" like position. 

## Step 2: Configuring Colliders and Ragdoll Joints
The Ragdoll Wizard does a very good job for you in setting the raw ragdoll, however from my experience, if you want your character to be working properly and avoid having to redo work I strongly encourage you to spend some time doing a more detailed tuning on your character. 

### 2.1: Colliders
The Unity physics engine uses the [colliders boxes](http://docs.unity3d.com/Manual/class-BoxCollider.html) to calculate collisions and you can use them in your scripts to trigger events. By default the Ragdoll Wizard usually makes the colliders way bigger than what you really need (see raw the colliders in green on the figure bellow), what makes the ragdoll to move in a clumsy and sometime even unexpected ways. For my character I made sure they were almost overlapping with  the real size of each respective rigid body. 

![Colliders](https://github.com/guiklink/portfolio/blob/gh-pages/public/images/twig/colliders.png?raw=true)

### 2.2: Joints
Once again Unity does not do everything automatically for us. The ragdoll uses the [Character Joint](http://docs.unity3d.com/Manual/class-CharacterJoint.html) component that gives you three preconfigured axes of rotations. Make sure to put the orange gizmos as the axis of rotations for places that do not rotate back like knees and elbows, this is the only axis you can have different values for maximum and minimum rotation, while the others axis will rotate the same amount of degrees for both sides (if you configure with 40 it will rotate from -40 to +40). Finally, make sure each joint is rotating as much as you will need on each axis, usually the arms require the most attention. In the figure bellow I show an example of how the joints in the arm should be set.

![Joints](https://github.com/guiklink/portfolio/blob/gh-pages/public/images/twig/JointsConfig.png?raw=true)

### 2.3: Add the Foot Rigid Body
By default the Ragdoll wizard does not add a [rigid body](http://docs.unity3d.com/Manual/class-Rigidbody.html) for the foot. In my implementation of Twig the forces that move the leg are added in the foot, the upper leg controller's only function is to lock undesired rotations so the character does not move in undesired ways. Therefore, I added a rigid body for the each foot and attached it to its respective leg through a [Character Joint](http://docs.unity3d.com/Manual/class-CharacterJoint.html). 

## Making the Ragdoll Stand
Before being able to walk the ragdoll needs to be able to stand. This was perhaps the most important step, since it needs to stand stable and general enough so it do not complicate the implementation of other movements. For standing, PID controller applies force both in the hips and head, in order to maintain the center of mass of the body alligned according to a computated reference point. 

### Computing the Reference Point {#index-ignore}
![Mid Point](https://github.com/guiklink/portfolio/blob/gh-pages/public/images/twig/midpoint.png?raw=true)

The figure above exmplifies how the reference point for the hips is calculated. Having a 3D coordinates dictaded by the world-frame at the top right, Y is an arbitrary heigh from the floor and X and Z is the mid-point between the feet. There are two big advantages of calculating the reference point this way. First, by simply decreasing the value of Y you can easily make the ragdoll crouch, or by increasing and decreasing in a sequence you can make the ragdoll jump. Second, since X and Z is always updated as the mid-point of the feet, when one of the foot moves forward the mid-point will also move forward (the value of Z changes), thus the force that makes the upper body moves when walking will be automatically generated by the PID trying to allign to the new reference.
 
### PID Gains {#index-ignore} 
Configuring the gains of the controller was challenging, as I said before getting a stable position was crucial for the walking. Too big gains makes the ragdoll to move unrealistic fast or keep contantly shaking, while to small gains would not be enought to rise the doll in a appropriate standing pose. After I got a raw tuning of the proportional gain, I tunned the other PID constants by throwing different weights on my ragdoll and looking at its response, as shown in the video bellow.

<iframe src="https://player.vimeo.com/video/137815980" width="500" height="281" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe> <p><a href="https://vimeo.com/137815980">Testing PID gains</a> from <a href="https://vimeo.com/user43396191">Guilherme Klink</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

## The State Machine for walking
The ragdoll has over fifteen rigid bodies and more than double number of joints, keep tracking of what it of this parts will be doing during a walking is not an ordinary task, many bugs were conceived by actuators or scripts not synchronized appropriately. Moreover, in the future my ragdoll will have more gaits other than walking (e.g. running, kicking, seating). To manage how all body parts are actuated, which movement will be performed and facilitate debugging I used a state machine script, perhaps the biggest difference of my approach of Twig in comparison to the one original work from Professor Horswill.

The diagram below shows the schematic of the state transition:

![State Diagram](https://github.com/guiklink/portfolio/blob/gh-pages/public/images/twig/statemachinediagram.png?raw=true)

**Stand:** at this state the ragdoll stays idle and the joints of the leg lock in the respective position.  
**Crouch to Walk:** here the ragdoll crouches by an arbitrary amount so one of its legs have enough length to reach for a step.   
**Pick Leg:** the positions of the legs are calculated with respect to the hips coordinate frame, the leg that is behind will be picked as the one to step.   
**Calculate Left/Right Step:** Using the maximum length of the leg, the position of both feet and the position of the hips, an algorithm calculates where the respective step should end.   
**Left/Right Step:** A force moves the respective foot to the position calculated in the previous state.   
**Rise to Stand:** When the system stops getting an input to walk, the next step will be made so both legs are aligned, then the ragdoll will raise its hips to the same height from where it started moving.  

## Step 5: Moving the Arms
Moving the arms for walking in theory is very simple, a torque is added on the shoulder joint and elbow joint as soon as the opposite step is given. Still, a lots of constants needs to be properly calibrated, the initial amount of torque needs to be adjusted according to different types of ragdolls and different walking speeds.  
A mathematical function actuating as a spring damping gives the arm a more natural oscillating behavior. The amount and damping frequency of this function also can be customized to different ragdoll models.

# Next Steps

* Develop a running GAIT
* Make two ragdolls play tag
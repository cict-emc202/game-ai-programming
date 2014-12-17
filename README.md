AI Programming
======
Helpful articles about AI programming

## Outline

Reference: https://developer.valvesoftware.com

AI stands for Artificial Intelligence and controls the behaviour of all but the simplest of NPC actions.

* Decision making
    * Overview
    * Sensing
    * Conditions
    * States
    * Schedules
    * Tasks
* AI concepts
    * Memory
    * Relationships
    * Readiness
    * Squads
    * Behaviors
        * Act Busy
        * Assault
        * Follow
        * Lead
    * Interactions

## Scripting
Reference: http://rfactory.org/aiscriptex.html
###Player Scripting

A quick look through the list of supported script events will reveal that the player only has three events avalible, spawn, playerstart and trigger. The reason for this, of course, is that the player doesn't need any AI support as we provide that ourselves. The three supported events, though, do serve a purpose which we will explore further.

* The spawn event block, for the player, is used to adjust the player's persistent attributes so he has the proper items, health and weapons for the current level. In a game where you progress linearly from level to level, preserving the persistent attributes as you go, there isn't much call to adjust the attributes. However, you may still wish to restore the health to the maximum or top up the ammunition. In games where each level is a seperate mission, though, you have to give the player the necessary equipment and health at the start of the level. In these instances the spawn event block comes into play.

As an example we will look at a game where you start the first level requiring the machinegun and some ammunition. The single player game code is designed to have the player start with no weapons so we must provide them in the spawn event.
```
spawn
{
 giveweapon machinegun;
 setammo bullets 100;
 selectweapon machinegun;
}
```
This will, when the player spawns, give him the machinegun and 100 rounds of ammunition and make him choose that as his current weapon. As well, we can give the player other weapons and items so he is fully equipped at the start of the level.
```
spawn
{
 giveweapon machinegun;
 setammo bullets 100;
 giveweapon shotgun;
 setammo shells 20;
 selectweapon machinegun;
 giveinventory flight;
}
```
Here the player gets the machinegun, the shotgun, some ammunition and the Flight powerup to start of the level.

* The playerstart event block is unique to the player. This event is run 0.5 seconds after the player spawns into the level, which gives enough time for all other entities to settle down and become stable. Unlike the spawn event, this event is the actual start of the game. After this event occurs the screen fades in from black in about 1 second. Therefore, anything that happens in the playerstart event block is not immediately visible to the player. This is important, as it allows you to spawn entities without the player seeing them suddenly appear in the game. It also allows seamless switching to a cut scene to start the level.

    As an example, we will look at a game where an NPC is spawned just as the game starts.
```
playerstart
{
 alertentity ogre1;
}
```
Here we use the alertentity command to "use" the entity with the ainame of ogre1. If this entity had it's spawnflags set to start suspended then "using" it will cause it to spawn. Another use for the playerstart event block is the starting of the opening cut scene.

The usual method creating a cut scene has a script_play entity as the scene director with the cameras being switched by the player using trigger events. An opening cut scene requires the script_play entity to be triggered from the playerstart event block to kick things off. The switch to the first camera is usually done here as well.
```
playerstart
{
 trigger director cinematic1;
 trigger player cine1_cam1;
}
```
Here we trigger our script_play entity (with the ainame of director) to run the trigger event block named cinematic1. This starts the director side of the cut scene. Then we trigger the player's trigger event block named cine1_cam1 which will switch to the first camera of the cut scene. Because the screen is black when the playerstart event block is run the cut scene will fade in smoothly.

* The trigger event blocks are used to control events during the game. Because the player has no other events happening during game play (or the running of a cut scene) the player script block is a good place to contain triggered events, as triggering them will not upset any other script actions that may be taking place. Other entities (such as our script_play director entity) can trigger the player trigger event blocks to do things like switch cameras or spawn in entities without worrying about messing up a running script event.

As an example, here are several trigger event blocks that are used to switch cameras during a cut scene.
```
trigger cine1_cam1
{
 startcamblack crypt01;
}
trigger cine1_cam2
{
 startcam crypt02;
}
```
When the cine1_cam1 trigger event is run it starts the crypt01.camera script running, fading it in from black at the start. When the cine1_cam2 trigger event is run it starts the crypt02.camera script running. As seen above, the cine1_cam1 trigger event is run from the playerstart event block while the cine1_cam2 trigger event would be triggered by the script_play director entity at the correct time.

Important : There is one special trigger event block for the player that is run from the program. When we are running a cut scene it can be interrupted at any time by pressing the ESC key. When this happens we abort the camera script and return to normal game play. In doing so we may bypass any cleanup or spawning scripts that would normally occur at the end of the director's script. As well, the director's script keeps running so we must stop it or we will switch back to the cut scene with the next camera change. To correct these problems, if we interrupt a cut scene with the ESC key, the program will cause the player's trigger event block named cameraInterrupt to be run. In this block we trigger the director to stop the cut scene and do the cleanup.
```
trigger cameraInterrupt
{
 trigger director cleanup;
}
```
NPC Scripting

* A major area in an NPCs AI scripting is the attributes event block. This is where we can make each NPC in the level an individual. By altering the attribute values, we no longer have all NPCs of one type being exactly the same. They can be different in such areas as hearing distance, sight distance, health levels, skin used for texturing, etc. This allows a much more varied group of NPCs in the level without having to create new NPC types.

* The use of skins for texturing the NPC model is one area that contributes greatly to the look of a level. While we can define the skin file in the entity definition, using the skin key, it can also be done in the attributes event block, which is quicker as a change doesn't require a map recompile. An example of using a skin to change a model's texture is :
```
attributes
{
 skin soldierrough;
}
```
This will use the soldierrough.skin file to texture the model for this particular NPC. Other NPCs will be unaffected.

By altering enough values in the attributes event block you can pretty much create a new type of NPC from an existing type. It can be faster, jump higher, have more health, look different and shoot better than the base NPC just by a few changes in the script. For example :
```
attributes
{
 health 150;
 jumpheight 80;
 walkspeed 50;
 runspeed 90;
 skin soldierrough;
 aim_accuracy 1.0;
}
```
would change a regular soldier NPC into a "super" soldier.

* A common feature in a level is an NPC walking a route doing a patrol. This can be accomplished in the level design by creating a path of npcpath entities and linking this to the NPC entity but it has certain drawbacks. Once an NPC leaves off walking a path defined this way, he can never return to it. He is also easily distracted from walking the path by external events and may not respond exactly as you wish. By using the AI script to make the NPC walk it's path you can control exactly how he responds to events and can make him return to the path at any time.

The path an NPC will walk is made up of unconnected npcpath entities. The NPC will use the bot navigational information to move to each npcpath entity so they do not have to be in sight of each other. As an example, let us suppose we have four npcpath entities in our level, with targetnames of p1, p2, p3 and p4. If we wish the NPC to walk from p2 to p3 to p1 to p4 and then restart at p2 again our script would look something like :
```
soldier1
{
 spawn
 {
   trigger soldier1 walkpath;
 }
 trigger walkpath
 {
   walktomarker p2;
   walktomarker p3;
   walktomarker p1;
   walktomarker p4;
   trigger soldier1 walkpath;
 }
}
```
While this may look a little odd, it works just fine. When the NPC (with the ainame of soldier1) spawns it triggers it's own trigger walkpath event. This event causes the NPC to walk from wherever it is to the p2 entity location. When it reaches that point it will then walk to the p3 point, then to the p1 point and finally to the p4 point. After that it triggers it's own trigger walkpath event again so it starts walking the path all over again.

This example uses an NPC that is walking the path as soon as it spawns. If we want it to start walking the path after some event occurs then we'd remove the trigger command from the spawn event block and let another entity trigger the trigger walkpath event.

If we want the NPC to walk the path without ever stopping at any of the points we can make it look better by changing the script to :
```
soldier1
{
 spawn
 {
   trigger soldier1 walkpath;
 }
 trigger walkpath
 {
   walktomarker p2 nostop;
   walktomarker p3 nostop;
   walktomarker p1 nostop;
   walktomarker p4 nostop;
   trigger soldier1 walkpath;
 }
}
```
The nostop parameter on the walktomarker command expands the distance from the point that the NPC can be and still be regarded as reaching the point. This gives a larger area for the NPC to turn in when it starts moving toward the next point and will make the movement look much smoother.

A more realistic looking path walk would probably have the NPC stopping at various locations and looking around for enemies. Lets suppose we want the NPC to stop at point p1 and look in two different directions before resuming his walk. We will use different methods of giving the NPC a direction to look in for each direction we want him to look.

The first method of providing a direction requires us to define an angle key in the npcpath entity p1. We will have our NPC turn to this angle to face in one of our directions. The other method will have the NPC turn to face an entity in the level. In this case we will have him turn to face the p2 npcpath entity.
```
soldier1
{
 spawn
 {
   trigger soldier1 walkpath;
 }
 trigger walkpath
 {
   walktomarker p2 nostop;
   walktomarker p3 nostop;
   walktomarker p1;
   facetargetangles p1;
   wait 3000;
   wait 3000 p2;

   walktomarker p4 nostop;
   trigger soldier1 walkpath;
 }
}
```
Once the NPC reaches point p1 we have him turn to face the direction defined by the angle key in that entity. His speed of turning is governed by his walking rotation speed, which is defined in the NPC config.txt file or in the attributes event block. He will wait at this location 3 seconds to allow the rotation to take place. He will then wait another 3 seconds while turning to face the p2 npcpath entity before resuming his walkabout.

A note about path walking using the gotomarker and gotonpc (and their walk and run variations) commands. While the NPC is executing these commands he will not respond to any external events, such as an enemy appearing or being shot at. He has a one track mind devoted only to moving to a location. If we want the NPC to respond to external events we must force him to do so using the resetscript command.

One of the most common events the NPC should respond to is pain. If the NPC is hit and is in pain he should stop walking the path and be alert to whoever shot him. To do this we add the pain event block to the script as so :
```
soldier1
{
	spawn
	{
   		trigger soldier1 walkpath;
 	}
 	trigger walkpath
 	{
     walktomarker p2 nostop;
     walktomarker p3 nostop;
     walktomarker p1;
     facetargetangles p1;
     wait 3000;
     wait 3000 p2;
     walktomarker p4 nostop;
     trigger soldier1 walkpath;
 	}
 	pain
 	{
   	resetscript;
   	statetype alert;
 	}
}
```
This addition will, when the NPC is in pain, reset the script to break him out of the path walking and set his behavior state to alert, where he can now look about for enemies.

In real life, a guard walking the beat would investigate shots being fired rather than continuing on his way. The NPC can be made to do this as well by use of the following :
```
soldier1
{
 spawn
 {
   trigger soldier1 walkpath;
 }
 trigger walkpath
 {
   walktomarker p2 nostop;
   walktomarker p3 nostop;
   walktomarker p1;
   facetargetangles p1;
   wait 3000;
   wait 3000 p2;
   walktomarker p4 nostop;
   trigger soldier1 walkpath;
 }
 bulletimpact
 {
   resetscript;
 }
}
```
The bulletimpact event block is run whenever the NPC hears or sees a projectile and we use the resetscript command to break him out of the path walking and allow him to start investigating the sound or impact.

Because NPCs are used in limited numbers and in localized areas, we know the names of the friendly and enemy NPCs he will encounter. We can make use of that knowledge to have him return to walking the path if the event turned out to be something he didn't have to respond to. Lets suppose that in the area of the soldier1 NPC there is one other NPC named soldier2 and the player. If we want the soldier1 NPC to ignore shots from soldier2 and the player, but to respond to shots from other NPCs, we could do something like :
```
soldier1
{
 spawn
 {
   trigger soldier1 walkpath;
 }
 trigger walkpath
 {
   walktomarker p2 nostop;
   walktomarker p3 nostop;
   walktomarker p1;
   facetargetangles p1;
   wait 3000;
   wait 3000 p2;
   walktomarker p4 nostop;
   trigger soldier1 walkpath;
 }
 bulletimpact
 {
   resetscript;
 }
 inspectbulletstart soldier2
 {
   walktomarker p3 nostop;
   trigger soldier1 walkpath;
 }
 inspectbulletstart player
 {
   walktomarker p4 nostop;
   trigger soldier1 walkpath;
 }
}
```
Here the NPC breaks out of the path walking on the sound or sight of a projectile. His normal reaction would then be to investigate the event. However, if the projectile was fired by the soldier2 NPC we make him walk to the p3 point and then resume his path walking. If it was the player shooting then he goes to the p4 point and resumes the path walk. A shot from any other NPC will make him investigate. If the soldier2 NPC was fighting an enemy rather than just shooting wildly, this NPC would go to investigate since the order of investigation is :

friendly NPC fighting
sound of enemy weapon being fired
sound/sight of projectile

* The response of an NPC can be tailored to fit a certain scenario by use of the deny or denyaction commands (these commands are identical). Suppose we have our soldier1 NPC standing guard at a point and we don't want him to leave unless his partner (the soldier2 NPC) is seen fighting an enemy NPC named ank1. We have to stop the NPC from investgating sounds and projectiles until the combat is visible. One way to do this is :

```
soldier1
{
 spawn
 {
   nosight forever;
 }
 bulletimpact
 {
   deny;
 }
 inspectsoundstart ank1
 {
   deny;
 }
 inspectfriendlycombatstart
 {
   sight;
 }
}
```
When the NPC spawns we stop him from ever seeing an enemy so he doesn't start attacking at the wrong time. In the bulletimpact event block the deny command will abort any investigation of projectiles, while the deny command in the inspectsoundstart ank1 event block will stop him from investigating weapon firing from the ank1 NPC. In the inspectfriendlycombatstart event block we restore the enemy sight when we see our friend fighting so we can go and attack as well.

Cut Scenes

Cut scenes are a special case of AI scripting. In these scripts the NPCs are under strict control and only do what they are told. The player script is used to trigger camera changes and a script_play entity is used as the director. Let's look at how one is put together.

To keep the NPCs under strict control they must have their normal reactions to events halted. A basic script to do that is :
```
cine_soldier1
{
 spawn
 {
   nosight forever;
 }
 bulletimpact
 {
   deny;
 }
 inspectfriendlycombatstart
 {
   deny;
 }
}
```
This keeps them from attacking any enemies or investigating events occuring around them. All their actions will be under the control of an outside agent, usually the director, so they will use trigger event blocks to cause the action to happen. Suppose, as part of the cut scene, the soldier must move to a specific point, p1, and then look at another point, p2, until further instructions. The script would be :
```
cine_soldier1
{
 spawn
 {
   nosight forever;
 }
 bulletimpact
 {
   deny;
 }
 inspectfriendlycombatstart
 {
   deny;
 }
 trigger movelook
 {
   walktopoint p1;
   wait forever p2;
 }
}
```
To achieve the desired action, the director would trigger the trigger movelook event. The director would control the timing of each NPC's action and trigger them at the correct point in the cut scene.

As pointed out above, the camera switching is done by use of trigger events in the player's script. This is not strictly required, as we could use another script_play entity as the cameraman but it does save us one entity in the level. Regardless of which you use, player or cameraman entity, their script would consist of a number of trigger event blocks that start a camera script running. The director will trigger these at the correct time to switch cameras for the cut scene. To achieve a smooth transition into the cut scene the first camera should be started using the startcamblack command. This will cause the screen to go black and then fade in to the scene rather than abruptly changing to it.
```
player
{
 trigger firstcamera
 {
   startcamerablack firstcam;
 }
 trigger secondcamera
 {
   startcamera secondcam;
 }
}
```
When the director executes the trigger player firstcamera command you'll get the smooth transition you desire. When he executes the trigger player secondcamera command the change will be abrupt, as you would expect.

The director must keep track of the elapsed time after a camera change and switch to another as the current camera's time expires. As you know from the scripted camera tutorials, a camera script will only run for a predetermined amount of time before the script ends and the view is switched back to the player. To keep from having a jumpy view, the director must switch to a new camera script just as the old one ends. Suppose the firstcam script runs for 15 seconds and the secondcam script runs for 7 seconds. The director script would look something like the following when switching cameras :
```
director
{
 trigger cine1
 {
   trigger player firstcamera;
   wait 15000;
   trigger player secondcamera;
   wait 7000;
 }
}
```
If a camera script is allowed to run out without switching to a new camera, as would happen for the last camera script used in the cut scene, the view will fade to black and then fade in to the player's view, giving a smooth transition back.

One important part of the director's script is the cut scene cleanup. As mentioned above, this is a script event block that is used to remove cut scene entities, stop any music and do other finalization work. It should be contained in it's own trigger event block so the player's trigger cameraInterrupt event block can trigger it if the cut scene is aborted early. If nothing else, the cleanup event block should reset the script (using the resetscript command ) just so it can handle cut scene interruption (an empty event block won't stop the cut scene from running).
```
player
{
 trigger firstcamera
 (
   startcamerablack firstcam;
 }
 trigger secondcamera
 {
   startcamera secondcam;
 }
 trigger cameraInterrupt
 {
   trigger director cleanup;
 }
}
director
{
 trigger cine1
 {
   trigger player firstcamera;
   wait 15000;
   trigger player secondcamera;
   wait 7000;
   trigger director cleanup;
 }
 trigger cleanup
 {
   deletetarget soldier1;
   ...
 }
}
```

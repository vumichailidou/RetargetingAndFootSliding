# Retargeting in UE

This Unreal Engine project demonstrates several animation retargeting scenarios across different characters. It showcases various techniques used to minimize and prevent foot sliding issues that can occur during the retargeting process.

You can find the different retargeting examples in the Retargeting folder. The files are named 
- UE_RET-01
- UE_RET-02
- UE_RET-03.

1. In the parameters, assign the IK Rig of the Mannequin and the IK Rig of your character respectively.

![alt text](image-2.png)

    This step defines the mapping of the corresponding joints between the two rigs.

2.  Create your joint chains. For human skeletons, it is recommended to follow the same structure as the Mannequin:

- Spine
- Legs
- Arms
- Neck
- Head (separated from the Neck)

- The Clavicle bones should be handled separately, as they can contain translation values in addition to rotation.

In the lower-left section, create a Full Body IK solver by clicking + Add New Solver.

![alt text](image-3.png)

3. Select the end joint of the hand and leg chains, which in most cases are the hand and foot bones. Right-click on the selected joint and choose New IK Goal to create an IK chain from it.

    Select the entire joint chain (here, for example, shoulder to right hand) while having Full Body IK selected at the bottom. Right-click on one of the joints and you can apply Full Body IK to it using "Add settings to selected bone".

    It creates a more natural deformation and lets the IK solver affect the connected joints while maintaining the overall pose.

![alt text](image-4.png)


4. Afterward, open the Animation Blueprint (ABP_Unarmed) of each retargeted character. In the Anim Graph, connect the corresponding retargeting asset (UE_RET-01, UE_RET-02, or UE_RET-03) to the Output Pose node.



![alt text](image.png)

    In the Blueprint of the respective character, the Skeletal Mesh needs to be attached as a child component of the Mannequin.

    The Mannequin itself should be hidden by searching for Visible in the Details panel and disabling this option.

    After that, the previously created Animation Blueprint (ABP) can be assigned. This ABP only consists of the Retarget Pose from Mesh node.

    The character should now be playable and has inherited all the logic from the mannequin.

![alt text](image-1.png)
_____________________________________________________________

# Root Motion

To use Root Motion, we first need an animation that actually contains Root Motion. You can often recognize such animations by the red trail they leave behind.

![alt text](image-5.png)

1. First, enable Root Motion by checking the "Enable Root Motion" option.

![alt text](image-6.png)

2. To prevent the animation from snapping back to its starting position, open your Animation Blueprint, go to Class Defaults, and set the Root Motion Mode to either Root Motion from Everything or Root Motion from Montages Only.

![alt text](image-7.png)

The Root Motion animation should now work correctly.

In the Michailidou_Vu_BA_Echtzeitverfahren project, I've bound several Root Motion dance animations to the 5 key. You can use them to easily test whether Root Motion is working correctly.
_____________________________________________________________

# Foot Sliding Solver

The ABP_Unarmed already includes several techniques for reducing foot sliding, which can be enabled or disabled using the 1–4 keys. The Control Rig implementation is, in theory, the same as the Foot Placement node.

![alt text](image-8.png)

I recreated my own Foot Placement system to take a closer look at how it works. Since both implementations calculate the target position for the Leg IK, the built-in Foot Placement node and the custom Foot Placement system cannot run at the same time. Therefore, one of them must always be disconnected.

![alt text](image-9.png)

The logic can be found in the Event Graph.
_____________________________________________________________

# Stride Warp

Stride Warping dynamically adjusts the character's stride length based on its movement speed. 

Since the character's movement is driven by a Blend Space according to its current speed, it is recommended to switch the Mode to Graph and set the Ground Speed variable as the Locomotion Speed. Finally, in the Settings, simply assign the Pelvis Bone and define the IK bone chains.

![alt text](image-11.png)

For systems like these to work, the skeleton needs IK bones that define the target positions. By default, the Unreal Engine Mannequin already includes this setup.

Unlike the Mannequin, the MetaHuman does not include this IK hierarchy by default. To add it, open the Skeleton Editor, right-click the Root bone, and select Add Virtual Bone. Then recreate the hierarchy exactly as shown in the image, using the same bone names.

The basic hierarchy should match the Unreal Engine Mannequin. Start by creating an ik_foot_root as a child of the root bone. Under ik_foot_root, create ik_foot_l and ik_foot_r. If you also plan to use hand IK, create an ik_hand_root with ik_hand_l and ik_hand_r as its children.

When creating a Virtual Bone, Unreal Engine asks for a source bone and a target bone. The Virtual Bone is automatically positioned so that it follows the target bone while remaining part of the hierarchy under the selected parent. For the foot IK setup, each Virtual Bone should reference the corresponding left or right foot bone so that it always matches the foot's position.

Virtual Bones automatically snap to their target bones, so there is no need to position them manually. Keep in mind that they are only visible in the Skeleton Editor—they do not appear in the Animation Sequence editor, so don't worry if you can't see them there.

Once this hierarchy has been created, it can be used as the target for systems such as the built-in Foot Placement node or a custom Foot Placement implementation.

![alt text](image-10.png)

I replaced the Walk animation with the Run animation in the Blend Space to make the effects of Stride Warping more noticeable. You can also test this setup in the Michailidou_Vu_BA_Echtzeitverfahren project.

![alt text](image-12.png)
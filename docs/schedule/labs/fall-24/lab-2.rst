Lab 2: Forward Kinematics
=========================

Goal
----
Implement forward kinematics for the right front leg of the Pupper robot using ROS2 and Python.

Fill out the `lab document <https://docs.google.com/document/d/1uAoTIHvAqEqXTPVWyHrLkuw0ZJ24BPCPn_Q6XIztvR0/edit?usp=sharing>`_ as you go. Make a copy and add your responses.

Part 1: Hardware Build
-------------

1. Follow the build instructions for lab 2: `lab 2 assembly instructions <https://drive.google.com/file/d/1xkli-Mg0iUog6XsUrviYll4hlnVv-qmk/view?usp=sharing>`_. You will build a front right leg for Pupper in this lab. Begin by checking to see that your kits contain all the pieces, if not, please ask a TA. 

Part 2: Setup
-------------

1. Make sure you have completed Lab 1 and are familiar with the ROS2 environment on your Raspberry Pi 5.

2. Clone the lab 2 code repository on the Raspberry Pi:

   .. code-block:: bash

      cd ~/
      git clone https://github.com/cs123-stanford/lab_2_2024.git lab_2

3. Open the workspace (lab_2 directory) in VSCode and examine the ``lab_2.py`` file.

4. Change all the 12 occurances of ``homing_kp`` values to ``1.0`` and ``homing_kd`` values to ``0.1`` in the ``~/ros2_ws/src/pupper_v3_description/description/components.xacro`` file. 

**DELIVERABLE:** Why do you think that there are 12 occurances of these values in the xacro file? What do you think that changing them from the previous value does?

Part 3: Understanding the Code Structure
----------------------------------------

Before we start implementing the TODOs, let's understand the structure of the ``lab_2.py`` file:

1. The code defines a ``ForwardKinematics`` class that inherits from ``rclpy.node.Node``.
2. It subscribes to the ``joint_states`` topic and publishes to the ``leg_front_r_end_effector_position`` topic.
3. The ``forward_kinematics`` method is where we'll implement the forward kinematics calculations.
4. The code uses NumPy for matrix operations.
5. Note that it is convention to orient the coordinate frame so that the rotation about each motor is the z axis.

Part 4: Implementing Forward Kinematics
---------------------------------------

Step 1: Implement Rotation Matrices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Open ``lab_2.py`` and locate the ``forward_kinematics`` method.

2. Implement the rotation matrices about the x, y, and z axes. Follow the homogeneous coordinates representation as presented in lecture.

**DELIVERABLE:** Which axis is typically used as the default axis for rotations in robotic systems? Why?

Step 2: Implement Transformation Matrices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Note that theta is the motor angle

1. The transformation matrix from the base link to leg_front_r_1 has been implemented for you in ``T_0_1``. This involves a translation and two rotations. Understanding this transformation will help you complete the remainder of the transformations. 

**DELIVERABLE** Explain the reasoning behind this implementation. What does the translation and each of the rotations do in ``T_0_1``?

2. Implement the transformation matrix from leg_front_r_1 to leg_front_r_2 in ``T_1_2``. Follow the same thought process as with ``T_0_1``.

3. Implement the transformation matrix from leg_front_r_2 to leg_front_r_3 in ``T_2_3``.

4. Implement the transformation matrix from leg_front_r_3 to the end effectorin ``T_3_ee``.

5. Compute the final transformation matrix following the described process from lecture in ``T_0_ee``. Remember that the end effector position is not in homogeneous coordinates. Calculate ``end_effector_position`` from ``T_0_ee``.

   Note: The translation values may need to be adjusted based on the actual dimensions of your robot. Make sure to verify these values with your robot's specifications.

**DELIVERABLE:**

1. Write out the full equation you used to calculate the forward kinematics (in math), please use Latex and take a screenshot, or use the equation functionality in google docs
What is the benefit of using homogeneous transformations? 

2. Why is there a 1 in the bottom-right corner of a homogeneous transformation matrix?


Part 5: Testing Your Implementation
-----------------------------------

1. Save your changes to ``lab_2.py``.

2. Run the ROS2 nodes:

   .. code-block:: bash

      ros2 launch lab_2.launch.py

3. In another terminal, use the following command to run the main code:

   .. code-block:: bash

      python lab_2.py

4. Move the right front leg of your robot and observe the changes in the published positions.

To test your code in simulation to make sure that the code works as expected, you can use RVIZ. RVIZ will show the Pupper model as well as a marker that shows the output from the forward kinematics.

   .. code-block:: bash

      rviz2 -d lab_2.rviz

The above command will load the RVIZ config file. If you just run ``rviz``, you can manually add the configuration. After running `rviz`, click the "Add" button, and then select a Robot Model type. Select the /robot_description topic. Next, add the marker by selecting "Add" again, and select a Marker type. Select the topic /marker.

Part 6: Analyzing the Results
-----------------------------

1. Record the end-effector positions for at the front right leg configurations.

2. Compare these positions with the expected positions based on the physical dimensions of your robot. (Why are the numbers printed in the terminal so small?)

3. If there are discrepancies, try to identify the source of the errors. It could be due to:
   
   - Incorrect transformation matrices
   - Inaccurate joint angle readings
   - Errors in the physical measurements of the robot

**DELIVERABLE:**

1. Measuring the correct physical parameters of the robot (leg lengths, motor angles, etc) is essential to compute accurate kinematics. This process is called system identification. How would your estimate of the end effector (EEF) position change if your estimate of leg link 2 is off my 0.2 cm? What about 0.4cm, or 0.8cm? Write out the number you computed, and how you calculated them, for both 0 degrees rotation in each of the joints, and 45 degrees rotation in each of the joints. Qualitatively, how does error in estimated EEF position change with respect to error in leg length? 

2. How does computational complexity of FK scale with respect to degree of freedom (number of motor angles)? Please use big O notation.


Additional Challenges (Optional)
--------------------------------

If you finish early or want to explore further:

1. Extend your implementation to calculate forward kinematics for all four legs of the Pupper robot.
2. Create a visualization of the leg's end-effector position using RViz or another visualization tool.

Remember, understanding forward kinematics is crucial for robot control and motion planning. Take your time to ensure you understand each step of the process.

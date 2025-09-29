Lab 2: Forward Kinematics
=========================

Goal
----
Implement forward kinematics for the left front leg of the Pupper robot using ROS2 and Python.

Here's what your implementation should look like when complete (this quarter it will be the left front leg instead):

.. raw:: html

   <iframe width="560" height="315" src="https://www.youtube.com/embed/lVcSik8-KIg" title="Kinematics RViz Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Fill out the `lab document <https://docs.google.com/document/d/1uAoTIHvAqEqXTPVWyHrLkuw0ZJ24BPCPn_Q6XIztvR0/edit?usp=sharing>`_ as you go. Make a copy and add your responses.

Part 1: Hardware Build
------------------------

1. Follow the build instructions for lab 2 

    .. raw:: html

        <a href="https://docs.google.com/presentation/d/1LWhURxF0z4iUYnUWLJexuQ4GSN4Q30BKO6-dLN8Wb0w/edit?usp=sharing" target="_blank" style="font-size: 1.2em; font-weight: bold; color: #E53E3E; background-color: #FED7D7; padding: 10px 15px; border-radius: 5px; text-decoration: none; display: inline-block; margin: 10px 0;">📝 Open build instructions in new tab 📝</a>

    
    .. raw:: html

       <iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRHYkkOR1CYsA7x-u3RZAFrIjvhlZBjibNNWEvTePSsiXtnQ3fwN75Bu6I5iVGKe202sfwx_FWMzLbF/pubembed?start=false&loop=false&delayms=60000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
    
    You will build a front left leg for Pupper in this lab. Begin by checking to see that your kits contain all the pieces, if not, please ask a TA. 


Part 2: Setup
---------------

1. Make sure you have completed Lab 1 and are familiar with the ROS2 environment on your Raspberry Pi 5.

2. Open the lab 2 code repository (`lab 2 code repository <https://github.com/cs123-stanford/lab_2_fall_2025>`_) on your GitHub account. Then, fork the repository to your own GitHub account following the instructions in :doc:`forking_repositories`.

3. Open the lab 2 folder in VSCode

   .. code-block:: bash

      cd ~/lab_2_fall_2025
      code .

Part 3: Understanding the Code Structure
-------------------------------------------

Before we start implementing the ``TODOs``, let's understand the structure of the ``lab_2.py`` file:

1. The code defines a ``ForwardKinematics`` class that inherits from ``rclpy.node.Node``.
2. It subscribes to the ``joint_states`` topic and publishes to the ``leg_front_l_end_effector_position`` and ``marker`` topics.
3. The ``forward_kinematics`` method is where we'll implement the forward kinematics calculations.
4. The code uses NumPy for matrix operations.
5. Note that it is convention to orient the coordinate frame so that the rotation about each motor is the z axis.

Part 4: Implementing Forward Kinematics
------------------------------------------

For the following steps, you can view the Pupper CAD to help you understand the kinematic chain `CAD <https://cad.onshape.com/documents/97a1bc3e752ec66822dbb5bb/w/c7f9232ccbc53a2e3f6ee909/e/74c0b3caf828b9fd1994bcd6?renderMode=0&uiState=67f1c37599fde447b364a89c>`_

Step 1: Implement Rotation Matrices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Open ``lab_2.py`` and locate the ``forward_kinematics`` method.

2. Implement the rotation matrices about the x, y, and z axes. Follow the homogeneous coordinates representation as presented in lecture.

**DELIVERABLE:** Which axis is typically used as the default axis for rotations in robotic systems? What angles are we rotating along the default axis? Why?

Step 2: Implement Transformation Matrices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::
   In the following steps, :math:`\theta` (theta) represents the motor angle. Figuring out the sign of :math:`\theta` will be trickier than you might expect!

1. The transformation matrix from the base link to leg_front_l_1 has been implemented for you in ``T_0_1``. This involves a translation and two rotations. We include a visualization of this transformation below to facilitate your understanding (keeping all these in mind can be tricky!). Understanding this transformation will help you complete the remainder of the transformations. 

   .. figure:: ../../../_static/kinematics/base_l1_kinematics.png
      :align: center
      :width: 75%

      Transformation from base link to leg_front_l_1

**DELIVERABLE:** Explain the reasoning behind this implementation. What does the translation and each of the rotations do in ``T_0_1``?

2. Implement the transformation matrix from leg_front_l_1 to leg_front_l_2 in ``T_1_2``. Follow the same thought process as with ``T_0_1``. Check out the figure below for visual reference.

   .. figure:: ../../../_static/kinematics/l1_l2_kinematics.png
      :align: center
      :width: 75%

      Transformation from leg_front_l_1 to leg_front_l_2

3. Implement the transformation matrix from leg_front_l_2 to leg_front_l_3 in ``T_2_3``. Check out the figure below for visual reference.

   .. figure:: ../../../_static/kinematics/l2_l3_kinematics.png
      :align: center
      :width: 75%

      Transformation from leg_front_l_2 to leg_front_l_3

4. Implement the transformation matrix from leg_front_l_3 to the end effector in ``T_3_ee``. Check out the figure below for visual reference.

   .. figure:: ../../../_static/kinematics/l3_ee_kinematics.png
      :align: center
      :width: 75%

      Transformation from leg_front_l_3 to the end effector

5. Compute the final transformation matrix following the described process from lecture in ``T_0_ee``. Remember that the end effector position is not in homogeneous coordinates. Calculate ``end_effector_position`` from ``T_0_ee``.

Part 5: Debugging Your Implementation With RVIZ2
---------------------------------------------------

1. Save your changes to ``lab_2.py``.

2. Run the ROS2 nodes:

   .. code-block:: bash

      ros2 launch lab_2.launch.py

3. In another terminal, use the following command to run the main code:

   .. code-block:: bash

      python lab_2.py

4. Move the left front leg of your robot and observe the changes in the published positions.

To test your code in simulation to make sure that the code works as expected, you can use RVIZ. RVIZ will show the Pupper model as well as a marker that shows the output from the forward kinematics.

   .. code-block:: bash

      rviz2 -d lab_2.rviz

The above command will load the RVIZ config file. If you just run ``rviz``, you can manually add the configuration. After running `rviz`, click the "Add" button, and then select a Robot Model type. Select the /robot_description topic. Next, add the marker by selecting "Add" again, and select a Marker type. Select the topic /marker.

.. note::
   While we've tested this pipeline on a Pupper and it works as expected, rviz may fail on your robot due to heating in the Raspberry Pi. If this happens, reach out to a TA to check the implementation first, then turn off Pupper, wait a while to let it cool down, and try again.

**DELIVERABLE:** 

1. Take a video of the working implementation with you moving Pupper's leg and the simulation mimicking the results and upload it to the Google Drive

2. Write out the full equation you used to calculate the forward kinematics (in math). Please use LaTeX and take a screenshot, or use the equation functionality in Google Docs. What is the benefit of using homogeneous transformations? 

3. Why is there a 1 in the bottom-right corner of a homogeneous transformation matrix?

Part 6: Analyzing the Results
--------------------------------

1. Record the end-effector positions for the left front leg configurations.

2. Compare these positions with the expected positions based on the physical dimensions of your robot. (Why are the numbers printed in the terminal so small?)

3. If there are discrepancies, try to identify the source of the errors. It could be due to:
   
   - Incorrect transformation matrices
   - Inaccurate joint angle readings
   - Errors in the physical measurements of the robot

**DELIVERABLE:**

1. Measuring the correct physical parameters of the robot (leg lengths, motor angles, etc.) is essential to compute accurate kinematics. This process is called system identification. How would your estimate of the end effector (EEF) position change if your estimate of leg link 2 (r2) is off by 0.2 cm short from the actual distance to leg link 1 (r1)? What about 0.4 cm, or 0.8 cm? Write out the numbers you computed, and how you calculated them, for both 0 degrees rotation in each of the joints, and 45 degrees rotation in each of the joints. Qualitatively, how does error in estimated EEF position change with respect to error in leg length? 

2. How does computational complexity of FK scale with respect to degree of freedom (number of motor angles)? Please use big O notation.

Additional Challenges (Optional)
----------------------------------

If you finish early and want to explore further:

1. Extend your implementation to calculate forward kinematics for all four legs of the Pupper robot. Save your calculations for these other legs for lab 4, where we will need forward kinematics for all four legs.

   We provide the base link to leg_back_r1 transformation in the diagram below. The rest of the transformations are identical to the front leg:

   .. figure:: ../../../_static/kinematics/base_back_kinematics.png
      :align: center
      :width: 75%

      Base to back right leg transformation diagram
   
2. During the testing with rviz2, write a script that saves the sequence of your well-crafted motion, recorded as end effector positions into a file. You will have a chance to let Pupper replay this recorded motion in the next lab! You will need to use the ``joint_states`` topic to record the motor angles, and the ``leg_front_l_end_effector_position`` topic to record the end effector positions.

Friendly reminder: The first optional lab will be released next week, attempt at your own risk!

Remember, understanding forward kinematics is crucial for robot control and motion planning. Take your time to ensure you understand each step of the process!

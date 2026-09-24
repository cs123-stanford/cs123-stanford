Lab 2: Forward Kinematics
=========================

Goal
----
Implement forward kinematics for all four legs of the Pupper robot using ROS2 and Python, and
watch the computed foot positions follow the real legs in a 3D viewer in your browser.

Here's what your implementation should look like when complete (all four legs, watched in
the viser web viewer, see Part 5):

.. raw:: html

   <iframe width="560" height="315" src="https://www.youtube.com/embed/FatvKoDci50" title="Forward Kinematics Viser Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Fill out the `lab document <https://docs.google.com/document/d/1uAoTIHvAqEqXTPVWyHrLkuw0ZJ24BPCPn_Q6XIztvR0/edit?usp=sharing>`_ as you go. Make a copy and add your responses.

AI Use Suggestion
------------------
Coding agents likely won't be helpful in this lab, and we suggest against using Claude Code or
Codex for it. The forward kinematics you implement here is exactly the kind of content that will
be tested on the closed-book, pen-and-paper quiz in Week 4. Even if you fluke your way through the
lab by leaning on AI, you will have a bad time on the quiz. Work through the derivations and code
yourself; AI tools are still fine for understanding the material better.

Part 1: Hardware Build
------------------------

In Lab 1 you built the brain, the body, and one leg. In this lab you will build the full
rest of the robot: the three remaining legs, following the same leg build you have already
done once. Also put the lower leg back on the front-left knee in place of the spinning knob
from Lab 1.

.. raw:: html

        <a href="https://beepboopbeep.org/raise-a-robot" target="_blank" style="font-size: 1.2em; font-weight: bold; color: #E53E3E; background-color: #FED7D7; padding: 10px 15px; border-radius: 5px; text-decoration: none; display: inline-block; margin: 10px 0;">🤖 Open the Raise a Robot build videos in new tab 🤖</a>

.. note::
   The guide is password protected. The password is posted on Ed.

Begin by checking that your kit contains all the pieces; if not, please ask a TA. All the
pieces for each leg are labeled — each right leg piece has an **R** on it, and each left
leg piece has an **L** on it.

.. raw:: html

   <details style="margin: 15px 0; border: 1px solid #d9d9d9; border-radius: 5px; padding: 10px 15px; background-color: #fafafa;">
     <summary style="cursor: pointer; font-weight: bold; color: #404040;">📝 Optional: prefer slides? Click to show the slide deck instead</summary>
     <div style="margin-top: 15px;">
       <p>These slides cover the same leg build. They are optional &mdash; the videos above are the primary instructions.</p>
       <a href="https://docs.google.com/presentation/d/1LWhURxF0z4iUYnUWLJexuQ4GSN4Q30BKO6-dLN8Wb0w/edit?usp=sharing" target="_blank" style="font-size: 1.2em; font-weight: bold; color: #E53E3E; background-color: #FED7D7; padding: 10px 15px; border-radius: 5px; text-decoration: none; display: inline-block; margin: 10px 0;">📝 Open build instructions in new tab 📝</a>
       <br>
       <iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRHYkkOR1CYsA7x-u3RZAFrIjvhlZBjibNNWEvTePSsiXtnQ3fwN75Bu6I5iVGKe202sfwx_FWMzLbF/pubembed?start=false&loop=false&delayms=60000" frameborder="0" width="960" height="569" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>
     </div>
   </details>


Part 2: Setup
---------------

1. Make sure you have completed Lab 1 and are familiar with the ROS2 environment on your Raspberry Pi 5.

2. Open the forward kinematics lab code repository (`https://github.com/cs123-stanford/forward_kinematics_lab <https://github.com/cs123-stanford/forward_kinematics_lab>`_) on your GitHub account. Then, fork the repository to your own GitHub account following the instructions in :doc:`forking_repositories`.

3. Open the forward kinematics lab folder in VSCode

   .. code-block:: bash

      cd ~/forward_kinematics_lab
      code .

Part 3: Understanding the Code Structure
-------------------------------------------

Before we start implementing the ``TODOs``, let's understand the structure of the ``forward_kinematics.py`` file:

1. The code defines a ``ForwardKinematics`` class that inherits from ``rclpy.node.Node``.
2. It subscribes to the ``joint_states`` topic and publishes one end-effector position topic per leg (``leg_front_l_end_effector_position``, ``leg_front_r_end_effector_position``, ...) plus one colored sphere per leg on the ``marker`` topic.
3. The ``rotation_x``, ``rotation_y``, ``rotation_z`` and ``translation`` helpers build the homogeneous transforms, and the four methods ``fk_front_left``, ``fk_front_right``, ``fk_back_left`` and ``fk_back_right`` are where we'll implement the forward kinematics of each leg.
4. A leg whose method still returns ``None`` is simply skipped, so you can implement and check the legs one at a time.
5. The code uses NumPy for matrix operations.
6. Note that it is convention to orient the coordinate frame so that the rotation about each motor is the z axis.

Part 4: Implementing Forward Kinematics
------------------------------------------

For the following steps, you can view the Pupper CAD to help you understand the kinematic chain `CAD <https://cad.onshape.com/documents/97a1bc3e752ec66822dbb5bb/w/c7f9232ccbc53a2e3f6ee909/e/74c0b3caf828b9fd1994bcd6?renderMode=0&uiState=67f1c37599fde447b364a89c>`_

Step 1: Implement Rotation Matrices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Open ``forward_kinematics.py`` and locate the ``rotation_y``, ``rotation_z`` and ``translation`` methods (``rotation_x`` is done for you).

2. Implement the rotation matrices about the x, y, and z axes. Follow the homogeneous coordinates representation as presented in lecture.

**DELIVERABLE:** Which axis is typically used as the default axis for rotations in robotic systems? What angles are we rotating along the default axis? Why?

Step 2: Implement Transformation Matrices (Front-Left Leg)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Start with the front-left leg in ``fk_front_left``.


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

We recommend jumping to Part 5 now to check the front-left leg in the viewer before moving on to the other three legs.

Step 3: Extend to the Other Three Legs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Now implement ``fk_front_right``, ``fk_back_left`` and ``fk_back_right``. Each leg is the same
chain as the front-left leg (``T_0_1``, ``T_1_2``, ``T_2_3``, ``T_3_ee``, then ``T_0_ee``),
with two differences:

- ``T_0_1`` starts from a different hip position on the body (we provide the numbers you need as comments in the code).
- The right legs are mirror images of the left legs, and mirroring changes the sign of some of the terms.

We provide the base link to ``leg_back_r_1`` transformation in the diagram below. The rest of
the back-right chain follows the same pattern as the front leg:

   .. figure:: ../../../_static/kinematics/base_back_kinematics.png
      :align: center
      :width: 75%

      Base to back right leg transformation diagram

For the front-right and back-left legs, no diagram is given on purpose: work them out from
the front-left and back-right diagrams and the `CAD <https://cad.onshape.com/documents/97a1bc3e752ec66822dbb5bb/w/c7f9232ccbc53a2e3f6ee909/e/74c0b3caf828b9fd1994bcd6?renderMode=0&uiState=67f1c37599fde447b364a89c>`_.
The viewer in Part 5 tells you immediately whether a leg is right: move one joint at a time
by hand and check that the sphere stays on the foot. If you are truly stuck after trying, ask a
TA.

**DELIVERABLE:** For each of the three additional legs, state which terms change compared to the front-left leg and why.

Part 5: Checking Your Implementation in the Viewer
---------------------------------------------------

1. Save your changes to ``forward_kinematics.py``.

2. Run the ROS2 nodes. This also starts the 3D web viewer (viser) on the Pupper:

   .. code-block:: bash

      ros2 launch forward_kinematics.launch.py

3. In another terminal, use the following command to run the main code:

   .. code-block:: bash

      python forward_kinematics.py

4. Open the viewer in your browser. Both commands above print the exact link when they start. If your laptop is on the same Wi-Fi as the Pupper, browse to ``http://<pupper-ip>:8080`` (several people can have it open at once). If you can only reach the Pupper over SSH, forward the port from your laptop first and browse to ``http://localhost:8080``:

   .. code-block:: bash

      ssh -N -L 8080:localhost:8080 pi@pupper[YOUR_GROUP_NUMBER].local

   If you have trouble reaching the Pupper, see :doc:`ssh-over-wifi`.

5. The viewer shows the Pupper model following the real joint angles, plus one sphere per leg at the position your forward kinematics computed: front-left **green**, front-right **red**, back-left **blue**, back-right **yellow**. The side panel lists the joint angles and end-effector positions. A leg whose method still returns ``None`` has no sphere yet.

6. Move each leg of your robot by hand (the motors are held limp for you) and check that its sphere stays on the foot. If a sphere drifts off the foot when you move one particular joint, that joint's transform (or the sign of its angle) is the one to fix.

.. note::
   **No Wi-Fi connection?** If your laptop cannot reach the Pupper's web page (weak or no Wi-Fi in your area, or the ``ssh -L`` route above fails), fall back to RViz2, which draws the same robot model and the same four markers:

   .. code-block:: bash

      rviz2 -d forward_kinematics.rviz

   The above command will load the RViz config file. If you just run ``rviz2``, you can manually add the configuration: click "Add", select a Robot Model type and the ``/robot_description`` topic; then "Add" again, select a Marker type and the ``/marker`` topic. RViz2 needs a display (the monitor setup from Lab 1 or X forwarding) and may fail on your robot due to heating in the Raspberry Pi. If this happens, reach out to a TA to check the implementation first, then turn off Pupper, wait a while to let it cool down, and try again.

**DELIVERABLE:** 

1. Take a video of the working implementation with you moving each of Pupper's four legs and the viewer mimicking the results, and upload it to the Google Drive

2. Write out the full equation you used to calculate the forward kinematics (in math). Please use LaTeX and take a screenshot, or use the equation functionality in Google Docs. What is the benefit of using homogeneous transformations? 

3. Why is there a 1 in the bottom-right corner of a homogeneous transformation matrix?

Part 6: Analyzing the Results
--------------------------------

1. Record the end-effector positions for the left front leg configurations (the side panel of the viewer shows them, or run ``ros2 topic echo /leg_front_l_end_effector_position``).

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

1. While testing in the viewer, write a script that saves the sequence of your well-crafted motion, recorded as end effector positions into a file. You will have a chance to let Pupper replay this recorded motion in the next lab! You will need to use the ``joint_states`` topic to record the motor angles, and the ``leg_<leg>_end_effector_position`` topics to record the end effector positions.

2. Keep your four-leg forward kinematics handy: lab 3 builds on it to make Pupper walk.

Friendly reminder: The first optional lab will be released next week, attempt at your own risk!

Remember, understanding forward kinematics is crucial for robot control and motion planning. Take your time to ensure you understand each step of the process!

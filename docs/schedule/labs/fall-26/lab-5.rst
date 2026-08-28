Lab 5: Follow Me Pupper
========================

*Goal: give Pupper eyes — detect objects onboard, follow a person with a state
machine, and step around whatever is in the way.*

.. TODO(staff): add this offering's lab slides link
.. TODO(staff): add this offering's lab document link
.. TODO(staff): capture fresh figures on a robot: the viser viewer with boxes +
   masks, and a detour sequence. Old Foxglove screenshots no longer apply.

`Lab slides <#>`_ (TODO)

`Lab document <#>`_ (TODO)

Everything in this lab runs *on the robot*: YOLOv8n-seg on Pupper's Hailo AI
accelerator turns camera frames into detections, your state machine turns
detections into motion, and a browser viewer shows you what Pupper sees while
it happens. No API keys, no cloud, no voice — the tracking API you build here
(``begin_tracking()`` / ``end_tracking()``) is exactly what the robot
foundation model lab will drive with language later in the quarter.

The intellectual core of the lab is a single, lovely idea: **your robot's
camera has no depth sensor, and yet it can tell how far away things are** —
because everything stands on the same floor, and the floor recedes upward in
the image. The bottom edge of a bounding box *is* a distance sensor. You will
build all of the obstacle avoidance out of that one observation.

Step 0. Setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Fork the `follow_me_pupper <https://github.com/cs123-stanford/follow_me_pupper>`_
   repository to your own GitHub account, following
   :doc:`forking_repositories`.

2. Clone your fork onto the Pupper and fetch the detection model:

.. code-block:: bash

   cd ~/
   git clone https://github.com/YOUR_USERNAME/follow_me_pupper.git
   cd follow_me_pupper
   ./scripts/download_model.sh      # yolov8n_seg.hef, ~11 MB

3. Skim the README — especially the "How the pieces talk" diagram. Three
   programs cooperate over ROS topics: ``viser_camera.py`` (camera → YOLO →
   ``/detections``), ``follow_me.py`` (your state machine, ``/detections`` →
   ``/cmd_vel``), and ``karel.py`` (the API that switches tracking on and
   off over ``/tracking_control``).

Step 1. See What Pupper Sees
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The camera stack and viewer are given — get them running before writing any
code.

1. Launch the robot stack (motors, camera, detector, viewer):

.. code-block:: bash

   ros2 launch follow_me.launch.py

2. Open the viewer. On the same Wi-Fi, browse to ``http://<pupper-ip>:8080``.
   Over SSH, forward the port from your laptop first and browse to
   ``localhost:8080``:

.. code-block:: bash

   ssh -N -L 8080:localhost:8080 pi@pupper[GROUP_NUMBER].local

3. You should see the camera feed with bounding boxes *and* segmentation
   masks around detected objects — 80 COCO classes' worth. Two GUI settings
   matter when the Wi-Fi is weak: **Lock aspect ratio** and **Drop frames
   when behind** (both on by default; the README explains what each does).

4. Notice the **View** dropdown: the raw image is a fisheye, but detection
   always runs on the *equirectangular* (undistorted) view. The fisheye lens
   bends people into shapes YOLO has never seen — scores drop badly on the
   raw image. ``fisheye_converter.py`` holds the math if you are curious
   (totally optional).

**DELIVERABLE:** A screenshot of the viewer showing detections with bounding
boxes and masks. Upload to Gradescope.

**DELIVERABLE:** In a sentence or two: why do we run the detector on the
undistorted image instead of the raw fisheye?

Step 2. The Geometry of a Flat World
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Before the state machine, build its vocabulary. Open ``avoidance.py`` and
read the module docstring — it explains the floor-geometry idea everything
else rests on. Then implement the four geometry primitives:

- ``box_bottom`` — where a box meets the floor, in pixels;
- ``box_offset`` — how far off-centre a box is (the steering error signal);
- ``box_left_fraction`` — how a box splits across the image's two halves;
- ``obstacle_weight`` — nearer things count for more.

Detections arrive in a fixed 700×572 reference frame whatever resolution the
camera runs at, so pixel thresholds always mean the same thing.

You do not need the robot for any of this. The repo ships a desk-side test
harness — run it after every function:

.. code-block:: bash

   python3 test_avoidance.py

**DELIVERABLE:** Explain the distance-from-bottom-edge trick in your own
words: *why* does a lower bottom edge mean a nearer object? Then give two
concrete situations where this assumption lies to the robot, and what it
would do wrong in each.

Step 3. The Tracking State Machine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Now make Pupper follow things. ``follow_me.py`` is a ROS 2 node with a state
machine; the core states are:

- **IDLE** — tracking off; stand still (and stay quiet on ``/cmd_vel``, so
  manual Karel commands still work);
- **SEARCH** — target lost; rotate to find it;
- **TRACK** — target in view; steer toward it and walk.

Work through the marked sections:

1. **Detection processing** (``detection_callback``): split the incoming
   ``Detection2DArray`` into your target class and everything else, pick
   *the* target (``pick_target`` — most centred wins, for now), store its
   offset, and stamp the time. The
   `message documentation <http://docs.ros.org/en/kinetic/api/vision_msgs/html/msg/Detection2DArray.html>`_
   has the structure.
2. **Transitions** (``timer_callback``): when is a detection *stale*? What
   should ``TIMEOUT`` seconds of silence mean?
3. **Behaviors**: motion commands per state. TRACK is proportional control —
   steer against the offset with gain ``KP``. For SEARCH, think about *which
   way* to spin: where was the target last seen?
4. **Constants**: pick and tune ``TIMEOUT``, ``SEARCH_YAW_VEL``,
   ``TRACK_FORWARD_VEL``, ``KP``.

Test it. In two more terminals:

.. code-block:: bash

   python3 follow_me.py             # terminal 2: your state machine
   python3 test_tracking.py         # terminal 3: choose what to track

(or start tracking immediately with
``python3 follow_me.py --ros-args -p target:=person``; the ``KarelPupper``
API — ``begin_tracking("person")``, ``end_tracking()`` — is what
``test_tracking.py`` uses under the hood, and what the foundation-model lab
will call later. ``scripts/run_tracking.sh`` runs all three terminals at once
for quick demos.)

.. note::

   **Debugging with pdb**: add ``breakpoint()`` inside ``detection_callback``
   or ``timer_callback`` and run ``python3 follow_me.py`` to inspect what is
   actually arriving: what does ``msg.detections`` contain? Is
   ``self.target_pos`` sensible? Are transitions firing when you expect?
   (``p variable`` prints, ``n`` steps, ``c`` continues.)

**DELIVERABLE:** A state machine diagram of IDLE / SEARCH / TRACK with every
transition condition labeled. Upload to Gradescope.

**DELIVERABLE:** A video of Pupper tracking a person, showing both search and
track behavior. Upload to Gradescope.

**DELIVERABLE:** A video with *two* people in frame. Which one does your
robot follow, and why? (Notice anything unsatisfying? Hold that thought for
the optional part.)

Step 4. Something in the Way
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Put a chair between Pupper and the person: the TRACK controller walks
straight into it, because nothing it computes knows the chair exists. Fixing
that is the second half of the lab, and it has two parts — the *decisions*
(pure functions in ``avoidance.py``, desk-testable) and the *detour* (three
new FSM states in ``follow_me.py``).

**The decisions**, in ``avoidance.py``:

1. ``measure_crowding`` — how much stuff is in each half of the frame,
   weighted by nearness. This is measured on *every* frame, target or not,
   because it decides which way a detour should go: walking around one chair
   into another is no better than standing still.
2. ``find_blocking_obstacle`` — is something actually *in the way*? Three
   tests: between us and the target (bottom edge below the target's), close
   (within ``OBSTACLE_BOTTOM_MARGIN`` of the image bottom), and roughly
   ahead (inside ``OBSTACLE_CENTER_BAND``). Nearest survivor wins.
3. ``choose_side`` — which way around, with a tie-break cascade: emptier
   half of the frame, else away from the obstacle's side, else toward the
   target.

Run ``python3 test_avoidance.py`` until all 16 scenes pass — every one of
them is a bug you did not have to debug on a moving robot.

**The detour**, in ``follow_me.py`` — three timed states:

- **AVOID_TURN** — turn off the direct line, direction from ``choose_side``;
- **AVOID_PASS** — walk straight, past the obstacle;
- **AVOID_RETURN** — turn back for the same time at the same speed, which
  restores the original heading, displaced sideways: that displacement *is*
  the detour.

Wire them in: ``start_avoiding`` commits to a side, ``advance_avoidance``
steps the phase clock, ``ready_to_avoid`` gates when a detour may begin —
including a cooldown after each one. Detections keep arriving during the
detour, so the chase resumes seamlessly when it ends.

**DELIVERABLE:** The output of ``test_avoidance.py`` with all scenes passing.

**DELIVERABLE:** Your full six-state machine diagram, transitions labeled.

**DELIVERABLE:** A video of Pupper walking toward you, detouring around an
obstacle placed in its path, and resuming the chase.

**DELIVERABLE:** Two design questions, a short paragraph each: (a) The detour
is *open-loop* — during AVOID_PASS the robot does not re-check the obstacle.
We chose that deliberately: the onboard detector is unreliable exactly when an
object is very close and fills the frame. What would happen to a closed-loop
detour built on a detector with that failure mode? (b) Why is the cooldown in
``ready_to_avoid`` necessary — what does the robot do without it, right after
a detour ends?

Step 5 (Optional). Follow *Me*, Specifically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Your Step 3 two-person video exposed the weakness: "track the most centred
person" happily hops between people. The fix uses machinery you already have
on screen — the segmentation masks.

Every detection is published with the **mean color of its mask** (in the
message's ``id`` field, as ``"r,g,b"``) — for a person, that is mostly the
color of their clothes. In ``follow_me.py``, set ``ENABLE_COLOR_REID = True``
and implement the marked optional section of ``pick_target``: memorize the
target's color when the chase starts, prefer the candidate nearest that color
(within ``REID_MAX_DIST``, so a frame containing only strangers does not
steal the lock), and blend the memory slowly toward the current match so
gradual lighting changes do not shake it off.

**DELIVERABLE (optional):** The two-person video again — but now the robot
stays locked on the same person as they cross paths. Then break it on
purpose: what happens when both people wear the same color, and why is that
exactly what your algorithm predicts?

Congratulations — Pupper now sees. It finds a person, follows them across a
room, steps around furniture using nothing but bounding-box geometry, and
(optionally) knows *which* person is yours. In the robot foundation model
lab, a language model will drive the very ``begin_tracking()`` API you tested
today — "follow that person" is about to become a sentence.

Resources
-----------
`You Only Look Once: Unified, Real-Time Object Detection <https://arxiv.org/abs/1506.02640>`_

`Microsoft COCO: Common Objects in Context <https://arxiv.org/abs/1405.0312>`_

`The Double Sphere Camera Model <https://arxiv.org/abs/1807.08957>`_ — the
fisheye model behind ``fisheye_converter.py``

`Hailo-8 AI accelerator <https://hailo.ai/products/ai-accelerators/hailo-8l-ai-accelerator-for-ai-light-applications/>`_ — the chip the detector runs on

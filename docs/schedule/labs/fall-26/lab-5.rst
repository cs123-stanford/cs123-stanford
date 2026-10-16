Lab 5: Follow Me Pupper
========================

*Goal: give Pupper eyes — detect objects onboard, follow a person with a state
machine, and step around whatever is in the way.*

.. TODO(staff): add this offering's lab slides link
.. TODO(staff): add this offering's lab document link
.. TODO(staff): capture a detour sequence figure on a robot. Old Foxglove
   screenshots no longer apply.

`Lab slides <#>`_ (TODO)

`Lab document <#>`_ (TODO)

Everything in this lab runs *on the robot*: YOLOv8n-seg on Pupper's Hailo AI
accelerator turns camera frames into detections, your state machine turns
detections into motion, and a browser viewer shows you what Pupper sees while
it happens. No API keys, no cloud, no voice — the tracking API you build here
(``begin_tracking()`` / ``end_tracking()``) is exactly the kind of skill the
robot foundation model lab will drive with language later in the quarter.

The intellectual core of the lab is a single, lovely idea: **your robot's
camera has no depth sensor, and yet it can tell how far away things are** —
because everything stands on the same floor, and the floor recedes upward in
the image. The bottom edge of a bounding box *is* a distance sensor. You will
build all of the obstacle avoidance out of that one observation.

.. admonition:: AI Use Suggestion
   :class: note

   **This lab is designed to be completed without a coding agent.** Every
   TODO is a few lines, and the desk-side tests tell you exactly what each one
   must do. That said, you are free to use a coding agent (Claude Code or
   Codex) for the coding part, on two conditions:

   * **Understand the high-level ideas** — why the bottom edge of a box is a
     distance sensor, how the state machine moves between states, how the
     detour picks a side, and how the target lock tells people apart — well
     enough to explain them without your agent open.
   * **Understand every line your agent writes**: what it does and why it is
     there. If you cannot explain a line, ask the agent to explain it until
     you can.

   Answer every DELIVERABLE in your own words, from what you did and saw on
   your robot.

   .. important::

      **Quiz 2 tests this.** Some implementation details of this lab — what a
      function returns, why a check is written the way it is, what the robot
      does when a check is missing — will be on Quiz 2.

.. list-table:: Lab roadmap
   :header-rows: 1
   :widths: 10 50 40

   * - Step
     - What you do
     - Code and tests
   * - 0
     - **Setup:** fork, clone, download the model
     - (reading)
   * - 1
     - **See:** run the camera, detector and viewer
     - (nothing to write)
   * - 2
     - **Geometry:** the four bounding-box primitives
     - ``lab/avoidance.py`` · ``lab/test_avoidance.py geometry``
   * - 3
     - **Track:** the state machine, and stopping at a polite distance
     - ``lab/follow_me.py``, ``lab/target_lock.py`` (``STOP_BOTTOM``,
       ``approach_speed``) · ``lab/test_target_lock.py distance``
   * - 4
     - **Avoid:** decide when something is in the way, and detour around it
     - ``lab/avoidance.py``, ``lab/follow_me.py`` · ``lab/test_avoidance.py avoidance``
   * - 5
     - **Lock on:** follow *your* person, not whoever is most centred
     - ``lab/target_lock.py`` · ``lab/test_target_lock.py lock``

**The tests: what they check.** Two desk-side scripts run your code on fake
detections — no robot, no ROS. Each takes a section name, so you can test one
step at a time (``python3 lab/test_avoidance.py geometry``); with no name, it runs
everything. A check whose code you have not written yet is skipped (``--``),
not failed, so run them after every function. A failing check prints what your
code returned, what was wanted, and an example that passes. Each step below
ends with a table of *every* check its tests make: nothing is checked that is
not in the table. The state machine in ``follow_me.py`` has no desk tests — the
robot is its test.

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
   ./scripts/download_model.sh      # robot/yolov8n_seg.hef, ~11 MB

3. Skim the README — especially the "How the pieces talk" diagram. Everything
   you write is in ``lab/``: ``follow_me.py``, ``avoidance.py``,
   ``target_lock.py``, and the desk tests for them. ``robot/`` is provided
   (camera, detector, viewer, launch file, Karel API): read it, don't rewrite
   it. Run every command from the repo root. Three programs cooperate over ROS
   topics: ``robot/viser_camera.py`` (camera → YOLO → ``/detections``),
   ``lab/follow_me.py`` (your state machine, ``/detections`` →
   ``/cmd_vel``), and ``robot/karel.py`` (the API that switches tracking on
   and off over ``/tracking_control``).

   Every function you write has a card at the end of its docstring: what each
   input is, what it should return (with an example), what calls it, and the
   test to run.

Step 1. See What Pupper Sees
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The camera stack and viewer are given — get them running before writing any
code.

1. Launch the robot stack (motors, camera, detector, viewer):

.. code-block:: bash

   ros2 launch robot/follow_me.launch.py

2. Open the viewer. On the same Wi-Fi, browse to ``http://<pupper-ip>:8080``.
   Over SSH, forward the port from your laptop first and browse to
   ``localhost:8080``:

.. code-block:: bash

   ssh -N -L 8080:localhost:8080 pi@pupper[YOUR_GROUP_NUMBER].local

3. You should see the camera feed with bounding boxes *and* segmentation
   masks around detected objects — 80 COCO classes' worth. Two GUI settings
   matter when the Wi-Fi is weak: **Lock aspect ratio** and **Drop frames
   when behind** (both on by default; the README explains what each does).

   .. figure:: ../../../_static/lab5/pupper_viser.png
      :align: center

      The viser viewer on the equirectangular view: a person, a chair, a TV
      and a bed, each with a bounding box, confidence score, and
      segmentation mask.

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

   python3 lab/test_avoidance.py geometry

**What** ``python3 lab/test_avoidance.py geometry`` **checks.** Every check, with
an example that passes and a common way to fail it.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``box_bottom`` is the bottom edge
     - a box from y=200 to y=500 gives ``500``
     - ``350`` (the centre), or ``200`` (the top: image y grows *downward*)
   * - ``box_offset`` of a centred box is 0, and of a box at the right edge
       about +0.49
     - centre x=690 of 700 gives ``690 / 700 - 0.5 = 0.486``
     - ``0.986`` (forgot the ``- 0.5``), or ``340`` (not normalized)
   * - ``box_left_fraction`` is 1 for a box fully left of centre, and 0.25
       for a box with 30 of its 120 px left of x=350
     - ``1.0`` and ``0.25``
     - ``1.0`` for the straddling box (judged by its centre alone)
   * - ``obstacle_weight`` is ``SIDE_WEIGHT_NEAR`` at the image bottom, and a
       nearer object weighs more than a farther one
     - bottom 572 gives ``2.0``; bottom 30 gives about ``0.58``
     - weights that grow toward the *top* of the image

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

Here is the whole machine — keep it in front of you while you fill in the
transitions:

.. figure:: ../../../_static/lab5/state_machine_core.svg
   :align: center
   :width: 90%

   The core tracking state machine. Every arrow is a condition your
   ``timer_callback`` checks; every box is a motion command it publishes.

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
4. **Keeping your distance** (``approach_speed`` in ``target_lock.py``): a
   robot that walks at a constant speed walks straight into your shins.
   The bottom edge of the target's box is a distance sensor (Step 2), so
   steer *that* too. First, measure where Pupper should stop:

   a. With the launch running, open the viewer and stand in front of Pupper
      at the distance you want it to keep (close, but not on top of it —
      your feet should still be in frame).
   b. Save the **Live feed** image from the viewer's side panel (it is the
      equirectangular frame the detector sees, without padding).
   c. In the image, find the y of your box's bottom edge, in pixels, and
      scale it to the 572 px reference frame:
      ``STOP_BOTTOM = y_bottom / image_height * 572``. Put it into
      ``target_lock.py``.

   ``approach_speed`` is then a second proportional controller on the error
   to ``STOP_BOTTOM`` — walk in when the person is far, stand still inside
   ``STOP_TOLERANCE``, step back when they come too close. Check it with
   ``python3 lab/test_target_lock.py distance`` (table below).
5. **Constants**: pick and tune ``TIMEOUT``, ``SEARCH_YAW_VEL``,
   ``TRACK_FORWARD_VEL``, ``KP``.

**What** ``python3 lab/test_target_lock.py distance`` **checks.** The examples
use ``KP_DISTANCE = 1.5``, ``STOP_TOLERANCE = 15``, ``MAX_BACKUP_VEL = 0.2``
and a cap of 0.3 m/s. Every speed passes within ±0.01 m/s of the expected
one, so rounding or an equivalent formula never fails a check.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``STOP_BOTTOM`` is measured: a bottom edge inside the 572 px frame
     - the value you measured from your Live feed image, a number of pixels
       between 0 and 572
     - ``None`` (skipped until you measure it), or the raw pixel y from a
       screenshot of a different height, not scaled to 572
   * - At ``STOP_BOTTOM``, and halfway to the edge of ``STOP_TOLERANCE``:
       stand still
     - ``0.0``
     - a tiny speed like ``0.02`` (no dead band: Pupper shuffles in place)
   * - Far away: walk in at the cap
     - 200 px short gives ``0.3`` (1.5 × 200 / 572 = 0.52, capped)
     - ``0.52`` (not capped at ``max_forward``)
   * - A little far: proportional, ``KP_DISTANCE * error / 572``
     - 40 px short gives ``0.105``
     - ``0.3`` (full speed whenever it is far: not proportional)
   * - Too close: back away
     - 60 px past gives ``-0.157``
     - ``0.0`` or a positive speed (the sign of the error is flipped)
   * - Right on top of us: back away no faster than ``MAX_BACKUP_VEL``
     - bottom 572 gives ``-0.2``
     - ``-0.31`` (not clamped backward)

Test it. In two more terminals:

.. code-block:: bash

   python3 lab/follow_me.py             # terminal 2: your state machine
   python3 robot/test_tracking.py       # terminal 3: choose what to track

(or start tracking immediately with
``python3 lab/follow_me.py --ros-args -p target:=person``; the ``KarelPupper``
API — ``begin_tracking("person")``, ``end_tracking()`` — is what
``robot/test_tracking.py`` uses under the hood, and the shape of the follow tool the
foundation-model lab will give a language model. ``scripts/run_tracking.sh`` runs all three terminals at once
for quick demos.)

.. note::

   **Debugging with pdb**: add ``breakpoint()`` inside ``detection_callback``
   or ``timer_callback`` and run ``python3 lab/follow_me.py`` to inspect what is
   actually arriving: what does ``msg.detections`` contain? Is
   ``self.target_pos`` sensible? Are transitions firing when you expect?
   (``p variable`` prints, ``n`` steps, ``c`` continues.)

**DELIVERABLE:** A video of Pupper tracking a person, showing both search and
track behavior — including it stopping at the right distance, and backing
off when you step toward it — plus the Live feed image you measured
``STOP_BOTTOM`` from and the value you got. Upload to Gradescope.

**DELIVERABLE:** A video with *two* people in frame. Which one does your
robot follow, and why? (Notice anything unsatisfying? Hold that thought for
Step 5.)

Step 4. Something in the Way
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Put a chair between Pupper and the person: the TRACK controller walks
straight into it, because nothing it computes knows the chair exists. Fixing
that is the second half of the lab, and it has two parts — the *decisions*
(pure functions in ``avoidance.py``, desk-testable) and the *detour* (three
new FSM states in ``follow_me.py``).

The whole scheme in one picture — the left panel is the *decision* (made in
image space, using nothing but bottom edges), the right panel is the *detour*
(made in the world, using nothing but a clock):

.. figure:: ../../../_static/lab5/obstacle_avoidance.svg
   :align: center

   Obstacle avoidance from one observation: bottom edge = distance. An
   obstacle *blocks* when it passes all three tests on the left; the robot
   then sidesteps it with three timed states and resumes the chase.

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

Run ``python3 lab/test_avoidance.py avoidance`` until every check passes — each
one is a bug you did not have to debug on a moving robot.

**What** ``python3 lab/test_avoidance.py avoidance`` **checks.** The scene: the
person you chase stands mid-frame with their bottom edge at y=272.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``measure_crowding``: a near chair on the left outweighs a far one on
       the right
     - ``(left, right) = (1.95, 0.92)``
     - left and right swapped, or every box weighted the same
   * - ``find_blocking_obstacle``: an object *beyond* the target (bottom
       edge 222, above the target's 272) does not block
     - ``None``
     - its offset (test 1 missing: anything centred counts as in the way)
   * - ... an object between us and the target but still far (280 px from
       the image bottom, more than ``OBSTACLE_BOTTOM_MARGIN``) does not block
     - ``None``
     - its offset (test 2 missing, or measured from the top of the image)
   * - ... a near object off to a side (offset −0.37, outside
       ``OBSTACLE_CENTER_BAND``) does not block
     - ``None``
     - ``-0.37`` (test 3 missing, or compared without ``abs``)
   * - ... a near, centred object between us and the target blocks
     - ``-0.014``, the blocker's ``box_offset``
     - ``None``, or the detection itself instead of its offset
   * - ... of two blockers, the nearest wins
     - ``+0.029`` (bottom 532 beats bottom 472)
     - ``-0.014`` (the first one found, or the farther one)
   * - ``choose_side``: crowded right → left (``+1``); crowded left → right
       (``-1``)
     - ``(1.0, 'crowding L 0.50 / R 3.00')``
     - the signs swapped (``+1`` is *left*, as for yaw)
   * - ... even sides, obstacle right of centre → left
     - ``1.0``
     - ``-1.0`` (stepping toward the obstacle's side)
   * - ... all even, target right → right, the shorter way back
     - ``-1.0``
     - ``1.0``

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

.. warning::

   Let's be frank: this obstacle avoidance is not very good. It is a stack of
   hand-tuned heuristics, and on the real robot it will sometimes miss an
   obstacle, detour around nothing, or pick the wrong side. Do not expect it
   to work every time — that is not a bug in your code, and you will be asked
   to explain why below. One clean run is all the video deliverable needs.

**DELIVERABLE:** The output of ``test_avoidance.py`` with all scenes passing.

**DELIVERABLE:** We drew the three-state machine for you in Step 3 — now it
is your turn. Draw the full six-state machine (IDLE, SEARCH, TRACK,
AVOID_TURN, AVOID_PASS, AVOID_RETURN) with every transition condition
labeled. Upload to Gradescope.

**DELIVERABLE:** A video of Pupper walking toward you, detouring around one
obstacle placed in its path, and resuming the chase. It does not have to be
reliable: one successful run is enough.

**DELIVERABLE:** Three design questions, a short paragraph each: (a) The detour
is *open-loop* — during AVOID_PASS the robot does not re-check the obstacle.
We chose that deliberately: the onboard detector is unreliable exactly when an
object is very close and fills the frame. What would happen to a closed-loop
detour built on a detector with that failure mode? (b) Why is the cooldown in
``ready_to_avoid`` necessary — what does the robot do without it, right after
a detour ends? (c) You probably saw the avoidance fail a few times while
filming. Why is this heuristic approach so fragile? Look at what every
function in ``avoidance.py`` takes as input, then think about where those
inputs come from on *this* robot: a camera a few centimetres off the floor,
on a body that is walking. What would a sturdier approach need?

Step 5. Follow *Me*, Specifically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Your Step 3 two-person video exposed the weakness: "track the most centred
person" happily hops between people. Time to fix it.

The constraint that makes it interesting: no new sensors, no new models. To
YOLO, every person is just the class ``person`` — a bounding box cannot tell
your person from a stranger. But two things still can:

- **Where they were.** Detections arrive about ten times a second, and
  nobody teleports between frames. The person nearest to where your target
  just was is almost certainly your target — so search *there* first.
- **What they look like.** A box says *where* a person is; the segmentation
  mask says which pixels *are* the person — and a person's pixels are mostly
  their clothes. Every detection ships the **mean color of its mask** in
  the message's ``id`` field, as ``"r,g,b"``. When position cannot decide —
  two people crossing paths, or your target vanishing and reappearing
  somewhere else — the color does.

Build it in ``target_lock.py``: ``position_distance`` and ``color_distance``
measure the two cues, ``pick_target`` combines them (its TODO lays out the
order), and ``remember`` updates what the robot knows about its target after
every frame. ``follow_me.py`` already calls them (until ``pick_target`` is
written, it falls back to the most centred person). Run
``python3 lab/test_target_lock.py lock`` until every check passes.

**What** ``python3 lab/test_target_lock.py lock`` **checks.** The people wear red
(200, 40, 40), blue (40, 60, 200), or "reddish" (185, 55, 50): the red shirt
under slightly different light. The lock was just following a red person at
the centre of the frame, bottom edge at y=400. Distances pass with a small
margin: ±0.005 for positions, ±1 (on the 0–255 scale) for colors.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``position_distance`` is 0 to a detection's own position, and in image
       fractions (70 px sideways = 0.1)
     - ``0.1``
     - ``70.0`` (pixels, not fractions of the image)
   * - ``color_distance`` of the same shirt is 0; with no color on the
       detection, or nothing remembered, it is ``None``
     - ``None`` for ``id == ""``
     - ``0.0`` or a crash when a color is missing (missing is *no evidence*,
       not a perfect match)
   * - ``remember`` stores the color on first sight, and the position
       (``box_position``)
     - ``lock.color == [200, 40, 40]``
     - ``lock.color`` still ``None``
   * - ``remember`` after that blends the color by ``REID_BLEND``
     - 0.9 × red + 0.1 × reddish = ``[198.5, 41.5, 41]``
     - ``[185, 55, 50]`` (replaced, not blended), or the weights swapped
   * - ``pick_target``, new chase: the most centred person
     - the person at x=370, not the one at x=200
     - the first in the list
   * - ... nearest to the last position wins, even over a better color
       match far away
     - the reddish person 15 px from where we were
     - the red person at x=100 (color checked before position)
   * - ... a stranger alone in our spot does not steal the lock
     - ``None`` (blue is too far from red)
     - the stranger (position trusted without the color veto)
   * - ... a frame of only strangers has no target
     - ``None``
     - the stranger (falling back to the most centred)
   * - ... two people crossing, both nearby: the color decides
     - the reddish one
     - the blue one (nearest position wins even when several are nearby)
   * - ... a stranger in our spot and us elsewhere: re-acquire us by color
     - the reddish person at x=120
     - ``None`` (gave up after the color veto instead of looking further)
   * - ... after a search (no position remembered): re-acquire by color
       anywhere
     - the reddish person at x=600
     - the blue one at x=300 (most centred)
   * - ... with no masks (no colors at all): nearest position decides
     - the person 10 px away, not the one 40 px away
     - ``None`` (a missing color treated as a mismatch)

Some questions to settle before you write code: what does the robot do when
it lost you and spun around — is "where you were" still worth anything? A
frame contains only strangers: what stops one of them from stealing the lock?
And the lighting changes as you walk from the window to the hallway: how does
the remembered color keep up without forgetting who you are?

**DELIVERABLE:** Your design, before your code: explain how position and the
segmentation masks together tell one person from another when bounding boxes
alone cannot, and how ``pick_target`` decides — including what stops a frame
containing only strangers from stealing the lock.

**DELIVERABLE:** The output of ``test_target_lock.py`` with all scenes
passing.

**DELIVERABLE:** The two-person video again — but now the robot stays locked
on the same person as they cross paths. Then break it on purpose: find a
situation where your approach *must* fail, show it failing, and explain why
that failure is exactly what your algorithm predicts.

Congratulations — Pupper now sees. It finds a person, follows them across a
room, steps around furniture using nothing but bounding-box geometry, stops at a
polite distance, and knows *which* person is yours. In the robot foundation model
lab, a language model will drive a follow skill just like the
``begin_tracking()`` API you tested today — "follow that person" is about to
become a sentence.

Resources
-----------
`You Only Look Once: Unified, Real-Time Object Detection <https://arxiv.org/abs/1506.02640>`_

`Microsoft COCO: Common Objects in Context <https://arxiv.org/abs/1405.0312>`_

`The Double Sphere Camera Model <https://arxiv.org/abs/1807.08957>`_ — the
fisheye model behind ``fisheye_converter.py``

`Hailo-8 AI accelerator <https://hailo.ai/products/ai-accelerators/hailo-8l-ai-accelerator-for-ai-light-applications/>`_ — the chip the detector runs on

Optional Lab 2: Reference Bootstrapping and Multi-Gait Distillation
====================================================================

Goal
----
Make Pupper *fast*. The ``Mjlab-MixedGaits-Flat-Pupper-v3`` task commands
velocities out to **±1.5 m/s** — more than twice what your lab 4 policies were
asked for — and blends three reference slots by command: the trot for ordinary
fore/aft walking, your lift gait for turning and sidestepping, and a **fast
branch** that engages above a speed threshold. Out of the box, the fast branch
plays... the trot. It is not good at 1.5 m/s. Finding a reference that beats
it — first by designing one, then by *bootstrapping better references out of
your own trained policies* — is this lab. The endpoint is a single velocity-
and reference-conditioned policy that covers the entire command range and runs
on your robot: the same species of policy your Pupper shipped with, except
yours is faster.

(Why 1.5? In our stress testing, everything beyond ±1.5 m/s made sim2real
fall apart — policies that scream in simulation and faceplant on carpet — so
the task caps commands there. The cap is a config value, not a law of nature.
If you believe your gait can beat it, raise it. Bring a spare set of legs.)

Prerequisites
-------------
Lab 4, fully — this lab assumes the StableGait sections are muscle memory:
you know what the 12-dim reference observation is, what ``gait_tracking``
pays for, and how a run's curves read. Unlike lab 4, nothing here happens in
a curated notebook. You will work *in the codebase*:

* **Fork and clone** `pupper-mjlab <https://github.com/cs123-stanford/pupper-mjlab>`_
  — you need your own editable copy; you will be committing reference tables,
  capture scripts, and config changes to it.
* Drive the repo with a coding agent — **Claude Code (preferably Opus or
  Fable) or Codex**. The codebase is the documentation; the agent is your
  trail guide. Asking it to *explain* machinery before you modify it is the
  intended workflow, and honestly the point.
* A Colab Pro subscription (or your own NVIDIA GPU). Budget realistically:
  each bootstrap round is a full training run, so a serious pass at this lab
  is **4–6+ G4 runs** plus evaluation time.

Part 0: Spelunking
------------------
Before changing anything, know the machinery. In your clone, work through
``src/mjlab/tasks/pupper_gait/mdp/gait_reference.py`` end to end — with your
agent, and with the lab 4 mental model as your anchor. Three things to hunt
down:

1. **The parametric pipeline.** How do a touchdown tuple, a stride, a stance
   depth, and a swing lift become a ``(n_samples, 12)`` joint-angle table?
   Where do the keyframes, the IK, and the joint-limit clamp live?
2. **The capture sockets.** The loader knows how to consume captured gait
   tables from ``.npz`` files — files which the public repo *does not ship*.
   Find the load paths, the expected array shapes and keys, and the
   resampling applied on load. These sockets are where your Part 2 artifacts
   will go: the code is already waiting for files you have not made yet.
3. **The fast branch.** How does MixedGaits decide which reference plays for
   a given command? Find the speed threshold that gates the fast slot, the
   frequency multiplier applied above it, and where the command range's
   ±1.5 m/s cap is set.

**Report:** a one-page account of the reference pipeline (a diagram is worth
more than prose), the exact contract of the capture sockets (file, keys,
shapes, resampling), and the command-to-reference selection logic including
every constant involved.

Part 1: Author the Fast Gait
----------------------------
Design a parametric gait for the fast branch. The shipped gaits are your
intuition pump — study how their touchdown patterns differ and what that does
to the footfall rhythm:

.. list-table::
   :widths: 33 33 33

   * - .. figure:: ../../../_static/opt-lab2/ref_trot.gif

          Trot: diagonal pairs, half a cycle apart.

     - .. figure:: ../../../_static/opt-lab2/ref_gallop.gif

          Gallop: rotatory footfall, hind pair first.

     - .. figure:: ../../../_static/opt-lab2/ref_cheetah.gif

          Cheetah: the gallop's mirror — fore pair leads.

These three are starting points, not answers. What limits the trot at speed?
(Think: with diagonal pairs locked together, how much ground can one stride
cover before a planted foot has to slip?) What does staggering the touchdowns
buy? What does a longer swing buy, and what does it cost in stance time? Your
stride, stance depth, swing lift, and touchdown pattern are all in play — and
nothing says the fast gait must resemble any of these.

Workflow:

* Add your gait to the parametric tables and **visualize it before training**
  (``visualize_reference`` — same tool as lab 4). Choreography bugs are free
  to fix here and expensive to fix after an 18-minute run.
* Wire it into the fast branch, train ``Mjlab-MixedGaits-Flat-Pupper-v3``,
  and evaluate **at speed**: drive commands near ±1.5 m/s in the viewer and
  read ``error_vel_xy`` conditioned on the command, not just the average.
* Baseline first: one run with the stock trot-in-the-fast-slot, so every
  claim you make has a number to beat.

**Report:** your gait's parameters and the reasoning behind each; visualizer
video; a table of baseline-vs-yours — high-speed ``error_vel_xy``,
``error_vel_yaw``, mean torque², slip velocity — and honest commentary on
what your design got right and wrong.

Part 2: Reference Bootstrapping
-------------------------------
Here is the core idea of this lab. Your Part 1 reference is kinematic
choreography — it knows nothing about physics. But the *policy you trained on
it* does: at 1.5 m/s it has already deformed your choreography into something
dynamically consistent — leaning, timing shifts, compliance your tables never
specified. That knowledge is sitting in the rollouts. Harvest it:

1. **Capture.** Roll out your best policy at speed and record its actual
   joint trajectories. Select steady-state windows (no resets, no transients,
   command held constant).
2. **Align and fold.** Phase-align the captured cycles, average or select
   into a single representative cycle, resample onto the reference grid, and
   enforce periodicity at the wrap point.
3. **Install.** Write the result as the ``.npz`` the capture socket expects.
   The loader you studied in Part 0 takes it from there — your captured gait
   is now a first-class reference.
4. **Retrain** on the captured reference. The policy now tracks a target that
   physics already endorsed.
5. **Repeat.** Each round: reference ← rollout of best policy; policy ←
   trained against the new reference. Two to three rounds is typically where
   the loop stops paying.

This is expert iteration where the expert is your own previous policy,
filtered through the simulator's physics. Whether it *converges to something
better* — rather than amplifying a quirk of round one — depends on your
capture hygiene: which episodes you harvest, how you handle the phase
alignment, whether you smooth. There is no scaffold for this; you and your
agent write the capture pipeline. Design decisions are yours to make and
defend.

**Report:** the capture pipeline (committed to your fork), and a per-round
table — high-speed tracking error, torque², slip, top achieved speed — with
videos. State clearly which round you stopped at and why. If a round made
things *worse*, that is a result: explain the failure mode.

Part 3: Multi-Gait Distillation
-------------------------------
Assemble the whole animal. Train the full MixedGaits task — trot fore/aft,
your lift gait for turns, your bootstrapped gait in the fast branch — into
**one** policy, then make it survive contact with reality:

* Verify **mid-rollout gait transitions** in the viewer: sweep the command
  from 0 through the fast-branch threshold and back. The blend logic hands
  the policy a different choreography mid-stride; does it stumble at the
  seam?
* Run the ``Bumpy`` variant with your lab 4 DR ranges as the robustness pass.
* **Deploy.** The controller already reproduces every reference table from
  ``policy.json`` with its own phase clock — your multi-gait policy runs on
  hardware with zero controller changes. The trained ±1.5 m/s command clip
  ships inside the export (recall the heading-hold deliverable: the
  constants travel with the policy). If you raised the cap, this is where
  you find out whether you should have.
* **Measure it.** Tape measure, stopwatch, a straight hallway. Report actual
  m/s on the real robot, not the commanded value.

**Report:** transition videos (sim), deployed videos (real), measured top
speed on hardware vs. commanded, and where the sim2real gap bit hardest at
speed.

Part 4: ???
-----------
There is one more registered task. You may have seen it in the registry while
spelunking:

.. code-block:: python

   # ???
   register_mjlab_task(
     task_id="Mjlab-Mystery-Flat-Pupper-v3",
     ...
   )

What does it train? The task id will not tell you, and neither will the
docstring — go read it anyway. What we can say: it is MixedGaits with
something extra. An extra reference slot that the schedule opens now and
then. Reward machinery that pays for something no other task pays for. And,
out of the box, only the trot playing in that slot — earning almost none of
it.

The reference that slot is waiting for is not in the repo. It is not
parametric — no setting of the four dials produces it. It is something a
policy of yours might, under the right conditions, already know how to do —
if you can catch it doing it. You have a capture pipeline now. ???

.. figure:: ../../../_static/opt-lab2/ref_mystery.gif
   :align: center
   :width: 384px

   ``???``

**Report (open-ended):** whatever you find. If your Pupper does the thing —
in sim or, braver still, on hardware — the video goes on the course wall of
fame. Get it right and you will know.

Submission
----------
A link to your fork (capture pipeline, gait tables, and config changes
committed with real commit messages) plus a single technical report covering
the four parts' items above. Write it like a lab notebook for a colleague,
not a homework proof-of-effort: numbers in tables, claims tied to runs,
failures documented alongside wins.

Resources
---------
`DeepMimic: Example-Guided Deep Reinforcement Learning of Physics-Based Character Skills <https://arxiv.org/abs/1804.02717>`_

`AMP: Adversarial Motion Priors for Stylized Physics-Based Character Control <https://arxiv.org/abs/2104.02180>`_

`BeyondMimic: From Motion Tracking to Versatile Humanoid Control <https://arxiv.org/abs/2508.08241>`_

`Fast and Efficient Locomotion via Learned Gait Transitions <https://arxiv.org/abs/2104.04644>`_

`Concurrent Training of a Control Policy and a State Estimator for Dynamic and Robust Legged Locomotion <https://arxiv.org/abs/2202.05481>`_

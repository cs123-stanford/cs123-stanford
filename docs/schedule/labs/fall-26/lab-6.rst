Lab 6: Bringing Your Pupper to Life
===================================

*The title is a tribute to Haochen Shi's PhD thesis,*
`Bringing life to robots <https://purl.stanford.edu/yf485jt4430>`_
*(Stanford University, 2026).*

*Goal: let a foundation model drive Pupper, safely. Gemini decides what to
do; the tools you write decide what it can do, and tell it what really
happened. Then train Pupper to sit, stay and shake a paw, and give Gemini
those moves too.*

.. TODO(staff): add this offering's lab slides link

`Lab slides <#>`_ (TODO)

`Lab document <https://docs.google.com/document/d/1kp3xRDpWT6fy1FBzcx5_VDpdCx2mRkD5-4JWlJbUOac/edit?usp=sharing>`_: make a copy, answer each DELIVERABLE under its
question number (Q1–Q19), and submit it to Gradescope.

.. note::

   **Gemini Pupper was created by** `Peng Xu <https://scholar.google.com/citations?user=460NWeQAAAAJ&hl=en>`_
   **(Google DeepMind).** This lab is built on his project
   (`sippeyxp/gemini-pupper <https://github.com/sippeyxp/gemini-pupper>`_), and the
   terminal chat you meet first (``lab/chat.py``) is based on his minimal Gemini
   Live example. Thank you, Peng!

Robot foundation models like Gemini can listen, look through a camera,
reason about what you want, and decide what to do. But a foundation model
cannot move a single motor. It can only **call tools**. In this lab, three
pieces work together:

* **Gemini Live** hears you, sees through the camera, decides what to do, and
  talks back.
* **The policies** drive Pupper's 12 motors. Pupper is in one of two modes:

  - **Locomotion mode**: our default neural controller, the locomotion policy
    from lab 4. It walks, turns and runs; Gemini Pupper calls it *walking*.
  - **Trick mode**: a **trick policy** holds the legs instead of the
    locomotion policy. A trick policy is a tracking policy: it was trained
    (BeyondMimic-style) to follow a reference motion and keep Pupper balanced
    while it does. You will train one in this lab.

* **Gemini Pupper**, the system you build in this lab, connects the two. It
  decides which tools exist, what Gemini is told about them, what each one
  does on the robot, what it refuses, and what it reports back.

The idea to keep in mind: *a foundation model is only as good as its tools,
and it only knows what its tools tell it.* A tool that fails silently is a
robot that lies to you. Remember that every time Gemini says "Okay, I'm
walking!" while Pupper stands perfectly still.

Before You Start
----------------
In this lab you build a **human-robot interaction agent pipeline**, piece by
piece: from the words a person says, to the tool Gemini picks, to the policy
that moves the motors, and back to what Gemini tells the person. The amount
of code is moderate: most TODOs are a few lines to a few dozen. **The ideas
behind them are heavy.** Almost every TODO is about something that happens at
the same time as something else, later than you'd think, or out of Gemini's
sight. Read the explanations, not just the TODO cards, and give this lab real
time: start early, and don't leave Part 5 (which trains a policy) for the
last days.

It is worth it. Gemini Pupper is one of the best starting points for a final
project that any CS 123 quarter has had: a robot you can talk to, that walks,
follows people and does tricks you trained yourself. We expect great things
from the projects that build on it.

AI Use Suggestion
------------------
**This lab is designed so that you can complete it without AI.** You are
welcome to use a coding agent (Claude Code or Codex) to understand the code
and for light coding, as long as you understand what every line it writes
means. The repo comes with notes for your agent (``AGENTS.md``).

* **The deliverables are yours.** Answer every DELIVERABLE in your own words,
  from what you did and saw on your robot. Don't hand the questions to your
  agent.
* **Understand everything your agent does.** For every TODO, you should be
  able to explain what the code does, why it is written that way, and what
  would go wrong without it, without your agent open. Quiz 2 asks about this
  code.

Good ways to work with an agent are at the end of this handout, in
`Working With a Coding Agent`_. One rule always holds: never let an agent
move the robot, raise a limit, or weaken a test to make something pass.

.. important::

   **Safety rules for this lab.** Read them now. They apply to you *and* to
   your coding agent.

   * Test anything new with Pupper **on the ground, clear space around it, and
     your hand near the e-stop** (press the gamepad's right stick: it relaxes
     the motors, whatever Gemini is doing). Never on a table. On the stand,
     tricks look shakier than they are: test them on the ground.
   * **TA gate:** before your newly trained trick policy runs on the robot for
     the first time, show a TA its score and the simulation viewer. A badly
     trained trick can slam a paw or the body into the ground.
   * A coding agent never moves the robot on its own: no ``start.sh``, no
     ``chat.py --robot``, no ``curl`` to a POST endpoint, no ``ros2 topic
     pub``, unless you say yes, right then, standing at the robot.
   * **Pupper overheats.** Sitting and holding poses load the motors hard. If
     Pupper is hot to the touch or a motor faults, stop, relax the motors
     (``robot/stop.sh``), and let it cool.
   * Never put an API key in a file in your repo, a commit, or a chat you
     submit. The key goes in ``~/.config/gemini-pupper/api_key`` on the
     Pupper, never in the repo.

Lab roadmap
-----------
This lab is expected to take **two weeks**.

.. list-table::
   :header-rows: 1
   :widths: 10 64 26

   * - Part
     - What you do
     - TODOs
   * - 0
     - **Setup:** meet the loop on a fake robot; wake Pupper up; try what it can already do
     - (reading)
   * - 1
     - **Talk:** the prompt, first pass
     - 1
   * - 2
     - **Walk:** refuse unsafe moves; read the walking job; the turn, move and arc tools; trace one command
     - 2, 3, 4
   * - 3
     - **Follow:** the rule that ends follow mode, and the follow tool
     - 5, 6
   * - 4
     - **Tricks:** play any trick; read the trick job; tell Gemini which tricks exist
     - 7, 8
   * - 5
     - **Sit, stay, shake:** build a clip library, train it, give Gemini the actions
     - 9, 10 (+ notebook)
   * - 6
     - **The spider:** a trick that walks: build its clips and train its walking policy (reward weights, as in lab 4)
     - (notebook)
   * - 7
     - **Together:** the prompt, second pass; demo; reflection
     - 11

**The big picture: you are building one big state machine.** Like the state
machine in lab 5, only much bigger. Pupper is always in exactly one mode, and
every tool Gemini calls either moves it to another state, or is refused because
it can't happen from the state Pupper is in (or will be in, once everything
already queued has run). Keep this picture in mind: each TODO adds an arrow or
a rule to it.

.. figure:: ../../../_static/lab6/state_machine.svg
   :align: center
   :width: 95%

   Pupper's states, and the tools that move it between them. **Locomotion
   mode** is one state, with follow mode on or off inside it. **Trick mode**
   is one state per trick, of two kinds: a **playback trick** is one policy
   that plays its motion once; an **interactive trick** is a state Pupper
   stays in and takes requests in, held by one policy (``sit_stay_shake``) or
   more (the spider, Part 6). Each has a state machine of its own inside
   (Parts 5 and 6), but here it is one state. **Motors idle** is
   Pupper lying down with no controller holding the legs, and **fallen** is
   where a tilt e-stop leaves it. Any motion tool switches straight to the
   state it needs: a walk from motors idle goes to locomotion mode, a trick to
   that trick. Follow mode is the follower you built in lab 5, without its
   obstacle avoidance.

The TODOs are numbered 1–11 and run in the same order as this handout. In the
code, each one sits between ``TODO n: YOUR CODE HERE`` banners, under a short
**card** that says what happens now, what happens later, what Gemini hears,
what must always be true, which helpers to use, and what its tests check. Some
functions are already written, and marked *finished: read it*: you don't write
them, but you should be able to explain them (Q4 and Q8 ask you to).

Part 0: Setup
-------------
1. **Your Gemini API key.** Create one in
   `Google AI Studio <https://aistudio.google.com/apikey>`_.

   .. important::

      * **Sign in to Google AI Studio with your own personal Gmail account,
        not your Stanford account.**
      * **You might need some credits.** The Live API models the app uses
        may require billing or credits on your AI Studio account before your
        key works for them.

2. **Meet the loop (no robot needed).** Before you touch Pupper, look at the
   loop itself. Open the
   `Meet the Loop notebook <https://colab.research.google.com/github/cs123-stanford/gemini_pupper/blob/main/notebooks/Meet_the_Loop.ipynb>`_
   in Colab (**File → Save a copy in Drive**) and follow it: it puts your key
   in Colab's Secrets, never in a cell. It runs ``lab/chat.py``, which is
   based on a short example by Peng Xu: the lab's prompt and tools, the exact
   list the web app gets, on a **fake robot**. You type, Gemini answers, and
   every tool call, every reply and everything the fake robot did is printed.
   Nothing moves. (No Colab? On your laptop: clone the repo, ``python3 -m pip
   install --user google-genai pyyaml``, put your key in ``GEMINI_API_KEY``,
   and run ``python3 lab/chat.py``.)

   Type *"lie down, then stand up, then walk forward"*. You will see something
   like this:

   .. code-block:: text

      You > lie down, then stand up, then walk forward
        -> Gemini calls lie_down({})
        <- reply Gemini hears now: {"success": true, "message": "lie down queued; the queue should finish in 1.0 s"}
           ...later, when the queue gets to it:
           [fake robot] legs: relaxed gemini_walk
           [fake robot] outcome: lie down: ok
        -> Gemini calls stand_up({})
        <- reply Gemini hears now: {"success": true, "message": "stand up queued; the queue should finish in 5.2 s"}
           ...later, when the queue gets to it:
           [fake robot] legs: gemini_walk took over
           [fake robot] outcome: stand up: ok
        -> Gemini calls move({"direction": "forward"})
        <- reply Gemini hears now: {"success": false, "message": "TODO 3 is not written yet (move in lab/part2_walk.py)"}
      Pupper > ...

   That is the whole loop the lab is about. Gemini never runs code: it reads
   the description of each tool (the ``@tool(...)`` lines in ``lab/``), asks
   for a call, and *your* code runs it and sends back a reply. The reply is
   how Gemini gets feedback on what the robot did.

   **Already done for you, in** ``lab/basics.py``: ``stand_up``,
   ``lie_down``, ``stop`` and ``robot_status``. The other tools reply that
   their TODO isn't written yet. Watch what Gemini does with that reply.

   Two experiments in ``lab/basics.py`` (the notebook shows you how to edit
   it there; ``chat.py`` reads ``lab/`` when it starts, so restart it after
   each change, and put everything back afterwards):

   a. **The words drive the call.** Ask Pupper to *"take a nap"*. Then delete the
      sentence *"Use when the user says sleep, relax or lie down."* from
      ``lie_down``'s description (it spans two lines) and ask again. Last, change the description
      to ``"Makes the robot jump."`` and ask it to jump. What does Gemini call
      each time?
   b. **The reply is how Gemini gets feedback.** Make ``stand_up_tool`` return
      ``Result(False, "a wall is in the way")`` instead of calling
      ``stand_up``, and ask Pupper to stand up. Then make it return
      ``Result(True, "standing up")``, still without calling ``stand_up``:
      nothing reaches the fake robot (no ``[fake robot]`` lines). What does
      Gemini tell you each time?

   From now on, this is the fastest way to try a change to what Gemini reads:
   edit, run ``python3 lab/chat.py`` (on the Pupper, in your own repo), type. (``python3 lab/chat.py --robot`` sends the
   calls to the real robot server instead, the path a voice command takes,
   and Pupper moves.)

3. **Your code, on the Pupper.** Fork the `gemini_pupper <https://github.com/cs123-stanford/gemini_pupper>`_
   repository to your own GitHub account, following
   :doc:`forking_repositories`, and clone your fork onto the Pupper:

   .. code-block:: bash

      cd ~/
      git clone https://github.com/YOUR_USERNAME/gemini_pupper.git

4. **Set it up.** On the Pupper:

   .. code-block:: bash

      cd ~/gemini_pupper
      robot/setup.sh

   It installs what the app needs (without ``sudo``), builds it, and asks for
   your key. The key goes in ``~/.config/gemini-pupper/api_key`` on the
   Pupper, never in the repo. You can run ``setup.sh`` again at any time.

   Keep your Lab 5 repo (``~/follow_me_pupper``) on the Pupper: follow mode
   runs it in Part 3.

5. **Wake Pupper up.** Put Pupper on the ground in clear space. There are two
   ways to run the app:

   * **From your laptop: use this while you build and debug.** It is much,
     much easier: the debug view gets a big screen, and you can type as well
     as talk. On the Pupper, run ``robot/start.sh``. On your laptop, forward
     the port, browse to ``http://localhost:8000``, open **Settings**, paste
     your key, and press **WAKE PUPPER**:

     .. code-block:: bash

        ssh -N -L 8000:localhost:8000 pi@pupper[YOUR_GROUP_NUMBER].local

   * **On Pupper itself: use this for demos.** ``robot/start.sh --kiosk``
     runs the app on Pupper's own screen, with its mic and speaker, and no
     laptop. This is how you record your demo videos.

   Say hi. The **debug view** shows every tool call Gemini makes and what the
   robot said back: keep it open for the whole lab. ``robot/stop.sh`` relaxes
   the motors and stops everything.

   .. admonition:: TODO (staff): screenshot of the debug view

      A screenshot of the app's debug view, with a few tool calls and their
      replies, goes here.

   .. admonition:: Debugging tips
      :class: tip

      * **Start from the debug view.** It shows every function call Gemini
        made and the reply it heard. When Gemini says something odd, find the
        reply it was answering.
      * **Read the logs on the Pupper**: ``/tmp/gemini-pupper/server.log``
        (your ``lab/`` code: every tool call, every job's outcome, every
        error) and ``/tmp/gemini-pupper/robot.log`` (ROS 2: controllers and
        policies). ``tail -f`` one while you talk.
      * **Ask the robot, not Gemini.** Gemini only knows what the replies
        told it. ``curl localhost:8000/system/mode`` on the Pupper says what
        the robot is really doing.
      * **Try it without the robot.** ``python3 lab/chat.py`` runs the same
        conversation on the fake robot, and ``python3 lab/test.py todoN``
        checks one TODO.
      * **Changed a** ``.py`` **file in** ``lab/``? Restart
        (``robot/stop.sh && robot/start.sh``). Changed ``lab/prompt.md``?
        Just reconnect in the app.
      * **The app won't start?** ``robot/start.sh`` prints where it got
        stuck: controllers still loading (see ``robot.log``), or the server
        didn't start (see ``server.log``).

6. **What already works.** The provided tools are ``stand_up``, ``lie_down``,
   ``stop`` and ``robot_status``. Try *"stand up"*, *"lie down"*, *"what are
   you doing?"*. Then ask Pupper to walk to you, and to do a trick: those
   tools exist, but they reply that their TODO isn't written yet. Watch what
   Gemini does with that reply.

   .. admonition:: TODO (staff): video

      A short video of Pupper not working: Gemini says it is walking while
      Pupper stands still.

7. Run the tests once: ``python3 lab/test.py``. The provided robot's checks
   pass, and every TODO is listed as not written yet.

**DELIVERABLE (Q1):**

(a) Step 2a: what did Gemini call after each change to ``lie_down``'s
description?

(b) Step 2b: what did Gemini tell you when ``stand_up`` failed? And when it
"succeeded" without the robot moving?

(c) Step 6: when you asked Pupper to walk, what did Gemini say, and what did
Pupper actually do?

(d) A robot that says it is doing something it isn't doing is bad
human-robot interaction. Why is it worse on a robot than in a chatbot? Give a
possible explanation for why Gemini said it.

How Gemini Pupper Works
-----------------------
You have seen the loop from the outside. Here is how it works inside,
starting from the code. Read this once now, and come back to it whenever a
TODO refers to it.

**You work in one folder: lab/.** When Gemini connects, the robot sends it a
prompt and a list of tools. When Gemini calls a tool, the robot runs it and
sends back a reply. The prompt and the tools are yours, in ``lab/``:

.. figure:: ../../../_static/lab6/tool_loop.svg
   :align: center
   :width: 95%

   The loop you are building. Gemini picks a tool from its *description*;
   your code on the robot does the work and replies; the reply is how Gemini
   gets feedback on what actually happened.

.. code-block:: text

   gemini_pupper/
   ├── lab/                         ← you work here
   │   ├── prompt.md                who Pupper is and how it behaves        TODO 1, 6, 11
   │   ├── basics.py                read first: finished examples
   │   ├── part2_walk.py            check_move, walk, turn, move, arc       TODO 2, 3, 4
   │   ├── part3_follow.py          the follow rule, the follow tool        TODO 5, 6
   │   ├── part4_tricks.py          play_trick, the trick job (finished)    TODO 7, 8
   │   ├── part5_sit_stay_shake.py  interactive_action, its tool                  TODO 9, 10
   │   ├── test.py                  python3 lab/test.py todo3   (tests on a fake robot)
   │   └── chat.py                  python3 lab/chat.py         (talk to Gemini in text)
   ├── tricks/                      one folder per trick
   ├── robot/                       provided: the Robot API, the robot server, launch files, scripts
   └── web/                         provided: the web app; you never need it

**The one idea: now and later.** Every motion tool runs in two moments:

* **Now**, the moment Gemini calls it: check that the request is allowed, put
  a **job** in line for the robot, and reply right away. Gemini hears this
  reply: refused (and why), or accepted.
* **Later**, when the robot gets to it: the job moves the robot. What
  happened (done, fell over, controller failed) is the job's **outcome**. It
  goes to the debug log and to ``robot_status``.

Why two moments? Gemini often asks for several things in one breath
(*"stand up, then walk forward"*). The robot can only do them one at a time,
and Gemini shouldn't go silent until they're done. So every motion waits in a
**queue**, and every tool replies as soon as its job is in line.

**In the code: stand_up.** Here are both halves of the first finished
example in ``lab/basics.py``: the tool Gemini reads, and the behavior it
runs.

.. code-block:: python

   @tool("stand_up", "Stand up and turn on walking. Needed before moving, ..." + ENDS_FOLLOW)
   async def stand_up_tool(robot: Robot) -> Result:            # what Gemini reads
       return await stand_up(robot)                              # (1)

   async def stand_up(robot: Robot) -> Result:                 # what the robot does
       note = await prepare_to_move(robot)                       # (2) NOW

       async def job() -> Result:                                # (3) defined now, run LATER
           return await robot.switch_to(LOCOMOTION_CONTROLLER)

       return with_note(note, await robot.queue("stand up", job, WALK_READY_S, then=LOCOMOTION))  # (4) NOW

Follow it from the words *"stand up"*:

1. **Gemini calls the tool.** From the ``@tool(...)`` words alone, Gemini
   decides that ``stand_up`` is the one and makes a **function call** (the
   Gemini API's name for a tool call). The robot server runs
   ``stand_up_tool``, which calls the behavior, ``stand_up``.
2. **Now: get ready.** ``prepare_to_move`` (your TODO 5) gets the robot ready
   for a new motion: it stops anything that runs *on its own, outside the
   queue*, and would fight with the motion. Right now nothing does, so it
   does nothing and returns ``""``. In Part 3 you make it end follow mode
   (the person follower drives the legs by itself), and it returns a note
   for Gemini saying so.
3. **Now: write down the work for later.** ``async def job()`` *defines* the
   work without running it: give the legs to the locomotion policy. It can
   use ``stand_up``'s variables (``robot`` here; ``vx`` and ``duration`` in
   ``walk``). A ``lambda`` is a one-line function:
   ``job = lambda: play_policy_trick(robot, trick)`` is a job too.
4. **Now: queue it and reply.** ``robot.queue(name, job, seconds, then=...)``
   puts the job in line and returns at once with the *accepted* reply,
   ``"stand up queued; the queue should finish in 4.2 s"``. ``then=`` is the
   state Pupper will be in once the job is done (here ``LOCOMOTION``): the
   queue keeps it as ``robot.state_after_queue``, so the next request knows
   where Pupper will be by the time it gets its turn. ``with_note`` puts the
   note in front. This reply is what Gemini hears.
5. **Later: the job runs.** When every job before it is done, the queue calls
   ``job()``. Its outcome, ``"stand up: ok"``, goes to the debug log
   and to ``robot_status``.

**Every motion in the lab has this shape: check now, queue a job, reply.**

**On a timeline.** Someone says *"stand up and walk forward"*, and Gemini
makes two function calls, back to back:

.. list-table::
   :header-rows: 1
   :widths: 11 33 30 26

   * - Time
     - Gemini: function call → the reply it hears
     - The robot
     - The debug log (outcomes)
   * - 0.00 s
     - ``stand_up()`` → *"stand up queued; the queue should finish
       in 4.2 s"*
     - ``stand_up``: not following, nothing to stop; its job is queued.
     -
   * - 0.01 s
     - ``move(direction="forward")`` → *"move vx=0.4 … queued; the queue
       should finish in 5.7 s"*
     - ``walk``: ``check_move`` is happy; its job is queued behind
       stand_up's.
     -
   * - 0.01–4.2 s
     - (nothing: Gemini is talking to you)
     - stand_up's job: ``switch_to(gemini_walk)``, wait for the policy.
     - *"stand up: ok"*
   * - 4.2–5.7 s
     - (nothing)
     - walk's job: send the speed 20 times a second, then a zero.
     - *"move vx=0.4 …: moved vx=0.4 … for 1.5 s"*

Gemini heard both replies before Pupper took a single step. Tricks work the
same way: *"sit, then shake"* is two function calls,
``play_trick("sit_stay_shake")`` then ``interactive_action("shake_left")``, both
answered at once; the shake's job starts only when the sit's job is done.

.. important::

   **Gemini is not told when a job finishes.** A tool replies the moment its
   job is *accepted*. When the job finishes later (done, fell over,
   controller failed), its outcome goes to the debug log and to
   ``robot_status``, and nowhere else: nothing is sent back to Gemini. Right
   after *"stand up and walk forward"*, Gemini knows that both moves were
   queued, and nothing more. To learn what really happened, it has to ask
   (call ``robot_status``). Q4 asks what this means for a long walk.

**The Python behind it: async and await.** The robot server does many things
at once: it runs a job, answers Gemini, and listens for *"stop"* and the
e-stop. ``async`` functions are how: one can pause while it waits, and
everything else keeps running meanwhile. Two rules are all you need:

* **Put** ``await`` **in front of every** ``async`` **call**:
  ``await robot.switch_to(...)``, ``await robot.sleep(0.05)``,
  ``await robot.queue(...)``. Without ``await``, the call never runs: Python
  makes it and throws it away, with at most a warning that it "was never
  awaited".
* **Never** ``time.sleep``. It doesn't pause your function: it freezes the
  whole server, the ``stop`` tool and the e-stop included. Use
  ``await robot.sleep(seconds)``.

.. raw:: html

   <details style="margin-bottom: 1.5em">
   <summary><b>New to async? A two-minute example (click to open)</b></summary>

Run this on your laptop (``python3 example.py``):

.. code-block:: python

   import asyncio


   async def walk():
       print("walk: start")
       await asyncio.sleep(1.2)  # pause walk; everything else keeps running
       print("walk: done")


   async def listen():
       for _ in range(3):
           print("listen: did anyone say stop?")
           await asyncio.sleep(0.5)


   async def main():
       await asyncio.gather(walk(), listen())  # run both at once


   asyncio.run(main())

.. code-block:: text

   walk: start
   listen: did anyone say stop?
   listen: did anyone say stop?
   listen: did anyone say stop?
   walk: done

``await asyncio.sleep(1.2)`` pauses ``walk``, and ``listen`` runs in the
meantime. Now break it, twice:

* Change it to ``time.sleep(1.2)`` (and ``import time``): ``listen`` stays
  silent until the walk is over. On Pupper, ``listen`` is the ``stop`` tool.
* Delete the ``await``: ``walk`` doesn't wait at all, and Python warns
  ``coroutine 'sleep' was never awaited``.

The robot server works the same way: ``robot.queue`` runs the jobs one at a
time, ``await``-ing each, while the server keeps answering Gemini and
listening for *"stop"*. More: the Python docs'
`Coroutines and Tasks <https://docs.python.org/3/library/asyncio-task.html>`_,
and Real Python's `Async IO in Python <https://realpython.com/async-io-python/>`_.

.. raw:: html

   </details>

**Result.** Every tool and every job returns ``Result(ok, message)``: ``ok``
is ``True`` or ``False``, and ``message`` is the sentence Gemini (or the
debug log) reads. Write messages for a person: what happened, or what was
wrong and what would work instead.

Every ``lab/`` file has the same two halves, in order:

* **What the robot does**: behaviors, like ``stand_up``. Is the request
  allowed? Do it. Report what happened.
* **What Gemini reads**: tools, each an ``@tool(...)`` line with the tool's
  name (the one Gemini calls), a description, and a description of each
  parameter, over a short function that turns Gemini's words into numbers and
  calls a behavior. Gemini picks a tool, and fills in its parameters, from
  these words alone.

**Pupper's state.** The robot is always in exactly one state, and the state
is whatever holds the legs: **locomotion mode** (``LOCOMOTION``: the default
locomotion policy, on the ``gemini_walk`` controller; following or not),
**trick mode** (one trick: its trick policy, on its ``trick_<name>``
controller), **motors idle** (``MOTORS_IDLE``: nothing holds the legs, so
Pupper lies down) or **fallen** (``FALLEN``). Every job changes the state
with one call, ``robot.switch_to(...)``, and the robot keeps track of two
states for you:

* ``robot.state_now``: the state Pupper is in right now. Only ``switch_to``
  (and a fall, the e-stop or ``stop``) changes it.
* ``robot.state_after_queue``: the state it will be in once everything already
  queued has run. Each ``robot.queue(..., then=...)`` sets it.

A request asked for *now* checks ``state_after_queue`` (where will Pupper be
when its turn comes?); a job checks ``state_now`` when it runs (where is it
really?). You will use both in Part 5.

**The tests: what they check, and why.** ``python3 lab/test.py`` runs your
code on a fake robot. Each TODO below ends with a table of *every* check its
tests make, with an example that passes and one that fails: nothing is checked
that isn't in the table. When a check fails, it prints what your code produced
and an example that passes.

Many checks look for a word or a number in what Gemini reads: a tool's reply
(``"... at most 20.0 s"``) or a description (``"follow"``). That is on purpose.
Gemini knows only what you tell it. A reply that says ``"too fast"`` leaves it
guessing; one that says ``"too fast: |vx|<=1.5 m/s"`` lets it retry at a speed
that works, or tell the person why it can't. **When you build tools for a
foundation model, put a bit more in every reply than you think you need: what
went wrong, what the limit is, and what the real choices are.**

**The Robot API.** These are the only parts of ``robot/robot_api.py`` you need:

.. list-table::
   :header-rows: 1
   :widths: 42 58

   * - Call
     - What it does
   * - ``await robot.queue(name, job, seconds, then=state)``
     - Put ``job`` (an ``async`` function with no arguments) in line, and
       return the *accepted* Result now. ``then``: the state Pupper is in once
       this job is done (``LOCOMOTION``, ``MOTORS_IDLE`` or a trick).
   * - ``await robot.switch_to(controller)``
     - Give the legs to one controller (``None``: relax the motors), and update
       ``robot.state_now``. Also stands up safely after a fall, and leaves an
       interactive trick through its exit clip before another motion takes
       the legs.
   * - ``robot.state_now``, ``robot.state_after_queue``
     - The state Pupper is in right now, and the one it will be in once
       everything already queued has run.
   * - ``robot.send_speed(vx, vy, wz_deg)``
     - One speed command to the locomotion policy.
   * - ``await robot.sleep(s)``, ``robot.now()``
     - Wait without freezing the server; the time now. Never ``time.sleep``.
   * - ``robot.fallen``, ``FELL_MESSAGE``
     - Which controller fell over (``None`` while upright), and what to say.
   * - ``robot.following``, ``await robot.stop_following()``
     - Follow mode.
   * - ``await robot.is_active(controller)``
     - Is this controller driving the legs right now?
   * - ``await robot.play_clip(trick, clip)``
     - Play one clip of an interactive trick, and wait for it.
   * - ``robot.tricks``, ``robot.action_ids()``
     - The installed tricks; interactive actions → (trick, clip).

Part 1: Talk
------------
Open ``lab/prompt.md``. It has six headings with a question under each;
you write it in two passes.

**TODO 1:** Write the first three sections: *Who you are*, *How you move*,
and *When a tool says no*.

- **Who you are.** Give Pupper a personality and a way of talking. Everything
  Gemini says is spoken out loud, so short sentences work best.
- **How you move.** Pupper can only move by calling a tool. Should it call the
  tool first and talk after, or the other way round? (Think about what
  happens if it says "I'm walking!" and the tool then refuses.) What must it
  never say?
- **When a tool says no.** Every tool reply says what really happened. What
  should Pupper do when a tool refuses, or says its TODO isn't written yet?
  What should it never pretend?

*Writing rules Gemini follows.* Gemini follows short, concrete rules much
better than general wishes. Compare:

.. code-block:: text

   Vague:     Be honest about what you can do.
   Concrete:  You can only move by calling a tool. Never say you are moving,
              sitting or doing a trick unless you called the tool for it.

A good rule says *when* (in which situation), *what to do*, and, if it
matters, *what never to do*. Use bullet points: one rule per bullet.

After editing the prompt, try it in text first: ``python3 lab/chat.py``
reads ``prompt.md`` every time it starts. Then reconnect in the app (no
restart needed), ask for a walk again, and compare with Q1.

**What** ``python3 lab/test.py todo1`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - The three sections are written
     - your own rules under each heading
     - a heading still says TODO (the checks are skipped)
   * - They have 60+ words together
     - a few bullet points under each heading
     - "Be nice. Use tools."
   * - They say ``"tool"``
     - "You can only move by calling a tool."
     - "You can walk, turn and do tricks."

These checks can only look for words. Whether Gemini *follows* your rules, you check by talking to it (Q2).

**DELIVERABLE (Q2):** Pick two rules from your prompt. For each: what did
Gemini do before you wrote it, and what does it do now? Is the sentence in
the prompt enough, or will the robot also have to make sure of it in code?

Part 2: Walk
------------
In this part you make Pupper walk, turn, run and curve by voice, and refuse
what it must not do. A walk or a turn asked for by Gemini goes through three
functions in ``lab/part2_walk.py``, from the words to the motors:

.. code-block:: text

   "walk forward"  ->  move (TODO 3)                     words  -> vx, vy, duration
   "turn left"     ->  turn (TODO 3)                     words  -> wz_deg, seconds
   "go in a circle" -> arc (TODO 4)                      words  -> vx, wz_deg, seconds
                   ->  walk (finished: read it)          check now, queue the job
                         -> check_move (TODO 2)          is this move allowed?
                         -> the job, later               20 speed messages a second

We build them in the order of the file: the safety check, then reading the
walking job, then the words Gemini reads (``turn`` first, then ``move``),
and last ``arc``, which walks and turns at the same time.

**TODO 2:** ``check_move`` in ``lab/part2_walk.py``: refuse what the robot
can't or shouldn't do.

*Why:* the most common reason a tool refuses is not a missing tool. Gemini
finds the right tool, but calls it with values the robot can't or shouldn't
do: 3 m/s, a 30-second walk, a speed so small the legs just shuffle. (*"Sit,
then run 100 mph"*: Gemini calls ``move``, with a speed far over the limit.)
The other kind, a request with no tool at all (*"do a backflip"*), you meet
in Part 4. Your tool descriptions *ask* Gemini to stay within the limits;
``check_move`` *makes sure* of it. It is a plain function (no robot, no ``await``), so it is the
easiest place to start.

It gets a velocity (``vx`` forward and ``vy`` left, in m/s; ``wz_deg``
counter-clockwise, in deg/s) and a ``duration`` in seconds, and returns
``None`` if the move is allowed, or a sentence saying why not. The limits are
in ``MoveLimits`` at the top of ``robot/robot_api.py``. Check them one at a time and
return the first problem you find:

1. **Too fast:** ``abs(vx) > limits.vx_max``, or the same for ``vy`` and
   ``wz_deg``. (``abs``, because backward and right are negative.)
2. **Too slow to move at all:** ``vx``, ``vy`` *and* ``wz_deg`` are all below
   their minimums (``vx_min``, ``vy_min``, ``wz_min``). Only if all three are:
   a turn in place has ``vx = vy = 0``, and that's a fine move.
3. **Duration:** it must be more than 0 and at most ``limits.duration_max``.
4. **Too far:** the distance is speed × duration, where speed is
   ``(vx**2 + vy**2) ** 0.5``. Refuse if it is more than
   ``limits.distance_max``.
5. Nothing wrong: return ``None``.

.. important::

   **Your reason must name the limit, and the tests check for it.** The string
   you return goes to Gemini word for word as the tool's reply: it is how
   Gemini learns why it was refused. ``"too fast"`` leaves it guessing a new
   speed (and maybe guessing wrong again, or giving up).
   ``"too fast: |vx|<=1.5 m/s"`` lets it retry at a speed that works, or tell
   you why it can't. Write the number with the ``MoveLimits`` field in an
   f-string, so it stays right if a limit changes.

**What** ``python3 lab/test.py todo2`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - Allows good moves, even exactly at a limit
     - ``check_move(1.5, 0, 0, 1.0)`` returns ``None``
     - refusing it because you wrote ``>=`` instead of ``>``
   * - Refuses each bad move with a reason: too fast (forward, backward, sideways, turning), too slow on all three, a zero or negative duration, longer than ``duration_max``, farther than ``distance_max``
     - ``"duration must be more than 0 and at most 20.0 s"``
     - ``None``, or ``""``
   * - The reason contains the limit's number, as Python prints it: ``1.5`` (vx_max), ``0.5`` (vy_max), ``114.0`` (wz_max), ``20.0`` (duration_max), ``7.0`` (distance_max)
     - ``f"... at most {limits.duration_max} s"`` gives ``"... at most 20.0 s"``
     - ``"too long"`` (no number); ``f"{limits.duration_max:.0f}"`` gives ``"20"``
   * - (finished ``walk``) Gemini hears your reason, limit included
     - ``walk(robot, 5.0, 0, 0, 1.0)`` fails with ``"too fast: limits are |vx|<=1.5 m/s, ..."``
     - ``"too fast"``

The other checks under *(provided) walk* test the finished ``walk``: they pass once ``check_move`` works.

Check it: ``python3 lab/test.py todo2``. It also runs the finished ``walk``
(below) with your ``check_move``, so you see refused moves come back as
failures with your reason.

**Read: walk** (finished, in ``lab/part2_walk.py``). This is ``stand_up``'s
shape, with a loop in the job. You don't write it, but read it now, line by
line, and make sure you could explain it (Q4):

* *Now:* it asks your ``check_move`` and refuses with its reason; it calls
  ``prepare_to_move`` to get the robot ready (it does nothing yet: you write
  it in Part 3, TODO 5); it refuses if the robot has already fallen.
* *Later, in the job:* it hands the legs to the locomotion policy, then sends the
  speed with ``robot.send_speed`` 20 times a second (``LLM_CMD_RATE``) until
  the duration is up, and ends with a zero speed.
* *Why the loop?* The locomotion policy doesn't take "walk for 2 seconds". It
  takes a speed, over and over. Between Gemini and the policy sits
  ``cmd_vel_mux``, the node that lets your joystick win over Gemini; it
  forgets a speed it hasn't heard for half a second, so a crashed program
  can't leave Pupper walking forever.
* *Why check* ``robot.fallen`` *three times?* The robot may already have
  fallen when Gemini asks, may fall while the job waits in line, or while it
  walks. Each time, the outcome should say so, not "moved".
* *Who hears the outcome?* Not Gemini. It heard *"move ... queued"* the
  moment ``walk`` returned. ``"moved ..."`` or ``"the robot fell over ..."``
  goes to the debug log and ``robot_status`` only (see *Gemini is not told
  when a job finishes*, above).

**TODO 3:** ``turn``, then ``move``, near the bottom of ``lab/part2_walk.py``:
the tools Gemini calls.
Write ``turn`` first. It's short, and it shows you the whole path from
Gemini's words to the robot. Then ``move``, the real challenge of this part.

*What Gemini reads.* Gemini picks a tool, and fills in its parameters, only
from the words in ``@tool(...)``: the tool's description and each parameter's
description. For each tool, replace every ``TODO 3`` there:

- Say what the tool does and when to use it. Put the robot's real limits in
  the words: the ``f"..."`` strings can use ``LIMITS.vx_max``,
  ``LIMITS.duration_max`` and the others, so they stay right if a limit
  changes. Add ``+ ENDS_FOLLOW`` to the description (Part 3 explains it).
- For each parameter: its unit (m/s, seconds, degrees), its range, and its
  default.
- Give a worked example wherever Gemini has to do arithmetic or pick a number.
  When we tested the answer key, *"walk forward 2 meters"* walked 0.6 m until
  ``duration``'s words said *"Distance = speed x duration: 2 m at 0.4 m/s is
  5 s"*, and *"walk in a square"* came out with 45° corners until ``angle``'s
  words said *"90 a square's corner, 120 a triangle's"*.

**turn: the warm-up.** Turning in place means no forward or sideways speed,
only a turning speed, for as long as the angle takes:

1. ``wz_deg`` is ``+TURN_SPEED`` (60 deg/s) to the left (counter-clockwise),
   and ``-TURN_SPEED`` to the right.
2. The time it takes: ``seconds = abs(angle) / TURN_SPEED``. 90 degrees at
   60 deg/s takes 1.5 s.
3. ``return await walk(robot, 0.0, 0.0, wz_deg, seconds)``

Run ``python3 lab/test.py todo3``: it checks ``turn`` first, on its own, so
you know it works before you start on ``move``.

**move: the real challenge.** Gemini calls ``move`` with a ``direction`` and,
if it wants, ``run``, ``speed`` and ``duration``. Anything it leaves out keeps
the default in the function's signature (``speed`` is ``None`` when Gemini
gave none). Your job is to turn that into one speed, and then split it into
``vx`` and ``vy``.

1. **Work out the speed.**

   - *Running* means ``run`` is true **and** the direction is forward or
     backward. A running speed is at least ``RUN_MIN_SPEED`` (1.0 m/s), and
     ``RUN_DEFAULT_SPEED`` (1.2 m/s) if Gemini gave no speed. ``max(a, b)``
     gives the bigger of two numbers: ``max(0.6, RUN_MIN_SPEED)`` is 1.0.
   - *Otherwise* (walking, or any sideways move, which never runs) it's
     ``speed``, or 0.4 m/s if Gemini gave none.

2. **Split it by direction.** The robot's convention is +x forward and
   **+y left**, the opposite of what many people guess:

   .. list-table::
      :header-rows: 1
      :widths: 25 25 25

      * - direction
        - vx
        - vy
      * - forward
        - +speed
        - 0
      * - backward
        - −speed
        - 0
      * - left
        - 0
        - +speed
      * - right
        - 0
        - −speed

3. **Say what you changed.** If Gemini asked to run sideways, your code walks
   instead. Nothing fails, so the reply would be a plain "queued", and Gemini
   would happily tell the person "Running left!" while Pupper walks. Start the
   reply with a note:

   .. code-block:: python

      note = "Sideways never runs; walking instead. " if run and sideways else ""
      return with_note(note, await walk(robot, vx, vy, 0.0, duration))

   ``with_note`` (provided) puts the note in front of ``walk``'s message.

Check your logic against these examples before you run anything. They are
exactly what the tests ask:

.. list-table::
   :header-rows: 1
   :widths: 48 26 26

   * - Gemini calls
     - vx (m/s)
     - vy (m/s)
   * - ``move(direction="forward")``
     - 0.4
     - 0
   * - ``move(direction="backward", speed=0.3)``
     - −0.3
     - 0
   * - ``move(direction="left", speed=0.3)``
     - 0
     - 0.3
   * - ``move(direction="backward", run=True)``
     - −1.2
     - 0
   * - ``move(direction="forward", run=True, speed=0.6)``
     - 1.0 (run means at least 1.0)
     - 0
   * - ``move(direction="right", run=True)``
     - 0
     - −0.4 (sideways never runs)

*What "run" means.* The locomotion policy switches to its running gait above
about 0.5 m/s, and in this lab "run" means **1.0 m/s or faster**. Gemini might
call ``move(run=true, speed=0.6)``. That's why the rule has to be in your
code, not only in the description. (Q3 asks about it.)

.. tip::

   The same lesson as TODO 2's limits, from the other side: **whenever your
   code changes what Gemini asked for, the reply says so.** Gemini only knows
   what the reply tells it, and a silent change becomes a confident wrong
   sentence out loud. We missed this case ourselves when we wrote the lab.

**What** ``python3 lab/test.py todo3`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``turn`` and ``move`` each have a description of 20+ characters, and every parameter is described
     - ``"Turn in place to the left or right." + ENDS_FOLLOW``
     - ``"turn"``, or a parameter still saying ``"TODO 3"``
   * - ``turn("left", 90)`` sends a positive turn rate, about 90° in all (70–110), with ``vx = vy = 0``
     - ``wz_deg = +TURN_SPEED`` for 1.5 s
     - ``-TURN_SPEED``; or ``seconds = angle`` (90 s)
   * - ``turn("right", 90)`` sends a negative turn rate
     - ``wz_deg = -TURN_SPEED``
     - ``+TURN_SPEED``
   * - ``move``'s words contain ``LIMITS.vx_max`` (``1.5``)
     - ``f"running {RUN_MIN_SPEED}-{LIMITS.vx_max} m/s"``
     - ``"runs fast"``, or a typed-in ``1.5`` (it passes, but goes stale)
   * - ``move``'s words contain ``"follow"``
     - ``"..." + ENDS_FOLLOW``
     - leaving ``ENDS_FOLLOW`` out
   * - The six ``move`` examples in the table above send exactly those speeds
     - ``move("right", run=True)`` sends ``vy = -0.4``
     - ``vy = -1.2`` (running sideways)
   * - ``move("right", run=True)``'s reply says ``"walk"``
     - ``"Sideways never runs; walking instead. move vx=0 vy=-0.4 ... queued"``
     - ``"move vx=0 vy=-0.4 ... queued"`` (Gemini then says it's running)
   * - A move ``check_move`` refuses comes back as a failure with its reason
     - ``move("forward", speed=1.5, duration=10)``: ``"that would travel 15.00 m, more than 7.0 m in one move: split it into shorter moves"``
     - a success, or a crash

**TODO 4:** ``arc``, at the very bottom of ``lab/part2_walk.py``: walk and
turn at the same time.

``turn`` sends only a turning speed, and ``move`` only a straight one. Ask for
"a circle" with just those, and the best Gemini can do is chain them: Pupper
walks, stops, turns, walks again. But ``walk`` takes ``vx`` and ``wz_deg``
together, and the locomotion policy was trained on both at once. ``arc`` is how
Gemini asks for a smooth curve.

Its parameters are given, so the tests can check them: ``direction`` (left or
right), ``angle`` (degrees of the circle to go around, **required**), ``radius``
(m, default 0.5) and ``speed`` (m/s, default 0.4). You write the words and the
body. ``angle`` has no default on purpose: with a default of 90, Gemini asked
for "a circle" by leaving it out, and Pupper walked a quarter of one.

**The math.** Going around a circle of radius *r* at speed *v*, Pupper turns
*v / r* radians per second. ``walk`` wants degrees per second, and the time
comes from the angle:

.. code-block:: python

   wz_deg = math.degrees(speed / radius)   # + to the left, - to the right
   seconds = abs(angle) / abs(wz_deg)
   return await walk(robot, speed, 0.0, wz_deg, seconds)

.. list-table::
   :header-rows: 1
   :widths: 46 54

   * - Gemini calls
     - What happens
   * - ``arc(direction="left", angle=90)``
     - vx 0.4, wz +45.8 deg/s for 1.96 s: a quarter circle
   * - ``arc(direction="right", angle=180, radius=1.0, speed=0.6)``
     - vx 0.6, wz −34.4 deg/s for 5.24 s: a U-turn
   * - ``arc(direction="left", angle=360, radius=1.0)``
     - vx 0.4, wz +22.9 deg/s for 15.7 s: a circle 2 m across (6.3 m of path)
   * - ``arc(direction="left", angle=90, radius=0)``
     - a sharp corner: turns in place, like ``turn``, and says so: ``"Radius 0: turned in place (a sharp corner). move ..."``
   * - ``arc(direction="left", angle=90, radius=0.1)``
     - refused by *your* code, with what works: ``"too tight: at 0.4 m/s the radius must be at least 0.25 m"``
   * - ``arc(direction="left", angle=360, radius=1.5, speed=1.2)``
     - refused by ``check_move``: ``"that would travel 9.42 m, more than 7.0 m in one move: split it into shorter moves"``
   * - ``arc(direction="left", angle=90, radius=-1)``
     - refused by your code: ``"no arc: radius must be 0 m or more ..."``

Three things to get right. Each one comes from watching Gemini use an ``arc``
that didn't have it:

- **A too-tight arc: say what works.** ``check_move`` would refuse it too, but
  its reason is ``"|wz|<=114.0 deg/s"``. Gemini can't tell from that that the
  *radius* is the problem: asked to "run in a circle", it sent the same arc five
  times. Check it yourself first, and reply with the smallest radius that works
  at that speed, ``speed / math.radians(LIMITS.wz_max)``, rounded up.
- **Radius 0 is a sharp corner, not an error.** Gemini plans a whole shape
  and sends every call at once, and it likes to ask for corners as
  ``arc(radius=0)``. When those were refused, the sides still ran, and "walk
  in a rectangle" came out as one straight line. ``speed / 0`` would crash
  anyway, so handle it before the math: turn in place, as ``turn`` does, and put
  a note in front of the reply (``with_note``, as in TODO 3). A negative radius,
  or a speed of 0 or less, is refused.
- **Give Gemini the limits in the words,** computed from ``LIMITS``. The
  tighter the circle, the faster the turn, and the robot turns at most
  ``LIMITS.wz_max`` deg/s: at speed *v* the smallest radius is
  ``v / math.radians(LIMITS.wz_max)``, about 0.2 m at a walk and 0.6 m at a
  run. One arc goes at most ``LIMITS.distance_max`` m. And say what ``arc``
  is *not* for: straight sides with sharp corners (a square, a star) are
  ``move`` and ``turn``.

**What** ``python3 lab/test.py todo4`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``arc`` has a description of 20+ characters, and every parameter is described
     - ``"Walk forward and turn at the same time, around part of a circle ..."``
     - ``"arc"``, or a parameter still saying ``"TODO 4"``
   * - Its words contain ``LIMITS.distance_max`` (``7.0``) and ``"follow"``
     - ``f"... One arc goes at most {LIMITS.distance_max} m." + ENDS_FOLLOW``
     - ``"Walk in a curve."``
   * - ``arc("left", angle=90, radius=0.5)`` sends ``vx = +0.4``, ``vy = 0``, ``wz = +45.8`` deg/s, and turns about 90° (70–110)
     - ``wz_deg = math.degrees(speed / radius)``
     - ``vx = 0`` (a turn in place); or ``wz_deg = speed / radius`` (0.8: that's rad/s, and ``check_move`` refuses the 112 s it then takes)
   * - ``arc("right", angle=90, ...)`` sends a negative ``wz``, still going forward
     - ``wz_deg = -45.8``, ``vx = +0.4``
     - ``vx = -0.4`` (that backs up)
   * - ``arc("left", angle=90, speed=0.6, radius=1.0)`` sends ``vx = 0.6``, ``wz = +34.4`` deg/s
     - ``vx = speed``
     - ``vx = 0.4`` (the speed ignored)
   * - ``arc("left", angle=90, radius=0)`` turns in place, left, about 90°, and the reply says ``"place"``
     - ``"Radius 0: turned in place (a sharp corner). move vx=0 ... queued"``
     - a crash (dividing by 0), or a refusal
   * - ``arc("left", angle=90, radius=-1)`` is a failure that says ``"radius"``, and sends nothing
     - ``"no arc: radius must be 0 m or more ..."``
     - a success, or a crash
   * - ``arc("left", angle=90, radius=0.1)`` (229 deg/s) fails, says ``"radius"``, and gives the smallest radius (0.2–0.3 m)
     - ``"too tight: at 0.4 m/s the radius must be at least 0.25 m"``
     - ``check_move``'s ``"too fast: ... |wz|<=114.0 deg/s"``; or a success that quietly turned slower

Check it: ``python3 lab/test.py part2`` (TODOs 2–4). Then try your words in
text: ``python3 lab/chat.py``, and ask for a run, a sidestep, a turn,
*"run in a circle around me"* (did Gemini pick a radius the robot can do?), and
a shape: *"walk in a rectangle"*, *"walk a star"*, *"trace the letter D"*. You see
the exact call Gemini makes and the speeds your code sends the (fake) robot. If
Gemini struggles with shapes, one sentence in your prompt's *How you move* can
help: shapes and letters are several calls in a row (a straight line, a sharp
corner, a curve), planned first and then called in order.
Last, restart (``robot/stop.sh && robot/start.sh``), reconnect in the app, and
try it by voice.

.. note::

   **Checkpoint: a refusal Gemini can explain.** Get Gemini to call your
   motion tool with a request the robot refuses, and explain the refusal to
   you using *only the robot's reply*. If your descriptions are so good that
   Gemini never asks for anything impossible, great: try something that fits
   the tool but breaks another rule. "Run forward for ten seconds" is more
   than 7 m. Show a TA the conversation and the debug view.

**DELIVERABLE (Q3):** Your ``turn``, ``move`` and ``arc`` descriptions, and one
sentence each on a choice you made (which parameters, which defaults, what the
description promises). Where is "run means at least 1.0 m/s" enforced, and
why there?

**DELIVERABLE (Q4):** About the finished ``walk``. (a) Why does it send the
speed 20 times a second, and what would Pupper do if it sent it only once?
What would happen if it waited with ``time.sleep`` instead of ``await
robot.sleep``? (b) Ask Pupper for a 5-second walk. What does Gemini hear right
away, and what does it believe at that moment? When, if ever, does it learn
that the walk failed (say, Pupper fell over halfway)? Describe one situation
where this matters, and one way you could fix it.

**DELIVERABLE (Q5):** Trace one voice command, *"stand up"*, from Gemini to
the controller switch. The first and last steps are given; fill in every file
and function in between, and mark each step *now* (before the reply goes back
to Gemini) or *later* (when the queue gets to the job):

.. code-block:: text

   1. web/services/geminiLive.ts   a toolCall arrives; the app calls robotBridge.callTool("stand_up", {})   now
   2. ...
   ...
   n. robot/robot_api.py           Robot.switch_to(LOCOMOTION_CONTROLLER): the locomotion policy takes the legs   later

Use a coding agent or read the code yourself (start at the ``stand_up`` tool
in ``lab/basics.py``, and work outward in both directions). Check your trace
against the robot server's log (``/tmp/gemini-pupper/server.log``). If an
agent helped, point out anything it got wrong, skipped or made up.

Part 3: Follow
--------------
Follow mode is the person follower you built in lab 5: camera → person
detection → your state machine → walking, with your target lock and stop
distance. It runs without lab 5's obstacle avoidance, which is not stable
enough to trust while Gemini is driving: Pupper follows you straight, and it is
up to you to lead it around furniture. Follow mode needs one rule:

   *Questions and conversation keep following. Any motion (walk, turn, arc,
   trick, stand up, lie down) ends follow mode, and Pupper says so first:
   "Let me stop following you and ..."*

.. note::

   **What follow mode needs.** When Gemini follows someone, it runs your
   ``lab/follow_me.py`` state machine, with your target lock and stop
   distance, through ``robot/follow_service.py`` in ``~/follow_me_pupper``.
   Keep that repo on the Pupper with its model downloaded, and Lab 5's Steps
   2–4 working (``python3 lab/test.py`` all passing): follow mode runs your
   ``process_detections``, ``pick_target`` and ``approach_speed``, and crashes
   on a function that is not written yet. Lab 5's optional steps stay off:
   color re-identification (Step 5) and obstacle avoidance (Step 6) are not
   stable enough to run while Gemini is driving. The Hailo only runs while
   Pupper is following, which keeps it cooler, so Pupper's on-screen eyes
   track people only while it follows. For eyes that track people all the
   time, start with ``FOLLOW_ALWAYS_DETECT=1 robot/start.sh`` (Pupper runs
   warmer). Without ``~/follow_me_pupper``, follow mode falls back to the
   monorepo's simpler follower.

Turning the follower on and off is provided (``robot.set_following``). The
rule is yours, in two halves: the code that makes it true, and the words that
let Gemini talk about it.

**TODO 5:** ``prepare_to_move`` in ``lab/part3_follow.py``. It gets the
robot ready for a new motion, and every motion calls it first, in its *now*
part: look for it in ``stand_up`` and ``lie_down`` (``basics.py``) and
``walk`` (``part2_walk.py``). So whatever you write here applies to every
motion at once.

*What it is for.* ``prepare_to_move`` stops anything that runs on its own,
outside the queue, before a new motion is queued. In Gemini Pupper that is
one thing, the person follower, so its one job is to end follow mode. But it
is the hook for anything like it you add later. Getting the *legs* ready is a
different job, done later, inside the job, by ``switch_to``: the state when
the job runs may not be the state when it was queued.

.. list-table::
   :header-rows: 1
   :widths: 40 26 34

   * - If you add…
     - put it in…
     - because
   * - A behavior that runs by itself, outside the queue: a patrol or
       look-around loop, dancing to music, head tracking that sends speeds
     - ``prepare_to_move`` (now)
     - a new request should end it at once, like the follower
   * - A new way to get the legs into or out of something: a trick that
       must be left a special way, a pose that needs a warm-up
     - ``switch_to``, or the trick's exit clip (later)
     - it depends on the state when the job runs, not when it was asked

*Why right away, and not in the job?* Say Pupper is following you and two
walks are queued. You say "sit". If following only ended when the queue got
to the sit, Pupper would keep chasing you for several seconds after you asked
it to stop. The person asked for something else *now*, so it ends now.

*Why a note?* Gemini can only say "Let me stop following you and …" if the
reply tells it that's what happened. The note goes in front of the tool's
own reply (``with_note`` in ``stand_up`` puts it there).

1. If ``robot.following`` is false, there's nothing to do: return ``""``.
2. ``await robot.stop_following()`` turns the person follower off.
3. ``robot.send_speed(0, 0, 0)``: stand still (the follower may have been
   walking).
4. Return the note, e.g. ``"Stopped following the person first."``
   (``with_note`` puts it in front of the tool's own reply).

**What** ``python3 lab/test.py todo5`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``stand_up``, ``lie_down`` and ``walk`` stop the follower right away, before the queue gets to them
     - ``await robot.stop_following()`` in ``prepare_to_move``
     - stopping it inside a job
   * - Their reply contains ``"follow"``
     - ``"Stopped following the person first. stand up queued; ..."``
     - ``"Done. stand up queued; ..."``
   * - When it isn't following, the reply does *not* contain ``"follow"``
     - ``"stand up queued; ..."``
     - the note every time (return ``""`` when not following)
   * - ``robot_status`` (a question) keeps following
     - following is still on afterwards
     - calling ``prepare_to_move`` from a question

Check it: ``python3 lab/test.py todo5``

**TODO 6:** The ``follow`` tool at the bottom of ``lab/part3_follow.py``, and the
*Follow mode* section of ``lab/prompt.md``.

- The body is one line: ``return await robot.set_following(enabled)``.
- Its description tells Gemini what follow mode is, what keeps it on
  (talking, questions like "what do you see?") and what ends it (any motion
  tool).
- The *Follow mode* section of the prompt tells Gemini what to *say* before a
  motion ends follow mode.

``ENDS_FOLLOW``, the sentence you added to every motion tool's description in
TODOs 3 and 4, is the third place the rule now lives.

**What** ``python3 lab/test.py todo6`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``follow`` has a description of 20+ characters, and ``enabled`` is described
     - ``"true to start following, false to stop."``
     - ``"TODO 6"``
   * - Its description says what ends it (contains ``"end"`` or ``"stop"``)
     - ``"... any motion tool (move, turn, arc, trick, stand up, lie down) ends it."``
     - ``"Follow the nearest person."``
   * - ``prompt.md``'s *Follow mode* says what ends it (``"end"`` or ``"stop"``)
     - "Any motion ends follow mode. Before you call that tool, say so."
     - "You can follow people."
   * - ``follow(enabled=True)`` starts the follower; ``enabled=False`` stops it
     - ``return await robot.set_following(enabled)``
     - ``robot.set_following(True)`` every time

Check it: ``python3 lab/test.py part3``, then in text (``python3 lab/chat.py``:
*"follow me"*, *"what do you see?"*, *"now sit"*), then restart and try it on
the robot.

**DELIVERABLE (Q6):** The follow rule now lives in three places: the prompt,
the tool descriptions, and ``prepare_to_move``. Make a table: for each
place, does it *tell* Gemini about the rule or *enforce* it? Then make
``prepare_to_move`` return ``""`` without stopping anything, and ask
Pupper to sit while it follows you. What happens? (Put it back.)

**DELIVERABLE (Q7): Video 1**, by voice only: Pupper walks forward and
backward, turns both ways, sidesteps, runs, and walks a circle; refuses one request and
explains why (the refusal checkpoint); then follows you, keeps following while
you ask it a question, and tells you it will stop following before it does
something else.

Part 4: Tricks
--------------
Every trick is a policy, played by its own controller: the 14 tricks in
``tricks/`` (``sit``, ``push_up``, ``swim``, …) were recorded on the Pupper
monorepo and trained as trick policies in simulation. Each folder has a
``policy.json`` and a ``trick.yaml`` (``robot/tricks.py`` lists the fields):
what Gemini reads about the trick, and the pose it starts and ends in. The
launch file loads one controller per trick, called ``trick_<name>``, all
switched off.

**Two kinds of trick.** A **playback trick** plays one reference motion from
start to finish; its trick policy keeps Pupper balanced while it plays, so it
holds up where a plain replay of the recording would wobble or fall. An
**interactive trick** is one Pupper *stays in* and that takes requests: it is
made of several short pieces of motion the robot plays one at a time, on
request. Compare ``sit``, which you already have,
with ``sit_stay_shake``, which you will train in Part 5:

.. list-table::
   :header-rows: 1
   :widths: 22 39 39

   * -
     - ``sit``: a playback trick
     - ``sit_stay_shake``: an interactive trick
   * - What it is
     - One recorded motion: sit down, hold for two seconds, stand back up.
       It always plays the whole thing.
     - Five pieces of motion, called **clips**: ``sit_down``, ``sit_idle``
       (sitting still), ``shake_left``, ``shake_right`` and ``stand_up``.
   * - When you play it
     - It sits, holds for two seconds, and stands up again, all by itself.
     - It plays ``sit_down``, then ``sit_idle`` over and over: Pupper **stays**
       sitting, for as long as you like.
   * - While it plays
     - Nothing to ask for: it's busy.
     - Gemini can ask for a clip by name: ``interactive_action("shake_left")``
       plays ``shake_left``, then goes back to ``sit_idle`` (Part 5).
   * - How it ends
     - By itself, standing.
     - When Gemini asks for ``stand_up``, its **exit clip**, or when any other
       motion is asked for (``switch_to`` plays the exit clip first).
   * - In the code
     - ``trick.interactive`` is ``False`` (``trick.clips`` is empty)
     - ``trick.interactive`` is ``True``: ``trick.clips`` lists the five clips; ``trick.actions`` the ones Gemini
       may ask for; ``trick.exit_clip`` is ``"stand_up"``

Until Part 5 you have no interactive trick, but ``play_trick`` has to handle them
already. The tests use a made-up one called ``hold_pose``.

**Playing a trick is a change of state**, from locomotion mode to that trick
(trick mode). Before, the locomotion policy has the legs. You give them to
``trick_<name>``, wait while it plays, and then decide which state Pupper is
in afterwards:

.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - The trick…
     - The state afterwards
     - Examples
   * - is a playback trick that ends standing (``end_pose: standing``)
     - ``LOCOMOTION``: the locomotion policy has the legs again, so Pupper can
       walk away
     - ``sit``, ``push_up``, ``sneeze``
   * - is a playback trick that ends lying down (any other ``end_pose``)
     - the trick: its controller holds the last pose until the next motion
     - ``swim``, ``twerk``, ``upward_dog``
   * - is an interactive trick
     - the trick: its controller keeps playing its looping clip (the "stay")
     - ``sit_stay_shake``

**TODO 7:** ``play_trick`` in ``lab/part4_tricks.py``: the *now* part, the
same shape as ``walk``. Its job, ``play_policy_trick``, is finished: you read
it next.

1. ``trick = robot.tricks.get(name)``. If it's ``None``, fail, and list the
   real names (``list(robot.tricks)``), so Gemini can pick a real one.
2. ``note = await prepare_to_move(robot)``.
3. The job is ``play_policy_trick(robot, trick)``. You don't want to run it
   now, so wrap it in a one-line function: ``job = lambda:
   play_policy_trick(robot, trick)``.
4. Queue it with ``robot.queue(f"trick {name}", job, trick.total_time,
   then=...)``, with the note in front. ``then`` is the state Pupper is in
   once the job is done, from the table above: ``LOCOMOTION`` after a
   playback trick that ends standing, else ``trick``. Part 5 relies on it:
   it's how "sit, then shake" said in one breath knows the sit is coming.

Examples, *now* (what Gemini hears straight away):

.. list-table::
   :header-rows: 1
   :widths: 30 22 48

   * - Gemini calls
     - ``then``
     - Gemini hears now
   * - ``play_trick("backflip")``
     - (not queued)
     - ``unknown trick 'backflip'; available: ['lie_downward_dog', 'lie_sit_lie', ...]``
       (From Gemini, the tool registry refuses it first, since ``name`` only
       takes the listed names: ``"play_trick: name must be one of [...]"``.
       Your check is the one behind it.)
   * - ``play_trick("sit")``
     - ``LOCOMOTION``
     - ``trick sit queued; the queue should finish in 10.7 s``
   * - ``play_trick("sit_stay_shake")``
     - the trick
     - ``trick sit_stay_shake queued; the queue should finish in 5.5 s``

**Read: play_policy_trick** (finished, in ``lab/part4_tricks.py``), the job
your ``play_trick`` queues. It gives the legs to the trick's controller, waits
while the trick plays, then leaves the robot in the right state, in four steps:

1. An interactive trick Pupper is already in (``robot.state_now is trick``):
   nothing to do.
2. A playback trick that is still holding the end of its last run (Pupper is
   still in its state) is relaxed first, with ``switch_to(None)``. Switching to a
   controller that is already on does nothing at all, so without this, "swim
   again" would just keep lying there.
3. ``switch_to(trick.controller)``, then wait ``trick.total_time + 0.3`` s
   while it plays. If the robot fell, say so.
4. Leave the robot in the right state, using the table above, and say which:
   an interactive trick stays (and the reply lists its actions); a playback
   trick that ends standing hands the legs back to the locomotion policy; any
   other trick just stays where it is.

Every outcome says which pose the robot is in now: Gemini (and
``robot_status``) can only plan the next move if it knows.

A trick policy plays its motion *by itself*: unlike a walk, it doesn't need
speeds sent to keep going. So the provided ``stop`` tool can't stop a trick
that is still playing by standing still; it relaxes the motors instead, like
the gamepad e-stop, and the reply says so. A trick that has finished and only
holds its pose is left as it is.

What it does, *later*, in each case (read the code with this table next to
it, and check that you can see each row happen):

.. list-table::
   :header-rows: 1
   :widths: 24 22 30 24

   * - The job for…
     - when the robot is…
     - it does (steps)
     - and returns
   * - ``sit`` (ends standing)
     - walking
     - 3: switch to ``trick_sit``, wait 11.0 s; 4: switch to ``gemini_walk``
     - ``played sit; back in locomotion mode``
   * - ``swim`` (ends lying)
     - walking
     - 3: switch to ``trick_swim``, wait 11.5 s; 4: nothing (it stays)
     - ``played swim; it holds its final pose (lying)``
   * - ``swim`` again
     - lying at the end of ``swim`` (``trick_swim`` is still on)
     - 2: switch to ``None`` first; then the same as above
     - the same as above
   * - ``sit_stay_shake`` (an interactive trick)
     - walking
     - 3: switch to ``trick_sit_stay_shake`` (``state_now`` becomes the trick),
       wait 5.8 s; 4: nothing (it stays)
     - ``played sit_stay_shake; it now stays sitting. Interactive actions:
       shake_left, shake_right, stand_up. Any other motion first plays
       stand_up.``
   * - ``sit_stay_shake`` again
     - already sitting in it
     - 1: nothing at all
     - ``already doing sit_stay_shake``
   * - ``sit``
     - walking, but it falls over during the trick
     - 3: switch to ``trick_sit``, wait; ``robot.fallen`` is set
     - ``sit ended in a fall: the robot fell over: ...``

**What** ``python3 lab/test.py todo7`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - An unknown trick fails, and the message lists the real names
     - ``"unknown trick 'backflip'; available: ['bow', 'roll_over', ...]"``
     - ``"unknown trick"``
   * - A real trick is accepted right away
     - ``"trick bow queued; the queue should finish in ... s"``
     - awaiting ``play_policy_trick`` yourself (nothing is queued)
   * - The queued job is ``play_policy_trick``: the trick's controller takes the legs for ``trick.total_time``, and its outcomes appear (``"back in locomotion mode"``, ``"holds its final pose"``, relax and restart)
     - ``job = lambda: play_policy_trick(robot, trick)``
     - ``job = play_policy_trick(robot, trick)`` (runs it now, not in the queue)
   * - ``then=``: ``LOCOMOTION`` after a trick that ends standing, else the trick, so asking for an interactive trick again while Pupper is in it does nothing
     - outcome ``"already doing hold_pose"``
     - ``then=LOCOMOTION`` for every trick: it replays
   * - With follow mode on, the reply contains ``"follow"``
     - ``with_note(note, await robot.queue(...))``
     - dropping the note

The fake robot's tricks: ``bow`` ends standing, ``roll_over`` ends lying down, and ``hold_pose`` is an interactive trick (actions ``wave_left``, ``wave_right``, exit clip ``leave``).

Check it: ``python3 lab/test.py todo7``. It runs your ``play_trick`` with the
finished job behind it, on the fake robot, which records every controller
switch and when.

**TODO 8:** The ``play_trick`` tool at the bottom of ``lab/part4_tricks.py``:
what Gemini reads. The body is already written; replace the ``TODO 8`` words.

- Put ``{tricks}`` in the description: the robot turns it into the list of
  installed tricks, each with the description from its ``trick.yaml``. Look
  at the list it makes: ``curl localhost:8000/tools`` on the Pupper.
- Tell Gemini what it can't see from the list: a trick that starts standing
  looks wrong from lying down, and a trick keeps its final pose until the next
  motion.

**What** ``python3 lab/test.py todo8`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``play_trick`` has a description of 20+ characters, and ``name`` is described
     - ``"The exact trick name."``
     - ``"TODO 8"``
   * - Its description contains ``{tricks}`` (every trick's own description appears in it)
     - ``"... \nTricks:\n{tricks}"``
     - typing the trick names yourself (stale when a trick is added)
   * - ``name`` keeps ``enum=TRICK_NAMES``
     - leave the enum as it is
     - removing it
   * - It says a trick keeps its final pose (contains ``"pose"``)
     - "A trick keeps its final pose until the next motion."
     - "Plays a trick."

Check it: ``python3 lab/test.py part4``, then ask for tricks in text
(``python3 lab/chat.py``): does Gemini pick the trick you meant from your
words? Then restart and ask for a few on the robot, some that end standing and
some that end lying down.

**DELIVERABLE (Q8):** About the finished ``play_policy_trick``. Ask for
``swim`` twice in a row: why does a trick that is holding its own end pose
need to be relaxed before it plays again? And why should every reply say which
pose the robot ends in: what would Gemini get wrong without it?

Part 5: Sit, Stay, Shake
------------------------
Pupper already knows ``sit``: it sits like a dog, holds for two seconds, and
stands right back up. By the end of this part, Pupper will **sit when told,
stay sitting for as long as you like, shake whichever paw you ask for, and
stand up when you say so**, all by voice.

.. note::

   Once you fetch your ``sit_stay_shake`` (end of Part C), **it takes over
   "sit"**. The old ``sit`` (sit, hold two seconds, stand back up) and
   ``stand_sit_shake_sit_stand`` step aside: Gemini no longer sees them. So
   if Pupper now sits and *stays* where it used to stand right back up,
   that's your new trick, not a bug. To get the old ones back, move
   ``tricks/sit_stay_shake`` out of ``tricks/`` and restart
   (``robot/stop.sh && robot/start.sh``).

You won't design any new poses. You get two finished motions, *sit down and
stand up* and *shake the left paw while sitting*, and you cut and join them
into a **clip library**: named pieces of motion the robot plays one at a time,
on request.

**What is a clip, and why cut a motion into pieces?** A reference motion is
one long recording: the body's pose and the 12 joint angles, frame by frame.
A playback trick plays its whole recording, start to finish, every time. But
"stay until I say so" and "shake when I ask" can't be one recording: nobody
knows in advance how long the stay lasts, or which paw comes next. So you cut
the recording into short named pieces, **clips** (sit down, sit still, shake
the left paw, ...). Each clip starts and ends at a pose it shares with its
neighbours, so any clip can follow the one before it without a jump. On the
robot, the trick plays one clip at a time, and *which clip comes next* is
chosen while it runs: the looping clip repeats until Gemini asks for another
one (``interactive_action``). For training, you join the clips back together into
one long reference motion that passes through every change the robot can make
on request, so the policy has practiced each one.

Then you train one **tracking policy** on the whole library,
BeyondMimic-style: in simulation, the policy watches the reference motion and
is rewarded for following it, the same idea as lab 4's reference gaits.

Part A: Build the Clip Library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Open the `CS123 Sit, Stay, Shake notebook <https://colab.research.google.com/github/cs123-stanford/pupper-mjlab/blob/main/notebooks/CS123_Pupper_Sit_Stay_Shake.ipynb>`_
in Colab, **File → Save a copy in Drive**, and pick the same G4 GPU runtime
as lab 4. Log into W&B with the **same account your Pupper is logged into**:
W&B carries your motion to training, and your policy to the robot. (Prefer
your laptop? The same TODOs are in ``tricks/sit_stay_shake.py`` in
`pupper-mjlab <https://github.com/cs123-stanford/pupper-mjlab/tree/main/src/mjlab/tasks/tracking/config/pupper>`_.)

This is the library you build:

.. figure:: ../../../_static/lab6/clip_graph.svg
   :align: center
   :width: 95%

   ``sit_stay_shake`` as the state machine inside the trick. Every clip starts
   and ends at a pose it shares with its neighbours, so the robot can go
   straight from the end of one clip to the start of the next: ``sit_idle``
   loops until something is asked for, each shake goes back to ``sit_idle``,
   and ``stand_up`` leaves. **To Gemini Pupper, this whole graph is a single
   state**, the trick ``sit_stay_shake``: ``robot.state_now`` is the trick from
   ``sit_down`` to the end of ``stand_up``, whichever clip is playing. Any
   interactive trick you design works the same way: a few clips that share a
   pose, one that loops, actions that return to it, and an exit clip (the
   spider's are ``spider_down``, ``spider_idle`` and ``spider_up``).

Fill in the four TODOs in the notebook:

1. ``cut``: a piece of a motion between two times. Cut the sit into
   ``sit_down`` and ``stand_up``.
2. ``hold``: stand still in one pose. This is ``sit_idle``, the **stay**.
3. ``mirror``: swap left and right, to get ``shake_right`` from
   ``shake_left``. You will need to work out how Pupper's left and right
   motors are mounted: does a mirrored leg copy the other leg's angles, or
   flip some signs? Your lab 2 kinematics helps.
4. **The clip table and the training order.** Which clip loops? What plays
   after each one? And in what order are the clips joined for training, so
   the policy sees every change the robot can make?

The notebook checks your code and prints ``ok`` or ``FAIL`` for each check.

**DELIVERABLE (Q9):** Draw your clip graph, like the figure above: every clip,
the pose it starts and ends in, which one loops, and each clip's ``next``. Why
must clips meet at matching poses: what would the robot do at a mismatched
boundary? And your training order: why does every change the robot can make on
request need to appear in it?

**DELIVERABLE (Q10):** Your ``mirror``. Which joint angles did you negate, and
why does Pupper need that? How did you check it?

Part B: Watch It Fall, Then Train It
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Keep going in the notebook:

* **Replay with physics.** The robot plays the joined motion as a fixed
  recording, with no policy. Expect it to fall.
* **Upload** the motion to W&B, and **train** one policy on the whole library
  (3,000 iterations, about 18 minutes: work on Part C's code meanwhile).
* **Score** your policy with the same physics check, and watch it in the
  simulation viewer.

**DELIVERABLE (Q11):** The plain-replay verdict and worst errors, next to your
policy's. In which clip does plain replay fail, and why can't a fixed recording
recover from it? And in your own words: why did the "stay" not need a policy
trained on a longer sit? What does the policy observe, frame by frame, that
makes an open-ended stay possible?

Part C: Give Gemini the Actions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
An interactive trick is one Pupper stays in and that takes requests.
``play_trick`` (your TODO 7) already sits and stays; the requests (*shake*,
*the other paw*, *stand up*) are **interactive actions**, and they only make sense
while Pupper is in that trick. Until an interactive trick is installed, Gemini
is told the ``interactive_action`` tool has nothing to choose from.

**TODO 9:** ``interactive_action`` in ``lab/part5_sit_stay_shake.py``. You can write
and test it while your policy trains: the tests use a made-up interactive
trick called ``hold_pose`` (actions ``wave_left``, ``wave_right``, and the exit
clip ``leave``).

*The new idea: an action only makes sense in one state.* A shake is a clip of
``sit_stay_shake``: it can only play while Pupper is in that trick. So
``interactive_action`` has to ask "is Pupper sitting?". The interesting part is
*when* to ask, because of the queue:

- Someone says "sit, and give me your paw" in one breath. Gemini calls
  ``play_trick`` and ``interactive_action`` back to back, before Pupper has even
  started to sit. If ``interactive_action`` asked "is it sitting *right now*?", it
  would refuse a perfectly good request. So, **now**, it asks: "will it be
  sitting once everything already queued has run?" That's
  ``robot.state_after_queue``, kept up to date by the ``then=`` every motion
  passes to ``robot.queue``.
- But a lot can happen between the call and the shake: Pupper can fall over,
  someone can press the e-stop or say "stop". So **later**, in the job, it
  asks again, about right now: ``robot.state_now``.

*Now*, when Gemini calls:

1. ``found = robot.action_ids().get(action)``. ``action_ids()`` maps each
   action's name to ``(trick, clip)``. If it's ``None``, fail, listing the
   real names (``list(robot.action_ids())``).
2. ``trick, clip = found``
3. If ``robot.state_after_queue is not trick``, fail, and say which trick to
   play first: ``f"... play it first (play_trick {trick.name})"``.

*Later*, in ``async def job()``:

4. If ``robot.state_now is not trick``, fail the same way.
5. The exit clip (``clip == trick.exit_clip``) leaves the trick: hand the
   legs back with ``await robot.switch_to(LOCOMOTION_CONTROLLER)``, which plays
   the exit clip on the way out, and report how that went.
6. Any other clip: ``return await robot.play_clip(trick, clip)``. It plays the
   clip and waits until it is done; Pupper stays in the trick.

*Queue it*:

7. ``robot.queue(f"{trick.name} {clip}", job,
   trick.clip_time_estimate(clip), then=...)``: ``LOCOMOTION`` after the exit
   clip (it stands up), else ``trick`` (it keeps sitting).

.. note::

   **Gemini still isn't told when the shake finishes.** It hears the reply
   from step 1, 3 or 7, right away. The job's result (the shake played, or
   the sit was gone by then) goes to the debug log and ``robot_status``.

**What** ``python3 lab/test.py todo9`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - An unknown action fails, and the message lists the real ones
     - ``"unknown interactive action fetch_the_ball; available: ['wave_left', 'wave_right', 'leave']"``
     - ``"unknown action"``
   * - An action while Pupper isn't in its trick (or queued to be) fails right away, names the trick, and queues nothing
     - ``"wave_left only works while hold_pose is running; play it first (play_trick hold_pose)"``
     - ``"not sitting"`` (which trick?); or queuing it and failing later
   * - In the trick: accepted, the clip plays, nothing fails, and Pupper stays in the trick
     - ``"hold_pose wave_left queued; ..."``
     - switching back to locomotion after every clip
   * - The exit clip plays, then locomotion takes the legs back, and ``state_now`` is ``LOCOMOTION``
     - outcome ``"played leave; back in locomotion mode"``
     - ``play_clip`` for the exit clip too (Pupper stays on the trick's controller)
   * - After the exit clip, actions are refused again
     - ``then=LOCOMOTION`` when you queue the exit clip
     - ``then=trick``
   * - "Sit, then shake" in one breath: an action queued right behind its trick is accepted and plays
     - the *now* check asks ``robot.state_after_queue``
     - asking ``robot.state_now`` or ``is_active`` *now*
   * - If the trick was relaxed before the queue got to the action, the job refuses and sends no clip
     - the job first checks ``robot.state_now is trick``
     - checking only *now*

Check it: ``python3 lab/test.py todo9``

**On the robot:**

1. **TA gate.** Show a TA your policy's score and the simulation viewer.
2. Fetch the policy on the Pupper with the command the training log printed,
   naming it ``sit_stay_shake``:

   .. code-block:: bash

      python3 robot/get_trick.py <entity>/mjlab/<run-id> --name sit_stay_shake

   From now on, the old ``sit`` and ``stand_sit_shake_sit_stand`` step aside:
   their ``trick.yaml`` says ``replaced_by: sit_stay_shake``, so once yours is
   installed they get no controller and Gemini no longer sees them. A plain
   "sit" now goes to your interactive trick: Pupper sits and stays until you ask it
   to shake or stand up.

**TODO 10:** What Gemini reads about the actions.

- Fill in ``tricks/sit_stay_shake/trick.yaml`` (``get_trick.py`` wrote a
  template): the trick's ``description``, the ``actions`` Gemini may ask for
  while Pupper sits (with a description each), the ``exit_clip``,
  ``end_pose: sitting``, and ``max_body_angle_deg: 60`` (the sit leans back
  about 35 degrees).
  Gemini reads the ``description`` and the action descriptions word for word,
  so write them for Gemini: what the robot does, and what it does after
  (keeps sitting, or stands up). Say when to use the trick, too: when someone
  just says *"sit"*, they mean sit and stay, and Gemini picks a trick from
  these words alone (``stand_sit_stand`` also sits, briefly).
- The ``interactive_action`` tool at the bottom of ``lab/part5_sit_stay_shake.py``.
  Its body is one line, ``return await interactive_action(robot, action)``. Its description: put
  ``{actions}`` in it, and the robot turns it into the list of actions with
  your descriptions. Then tell Gemini what has to happen first (the trick),
  and what ends it (any other motion).
- ``python3 lab/test.py part5``. Try *"sit, and give me your paw"*
  in text first (``python3 lab/chat.py``: you see both calls, back to back,
  before the sit has happened), then restart and say it to Pupper in one
  breath.

**What** ``python3 lab/test.py todo10`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - ``interactive_action`` has a description of 20+ characters, and ``action`` is described
     - ``"The exact action name."``
     - ``"TODO 10"``
   * - Its description contains ``{actions}`` (each action's description appears in it)
     - ``"... Actions:\n{actions}"``
     - typing the action names yourself
   * - ``action`` keeps ``enum=ACTION_IDS``, and calling the tool plays the clip
     - ``return await interactive_action(robot, action)``
     - removing the enum
   * - ``trick.yaml`` (once fetched) has no TODO left, and its ``description`` has 20+ characters
     - ``description: "Sit, and stay sitting until asked to stand. Use it when someone says sit."``
     - ``description: "sit"``
   * - ... and every action is described in 10+ characters
     - ``shake_left: "Shakes the left paw, then keeps sitting."``
     - ``shake_left: "left"``

Until you fetch your policy, the ``trick.yaml`` checks are skipped.

**DELIVERABLE (Q12):** Ask for a paw shake while Pupper is standing. What does
it do, and what does it say? Which of your two checks fired? Then describe a
case where the *other* check (in the job) is the one that matters.

Part 6: The Spider: A Trick That Walks
--------------------------------------
``sit_stay_shake`` holds one pose. Some tricks are a *place to be*: Pupper
lies down like a spider, legs splayed wide and torso just off the ground,
wiggles its legs, **walks around like that**, and wiggles its way back up to
standing when you are done.

.. note::

   Once you fetch your ``spider_walk`` (Part C), **it takes over "spider"**.
   The old playback ``spider`` (lie down, wiggle, lie still) steps aside:
   its ``trick.yaml`` says ``replaced_by: spider_walk``, so it gets no
   controller and Gemini no longer sees it. To get it back, move
   ``tricks/spider_walk`` out of ``tricks/`` and restart
   (``robot/stop.sh && robot/start.sh``).

The locomotion policy can't do this: it was trained to walk standing up. So
the spider is two policies:

* **The spider's clip policy**, like ``sit_stay_shake``: three clips cut from a
  recorded spider. ``spider_down`` (stand, lie down, wiggle, settle into the
  spider stance), ``spider_idle`` (the stance) and ``spider_up`` (wiggle, stand
  back up).
* **The spider walk**, a walking policy like your lab 4 one, but trained in the
  spider stance: it takes the same speed commands, and walks low and splayed.

.. figure:: ../../../_static/lab6/spider_graph.svg
   :align: center
   :width: 95%

   ``spider_walk`` as the state machine inside the trick, like
   ``sit_stay_shake``'s in Part 5. Two policies take turns holding the legs:
   the clip policy plays ``spider_down`` and ``spider_idle``, hands the legs
   to the walking policy, and takes them back at ``spider_up``. **To Gemini
   Pupper, this whole graph is still a single state**, the trick
   ``spider_walk``.

**This is why an interactive trick is a state, not a motion.** Pupper is in
the spider trick from the moment it lies down until it is standing again, even
though three different policies hold its legs on the way. Everything you wrote
already works with it, unchanged:

* ``play_trick`` (your TODO 7) queues the trick with ``then=trick``: Pupper
  will be *in* it. ``play_policy_trick`` plays its opening clips, then (step 4)
  hands the legs to the trick's walking policy.
* ``walk`` (finished, Part 2: read it again) asks ``robot.state_after_queue``
  which policy walks, the same now-question as your TODO 9: in the spider
  trick, its walking policy does. So ``move``, ``turn`` and ``arc`` walk the
  spider, slowed to the speeds it was trained for (and longer, so it still
  goes as far), and Pupper stays in the trick.
* Any other motion leaves it, through its exit clip: ``switch_to`` gives the
  legs back to the clip policy, started at ``spider_up`` (the controller's
  ``start_clip``), then to whatever comes next. ``interactive_action("spider_up")``
  (your TODO 9) does the same on purpose.

You train both policies in one notebook, the
`CS123 Spider notebook <https://colab.research.google.com/github/cs123-stanford/pupper-mjlab/blob/main/notebooks/CS123_Pupper_Spider.ipynb>`_
(**File → Save a copy in Drive**, GPU runtime). The same TODOs are in
``tricks/spider.py`` in
`pupper-mjlab <https://github.com/cs123-stanford/pupper-mjlab/tree/main/src/mjlab/tasks/tracking/config/pupper>`_.

Part A: The Spider's Clips
^^^^^^^^^^^^^^^^^^^^^^^^^^
You get the recorded spider (11 s: stand, lie down, wiggle, lie still) and
``settle_into_stance``, which blends a motion's last frame into the spider
stance the walking policy starts from. Paste your ``cut`` and ``hold`` from
Part 5, then:

* **S1, reverse:** the motion played backwards: every array's frames in
  reverse order.
* **S2, the clips and ORDER:** ``spider_down`` is the recording up to 7 s
  (still wiggling), settled into the stance. ``spider_idle`` holds that
  stance. And ``spider_up``? It has to go from the stance back to standing,
  wiggling on the way, and you already have a motion that does exactly that,
  backwards.

Replay the library with physics: it **falls**. Lying down is easy to play
open loop; pushing back up from the belly is not. Upload it and train it, as
in Part 5.

**DELIVERABLE (Q14):** Your ``reverse`` and your clips. Why can ``spider_up``
be ``spider_down`` played backwards, and why doesn't the plain replay survive
it? Name a clip from Part 5 that could *not* be made by reversing another.

Part B: The Spider Walk
^^^^^^^^^^^^^^^^^^^^^^^
This is lab 4 again, in a new stance. ``Mjlab-SpiderWalk-Flat-Pupper-v3``
tracks a **reference gait**, a spider trot in place: the diagonal leg pairs
take turns stepping, twice a second, low and splayed. The leg motion comes
from the recorded spider: the front legs fold the knee and swing up and out,
the back legs swing up. At zero command the reference is the still stance;
the velocity command decides where the steps take the body.

.. figure:: ../../../_static/lab6/spider_gait.gif
   :align: center
   :width: 70%

   The spider reference gait, played open loop in the viewer: the joint
   angles the policy is rewarded for tracking, two cycles a second, torso
   1 cm off the ground.

All you set are the **reward weights**, the same terms as lab 4. Start from
your StableGait weights, and think about what is different here: the height
target is 4.2 cm, not 14.5; the motors alone can't hold this low stance (with
zero actions it sags onto its belly), so holding the body up is something the
policy has to learn; and ``abduction_angle`` pulls joints toward the zero the
spider splays away from on purpose. Train, then watch it in the viewer: low
(never on its belly), legs stepping like the reference, going where it is
told. It will be slower than commanded: the spider creeps.

**DELIVERABLE (Q15):** Your reward weights, next to your lab 4 StableGait
weights. Which ones did you change, and why? A short clip or screenshots of
your trained spider walk in the viewer.

Part C: On the Robot
^^^^^^^^^^^^^^^^^^^^
1. **TA gate.** Show a TA both policies in the viewer.
2. Fetch both into one trick on the Pupper, then restart:

   .. code-block:: bash

      python3 robot/get_trick.py <clip-trick-run> --walk <spider-walk-run> --name spider_walk
      robot/stop.sh && robot/start.sh

3. Fill in ``tricks/spider_walk/trick.yaml``: the ``description`` (say that
   ``move``, ``turn`` and ``arc`` walk slowly in the spider stance, and how to
   leave it), ``actions`` with ``spider_up`` (and what it does),
   ``exit_clip: spider_up``, ``end_pose: spider``, and the ``walk_policy:
   walk_policy.json`` line ``get_trick.py`` wrote. The old ``spider`` (lie
   down, wiggle, lie still) steps aside: its ``trick.yaml`` says
   ``replaced_by: spider_walk``.

Try it by voice: *"Be a spider. Walk forward a bit, turn around, come back.
Now stand up."* Watch the debug view: which replies say the move is slower?

**DELIVERABLE (Q16):** Trace *"be a spider, walk forward, then sit"* through
the state machine. For each step: the state Pupper is in, the controller that
holds the legs, and what plays between them. Why does ``move`` keep Pupper in
the spider trick, while ``play_trick("sit_stay_shake")`` leaves it? Where in
the code is each of those decided?

Part 7: Put It Together
-----------------------
**TODO 11:** The last two sections of ``lab/prompt.md``.

- **Tricks.** A trick keeps its final pose until the next motion. Trick
  actions only work while their trick holds its pose. When the person asks for
  several things in a row ("sit, shake, then stand up"), Gemini should call
  the tools in that order: the queue plays them one after the other.
- **Safety.** If someone says stop, Gemini should call ``stop`` right away,
  before saying anything. If Pupper falls over, it can't get up by itself
  (``robot_status`` and every motion tool will say so): it should ask a person
  to set it upright on its feet, and only then call ``stand_up``. The robot
  refuses to stand up until it is upright and still, in code; your prompt
  is what makes Gemini ask for help instead of trying again and again.

**What** ``python3 lab/test.py todo11`` **checks.** Every check, with an example that passes and one that fails. A failing check prints what your code produced and an example that passes.

.. list-table::
   :header-rows: 1
   :widths: 36 34 30

   * - Check
     - Passes
     - Fails
   * - *Tricks* and *Safety* are written
     - your own rules under each heading
     - a heading still says TODO
   * - They say what to do when someone says stop (``"stop"``)
     - "If someone says stop, call stop right away, before saying anything."
     - "Be careful."
   * - ... and when Pupper falls over (``"fall"`` or ``"fell"``)
     - "If you fall over, ask a person to set you upright, then call stand_up."
     - nothing about falling
   * - ... and several requests in a row (``"order"``, ``"row"`` or ``"sequence"``)
     - "If the person asks for several things in a row, call the tools in that order."
     - "Do what they ask."

As in TODO 1, these look for words. The live demo is where you see whether Gemini follows them.

Run ``python3 lab/test.py`` once more: everything passes.

* **Live demo (TA checkoff).** A short, unscripted conversation with Pupper
  that walks, turns and curves, plays a trick, sits and **stays** (wait at least ten
  seconds), shakes both paws, stands up, becomes a spider and walks a little
  in its stance, stands back up, and refuses at least one request with a
  reason.
* **Reflection.** A page, not more:

  - Where did you put each rule: in the prompt, in a tool description, or in
    code? Which one would you move, and why?
  - One time Pupper did something you didn't expect. What did the tool replies
    and the server log tell you, and what did you change?
  - If you used a coding agent: one prompt that worked well, one place it went
    wrong, and how you caught it. Attach short excerpts (no secrets).
  - What would you tell next year's students?

**DELIVERABLE (Q17):** The output of ``python3 lab/test.py`` with every check
passing.

**DELIVERABLE (Q18): Video 2**, your demo conversation (by voice only).

**DELIVERABLE (Q19):** Your reflection.

Optional: The Jump
------------------
Pupper's default locomotion policy can already jump: you have seen it on the
gamepad. The jump isn't a policy of its own: it is a *reference* the locomotion
policy was trained to follow, sent to it once. Give Gemini that jump:

1. Rename ``tricks/jump/trick.yaml.optional`` to ``trick.yaml`` and restart.
2. In ``play_trick`` (``lab/part4_tricks.py``), a trick with
   ``trick.kind == "reference"`` plays with ``robot.play_reference_trick(trick)``
   instead of ``play_policy_trick``.
3. ``python3 lab/test.py optional_jump``, then ask Pupper to jump, with clear
   space around it.

Want more? :doc:`opt-lab-3` teaches Pupper secrets: in words (a password, a
secret handshake), and in its legs (a dance move you describe in a sentence and
train), which it performs only for someone who knows the password.

Working With a Coding Agent
---------------------------
A few habits that help:

* **Give it checks it can run**: ``python3 lab/test.py todo7``, and the
  notebook's (or ``sit_stay_shake.py``'s) ``ok`` / ``FAIL`` checks.
* **Ask for the plan before the code.** "Read the TODO 9 card in
  ``lab/part5_sit_stay_shake.py`` and tell me how an interactive action should use the
  queue; don't edit anything yet."
* **Check its claims against the robot.** The server log, the debug view and
  ``curl localhost:8000/system/mode`` are the truth. The agent's summary is
  not.
* **Don't let it weaken a test or raise a limit** to make something pass.
* **Make it teach you.** After each TODO: "Explain this function line by line,
  and tell me what would break if I deleted each check." Then close the agent
  and explain it to your partner. If you can't, you're not done.

Congratulations! Pupper now listens, answers, and does what it is told, and
only what it can do safely: you brought it to life. You built the core of a
robot driven by a foundation model:
tools a model can reason about, honest reports of what happened, state and
preconditions, and a trick pipeline that turns motions into a policy on
hardware. Your final project can build on all of it.

Resources
---------
`Gemini Pupper <https://github.com/sippeyxp/gemini-pupper>`_ by
`Peng Xu <https://scholar.google.com/citations?user=460NWeQAAAAJ&hl=en>`_
(Google DeepMind), and his write-up, *Interactive Robot Dog with Gemini Live*
(``docs/interactive-robot-dog.md`` in your repo)

``lab/chat.py`` is based on Peng's minimal Gemini Live robot-tool example.

`Gemini Live API <https://ai.google.dev/gemini-api/docs/live>`_ and
`function calling <https://ai.google.dev/gemini-api/docs/function-calling>`_

`Gemini Robotics: Bringing AI into the Physical World <https://arxiv.org/abs/2503.20020>`_

`Do As I Can, Not As I Say: Grounding Language in Robotic Affordances (SayCan) <https://say-can.github.io/>`_

`Code as Policies: Language Model Programs for Embodied Control <https://code-as-policies.github.io/>`_

`BeyondMimic: From Motion Tracking to Versatile Humanoid Control <https://arxiv.org/abs/2508.08241>`_

`DeepMimic: Example-Guided Deep Reinforcement Learning of Physics-Based Character Skills <https://arxiv.org/abs/1804.02717>`_

`Pupper tricks: design, replay, train <https://github.com/cs123-stanford/pupper-mjlab/tree/main/src/mjlab/tasks/tracking/config/pupper>`__ (the README behind the notebooks)

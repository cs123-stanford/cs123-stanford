Lab 6: A Foundation Model on a Leash
====================================

*Goal: let a foundation model drive Pupper, safely. Gemini decides what to
do; the tools you write decide what it can do, and tell it what really
happened. Then train Pupper to sit, stay and shake a paw, and give Gemini
those moves too.*

.. TODO(staff): add this offering's lab slides link
.. TODO(staff): add this offering's lab document link

`Lab slides <#>`_ (TODO)

`Lab document <#>`_ (TODO): make a copy, answer each DELIVERABLE under its
question number (Q1–Q22), and submit it to Gradescope.

.. note::

   **Gemini Pupper was created by** `Peng Xu <https://scholar.google.com/citations?user=460NWeQAAAAJ&hl=en>`_
   **(Google DeepMind).** This lab is built on his project
   (`sippeyxp/gemini-pupper <https://github.com/sippeyxp/gemini-pupper>`_), and the
   terminal chat you meet first (``robot/chat.py``) is based on his minimal Gemini
   Live example. Thank you, Peng!

Robot foundation models are becoming the "brain" of robots: models like
Gemini can listen, look through a camera, reason about what you want, and
decide what to do. But a foundation model cannot move a single motor. It can
only **call tools**. So here is the picture for this lab:

* **The brain** is Gemini Live: it hears you, sees through the camera and
  talks back.
* **The muscles** are policies: the walking policy from lab 4, and trick
  policies, one of which you will train in this lab.
* **The leash** is the tool layer in between, and you build it. It decides
  which tools exist, what Gemini is told about them, what each one does on
  the robot, what it refuses, and what it reports back.

The idea to keep in mind: *a foundation model is only as good as its tools,
and it only knows what its tools tell it.* A tool that fails silently is a
robot that lies to you. Remember that every time Gemini says "Okay, I'm
walking!" while Pupper stands perfectly still.

AI Use Suggestion
------------------
**We encourage you to use a coding agent in this lab** (Claude Code or Codex),
unlike lab 3. Robots like Pupper are built this way now, and the repo comes
with notes for your agent (``AGENTS.md``). Two conditions:

* **The deliverables are yours.** Answer every DELIVERABLE in your own words,
  from what you did and saw on your robot. Don't hand the questions to your
  agent.
* **Understand everything your agent does.** For every TODO, you should be
  able to explain what the code does, why it is written that way, and what
  would go wrong without it, without your agent open. Read every line it
  writes, and ask it to explain anything you can't explain yourself.

.. important::

   **Quiz 2 tests this.** It asks about the code in this lab: what a TODO
   does, why it checks what it checks, and what happens when something goes
   wrong. If your agent wrote it and you never understood it, the quiz will
   show it.

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
     the first time, show a TA its score and the viewer. A badly trained trick
     can slam a paw or the body into the ground.
   * A coding agent never moves the robot on its own: no ``start.sh``, no
     ``chat.py --robot``, no ``curl`` to a POST endpoint, no ``ros2 topic
     pub``, unless you say yes, right then, standing at the robot.
   * **Pupper overheats.** Sitting and holding poses load the motors hard. If
     Pupper is hot to the touch or a motor faults, stop, relax the motors
     (``robot/stop.sh``), and let it cool.
   * Never put an API key in a file in your repo, a commit, or a chat you
     submit.

Lab roadmap
-----------

.. list-table::
   :header-rows: 1
   :widths: 14 60 26

   * - Part
     - What you do
     - TODOs
   * - 0
     - Setup; meet the loop in your terminal; try what Pupper can already do; trace one command
     - (reading)
   * - 1
     - **Talk:** the prompt, first pass
     - 1
   * - 2
     - **Walk:** refuse unsafe moves, the walking job, the turn and move tools
     - 2, 3, 4
   * - 3
     - **Follow:** follow mode, and the rule that ends it
     - 5, 6
   * - 4
     - **Tricks:** play any trick, and tell Gemini which ones exist
     - 7, 8
   * - 5
     - **Sit, stay, shake:** build a clip library, train it, give Gemini the actions
     - 9, 10 (+ notebook)
   * - 6
     - **Together:** the prompt, second pass; demo; reflection
     - 11

The TODOs are numbered 1–11 and run in the same order as this handout. In the
code, each one sits between ``TODO n: YOUR CODE HERE`` banners, under a short
**card** that says what happens now, what happens later, what Gemini hears,
what must always be true, which helpers to use, and which test checks it.

How Gemini Pupper Works
-----------------------
Read this once now, and come back to it whenever a TODO refers to it.

**Three files are yours.** When Gemini connects, the robot sends it a prompt
and a list of tools. When Gemini calls a tool, the robot runs it and sends
back a reply.

.. figure:: ../../../_static/lab6/tool_loop.svg
   :align: center
   :width: 95%

   The loop you are building. Gemini picks a tool from its *description*;
   your code on the robot does the work and reports back; the report is the
   only thing Gemini learns about what actually happened.

.. list-table::
   :header-rows: 1
   :widths: 28 52 20

   * - File
     - What it is
     - Yours?
   * - ``robot/prompt.md``
     - Gemini's system prompt: who Pupper is and how it behaves.
     - TODO 1, 6, 11
   * - ``robot/tools.py``
     - Every tool Gemini can call. The ``@tool(...)`` line is what Gemini
       reads (a description, and one for each parameter); the function below
       it is a few lines that turn Gemini's words into numbers and call a
       behavior.
     - TODO 4, 6, 8, 10
   * - ``robot/behaviors.py``
     - What the robot does: is the request allowed, do it, report what
       happened. ``stand_up`` and ``lie_down`` at the top are finished
       examples.
     - TODO 2, 3, 5, 7, 9
   * - ``robot/test_tools.py``
     - Tests with a fake robot. ``python3 robot/test_tools.py todo3`` runs one
       TODO's tests; ``part2`` runs a part's.
     - run them
   * - ``robot/robot_api.py``
     - The Robot API (provided): the queue, the leg controllers, falls, follow
       mode, the e-stop. The table at its top lists what you can call.
     - no
   * - ``robot/tool_registry.py``, ``server.py``, the web app
     - Turn your ``@tool`` functions into what Gemini reads; serve the app.
     - no

**Every motion has the same shape: check now, queue a job, report.** Here is
``stand_up``, the finished example at the top of ``behaviors.py``:

.. code-block:: python

   async def stand_up(robot: Robot) -> Result:
       note = await leave_follow_mode(robot)             # (1) NOW: a decision made right away

       async def job() -> Result:                        # (2) LATER: runs when the queue gets here
           return await robot.switch_to(WALKING_CONTROLLER)   # act, and report what happened

       return with_note(note, await robot.queue("activate walking", job, WALK_READY_S))  # (3)

Why a queue? Gemini often asks for several things in one breath ("sit, then
shake"). The queue runs them one at a time, in order, while Gemini keeps
talking. That means a tool call has **two messages at two moments**:

* **Accepted (now).** What the tool returns right away: refused with a
  reason, or queued. *This is what Gemini hears.*
* **Outcome (later).** What the job returns when it finishes: done, fell
  over, controller failed. It goes to the debug log and to ``robot_status``.

The robot is always in exactly one state: **walking** (the ``gemini_walk``
controller), **one trick** (its ``trick_<name>`` controller), or **motors
off**. Every job changes the state with one call, ``robot.switch_to(...)``.

.. admonition:: Python you'll need (read this even if you know Python)
   :class: note

   **async / await.** The robot server does many things at once: it runs a
   walk, answers Gemini, and listens for the e-stop. An ``async def``
   function can pause while it waits (``await robot.sleep(0.05)``, ``await
   robot.switch_to(...)``) and let everything else run meanwhile. Two rules:
   call an ``async`` function with ``await`` in front, and never use
   ``time.sleep``: it freezes the whole server, the ``stop`` tool included.

   **A job is a function that runs later.** In ``stand_up``, ``async def
   job()`` is defined *inside* ``stand_up``, but ``stand_up`` doesn't run it:
   it hands it to ``robot.queue``, which runs it when every job before it has
   finished. The job can still use ``stand_up``'s variables (``robot`` here;
   ``vx`` and ``duration`` in your ``walk``). A ``lambda`` is a one-line
   function: ``job = lambda: play_policy_trick(robot, trick)`` is a job too.

   **Result.** Every tool and every job returns ``Result(ok, message)``:
   ``ok`` is ``True`` or ``False``, and ``message`` is the sentence Gemini (or
   the debug log) reads. Write messages for a person: what happened, or what
   was wrong and what would work instead.

Here is what happens, in order, when someone says *"stand up and walk
forward"* and Gemini calls two tools back to back:

.. list-table::
   :header-rows: 1
   :widths: 12 44 44

   * - Time
     - What runs
     - What Gemini hears / the log says
   * - 0.00 s
     - ``stand_up``: no follow mode to leave; its job is queued.
     - Gemini: *"activate walking queued; the queue should finish in 4.2 s"*
   * - 0.01 s
     - ``walk``: ``check_move`` is happy; its job is queued behind stand_up's.
     - Gemini: *"move vx=0.4 … queued; the queue should finish in 5.7 s"*
   * - 0.01–4.2 s
     - stand_up's job: ``switch_to(gemini_walk)``, wait for the policy.
     - Log: *"activate walking: ok"*
   * - 4.2–5.7 s
     - walk's job: send the speed 20 times a second, then a zero.
     - Log: *"move vx=0.4 …: moved vx=0.4 … for 1.5 s"*

Notice that Gemini heard both replies before Pupper took a single step: it
hears what was **accepted**, and the **outcomes** go to the log.

**The Robot API.** These are the only parts of ``robot_api.py`` you need:

.. list-table::
   :header-rows: 1
   :widths: 42 58

   * - Call
     - What it does
   * - ``await robot.queue(name, job, seconds, then_holding=None)``
     - Put ``job`` (an ``async`` function with no arguments) in line, and
       return the *accepted* Result now. ``then_holding``: the clip trick
       holding a pose once this job is done.
   * - ``await robot.switch_to(controller)``
     - Give the legs to one controller (``None``: relax the motors). Also
       stands up safely after a fall, and plays a sitting trick's exit clip
       before another motion takes the legs.
   * - ``robot.send_speed(vx, vy, wz_deg)``
     - One speed command to the walking policy.
   * - ``await robot.sleep(s)``, ``robot.now()``
     - Wait without freezing the server; the time now. Never ``time.sleep``.
   * - ``robot.fallen``, ``FELL_MESSAGE``
     - Which controller fell over (``None`` while upright), and what to say.
   * - ``robot.following``, ``await robot.stop_following()``
     - Follow mode.
   * - ``robot.holding_now``, ``robot.holding_after_queue``
     - The clip trick holding a pose right now, and once everything already
       queued has run.
   * - ``await robot.is_active(controller)``
     - Is this controller driving the legs right now?
   * - ``await robot.play_clip(trick, clip)``
     - Play one clip of a clip trick, and wait for it.
   * - ``robot.tricks``, ``robot.action_ids()``
     - The installed tricks; trick actions → (trick, clip).

Part 0: Setup
-------------
1. Fork the `gemini_pupper <https://github.com/cs123-stanford/gemini_pupper>`_
   repository to your own GitHub account, following
   :doc:`forking_repositories`, and clone your fork onto the Pupper:

   .. code-block:: bash

      cd ~/
      git clone https://github.com/YOUR_USERNAME/gemini_pupper.git

2. **Your Gemini API key.** Create one in
   `Google AI Studio <https://aistudio.google.com/apikey>`_.

   .. TODO(staff): confirm whether students need billing enabled / course
      credits for the Live API models the app uses, and say so here.

3. **Set it up.** On the Pupper:

   .. code-block:: bash

      cd ~/gemini_pupper
      robot/setup.sh

   It installs what the app needs (without ``sudo``), builds it, and asks for
   your key. The key goes in ``~/.config/gemini-pupper/api_key`` on the
   Pupper, never in the repo. You can run ``setup.sh`` again at any time.

4. **Meet the loop (no robot needed).** Before Pupper, look at the loop
   itself, in your terminal. ``robot/chat.py`` is based on a short example by
   Peng Xu: you type, Gemini answers, and every tool call and every reply is
   printed. Its ``--toy`` mode has a single made-up tool, ``move_robot``, that
   only prints what it would do. On the Pupper (or on your laptop, after
   ``python3 -m pip install --user google-genai pyyaml`` and with your key in
   ``GEMINI_API_KEY``):

   .. code-block:: bash

      python3 robot/chat.py --toy

   Type *"move forward two meters, then back one"*. You will see this:

   .. code-block:: text

      You > move forward two meters, then back one
        -> Gemini calls move_robot({"direction": "forward", "distance": 2})
           [toy robot] moving forward by 2 units...
        <- reply Gemini hears now: {"status": "success", "executed_move": "forward", "distance": 2}
        -> Gemini calls move_robot({"direction": "backward", "distance": 1})
        ...
      Pupper > I moved forward two meters and then back one!

   That is the whole loop the lab is about. Gemini never runs code: it reads
   a description of a tool (``TOY_TOOL`` in ``chat.py``), asks for a call,
   and *your* code runs it (``move_robot``) and sends back a reply. The reply
   is all Gemini ever learns. Two experiments, both in ``chat.py`` (put them
   back afterwards):

   a. **The words drive the call.** Change ``TOY_TOOL``'s description of
      ``distance`` to say it is in *steps*, and ask again. What does Gemini
      send now? What if you remove ``'backward'`` from the description of
      ``direction``, and ask it to back up?
   b. **The reply is all Gemini learns.** Make ``move_robot`` return
      ``{"status": "failure", "reason": "a wall is in the way"}``, and ask for
      a move. Then make it return success without printing anything, as if
      the motors had quietly failed. What does Gemini tell you each time?

5. **Wake Pupper up.** Put Pupper on the ground in clear space, then:

   .. code-block:: bash

      robot/start.sh --kiosk     # the app on Pupper's own screen, with its mic and speaker

   Or run ``robot/start.sh`` and use the app from your laptop: forward the
   port, browse to ``http://localhost:8000``, open **Settings**, paste your
   key, and press **WAKE PUPPER**:

   .. code-block:: bash

      ssh -N -L 8000:localhost:8000 pi@pupper[YOUR_GROUP_NUMBER].local

   Say hi. The **debug view** shows every tool call Gemini makes and what the
   robot said back: keep it open for the whole lab. ``robot/stop.sh`` relaxes
   the motors and stops everything.

6. **What already works.** The provided tools are ``stand_up``, ``lie_down``,
   ``stop`` and ``robot_status``. Try *"stand up"*, *"lie down"*, *"what are
   you doing?"*. Then ask Pupper to walk to you, and to do a trick: those
   tools exist, but they reply that their TODO isn't written yet. Watch what
   Gemini does with that reply.

7. **The same thing, in text.** ``python3 robot/chat.py`` (no ``--toy``) runs
   *your* prompt and *your* tools, the exact list the web app gets, on the fake
   robot from the tests: nothing moves, and no Pupper is needed. Ask it to walk
   to you here too. From now on, this is the fastest way to try a change to
   what Gemini reads: edit, run ``chat.py``, type. (``python3 robot/chat.py
   --robot`` sends the calls to the real robot server instead, the path a voice
   command takes, and Pupper moves.)

8. Run the tests once: ``python3 robot/test_tools.py``. The provided robot's
   checks pass, and every TODO is listed as not written yet.

**DELIVERABLE (Q1):** Your two experiments from step 4. (a) What did Gemini
send after each change to the description? (b) What did it tell you when the
move failed, and when it "succeeded" without moving? Then, from step 6: when
you asked Pupper to walk, did Gemini say it was walking before it knew the
result? In a few sentences: why is a model that talks about moving, without
moving, a particularly bad failure for a robot?

**DELIVERABLE (Q2):** Trace one voice command, *"stand up"*, from the
microphone to the controller switch: name every file and function it passes
through, and say which line runs *now* and which runs *later*. Use a coding
agent or read the code yourself (start at ``stand_up`` in ``robot/tools.py``).
Check your trace against the robot server's log
(``/tmp/gemini-pupper/server.log``). If an agent helped, point out anything it
got wrong, skipped or made up.

Part 1: Talk
------------
Open ``robot/prompt.md``. It has six headings with a question under each;
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

After editing the prompt, try it in text first: ``python3 robot/chat.py``
reads ``prompt.md`` every time it starts. Then reconnect in the app (no
restart needed), ask for a walk again, and compare with Q1.

**DELIVERABLE (Q3):** Pick two rules from your prompt. For each: what did
Gemini do before you wrote it, and what does it do now? Is the sentence in
the prompt enough, or will the robot also have to make sure of it in code?

Part 2: Walk
------------
In this part you make Pupper walk, turn and run by voice, and refuse what it
must not do.

A walk or a turn asked for by Gemini goes through three of your functions, from the
words to the motors:

.. code-block:: text

   "walk forward"  ->  move (tools.py, TODO 4)          words  -> vx, vy, duration
   "turn left"     ->  turn (tools.py, TODO 4)          words  -> wz_deg, seconds
                   ->  walk (behaviors.py, TODO 3)       check now, queue the job
                         -> check_move (TODO 2)          is this move allowed?
                         -> the job, later               20 speed messages a second

We build them from the inside out: first the safety check, then the walking
job, then the words Gemini reads (``turn`` first, then ``move``).

**TODO 2:** ``check_move`` in ``behaviors.py``: refuse what the robot can't or
shouldn't do.

*Why:* Gemini will sometimes ask for something the robot can't do: 3 m/s,
a 30-second walk, a speed so small the legs just shuffle. Your tool
descriptions *ask* Gemini to stay within the limits; ``check_move`` *makes
sure* of it. It is a plain function (no robot, no ``await``), so it is the
easiest place to start.

It gets a velocity (``vx`` forward and ``vy`` left, in m/s; ``wz_deg``
counter-clockwise, in deg/s) and a ``duration`` in seconds, and returns
``None`` if the move is allowed, or a sentence saying why not. The limits are
in ``MoveLimits`` at the top of ``robot_api.py``. Check them one at a time and
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

The string you return is all Gemini learns about the refusal, so say what was
wrong *and* what the limit is: ``f"too fast: limits are |vx|<={limits.vx_max}
m/s, ..."``. The tests check that the reason contains the limit's number.

Check it: ``python3 robot/test_tools.py todo2``

**TODO 3:** ``walk`` in ``behaviors.py``: the walking job. This is your first
full "check now, queue a job, report" behavior. Put ``stand_up`` (at the top
of the file) next to it and copy its shape.

*Why the loop:* the walking policy doesn't take "walk for 2 seconds". It
takes a speed, over and over. Between Gemini and the policy sits
``cmd_vel_mux``, the node that lets your joystick win over Gemini; it
forgets a speed it hasn't heard for half a second (so a crashed program can't
leave Pupper walking forever). So the job keeps sending the speed, 20 times a
second (``LLM_CMD_RATE``), for as long as the walk lasts.

*Now*, when Gemini calls:

1. ``reason = check_move(vx, vy, wz_deg, duration)``. If there is a reason,
   return ``Result(False, reason)``: nothing gets queued.
2. ``note = await leave_follow_mode(robot)``. That's TODO 5; until then it
   does nothing, so just call it.
3. If ``robot.fallen``, return ``Result(False, FELL_MESSAGE)``.

*Later*, in ``async def job()``:

4. If ``robot.fallen`` (it may have fallen while this job waited in line),
   return ``Result(False, FELL_MESSAGE)``.
5. ``res = await robot.switch_to(WALKING_CONTROLLER)``. If ``not res.ok``,
   return ``res``: it already says what went wrong.
6. The loop. Remember when to stop, ``end = robot.now() + duration``, and
   while ``robot.now() < end``: send ``robot.send_speed(vx, vy, wz_deg)``,
   then ``await robot.sleep(1.0 / LLM_CMD_RATE)``. If ``robot.fallen``
   becomes true inside the loop, send a zero speed and return
   ``FELL_MESSAGE``.
7. After the loop, ``robot.send_speed(0, 0, 0)`` (stand still), and return a
   ``Result(True, ...)`` that says what it did.

*Queue it* (this is what Gemini hears now):

8. ``return with_note(note, await robot.queue(f"move vx={vx} vy={vy}
   wz={wz_deg} {duration}s", job, duration))``

The numbered steps are also in the code, right where you write them.

Check it: ``python3 robot/test_tools.py todo3``. The fake robot records every
speed you send and when, so the test can tell if you sent too few, stopped
early, or forgot the zero at the end.

**TODO 4:** ``turn``, then ``move``, in ``tools.py``: the tools Gemini calls.
Write ``turn`` first. It's short, and it shows you the whole path from
Gemini's words to the robot. Then ``move``, the real challenge of this part.

*What Gemini reads.* Gemini picks a tool, and fills in its parameters, only
from the words in ``@tool(...)``: the tool's description and each parameter's
description. For each tool, replace every ``TODO 4`` there:

- Say what the tool does and when to use it. Put the robot's real limits in
  the words: the ``f"..."`` strings can use ``LIMITS.vx_max``,
  ``LIMITS.duration_max`` and the others, so they stay right if a limit
  changes. Add ``+ ENDS_FOLLOW`` to the description (Part 3 explains it).
- For each parameter: its unit (m/s, seconds, degrees), its range, and its
  default.

**turn: the warm-up.** Turning in place means no forward or sideways speed,
only a turning speed, for as long as the angle takes:

1. ``wz_deg`` is ``+TURN_SPEED`` (60 deg/s) to the left (counter-clockwise),
   and ``-TURN_SPEED`` to the right.
2. The time it takes: ``seconds = abs(angle) / TURN_SPEED``. 90 degrees at
   60 deg/s takes 1.5 s.
3. ``return await behaviors.walk(robot, 0.0, 0.0, wz_deg, seconds)``

Run ``python3 robot/test_tools.py todo4``: it checks ``turn`` first, on its
own, so you know it works before you start on ``move``.

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

3. ``return await behaviors.walk(robot, vx, vy, 0.0, duration)``

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

*What "run" means.* The walking policy switches to its running gait above
about 0.5 m/s, and in this lab "run" means **1.0 m/s or faster**. Gemini might
call ``move(run=true, speed=0.6)``. That's why the rule has to be in your
code, not only in the description. (Q5 asks about it.)

Check it: ``python3 robot/test_tools.py part2`` (TODOs 2–4). Then try your
words in text: ``python3 robot/chat.py``, and ask for a run, a sidestep and a
turn. You see the exact call Gemini makes and the speeds your code sends the
(fake) robot. Last, restart (``robot/stop.sh && robot/start.sh``), reconnect
in the app, and try it by voice.

.. note::

   **Checkpoint: a refusal Gemini can explain.** Get Gemini to call your
   motion tool with a request the robot refuses, and explain the refusal to
   you using *only the robot's reply*. If your descriptions are so good that
   Gemini never asks for anything impossible, great: try something that fits
   the tool but breaks another rule. "Run forward for ten seconds" is more
   than 5 m. Show a TA the conversation and the debug view.

**DELIVERABLE (Q4):** The output of ``python3 robot/test_tools.py part2`` with
every check passing.

**DELIVERABLE (Q5):** Your ``turn`` and ``move`` descriptions, and one sentence
each on a choice you made (which parameters, which defaults, what the
description promises). Where is "run means at least 1.0 m/s" enforced, and
why there?

**DELIVERABLE (Q6):** Make ``walk`` send the speed only once (then put it
back). What does Pupper do, and why? What would happen if you waited with
``time.sleep`` instead of ``await robot.sleep``?

**DELIVERABLE (Q7):** A video of Pupper, by voice only: walking forward and
backward, turning both ways, sidestepping, and running. Include the refusal
checkpoint (a screenshot of the debug view is fine for that part).

**DELIVERABLE (Q8):** Ask for a 5-second walk. What does Gemini hear right
away, and what does it believe at that moment? When, if ever, does it learn
that the walk failed (say, Pupper fell over halfway)? Describe one situation
where this matters, and one way you could fix it.

Part 3: Follow
--------------
The robot has a person follower: camera → person detection → walking, like
your lab 5 state machine. Follow mode needs one rule:

   *Questions and conversation keep following. Any motion (walk, turn,
   trick, stand up, lie down) ends follow mode, and Pupper says so first:
   "Let me stop following you and ..."*

Turning the follower on and off is provided (``robot.set_following``). The
rule is yours, in two halves: the code that makes it true, and the words that
let Gemini talk about it.

**TODO 5:** ``leave_follow_mode`` in ``behaviors.py``. Every motion calls it
in its *now* part: look for it in ``stand_up``, ``lie_down`` and your
``walk``. So whatever you write here applies to every motion at once.

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
4. Return the note, e.g. ``"Stopped following the person first. "``: end it
   with a space, since the tool's own reply comes right after it.

Check it: ``python3 robot/test_tools.py todo5``

**TODO 6:** The ``follow`` tool in ``tools.py``, and the *Follow mode* section
of ``prompt.md``.

- The body is one line: ``return await robot.set_following(enabled)``.
- Its description tells Gemini what follow mode is, what keeps it on
  (talking, questions like "what do you see?") and what ends it (any motion
  tool).
- The *Follow mode* section of the prompt tells Gemini what to *say* before a
  motion ends follow mode.

``ENDS_FOLLOW``, the sentence you added to every motion tool's description in
TODO 4, is the third place the rule now lives.

Check it: ``python3 robot/test_tools.py part3``, then in text
(``python3 robot/chat.py``: *"follow me"*, *"what do you see?"*, *"now sit"*),
then restart and try it on the robot.

**DELIVERABLE (Q9):** The follow rule now lives in three places: the prompt,
the tool descriptions, and ``leave_follow_mode``. Make a table: for each
place, does it *tell* Gemini about the rule or *enforce* it? Then make
``leave_follow_mode`` return ``""`` without stopping anything, and ask
Pupper to sit while it follows you. What happens? (Put it back.)

**DELIVERABLE (Q10):** A video of follow mode: Pupper follows you, you ask it a
question while it follows (it keeps following), then you ask it to do
something else (it tells you it will stop following, then does it).

Part 4: Tricks
--------------
Every trick is a policy, played by its own controller: the 14 tricks in
``tricks/`` (``sit``, ``push_up``, ``swim``, …) were recorded on the Pupper
monorepo and trained as tracking policies in simulation. Each folder has a
``policy.json`` and a ``trick.yaml`` (``robot/tricks.py`` lists the fields):
what Gemini reads about the trick, and the pose it starts and ends in. The
launch file loads one controller per trick, called ``trick_<name>``, all
switched off.

*Playing a trick is a change of state.* Before, the walking policy has the
legs. You give them to ``trick_<name>``, wait while it plays, and then decide
who has the legs afterwards:

.. list-table::
   :header-rows: 1
   :widths: 30 35 35

   * - The trick ends…
     - Who has the legs afterwards
     - Examples
   * - standing (``end_pose: standing``)
     - the walking policy again, so Pupper can walk away
     - ``sit``, ``push_up``, ``sneeze``
   * - lying down (anything else)
     - the trick's controller, holding its last pose until the next motion
     - ``swim``, ``twerk``, ``upward_dog``
   * - holding a pose, taking requests (a *clip trick*, Part 5)
     - the trick's controller, which keeps playing its "stay" clip
     - ``sit_stay_shake``

**TODO 7:** ``play_trick`` and ``play_policy_trick`` in ``behaviors.py``: the
same "now, then a job" shape as ``walk``, split into two functions because
the job is long.

``play_trick`` (*now*):

1. ``trick = robot.tricks.get(name)``. If it's ``None``, fail, and list the
   real names (``list(robot.tricks)``), so Gemini can pick a real one.
2. ``note = await leave_follow_mode(robot)``.
3. The job is ``play_policy_trick(robot, trick)``. You don't want to run it
   now, so wrap it in a one-line function: ``job = lambda:
   play_policy_trick(robot, trick)``.
4. Queue it with ``robot.queue(f"trick {name}", job, trick.total_time,
   then_holding=...)``, with the note in front. ``then_holding`` is how the
   queue keeps track of "will a clip trick be holding a pose when this job is
   done?": the trick if it is a clip trick (``trick.clips`` isn't empty),
   else ``None``. Part 5 relies on it.

``play_policy_trick`` (*the job*):

1. A clip trick that is already holding its pose (``robot.holding_now is
   trick``): nothing to do, so say it is already doing it.
2. A trick that is still holding the end of its last run (``await
   robot.is_active(trick.controller)``) needs ``await robot.switch_to(None)``
   first. Switching to a controller that is already on does nothing at all,
   so without this, "swim again" would just keep lying there.
3. ``await robot.switch_to(trick.controller)``, then ``await
   robot.sleep(trick.total_time + 0.3)`` while it plays. If ``robot.fallen``,
   say it ended in a fall.
4. Leave the robot in the right pose, using the table above, and say which:
   a clip trick sets ``robot.holding_now = trick`` and lists its actions; a
   trick that ends standing does ``await robot.switch_to(WALKING_CONTROLLER)``;
   any other trick just stays where it is.

Every outcome says which pose the robot is in now: Gemini (and
``robot_status``) can only plan the next move if it knows.

Check it: ``python3 robot/test_tools.py todo7``. The fake robot records every
controller switch and when, so the test can tell if a trick never got the
legs, or didn't keep them long enough.

**TODO 8:** The ``play_trick`` tool in ``tools.py``: what Gemini reads. The
body is already written; replace the ``TODO 8`` words.

- Put ``{tricks}`` in the description: the robot turns it into the list of
  installed tricks, each with the description from its ``trick.yaml``. Look
  at the list it makes: ``curl localhost:8000/tools`` on the Pupper.
- Tell Gemini what it can't see from the list: a trick that starts standing
  looks wrong from lying down, and a trick keeps its final pose until the next
  motion.

Check it: ``python3 robot/test_tools.py part4``, then ask for tricks in text
(``python3 robot/chat.py``): does Gemini pick the trick you meant from your
words? Then restart and ask for a few on the robot, some that end standing and
some that end lying down.

**DELIVERABLE (Q11):** The output of ``python3 robot/test_tools.py part4`` with
every check passing.

**DELIVERABLE (Q12):** Ask for ``swim`` twice in a row. Why does a trick that
is holding its own end pose need to be relaxed before it plays again? And why
should every reply say which pose the robot ends in: what would Gemini get
wrong without it?

Part 5: Sit, Stay, Shake
------------------------
Pupper already knows ``sit``: it sits like a dog, holds for two seconds, and
stands right back up. By the end of this part, Pupper will **sit when told,
stay sitting for as long as you like, shake whichever paw you ask for, and
stand up when you say so**, all by voice.

You won't design any new poses. You get two finished motions, *sit down and
stand up* and *shake the left paw while sitting*, and you cut and join them
into a **clip library**: named pieces of motion the robot plays one at a time,
on request. Then you train one **tracking policy** on the whole library,
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

   A clip trick. Every clip starts and ends at a pose it shares with its
   neighbours, so the robot can go straight from the end of one clip to the
   start of the next. A looping clip repeats until something else is asked
   for.

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

**DELIVERABLE (Q13):** Draw your clip graph, like the figure above: every clip,
the pose it starts and ends in, which one loops, and each clip's ``next``. Why
must clips meet at matching poses: what would the robot do at a mismatched
boundary?

**DELIVERABLE (Q14):** Your ``mirror``. Which joint angles did you negate, and
why does Pupper need that? How did you check it?

**DELIVERABLE (Q15):** Your training order. Why does every change the robot
can make on request need to appear in it?

Part B: Watch It Fall, Then Train It
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Keep going in the notebook:

* **Replay with physics.** The robot plays the joined motion as a fixed
  recording, with no policy. Expect it to fall.
* **Upload** the motion to W&B, and **train** one policy on the whole library
  (3,000 iterations, about 18 minutes: work on Part C's code meanwhile).
* **Score** your policy with the same physics check, and watch it in the
  viewer.

**DELIVERABLE (Q16):** The plain-replay verdict and worst errors, next to your
policy's. In which clip does plain replay fail, and why can't a fixed
recording recover from it?

**DELIVERABLE (Q17):** In your own words: why did the "stay" not need a policy
trained on a longer sit? What does the policy observe, frame by frame, that
makes an open-ended stay possible?

Part C: Give Gemini the Actions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
A clip trick holds a pose and takes requests. ``play_trick`` (your TODO 7)
already sits and stays; the requests (*shake*, *the other paw*, *stand up*)
are **trick actions**, and they only make sense while the trick holds its
pose. Until a clip trick is installed, Gemini is told the ``trick_action`` tool
has nothing to choose from.

**TODO 9:** ``trick_action`` in ``behaviors.py``. You can write and test it
while your policy trains: the tests use a made-up clip trick called
``hold_pose`` (actions ``wave_left``, ``wave_right``, and the exit clip
``leave``).

*The new idea: an action only makes sense in one state.* A shake is a clip of
``sit_stay_shake``: it can only play while that trick's controller holds the
sit. So ``trick_action`` has to ask "is Pupper sitting?". The interesting
part is *when* to ask, because of the queue:

- Someone says "sit, and give me your paw" in one breath. Gemini calls
  ``play_trick`` and ``trick_action`` back to back, before Pupper has even
  started to sit. If ``trick_action`` asked "is it sitting *right now*?", it
  would refuse a perfectly good request. So, **now**, it asks: "will it be
  sitting once everything already queued has run?" That's
  ``robot.holding_after_queue``, kept up to date by the ``then_holding`` you
  pass to ``robot.queue``.
- But a lot can happen between the call and the shake: Pupper can fall over,
  someone can press the e-stop or say "stop". So **later**, in the job, it
  asks again, about right now: ``await robot.is_active(trick.controller)``.

*Now*, when Gemini calls:

1. ``found = robot.action_ids().get(action)``. ``action_ids()`` maps each
   action's name to ``(trick, clip)``. If it's ``None``, fail, listing the
   real names (``list(robot.action_ids())``).
2. ``trick, clip = found``
3. If ``robot.holding_after_queue is not trick``, fail, and say which trick
   to play first: ``f"... play it first (play_trick {trick.name})"``.

*Later*, in ``async def job()``:

4. If ``not await robot.is_active(trick.controller)``, fail the same way.
5. ``res = await robot.play_clip(trick, clip)`` plays the clip and waits
   until it is done.
6. If that failed, or the clip isn't the trick's exit clip
   (``trick.exit_clip``), return ``res``: the robot keeps sitting.
7. The exit clip stands Pupper up, so nothing is holding a pose any more:
   ``robot.holding_now = None``, then ``await
   robot.switch_to(WALKING_CONTROLLER)``, and report how that went.

*Queue it*:

8. ``robot.queue(f"{trick.name} {clip}", job,
   trick.clip_time_estimate(clip), then_holding=...)``: ``None`` after the
   exit clip (it stands up), else ``trick`` (it keeps sitting).

Check it: ``python3 robot/test_tools.py todo9``

**On the robot:**

1. **TA gate.** Show a TA your policy's score and the viewer.
2. Fetch the policy on the Pupper with the command the training log printed,
   naming it ``sit_stay_shake``:

   .. code-block:: bash

      python3 robot/get_trick.py <entity>/mjlab/<run-id> --name sit_stay_shake

**TODO 10:** What Gemini reads about the actions.

- Fill in ``tricks/sit_stay_shake/trick.yaml`` (``get_trick.py`` wrote a
  template): the trick's ``description``, the ``actions`` Gemini may ask for
  while Pupper sits (with a description each), the ``exit_clip``,
  ``end_pose: sitting``, and ``max_body_angle_deg: 60`` (the sit leans back
  about 35 degrees).
  Gemini reads the ``description`` and the action descriptions word for word,
  so write them for Gemini: what the robot does, and what it does after
  (keeps sitting, or stands up).
- The ``trick_action`` tool in ``tools.py``. Its body is one line, ``return
  await behaviors.trick_action(robot, action)``. Its description: put
  ``{actions}`` in it, and the robot turns it into the list of actions with
  your descriptions. Then tell Gemini what has to happen first (the trick),
  and what ends it (any other motion).
- ``python3 robot/test_tools.py part5``. Try *"sit, and give me your paw"*
  in text first (``python3 robot/chat.py``: you see both calls, back to back,
  before the sit has happened), then restart and say it to Pupper in one
  breath.

**DELIVERABLE (Q18):** The output of ``python3 robot/test_tools.py part5`` with
every check passing.

**DELIVERABLE (Q19):** Ask for a paw shake while Pupper is standing. What does
it do, and what does it say? Which of your two checks fired? Then describe a
case where the *other* check (in the job) is the one that matters.

**DELIVERABLE (Q20):** A video, by voice only: "sit", wait at least ten seconds
(it stays), "shake", "now the other paw", "stand up", "walk forward".

Part 6: Put It Together
-----------------------
**TODO 11:** The last two sections of ``prompt.md``.

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

Run ``python3 robot/test_tools.py`` once more: everything passes.

* **Live demo (TA checkoff).** A short, unscripted conversation with Pupper
  that walks and turns, follows someone and stops following, sits, stays,
  shakes both paws, stands, plays a trick, and refuses at least one request
  with a reason.
* **Reflection.** A page, not more:

  - Where did you put each rule: in the prompt, in a tool description, or in
    code? Which one would you move, and why?
  - One time Pupper did something you didn't expect. What did the tool replies
    and the server log tell you, and what did you change?
  - If you used a coding agent: one prompt that worked well, one place it went
    wrong, and how you caught it. Attach short excerpts (no secrets).
  - What would you tell next year's students?

**DELIVERABLE (Q21):** A video of your demo conversation.

**DELIVERABLE (Q22):** Your reflection.

Optional: The Jump
------------------
Pupper's default walking policy can already jump: you have seen it on the
gamepad. The jump isn't a policy of its own: it is a *reference* the walking
policy was trained to follow, sent to it once. Give Gemini that jump:

1. Rename ``tricks/jump/trick.yaml.optional`` to ``trick.yaml`` and restart.
2. In ``play_trick``, a trick with ``trick.kind == "reference"`` plays with
   ``robot.play_reference_trick(trick)`` instead of your policy job.
3. ``python3 robot/test_tools.py optional_jump``, then ask Pupper to jump, with
   clear space around it.

Want more? :doc:`opt-lab-3` takes you from your own reference motions to a
dance Gemini can perform.

Working With a Coding Agent
---------------------------
A few habits that help:

* **Give it checks it can run**: ``python3 robot/test_tools.py todo7``, and the
  notebook's (or ``sit_stay_shake.py``'s) ``ok`` / ``FAIL`` checks.
* **Ask for the plan before the code.** "Read the TODO 9 card in
  ``behaviors.py`` and tell me how a trick action should use the queue; don't
  edit anything yet."
* **Check its claims against the robot.** The server log, the debug view and
  ``curl localhost:8000/system/mode`` are the truth. The agent's summary is
  not.
* **Don't let it weaken a test or raise a limit** to make something pass.
* **Make it teach you.** After each TODO: "Explain this function line by line,
  and tell me what would break if I deleted each check." Then close the agent
  and explain it to your partner. If you can't, you're not done.

Congratulations! Pupper now listens, answers, and does what it is told, and
only what it can do safely. You built the leash for a robot foundation model:
tools a model can reason about, honest reports of what happened, state and
preconditions, and a trick pipeline that turns motions into a policy on
hardware. Your final project can build on all of it.

Resources
---------
`Gemini Pupper <https://github.com/sippeyxp/gemini-pupper>`_ by
`Peng Xu <https://scholar.google.com/citations?user=460NWeQAAAAJ&hl=en>`_
(Google DeepMind), and his write-up, *Interactive Robot Dog with Gemini Live*
(``docs/interactive-robot-dog.md`` in your repo)

``robot/chat.py`` is based on Peng's minimal Gemini Live robot-tool example.

`Gemini Live API <https://ai.google.dev/gemini-api/docs/live>`_ and
`function calling <https://ai.google.dev/gemini-api/docs/function-calling>`_

`Gemini Robotics: Bringing AI into the Physical World <https://arxiv.org/abs/2503.20020>`_

`Do As I Can, Not As I Say: Grounding Language in Robotic Affordances (SayCan) <https://say-can.github.io/>`_

`Code as Policies: Language Model Programs for Embodied Control <https://code-as-policies.github.io/>`_

`BeyondMimic: From Motion Tracking to Versatile Humanoid Control <https://arxiv.org/abs/2508.08241>`_

`DeepMimic: Example-Guided Deep Reinforcement Learning of Physics-Based Character Skills <https://arxiv.org/abs/1804.02717>`_

`Pupper tricks: design, replay, train <https://github.com/cs123-stanford/pupper-mjlab/tree/main/src/mjlab/tasks/tracking/config/pupper>`__ (the README behind the notebooks)

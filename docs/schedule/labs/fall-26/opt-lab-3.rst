Optional Lab 3: Teach Pupper to Dance
=====================================

.. note::

   **Draft.** This lab is still being written; the outline below is what it
   will cover. Ask a TA before you start.

.. TODO(staff): finish this lab: notebook / scripts, TODO numbering, deliverables (Q-numbered).

Goal
----
Give Pupper a dance that Gemini can perform on request. In lab 6 you trained a
tracking policy (BeyondMimic-style) on a clip library someone else designed.
Here you make the **reference motion** yourself, three different ways, use a
coding agent to clean it up, train it, and play it from Gemini Pupper.

Prerequisites
-------------
Lab 6, fully: you know how a clip library becomes a tracking policy, and how a
clip trick is played and stopped (``play_trick``, ``trick_action``, the exit
clip). Like Optional Lab 2, this lab works in the codebase
(`pupper-mjlab <https://github.com/cs123-stanford/pupper-mjlab>`_), with a
coding agent (Claude Code or Codex) as your guide.

Part 1: Three Ways to Get a Reference Motion
--------------------------------------------
Make at least one short move (2–4 s) each way, and compare them:

1. **Retarget a real dog.** Take a clip from a dog motion-capture dataset and
   retarget it onto Pupper's legs: scale it to Pupper's size, map the joints,
   keep the feet on the ground.
2. **Puppeteer Pupper by hand.** With the motors relaxed, move Pupper's legs
   through the move yourself while the robot records its joint angles; your
   lab 2 forward kinematics turns them into foot positions you can check.
3. **Describe it in words.** Use UniMate (text-to-motion): type a sentence,
   get a Pupper reference motion.

.. TODO(staff): name the dataset, the recording script, and how students get UniMate access.

Part 2: Clean It Up With a Coding Agent
---------------------------------------
Raw motions are messy: feet slide or dip below the ground, joints hit their
limits, the start and end don't match. Work with your agent to fix each move
until the motion report and the physics replay (``play_motion.py``) are happy:
smooth it, fix foot contact, make it loop, mirror it, and make every move start
and end in the same pose so they can follow each other.

Part 3: Train the Dance and Perform It
--------------------------------------
Join your moves into one clip library (each move a clip; a "groove" clip
that loops), train one tracking policy on it, score it, and (after a TA's OK)
fetch it with ``get_trick.py``. Your moves become trick actions, so Gemini can
perform a routine: *"dance for me: the wiggle, then the spin, then bow"*.

Stretch: a ``dance`` tool that checks a whole routine before the first move
(every move exists, and each one can follow the one before it).

Deliverables (draft)
--------------------
- One move made each of the three ways, side by side: which way was fastest,
  which looked most natural, and which survived physics best?
- Before/after motion reports for one move your agent cleaned up, and the
  agent conversation that did it (short excerpts).
- A video of Pupper dancing on request.

Optional Lab 2: the Pupper Parkour Challenge
=========================

Goal
----
This lab challenges you to train Pupper to navigate increasingly difficult obstacle courses! You'll develop and test your own walking policies, competing with other teams to achieve the highest score across four unique challenges. This optional lab is designed to be more open-ended and creative, allowing you to focus on policy development and testing rather than implementation details.

Setup
-----
To get started, make a copy of the `pupperv3-mjx <https://github.com/Nate711/pupperv3-mjx.git>`_ codebase. You can modify this codebase to train your Pupper under different environments. Remember to change the remote URL to your forked repo to avoid accidentally pushing changes to the original repository.

The Challenge
------------
We have prepared five different obstacle courses in the Gates basement, each with increasing difficulty. These obstacles will only be revealed to you after you have your policy ready for testing. This approach encourages you to develop robust policies that can handle a variety of challenges, rather than optimizing for specific obstacles.

Parkour Challenge Rules
----------------------
Each group will have 5 attempts to showcase their policy's performance across all obstacles. The obstacles will be revealed one at a time, allowing you to test and refine your policy between attempts.

Scoring System
~~~~~~~~~~~~~
Your policy's performance will be evaluated based on the following criteria:

- Forward control traversal: 10/t points (where t is the time taken in seconds)
- Backward control traversal: 5/t points
- Side step traversal: 3 points

The difficulty of each obstacle is reflected in score multipliers:

- First obstacle: 1x multiplier
- Second obstacle: 2x multiplier
- Third obstacle: 3x multiplier
- Fourth obstacle: 5x multiplier
- Fifth obstacle: 8x multiplier
  
TAs will time each traversal attempt and record the quickest performance. Note that you will only have access to the next obstacle after you successfully traverse the current one with at least one control method. Your total score will be the sum of your best performances across all obstacles. Note that even our default policy cannot successfully traverse the final obstacle, so this presents a significant challenge for your team!

Policy Development Suggestions
----------------------------
Here are some potential approaches you might want to experiment with to improve your policy's performance:

1. **Environment Modification**
   
   - Add obstacles directly into the training environment by understanding how obstacles and heightfields are implemented in the original lab 5 notebook
   - This allows you to train your policy on similar challenges to what it will face in the actual course

2. **Physics Parameter Tuning**
   
   - Experiment with different physics parameters beyond just reward tuning
   - For example, try adjusting friction coefficients to make the robot more stable
   - Consider how different surface properties might affect traversal

3. **Reward Engineering**
   
   - Introduce additional reward terms specifically targeting traversal objectives
   - Fine-tune the balance between different reward components
   - Consider adding rewards for smooth motion, more powerful movements, or specific traversal behaviors

4. **Controller Optimization**
   
   - Increase the ``lin_vel_range`` controller range in your notebook
   - This might allow for more dynamic movements and faster traversal times
   - Be careful to maintain stability while increasing speed!

5. **Advanced Policy Development**
   
   - Augment policy observations with privileged information available in the simulation environment
   - Develop a way to distill this enhanced policy into one that can be confidently deployed
   - Note: This approach requires significant effort but could yield impressive results

Remember that these are just suggestions - feel free to explore your own ideas! The most successful policies often come from creative combinations of different approaches.

Prize and Final Words
---------------------
The challenge begins on May 5th, and ends by the end of May. Before you test your policy, make sure that all the screws are tightened on Pupper! The winning team will receive a mysterious prize ;). Good luck, and may the best policy win!

Optional Lab 3: Poor Man's VLA
===============================

Goal
----
Embed the pupper_llm codebase with vision processing capabilities to create a Vision-Language-Action (VLA) system! This lab challenges you to enhance Pupper's intelligence by enabling it to understand and respond to both visual and textual inputs.

What is VLA?
-----------
Vision-Language-Action (VLA) is an emerging paradigm in robotics that combines visual perception, natural language understanding, and robotic control into a unified system. In this lab, you'll create a simplified version ("poor man's VLA") where you'll modify the existing GPT-based control pipeline to incorporate visual information alongside text commands. With vision integrated, Pupper will be capable of doing many more advanced tasks, giving us more freedom to add dynamic functionalities to our KarelPupper API calls.

Why "Poor Man's VLA"?
~~~~~~~~~~~~~~~~~~~~
While fancy VLA systems use end-to-end training to learn everything from pixels to actions in one go (like a rich person buying a fully automated mansion), we're taking the "poor man's" approach - building our system piece by piece, like assembling IKEA furniture with extra steps. Instead of one giant neural network that does everything, we're creating a hierarchical system where:

1. Vision processing happens separately (like having a security camera)
2. Language understanding is handled by GPT (like having a translator)
3. Action generation uses our existing API (like having a remote control)

It's not as elegant as the end-to-end approach, but hey, we're working with what we've got! Plus, this modular approach gives you more control over each component and makes debugging easier - when something goes wrong, you know exactly which part of your IKEA furniture is wobbly.

The Challenge
------------
Your task is to improve the pipeline from lab 7 by enabling the GPT model to process both text and vision inputs in a single inference pass, producing appropriate robot API action calls. This will allow Pupper to make more informed decisions based on its visual understanding of the environment.

Getting Started
--------------
We have provided two potential starting points:

1. **Debug Existing Implementation**
   
   - Location: `pupper_llm/vision_integration`
   - This is a barebone (not functional) implementation that currently includes depth and recognition mask inputs
   - Your task: Debug and modify this implementation to work with simpler vision inputs
   - Note: You'll need to remove the depth and recognition mask components

2. **Direct Modification**
   
   - Location: `pupper_llm`
   - Alternative approach: Modify the main codebase directly
   - Your task: Implement an image message publisher to feed visual information to the GPT model

Refining the KarelPupper APIs with Vision Integration
---------------------------------------------------
With vision capabilities integrated into Pupper, we can now expand our KarelPupper API to support more sophisticated and dynamic behaviors that leverage visual feedback. Here are some suggested API enhancements:

- Location: `pupper_llm/karel/karel.py`
- Your task: Implement new vision-enabled API functionalities
- Important Note: The VLM pipeline processes one image at a time, with each inference usually taking over 15 seconds to complete. This means you'll only get very sparse visual feedback (one image every 15+ seconds). Keep this timing constraint in mind when designing your APIs:
  - For real-time tasks (like obstacle avoidance or dynamic navigation), you'll need to implement a separate, faster vision processing pipeline (while not frying your Raspberry Pi)
  - For tasks that can work with sparse visual updates (like goal recognition or environment understanding), you can use the VLM's output directly
- Consider adding (from easiest to hardest):
  
  - Simple movement primitives (e.g., side steps, diagonal movements)
  - Sound feedback capabilities (e.g., playing victory music when reaching goals, warning sounds for obstacles)
  - Custom action sequences (e.g., a sequence of steps chained together to perform a dance with synchronized music)
  - Sensor feedback integration (e.g., images, IMU data, joint positions)
  - Advanced navigation capabilities (e.g., path planning, obstacle avoidance with **real-time** vision feedback, start from lab 6)
    - Note: This will require a separate, faster vision processing pipeline due to the VLM's sparse visual feedback
  - Complex movement primitives (e.g., switching gaits or adding yaw control - check neural controller for implementation details, you may need to train your own policies)
  
- These new APIs will give your VLA system more expressive power to handle complex tasks

Evaluation
----------
The goal is to benchmark how Pupper's decision-making capabilities improve when it can perceive its environment. You should:

- Implement a working vision integration system
- Demonstrate improved task completion with visual input
- Compare performance with and without vision capabilities
- Document any interesting behaviors or limitations

This is an open-ended lab that encourages creative solutions. Feel free to experiment with different approaches to vision integration and pipeline architecture modifications. The key is to make Pupper more intelligent by enabling it to "see" and understand its surroundings.

Remember to document your approach, challenges faced, and lessons learned. We're excited to see how you enhance Pupper's capabilities with vision!

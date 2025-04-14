Optional Lab 1: Implicit Compliance
===================================

Goal
----
Last quarter, Ankush and I worked on an experimental project where we replaced Pupper's right front leg with DenseTact, a highly sensitive tactile sensor. Our hypothesis was that with this additional touch feedback, Pupper could better understand the terrain while walking, potentially leading to more adaptive and dynamic walking patterns. Your challenge in this lab is to investigate whether this approach is effective.

Prerequisites
-------------
This optional lab requires strong fundamentals in machine learning and practical experience with PyTorch, particularly in dataset handling. You'll also likely need a Google Colab Pro subscription to run the training code. If you need to brush up on these topics, we recommend reviewing `this tutorial <https://www.youtube.com/watch?v=tHL5STNJKag>`_ before starting.

Setup
-----
.. raw:: html

   <iframe width="560" height="315" src="https://www.youtube.com/embed/odr5Y_b_yhs" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Check out our `presentation slides <https://docs.google.com/presentation/d/1gRUbmp1Byxsv4ouav9vDTrSvouK07nl9HR4YpHBjEwI/edit?usp=sharing>`_ to understand our experimental design.

To simplify the challenge of terrain detection using tactile sensors, we designed experiments with a fixed Pupper gait where the right front foot contacts an incline at a consistent point. We 3D printed five different ramps ranging from 15° to 75° (in 15° increments) for the leg to land on. We also collected data from flat surfaces for comparison.

For surface materials, we selected five options with distinct properties:
- Foam (highly compliant)
- White tile (rigid with low friction)
- Gray tile (rigid with high friction)
- Back of gray tile (very rigid with textured pattern)
- Carpet felt pad (moderately compliant with low friction)
We also gathered data using just the slopes without additional materials. Combined with the 6 possible angles, this gave us 36 unique terrain configurations to test.

Data Collection
---------------
We're building on the code from lab_3, using the triangular motion pattern you implemented. Pupper executes this motion with its right front leg, making contact with various slopes and surfaces using the DenseTact sensor. To optimize data collection, we programmed a 1-second pause when contact is detected, followed by a 1-second sliding motion along the programmed trajectory. This approach maximizes the number of contact-state images we can collect while minimizing the required motion repetitions.

For contact detection, we developed a reference-based system using pre-collected and preprocessed images for both contact and non-contact states. During operation, the system captures and processes frames (with center cropping and resizing), then computes mean absolute differences against the reference images. By comparing these differences, we can efficiently determine contact state at 10Hz. You will implement this in a later part of this lab.

Regarding deep learning approaches - you might wonder why we didn't use a CNN for contact detection. We actually tried this initially with some success. However, even with a minimal CNN architecture, running PyTorch inference on Pupper's Raspberry Pi 5 exceeded its computational limits, resulting in hardware failure and physical damage to the robot's brain :( This experience led us to develop our current, more efficient approach.

Dataset
-------
The complete dataset for this experiment is available `here <https://drive.google.com/file/d/1-Jcldkcl7cbHorDAKQsJM5b5Xl2mlVP-/view?usp=sharing>`_. Tactile data (images) were collected at a rate of 3 Hz, while joint states were captured at approximately 1000 Hz. In total, we recorded 1,885,507 joint state samples and around 5000 tactile images—with 2242 of them representing contact. Each material and slope combination underwent 6–10 triangular gait repetitions per set, and two complete sets were collected. Unzip the file, and upload the folder to your Google Drive.

Ablation Study
--------------
Your goal for this lab is to perform an ablation study to investigate the impact of tactile sensing on terrain detection. (So you will not be using Pupper for the majority of this lab!)

1. Complete the `Explicit Classifier <https://colab.research.google.com/drive/10U6rUKtP3hk2M-_QttxvAcWTUxo7D8Y6?usp=sharing>`_ notebook, which trains an image classifier to detect contact from tactile images.
2. Complete the `Implicit Classifier <https://colab.research.google.com/drive/14EKpFbiVQAwySBp26gdjnzv3Y2PlcjVB?usp=sharing>`_ notebook, which trains a model to detect contact from just the motor joint states.
3. What accuracy did you achieve for the explicit classifier? What about the implicit classifier?
4. Compare the performance of both classifiers. What does this tell us about the relationship between tactile sensing and motor proprioception? Could it be that the motors themselves are acting as implicit tactile sensors?

The surprising effectiveness of the implicit classifier suggests that motor proprioception alone can provide sufficient information for terrain detection. This finding has profound implications for quadruped locomotion: it suggests that the motors' ability to sense and respond to external forces (their implicit compliance) might be sufficient for stable locomotion, even without explicit tactile sensors. This could explain why reinforcement learning approaches using only proprioceptive feedback have been so successful in training quadruped robots (which you will do in lab 5).

Deployment Challenge (Open-Ended)
-----------------------------------
Now that you've trained your implicit classifier, let's try deploying it on Pupper. Your challenge is to modify the triangular trajectory code to pause for one second when contact is detected.

1. Export your trained implicit classifier model from the Colab notebook.
2. Integrate the model into your Pupper codebase, using it to detect contact during the triangular trajectory.
3. When contact is detected, pause the trajectory for one second before continuing.

This is an open-ended challenge that will likely require some creative problem-solving. Our collected dataset is very specific to terrain detection scenarios, so you might find that the model doesn't work perfectly out of the box. You'll need to think about:

- How to preprocess the motor states in real-time to match the training data format
- Whether to use the raw model predictions or apply additional filtering
- How to handle false positives and negatives
- Whether to collect more training data or modify the model architecture to better suit this use case

This challenge could also serve as a starting point for your final project. Getting this to work will be sufficient for a good final project.

The goal is not just to get it working, but to understand the challenges and limitations of deploying machine learning models on real robots. This experience will be valuable for your future robotics projects.


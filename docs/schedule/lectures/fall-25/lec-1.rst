🤖 Lecture 1: ROS Introduction and PD Control
============================================

.. raw:: html

   <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 25px; border-radius: 15px; margin: 20px 0; text-align: center;">
   <h2 style="color: white; margin-top: 0; font-size: 2.2em;">🚀 Welcome to CS123!</h2>
   <p style="margin-bottom: 0; font-size: 1.2em;">Your journey into robotics begins here with ROS2 and PD Control</p>
   </div>

.. note::
   **📚 Course Materials** 
   
   This lecture covers the fundamentals of ROS2 (Robot Operating System) and PD (Proportional-Derivative) control - the building blocks of modern robotics!

.. raw:: html

   <div style="background: #f8f9fa; border-left: 5px solid #28a745; padding: 20px; margin: 20px 0; border-radius: 8px;">
   <h3 style="color: #28a745; margin-top: 0;">📖 Lecture Resources</h3>
   <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 15px; margin-top: 15px;">
   </div>
   </div>

.. raw:: html

   <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; margin: 20px 0;">

   <div style="background: linear-gradient(135deg, #ff6b6b 0%, #ee5a24 100%); color: white; padding: 20px; border-radius: 10px; text-align: center;">
   <h4 style="color: white; margin-top: 0;">🎯 Intro Slides</h4>
   <p style="margin: 10px 0;">Course overview and introduction</p>
   <a href="https://drive.google.com/file/d/1qvrmODWmE-iC4CE7JrxLM49VD_J3MVra/view?usp=sharing" target="_blank" style="color: white; text-decoration: none; font-weight: bold; background: rgba(255,255,255,0.2); padding: 8px 15px; border-radius: 5px; display: inline-block;">📄 View Slides</a>
   </div>

   <div style="background: linear-gradient(135deg, #4ecdc4 0%, #44a08d 100%); color: white; padding: 20px; border-radius: 10px; text-align: center;">
   <h4 style="color: white; margin-top: 0;">🎓 Main Lecture</h4>
   <p style="margin: 10px 0;">ROS2 and PD Control concepts</p>
   <a href="https://docs.google.com/presentation/d/1yiRQ9m7rA-Ci4zR0SOiX-bAIji_ZBRpx7SxWVQP5qd0/edit#slide=id.g22c45b09435_0_1388" target="_blank" style="color: white; text-decoration: none; font-weight: bold; background: rgba(255,255,255,0.2); padding: 8px 15px; border-radius: 5px; display: inline-block;">📊 View Slides</a>
   </div>

   <div style="background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%); color: #333; padding: 20px; border-radius: 10px; text-align: center;">
   <h4 style="color: #333; margin-top: 0;">🔬 Lab Review</h4>
   <p style="margin: 10px 0;">Hands-on lab walkthrough</p>
   <a href="https://docs.google.com/presentation/d/1AMfv35pinMGrfdzTLc6-Fk2t2rOCrXFE/edit?usp=sharing&ouid=112164671976474020631&rtpof=true&sd=true" target="_blank" style="color: #333; text-decoration: none; font-weight: bold; background: rgba(0,0,0,0.1); padding: 8px 15px; border-radius: 5px; display: inline-block;">📋 View Slides</a>
   </div>

   </div>

.. raw:: html

   <div style="background: #fff3cd; border-left: 5px solid #ffc107; padding: 20px; margin: 20px 0; border-radius: 8px;">
   <h3 style="color: #856404; margin-top: 0;">🛠️ Hands-On Lab</h3>
   <p style="margin: 10px 0;">Put theory into practice with your first robotics lab!</p>
   <div style="display: flex; gap: 15px; flex-wrap: wrap; margin-top: 15px;">
   <a href="../../labs/fall-25/lab-1.html" style="background: #ffc107; color: #333; text-decoration: none; font-weight: bold; padding: 10px 20px; border-radius: 5px; display: inline-block;">🔬 Lab 1: ROS & PD Control</a>
   <a href="https://docs.google.com/document/d/1FZ3WAwX1zRO5ivQpqraeYcaJwmDZFZVPRNCVBTsuZrw/edit#heading=h.47t0k5pf0v4" target="_blank" style="background: #17a2b8; color: white; text-decoration: none; font-weight: bold; padding: 10px 20px; border-radius: 5px; display: inline-block;">📝 Lab Document</a>
   </div>
   </div>

.. raw:: html

   <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 25px; border-radius: 15px; margin: 30px 0; text-align: center;">
   <h3 style="color: white; margin-top: 0;">🏋️ Meet Pupper v2!</h3>
   <p style="margin-bottom: 20px; font-size: 1.1em;">Watch our amazing robot doing push-ups - this is what you'll be programming!</p>
   </div>

.. youtube:: io4QuRr5GCA
   :width: 720
   :height: 405
   :align: center

.. raw:: html

   <div style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); color: white; padding: 25px; border-radius: 15px; margin: 30px 0; text-align: center;">
   <h3 style="color: white; margin-top: 0;">🎯 What You'll Learn Today</h3>
   <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin-top: 20px;">
   <div style="background: rgba(255,255,255,0.2); padding: 15px; border-radius: 8px;">
   <h4 style="color: white; margin-top: 0;">🤖 ROS2 Basics</h4>
   <p style="margin: 0;">Understanding the Robot Operating System</p>
   </div>
   <div style="background: rgba(255,255,255,0.2); padding: 15px; border-radius: 8px;">
   <h4 style="color: white; margin-top: 0;">⚡ PD Control</h4>
   <p style="margin: 0;">Proportional-Derivative control theory</p>
   </div>
   <div style="background: rgba(255,255,255,0.2); padding: 15px; border-radius: 8px;">
   <h4 style="color: white; margin-top: 0;">🔧 Hands-On</h4>
   <p style="margin: 0;">Programming your first robot control</p>
   </div>
   </div>
   </div>

.. tip::
   **💡 Pro Tip** 
   
   Make sure to follow along with the lab after the lecture - hands-on experience is crucial for understanding robotics concepts!

.. warning::
   **⚠️ Important Note** 
   
   This content is from Spring 2025 and will be updated for Fall 2025. The core concepts remain the same, but some details may change.


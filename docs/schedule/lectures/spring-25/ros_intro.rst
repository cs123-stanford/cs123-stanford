ROS2 Introduction
=================

ROS 2 (Robot Operating System 2) is an open-source software framework designed to help developers build robotic applications more easily. If you're new to it, here's a simple breakdown:

1. What is ROS 2?
^^^^^^^^^^^^^^^^^

ROS 2 is like the "brain" of a robot. It provides tools, libraries, and conventions that allow different parts of a robot (like sensors, actuators, and decision-making systems) to communicate with each other.

2. Why was ROS 2 created?
^^^^^^^^^^^^^^^^^^^^^^^^^

It’s the successor to ROS 1, with improvements in security, real-time performance, and support for more complex and distributed robotic systems. ROS 2 was built to work better in commercial and industrial environments, where reliability and scalability are critical.

3. Key Concepts:
^^^^^^^^^^^^^^^^

* Nodes: Think of nodes as small programs or modules that perform specific tasks, like controlling a motor or processing sensor data.
* Topics: Nodes communicate with each other by sending and receiving messages through topics, which are like channels for specific types of information.
* Services: Besides passing messages, nodes can also request specific actions from each other using services, which is like making a phone call and asking for something to be done.
* Middleware: ROS 2 uses a middleware (DDS - Data Distribution Service) that helps with data transfer between nodes, even if they're running on different machines or systems.

4. Why use ROS 2?
^^^^^^^^^^^^^^^^^

ROS 2 simplifies the development of complex robotic systems by offering reusable components, so you don’t have to reinvent the wheel. It’s especially useful for robotics projects that require coordination between multiple devices or need to handle large amounts of data.

5. How to get started?
^^^^^^^^^^^^^^^^^^^^^^
#. Install ROS 2: Start by installing the ROS 2 distribution that fits your system. (We have done this for you on your Raspberry Pis already)
#. Learn the Basics: Get familiar with basic concepts like nodes, topics, and services. The talker listener tutorial here https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html will be a good introduction to how to use publishers and subscribers in ROS2. 
#. Try it Out on the Robot: Pupper is fully controllable via ROS, so doing our labs and final project will help you get acquainted with ROS2 and be able to use it on any other robot you interface with. 

Common ROS Commands
___________________
#. ros2 launch <package> <launch_file>: Launches the nodes that are specified in the fields of the launch description in the python launch file
#. ros2 run <package> <file>: Runs the node defined in the specified file
#. ros2 topic list: Display a list of all active topics currently available in the ROS 2 environment
#. ros2 topic echo <topic_name>: Displays the messages being published on a specific topic in real-time. It's a way to "listen in" on the data being transmitted over a topic, which is useful for debugging, monitoring, or just understanding what data is being exchanged in the system.
#. ros2 topic info <topic_name>: Provides detailed information about a specific topic, including its type, publishers, and subscribers.
#. ros2 topic pub <topic_name> <msg_type> <msg_data>: Publishes a message to a specified topic manually.
#. ros2 pkg list: Lists all available ROS 2 packages.
#. colcon build <package>: Builds the specified package after changes have been made. You must do this every time you make a change before running the Ros2 program again
#. source install/setup.bash: Configure your shell environment to use the ROS packages, nodes, and tools that were built in your workspace. This is necessary because ROS relies on environment variables to find the correct package paths and dependencies.
#. source ~/.bashrc: All the custom configurations and environment variables defined in your .bashrc are re-applied to your current session. This is inherently performed when you open a new terminal window, but ROS2 is sometimes buggy and doing this may resolve some issues.
#. source /opt/ros/<distro>/setup.bash: Sources the global ROS 2 setup file for a specific distribution.

ROS Debugging Tips
__________________
#. ROS is unable to find new information from changes you just made: sourcing install/setup.bash or the .bashrc can often help
#. When running ros2 topic echo, only the \rosout topic is visible: sourcing install/setup.bash or the .bashrc can often help


In essence, ROS 2 is a powerful tool that makes it easier to build and manage robotic systems, even if you're just getting started.

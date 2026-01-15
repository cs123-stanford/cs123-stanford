< SSH into Pupper over WiFi Guide
===================================

.. note::
   **What is SSH?** =

   SSH (Secure Shell) allows you to remotely access and control Pupper's Raspberry Pi from your computer. This guide shows you a robust way to connect via WiFi, even when IP address aliasing doesn't work.

.. warning::
   **Important**  

   Sometimes Pupper shows as connected to WiFi but isn't actually receiving internet packets. This guide will help you verify the connection is working properly before attempting to SSH.

.. raw:: html

   <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 20px; border-radius: 10px; margin: 20px 0;">
   <h3 style="color: white; margin-top: 0;">=€ Let's Get Connected!</h3>
   <p style="margin-bottom: 0;">Follow these 3 simple steps to establish a reliable SSH connection to Pupper!</p>
   </div>

Step 1: =á Verify WiFi Connection
----------------------------------

.. raw:: html

   <div style="background: #f8f9fa; border-left: 4px solid #28a745; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>=Ë Instructions:</strong>
   <ol style="margin: 10px 0;">
   <li>Ensure your computer and Pupper are connected to the same WiFi network</li>
   <li>Open a terminal window on Pupper</li>
   <li>Run the ping test to verify internet connectivity</li>
   </ol>
   </div>

.. code-block:: bash

   ping 8.8.8.8

.. raw:: html

   <div style="background: #d4edda; border-left: 4px solid #28a745; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong> Success Indicator:</strong>
   <p style="margin: 10px 0;">If you see output like <code>64 bytes from...</code>, your Pupper is connected! If not, proceed to reconnect.</p>
   </div>

.. raw:: html

   <div style="background: #fff3cd; border-left: 4px solid #ffc107; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>=¡ Troubleshooting:</strong>
   <p style="margin: 10px 0;">If the ping fails, go to WiFi settings, forget the network, and reconnect to establish a fresh connection.</p>
   </div>

Step 2: = Find Pupper's IP Address
------------------------------------

.. raw:: html

   <div style="background: #d1ecf1; border-left: 4px solid #17a2b8; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong><¯ What to Look For:</strong>
   <ul style="margin: 10px 0;">
   <li>Run the <code>ip a</code> command in Pupper's terminal</li>
   <li>This displays all network interfaces on the Raspberry Pi</li>
   <li>Look for an IP address like <code>10.1.1.101</code></li>
   <li>Note this IP address - you'll need it in the next step</li>
   </ul>
   </div>

.. code-block:: bash

   ip a

Step 3: = Connect via SSH
---------------------------

.. raw:: html

   <div style="background: #d4edda; border-left: 4px solid #28a745; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>=¥ Terminal Commands:</strong>
   <p style="margin: 10px 0;">On your computer, run the following command (replace with Pupper's IP address):</p>
   </div>

.. code-block:: bash
   :emphasize-lines: 1

   ssh pi@<ip_address>

For example:

.. code-block:: bash

   ssh pi@10.1.1.101

.. raw:: html

   <div style="background: #e2e3e5; border-left: 4px solid #6c757d; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>=Ý What Happens Next:</strong>
   <ul style="margin: 10px 0;">
   <li>You may be asked to confirm the SSH connection - type <code>yes</code> and press Enter</li>
   <li>Enter the default Pupper password: <code>rhea123</code></li>
   <li>You should now be connected to Pupper via SSH!</li>
   </ul>
   </div>

.. raw:: html

   <div style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); color: white; padding: 25px; border-radius: 10px; margin: 30px 0; text-align: center;">
   <h3 style="color: white; margin-top: 0;"><‰ You're Connected!</h3>
   <p style="margin-bottom: 0; font-size: 1.1em;">You've successfully established an SSH connection to Pupper over WiFi! If you have any issues or this doesn't work for you, please let us know! =€</p>
   </div>

.. tip::
   **Default Credentials** =

   - Username: ``pi``
   - Password: ``rhea123``

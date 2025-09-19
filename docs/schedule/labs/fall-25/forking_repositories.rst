🍴 Forking Repositories Guide
=============================

.. note::
   **What is Forking?** 🎯
   
   Forking a repository is essentially creating a copy of someone else's GitHub repository on your own personal GitHub account, where you can then make all the changes you want! In our case, we provide you with starter code in these repositories, which you will then fill in to complete the lab assignments.

.. warning::
   **Prerequisites** ⚠️
   
   Before you begin, you **must** have already set up your SSH keys as described in the `GitHub SSH Setup Guide <https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent?platform=linux>`_.

.. raw:: html

   <div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; padding: 20px; border-radius: 10px; margin: 20px 0;">
   <h3 style="color: white; margin-top: 0;">🚀 Let's Get Started!</h3>
   <p style="margin-bottom: 0;">Follow these 4 simple steps to fork your first repository and start coding!</p>
   </div>

Step 1: 🍴 Fork the Repository
------------------------------

.. raw:: html

   <div style="background: #f8f9fa; border-left: 4px solid #28a745; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>📋 Instructions:</strong>
   <ol style="margin: 10px 0;">
   <li>Open the URL for the GitHub Repository in your browser (the URL should be in the lab instructions)</li>
   <li>Look for the <strong>Fork</strong> button at the top right of the repository page</li>
   <li>Click the fork icon to create your own copy</li>
   </ol>
   </div>

.. image:: forking_imgs/create_fork.png
   :alt: Create fork button on GitHub
   :align: center
   :width: 80%

Step 2: ⚙️ Configure Your Fork
------------------------------

.. raw:: html

   <div style="background: #fff3cd; border-left: 4px solid #ffc107; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>💡 Pro Tips:</strong>
   <ul style="margin: 10px 0;">
   <li>You may change the repository name if you'd like, but it's not necessary</li>
   <li><strong>Important:</strong> Make sure the Owner field shows your own GitHub account</li>
   <li>You can add a description if you want to keep track of what this fork is for</li>
   </ul>
   </div>

.. image:: forking_imgs/make_repository.png
   :alt: Repository configuration window
   :align: center
   :width: 80%

Step 3: 🔗 Get the Clone URL
----------------------------

.. raw:: html

   <div style="background: #d1ecf1; border-left: 4px solid #17a2b8; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>🎯 What to Look For:</strong>
   <ul style="margin: 10px 0;">
   <li>You should be redirected to your forked repository page</li>
   <li>Notice the reference to the original repository at the top left</li>
   <li>Click the green <strong>"Code"</strong> button</li>
   <li>Select <strong>"SSH"</strong> tab</li>
   <li>Click the copy icon to copy the SSH URL</li>
   </ul>
   </div>

.. image:: forking_imgs/copy_link.png
   :alt: Copy SSH clone URL
   :align: center
   :width: 80%

Step 4: 💻 Clone to Your Local Machine
--------------------------------------

.. raw:: html

   <div style="background: #d4edda; border-left: 4px solid #28a745; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>🖥️ Terminal Commands:</strong>
   <p style="margin: 10px 0;">Open your terminal and run these commands:</p>
   </div>

.. code-block:: bash
   :emphasize-lines: 1,3

   git clone <paste-your-copied-ssh-url>
   cd <repository_name>
   code .

.. raw:: html

   <div style="background: #e2e3e5; border-left: 4px solid #6c757d; padding: 15px; margin: 15px 0; border-radius: 5px;">
   <strong>📝 What Happens Next:</strong>
   <ul style="margin: 10px 0;">
   <li>The repository will be downloaded to your local machine</li>
   <li>All your changes will automatically sync back to your forked repository</li>
   <li>VSCode will open with your project ready to code!</li>
   </ul>
   </div>

.. image:: forking_imgs/terminal_command.png
   :alt: Terminal commands for cloning and opening in VSCode
   :align: center
   :width: 80%

.. raw:: html

   <div style="background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%); color: white; padding: 25px; border-radius: 10px; margin: 30px 0; text-align: center;">
   <h3 style="color: white; margin-top: 0;">🎉 Congratulations!</h3>
   <p style="margin-bottom: 0; font-size: 1.1em;">You have successfully learned to fork repositories! You'll use this procedure for completing all your lab assignments. Happy coding! 🚀</p>
   </div>

.. tip::
   **Next Steps** 💡
   
   - Start working on your lab assignment
   - Make commits as you progress: ``git add .`` → ``git commit -m "Your message"`` → ``git push``
   - Submit your work when complete!

GitHub Setup Guide
=================

This page contains instructions for setting up your GitHub account on Pupper and uploading the labs into your shared repository.

Setup a Shared Repository
************************

1. Log into GitHub on your browser and go to github.com/new to create a new repository.

   .. figure:: ../_static/github_setup_pics/create_repo.png
       :align: center
       :alt: GitHub new repository page

2. Name your repository (e.g., "CS123") and initialize it with a README file. Leave other options at their defaults.

   .. figure:: ../_static/github_setup_pics/create_repo2.png
       :align: center
       :alt: GitHub repository creation options

After creating the repository, this is how the home page should look like:

    .. figure:: ../_static/github_setup_pics/repo_created.png
        :align: center
        :alt: GitHub new repo page. Initialize README and leave the rest of the options at their defaults.

You can access the repository by clicking the green "Code" button and copying the HTTPS URL.

   .. figure:: ../_static/github_setup_pics/https_url.png
       :align: center
       :alt: Repository HTTPS URL
       :scale: 33%

4. Share the repository with your team:

   - Click the "Settings" gear icon in the top right
   - Select "Manage access"
   - In the "Collaborators" section, click "Add people"
   - Enter your team members' usernames and add them

   .. figure:: ../_static/github_setup_pics/add_collabs.png
       :align: center
       :alt: GitHub manage access page

   .. figure:: ../_static/github_setup_pics/add_collabs2.png
       :align: center
       :alt: Adding collaborators

Set Up SSH Authentication on Pupper
*********************************

The most secure and convenient way to connect your GitHub account to Pupper is using SSH keys. This eliminates the need to enter your password for every push or pull operation.

1. Generate an SSH Key
   Replace your_email@example.com with your GitHub-linked email:

   .. code-block:: bash

      ssh-keygen -t ed25519 -C "your_email@example.com"

   - Press Enter to accept the default file path (~/.ssh/id_ed25519)
   - Optionally set a passphrase for additional security

2. Copy Your Public Key
   
   .. code-block:: bash

      cat ~/.ssh/id_ed25519.pub

   Copy the entire output.

3. Add the Public Key to GitHub
   
   - Log into GitHub in Pupper's browser
   - Go to Settings → SSH and GPG keys
   - Click "New SSH key"
   - Paste your key, add a descriptive title (e.g., "Pupper SSH Key"), and save. This is how the page should look like with my SAIL cluster key:

   .. figure:: ../_static/github_setup_pics/ssh_keys.png
       :align: center
       :alt: GitHub SSH keys page

4. Test the Connection
   
   .. code-block:: bash

      ssh -T git@github.com

   You should see a message like:
   "Hi your-username! You've successfully authenticated, but GitHub does not provide shell access."

5. After Rebooting Pupper in the Future
   
   To check if your key is still loaded:

   .. code-block:: bash

      ssh-add -l

   If your key isn't listed, add it manually:

   .. code-block:: bash

      ssh-add ~/.ssh/id_ed25519

Uploading Labs to Your Shared Repository
**************************************

Since you'll be managing multiple labs in one repository, create a new branch for each lab. Here's how to upload Lab 1:

1. Set your shared repository as the origin remote for the lab (replace the URL with your shared repository URL):
   
   .. code-block:: bash

      git remote set-url origin https://github.com/your-username/CS123.git

2. Create and switch to a new branch for the lab (replace lab1 with the appropriate lab number in future labs):
   
   .. code-block:: bash

      git branch -M lab1

   .. note::
      If you need to rename a branch due to a mistake, use:
    
      .. code-block:: bash

         git branch -m old_name new_name

3. Add and commit your changes (the message will be seen in the GitHub UI):
   
   .. code-block:: bash

      git add .
      git commit -m "Lab 1: Initial implementation"

4. Push to your shared repository:
   
   .. code-block:: bash

      git push -u origin lab1

   .. warning::
      Make sure you're pushing to the correct branch! Pushing to the wrong branch could overwrite other labs' code.

5. Verify your code appears in the lab1 branch on your shared repository:

   .. figure:: ../_static/github_setup_pics/branch_published.png
       :align: center
       :alt: Lab code published on GitHub

Need Help?
---------

If you encounter any issues:

- Check the GitHub documentation
- Ask your TAs or classmates
- Consult the course staff during office hours
- Use online resources like ChatGPT for troubleshooting

Happy coding!
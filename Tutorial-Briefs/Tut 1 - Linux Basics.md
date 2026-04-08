
# Tutorial 1: Environment Setup & Linux Basics

## Introduction

The base of any high-performance system is its operating system. In this tutorial, your team will set up a Virtual Machine (VM) and learn how to navigate the Linux terminal. Mastering these foundational terminal commands is critical for managing cluster nodes efficiently.

### Additional Learning Resources

If you are new to the Linux command line, we highly recommend practicing with these free online terminal simulators before and during this tutorial:

-   [DevOps Daily - Linux Terminal Game](https://devops-daily.com/games/linux-terminal)
    
-   [Webminal](https://webterm.app)
    
-   [VPSWala Free Online Linux Terminal](https://vpswala.org/free-online-linux-terminal.html)
    
-   [LearnCMD](https://www.learncmd.io)
    

----------

## Part 1: Environment Setup

1.  **Create a Team Repository:** Have one member of your team create a GitHub repository. You will use this to upload your deliverables for this and future tutorials.
    
2.  **Install VirtualBox:** Download and install VirtualBox on your host machine.
    
3.  **Download the OS:** Download the **Rocky Linux Minimal ISO**. (Ensure you get the "Minimal" version, as we need a terminal-only environment without a Graphical User Interface).
    
4.  **Create your Virtual Machine:** Create a new VM in VirtualBox using the downloaded Rocky ISO. Configure it with the following baseline settings:
    
    -   **RAM:** 1024 MB
        
    -   **CPU:** 2 Cores
        
    -   **Storage:** 20 GB
        

----------

## Part 2: Navigating the Filesystem

_Boot up your new Rocky Linux VM and log in. You will execute the following steps entirely within the terminal._

5.  **Navigate to the home directory:**
    ```
    cd /home
    
    ```
    
6.  **List the contents:** Check what is inside the `/home` directory to find your user folder.
    ```
    ls -l
    
    ```
    
7.  **Change into your specific user directory:** _(Assuming your user is named `rocky`)_
    ```
    cd /home/rocky
    
    ```
    
8.  **Create and enter a workspace:** Create a new directory specifically for this training and move into it.
    ```
    mkdir uwc-hpc
    cd uwc-hpc
    
    ```
    

----------

## Part 3: File Manipulation & System Info

9.  **Create your tutorial file:** Create an empty text file named after your team or yourself.
    ```
    touch <name>-tutorial-1.txt
    
    ```
    
10.  **Edit the file:** Open the file using a terminal text editor (like `nano` or `vi`) and type your team's name at the very top. Save and exit.
    ```
    nano <name>-tutorial-1.txt
    
    ```
    
11.  **Append system information:** Run the command to get your kernel/system info, and use the `>>` operator to append that output directly to the end of your text file.
    ```
    uname -a >> <name>-tutorial-1.txt
    
    ```
    
12.  **Read and append OS details:** First, display the contents of your OS release file to the screen.
    ```
    cat /etc/os-release
    
    ```
    
    Next, append that exact same output to the bottom of your tutorial file.
    ```
    cat /etc/os-release >> <name>-tutorial-1.txt
    
    ```
    
13.  **Copy a system file:** Finally, copy the actual `os-release` file from the `/etc/` directory into your current `uwc-hpc` directory.
    ```
    cp /etc/os-release .
    
    ```
    

----------

## Deliverables

Once you have completed all the steps, upload the following to your team's GitHub repository:

1.  **Screenshot 1:** Open your final `<name>-tutorial-1.txt` file in a terminal text editor (like `nano`) so all the appended text is visible, and take a screenshot.
    
2.  **Screenshot 2:** Exit the editor and run `ls -l` in your `uwc-hpc` directory to show the contents (it should show your text file and the copied `os-release` file). Take a screenshot of this output.

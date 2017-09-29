Installation: Setting up all components
=================================================

This article provides a step-by-step guide to setup the Caterros project including all dependencies. 



General installation & setup
------------------------------

Install ROS
^^^^^^^^^^^^^^^^^^^
Install and setup ROS indigo on Ubuntu 14.04. Therefore, follow the instructions from http://wiki.ros.org/indigo/Installation/Ubuntu. Recommended is to install the package ros-indigo-desktop-full. 

Install additional packages
^^^^^^^^^^^^^^^^^^^
    1. Install needed tools:: 
    
        sudo apt-get install ros-indigo-catkin ros-indigo-rospack python-wstool
        
    2. Install rosjava::
    
        sudo apt-get install ros-indigo-rosjava ros-indigo-rosjava-*
        
    3. Install the PR2 navigation::
    
        sudo apt-get install ros-indigo-pr2-navigation
        
Setup your Workspace
^^^^^^^^^^^^^^^^^^^
    1. Prepare a catkin workspace for the project (see http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment for further information):: 
    
        mkdir -p ~/suturo_ws/src
        cd ~/suturo_ws/
        catkin_make
        source devel/setup.bash
    
    2. Prepare another catkin workspace for the project dependencies:: 
    
        mkdir -p ~/suturo_dep/src
        cd ~/suturo_dep/
        catkin_make
        source devel/setup.bash
  
  
Clone repositories
------------------------------    

Clone the git repositories of all the modules you need on your machine into the src folder of your project workspace. To run the whole project, each module is needed but they don't have to run on the same machine. The corresponding git command is::

    git clone <URL>
      
You can find all the URLs in the following table: 

+--------------+------------------------------------------+----------------------------------------------+
| Name         | SSH                                      | HTTPS                                        |
+--------------+------------------------------------------+----------------------------------------------+
| planning     | git@github.com:suturo16/planning.git     | https://github.com/suturo16/planning.git     |
+--------------+------------------------------------------+----------------------------------------------+
| perception   | git@github.com:suturo16/perception.git   | https://github.com/suturo16/perception.git   |
+--------------+------------------------------------------+----------------------------------------------+
| knowledge    | git@github.com:suturo16/knowledge.git    | https://github.com/suturo16/knowledge.git    |
+--------------+------------------------------------------+----------------------------------------------+
| manipulation | git@github.com:suturo16/manipulation.git | https://github.com/suturo16/manipulation.git |
+--------------+------------------------------------------+----------------------------------------------+
| suturo_msgs  | git@github.com:suturo16/suturo_msgs.git  | https://github.com/suturo16/suturo_msgs.git  |
+--------------+------------------------------------------+----------------------------------------------+

After you have set up the corresponding dependencies, you can build your workspace again::

    cd ~/suturo_ws/src
    catkin_make

The description of the dependencies of each module can be found below.

suturo_msgs
------------------------------ 
This module is needed by all the other modules, so clone it first into the src folder of your project workspace.

Planning
------------------------------ 

Install EMACS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Since the planning module is implemented in Lisp, you can execute it from Emacs::

    sudo apt-get install ros-indigo-roslisp-repl
 
You can open Emacs using the command::

        roslisp_repl
         
Install CRAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The planning module is based on the Cognitive Robot Abstract Machine (CRAM, see http://ai.uni-bremen.de/research/cram  or http://cram-system.org for further information). You need the full installation of it to run the module. (note: we are using ros-indigo and cram2 for this.) Therefore execute the following commands within the src folder of your dependency workspace::

    git clone https://github.com/cram2/cram_3rdparty.git
    git clone https://github.com/cram2/cram_core.git
    git clone https://github.com/cram2/cram_beliefstate.git
    git clone https://github.com/cram2/cram_common.git
    git clone https://github.com/cram2/cram_json_prolog.git
    git clone https://github.com/code-iai/designator_integration.git

    rosdep install --ignore-src --from-paths cram_3rdparty cram_core cram_beliefstate cram_common cram_json_prolog designator_integration
    cd .. && catkin_make

Note: you also need the iai_common_msgs. If you don't have them yet, please pull them now and catkin_make. Otherwise a lot of things won't work.::
    git clone https://github.com/code-iai/iai_common_msgs.git



[Optional] Install the fast downward planner
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to use the plan generator, you have to install the fast downward planer from http://www.fast-downward.org/ in addtion. This package is not needed for building the planning module. You can find a detailled description of how to setup and use the fast downwards planner at http://www.fast-downward.org/ObtainingAndRunningFastDownward.

1. Create a new folder within your dependency workspace, e.g. "planner". 

2. Within this folder, create a new file named "setup.py" with the following structure::
   
   	#!/usr/bin/env python

	from distutils.core import setup

	setup(name='planner',
    version='1.0',
    description='pddl planning system',
    author='someone',
    author_email='someone@stuff.net',
    url='https://www.python.org/sigs/distutils-sig/',
    packages=['downward'],
    	)      

    You can choose arbitrary values for the given fields.
    
 3. To ensure that all necessary dependencies are installed, execute::
 
        sudo apt-get install cmake g++ g++-multilib mercurial make python
        
 4. Then, you can clone the planer to the folder that you created in step 1::
 
        cd planner
	    hg clone http://hg.fast-downward.org downward
        
 5. Build the planner::
 
        cd downward
	    ./build.py
 
 6. Create an empty file named "__init__.py" within the "downward"-folder.
 
 7. Go to the subfolder "driver" and within the file "main.py" uncomment the line "sys.exit(exitcode)"::
 
        # sys.exit(exitcode)
        
   This is needed because otherwise the plan generator's server won't be able to give a return value when being called.
   
 8. Now, you can finally install the planner as a python module. This is necessary so that the plan generator can get access to it. Go to the folder you created in step 1 and execute::
 
        sudo pip install -e .
	
If you don't have pip installed, you can execute::

	sudo apt-get -y install python-pip
 
Build the planning module
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Return to your project workspace and try to build it. 

If actionlib_lisp cannot be found, you are missing the roslisp_common package. It should have been automatically installed within the ros installation but if it was not, you can add it manually. Therefore, go into the src folder of your dependency workspace and execute::

        git clone git@github.com:ros/roslisp_common.git
        cd .. && catkin_make
       
Now try again to build your project workspace.


Since Lisp uses asdf as it's building tool, building with catkin_make just makes sure you have most of the necessary dependencies. To really properly test if the planning module builds, you have to use emacs. 
Plase take some time to get familiar with Emacs since many of the shortcuts you are familiar with, do completly different things in Emacs. There is a very nice overview of the most usefull shortcuts for Emacs in the context of planning and cram here http://cram-system.org/doc/ide under the point "Setup" and "Useful Tips". 
Keep in mind, that the notation of Emacs shortcuts is also different. (it's explained in the linked tutorial above.)

So now, open an Emacs and type::
    
    M-x slime RET       || loads slime and repl.
    ,                   || promts to enter an command
    r-l-s RET           || short for "ros-load-system"
    plan_execution RET  || package name of the package you want to use. Plan_execution is the high-lvl-package of this module
    ,
    in-package RET      || 
    pexecution          || name of the package you want to work from.

If this all runs through without any exeptions, it means that all your dependencies should be fine and you were able to build the planning module sucessfully. 



[Optional Package for the Turtles]
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If you try to drive around with the turtles and you are sure you've set up amcl and move_base correctly, your odom is fine but the turtle still seems to be getting lost in the map, try installing this package::

    git clone https://github.com/code-iai/snap_map_icp.git

Just put it into your workspace. You will have to adjuts the launchfile to use the topics of your your robot. (Ex: if your robots topic is /tortugabot1/odom then you should set it in the launch file accordingly.)

This will just generally make the navigation and orientation within the map more stable, even if the robot looses connection to the rosmaster sometimes. 


TurtleBot
^^^^^^^^^^
On the turtlebot itself there already should be everything necessary for it to run. To make sure, you can however check if this package https://github.com/code-iai/tortugabot is there. Also it should already have amcl, move_base, and tortugabot_bringup. 

If it has all of the above, the only thing you need to to is clone this package onto the robot::

    git clone https://github.com/suturo16/sut_tortugabot.git

It contains all the launch files which it needs to run in order to work within this system.
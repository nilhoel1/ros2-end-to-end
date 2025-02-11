
# End-To-End Timing Analysis

The repository is used to reproduce the evaluation from

*End-To-End Timing Analysis in ROS2*

for RTSS 2022.

This document is organized as follows:
- [End-To-End Timing Analysis](#end-to-end-timing-analysis)
  - [Environment Setup](#environment-setup)
    - [Requirements](#requirements)
    - [File Structure](#file-structure)
    - [Deployment](#deployment)
  - [How to run the experiments](#how-to-run-the-experiments)
    - [ROS2 system execution](#ros2-system-execution)
    - [ROS2 system simulation and upper bound analysis](#ros2-system-simulation-and-upper-bound-analysis)
    - [System and run time details](#system-and-run-time-details)
  - [How to use VM](#how-to-use-vm)
  - [Overview of the corresponding functions and algorithms](#overview-of-the-corresponding-functions-and-algorithms)
  - [Miscellaneous](#miscellaneous)
    - [Authors](#authors)
    - [Acknowledgments](#acknowledgments)

## Environment Setup
### Requirements

The experiments are split into three parts:

1. ROS2 system execution
2. ROS2 system simulation
3. Upper bound analysis

The ROS2 system execution requires a system running Ubuntu 20.04, while the ROS2 system simulation and upper bound analysis can be run on any system running Python 3. 
Please note that we are running Python 3.8.10, which is the current version that is provided with Ubuntu 20.04.

For the ROS2 system execution, ROS2-Foxy is required.

The installation steps can be found here:
```
https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html
```
Add ROS2 to your bashrc file:
```
echo 'source /opt/ros/foxy/setup.bash' >> ~/.bashrc
```
Install Eclipse Cyclone DDS:
```
sudo apt install ros-foxy-rmw-cyclonedds-cpp
```
Install colcon:
```
sudo apt install python3-colcon-common-extensions
```
To switch to Cyclone DDS, execute the following command after launching every terminal:
```
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```
Or add the line to the .bashrc file in your home folder:
```
echo 'export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp' >> ~/.bashrc
```
Note that you need to start a new terminal to run the commands in the bashrc file.

### File Structure

    .
    ├── ros2_system				# System execution in ROS2-Foxy
    │   └── src				# Package src folder
    │       ├── rt_msgs			# Custom real-time message package
    │       │   ├── msg			# Single ECU chains + results
    │       │   │   ├── RTEntry.msg		# Entries for end-to-end measurements
    │       │   │   └── RTHeader.msg		# Header for end-to-end measurements
    │       │   ├── CMakeLists.txt		# Build instructions
    │       │   └── package.xml		# Package details
    │       └── rt_nodes			# Output for the runtime evaluation
    │           ├── include			# Include files
    │           │   └── rt_nodes		#
    │           │       ├── rt_node.hpp	# Real-time node classes
    │           │       └── rt_register.hpp	# Real-time end-to-end registers
    │           ├── src			# Source files
    │           │   ├── main.cpp		# Main function
    │           │   └── rt_node.cpp		# Real-time node classes
    │           ├── CMakeLists.txt		# Build instructions
    │           └── package.xml		# Package details
    ├── simulation_and_analysis		# Python simulation and upper bound analysis
    │   ├── analysis.py			# Analysis and simulation main script
    │   ├── entry.py				# Entries for end/to/end measurements
    │   ├── executor.py			# ROS2 executor model
    │   ├── message.py				# RTHeader message model
    │   ├── node.py				# ROS2 node model
    │   ├── subscription.py			# ROS2 subscription model
    │   └── timer.py				# ROS2 timer model
    └── README.md
    
The experiments are split into three parts:

1. ROS2 system execution
2. ROS2 system simulation
3. Upper bound analysis

The ROS2 system execution can be found in the ros2_system folder, while the ROS2 system simulation and upper bound analysis can be executed in the simulation_and_analysis folder.
The results of the experiments will be displayed in the terminal that executes the scripts.

### Deployment

The following steps explain how to deploy the experiments on the machine.

First, clone the git repository or download the [zip file](https://github.com/HarunTeper/ros2-end-to-end):
```
git clone https://github.com/HarunTeper/ros2-end-to-end
```
Move into the ros2_system folder, change the permissions of the script to be executable, and execute make_script.sh natively:
```
cd ros2_system
chmod 777 make_script.sh
./make_script.sh
```

## How to run the experiments

We explain how to run each experiment by itself to reproduce the results of the evaluation. Note that we first explain the general command template structure that is required for each script, after which we also add an example command for each template.

### ROS2 system execution

Start a terminal and move into the ros2_system folder:
```
cd ~/ros2-end-to-end/ros2_system
```
Execute the make_script:
```
./make_script.sh
```
Source ROS2-Foxy:
```
source /opt/ros/foxy/setup.bash
```
Source the experiment packages:
```
source install/setup.bash
```
Then, select the experiment and set the system configuration and types using the following command template:
```
ros2 run rt_nodes $experiment $arg1 $arg2
```
You can set the experiment to either feature a navigation_system that is presented in Section 9 or the case_study system from Section 8.

If you select the navigation system, the $type argument specifies the number of sensors of the system. An example system with 4 camera can be launched with:
```
ros2 run rt_nodes run_system navigation_system 4
```
If you select the case study, you can set decide to use an over- or under-utilized system using $arg1, using the following options:
```
over | under
```
Additionally, you need to set the system type with $arg2, using the following options:
| Type                                          | Argument |
| --------------------------------------------- | -------- |
| Subscription fusion and subscription actuator | SS       |
| Subscription fusion and timer actuator        | ST       |
| timer fusion and timer actuator               | TS       |
| timer fusion and timer actuator               | TT       |

For example, to run an over-utilized system with a timer fusion and timer actuator, use the following command:
```
ros2 run rt_nodes run_system case_study over TT
```
When the system is running, it displays the maximum reaction time and maximum data age of each cause-effect chain in the system.

To stop the system execution, press CTRL+C in the terminal in which the system is running.

### ROS2 system simulation and upper bound analysis

Start a terminal and move into the simulaion_and_analysis folder:
```
cd ~/ros2-end-to-end/simulation_and_analysis
```
Select the experiment and set the system configuration and types using the following template command:
```
python3 main.py $simulation_analysis_selection $experiment $arg1 $arg2
```
You can decide if you want to run the simulation, analysis or both, using the following options for the $simulation_analysis_selection argument:
```
simulation | analysis | both
```
You can decide the experiment using the following options:
```
navigation system | case_study
```
If you select the navigation system, you can set the number of sensors using $arg1$ and do not specify an input for $arg2$. For example, to run both the analysis and simulation for the navigation system with 4 cameras, run the following command:
```
python3 main.py both navigation_system 4
```
If you select the case study, you can set decide to use an over- or under-utilized system using $arg1, using the following options:
```
over | under
```
Additionally, you need to set the system type with $arg2, using the following options:

| Type                                          | Argument |
| --------------------------------------------- | -------- |
| Subscription fusion and subscription actuator | SS       |
| Subscription fusion and timer actuator        | ST       |
| timer fusion and timer actuator               | TS       |
| timer fusion and timer actuator               | TT       |

For example, if you want to run the analysis and simulation on the case study with an over-utilized system and a timer fusion and timer actuator, use the following command:
```
python3 main.py both case_study under TT
```
The output of the simulation and analysis includes the end-to-end latencies of all cause-effect chains that are part of the system.

### System and run time details

As a reference, we utilize a machine running Ubuntu 20.04, with an AMD Ryzen 5900 12-Core Processor (12 Cores, 24 Threads) with 3,7GHz and 32GB RAM.

For the virtual machine, we enabled 4 cores and 4096 MB of RAM.

During our experiments, our machine only required a few seconds to run the upper bound analysis and system ROS2 system simulation. For the ROS2 system execution, we only required a few seconds to generate the results for the evaluation. Please note that the timing values of the ROS2 system execution may be different compared to our evaluation due to the system configuration.

## How to use VM

- Please download the [zip file](https://tu-dortmund.sciebo.de/s/Po3B0vSe3UGjEEV), which contains the virtual disk and the machine description. The credential is: ros2end2end/rtss2022
- The source code is deployed on the desktop already. Some common software are installed accordingly, e.g., vscode, git.
- Please follow the above description to test out the provided analyses.
- Please note that the original scripts are for reproducing the results on the paper (so time consuming).
- Please update the installed repository before running any scripts. You can update the repository using the following commands:
```
cd ~/ros2-end-to-end
git pull
```

## Overview of the corresponding functions and algorithms

The following tables describe the mapping between content of our paper and the source code in this repository.

**Section 6** (Cause-Effect Chain Upper Bound Analysis):
| Paper theorem             | Source code | methods                  |
| ------------------------- | ----------- | ------------------------ |
| (1) Maximum Reaction Time | analysis.py | calculate_upper_bounds() |

**Section 7.A** (Executor Simulation):
| Paper algorithm         | Source code | methods                           |
| ----------------------- | ----------- | --------------------------------- |
| (1) Executor Simulation | analysis.py | simulate_next_processing_window() |

**Section 7.B** (Online timing measurement):
| Paper algorithm            | Source code | methods                         |
| -------------------------- | ----------- | ------------------------------- |
| (2) Message Creation       | rt_node.cpp | RTNode::*Callback()             |
| (3) Subscription Execution | rt_node.cpp | RTNode::*SubscriptionCallback() |
| (4) Timer Execution        | rt_node.cpp | RTNode::*TimerCallback()        |

## Miscellaneous

### Authors

* Harun Teper
* Mario Günzel
* Niklas Ueter
* Georg von der Brüggen
* Jian-Jia Chen

### Acknowledgments

This work has been supported by the Federal Ministry of Education
and Research (BMBF) in the course of the project 6GEM under
the funding reference 16KISK038.


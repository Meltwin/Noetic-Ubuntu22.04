The process is pretty similar to the standard [installation from source](http://wiki.ros.org/noetic/Installation/Source) except
for some fixes that you will have to make during the installation. 
I will keep the same header so you can keep a track on both guide at the same time (follow the official website 
for more explication, I will only write the steps to make).

## 1 - Prerequisites

### 1.1 - Installing bootstrap dependencies

To install bootstrap dependencies

```shell
sudo apt-get install python3-rosdep python3-rosinstall-generator python3-vcstools python3-vcstool build-essential
```

### 1.X - Changing rosdep repository adress

The first problem that you will encounter while following the official guide is the lacking of some packages on the apt.
In order to resolve this problem, do these steps:

1. Modify */etc/ros/rosdep/sources.list.d/20-default.list* as following: 
```YAML
# os-specific listings first
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/osx-homebrew.yaml osx

# generic
yaml <enter base.yaml address>
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
yaml https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/ruby.yaml
gbpdistro https://raw.githubusercontent.com/ros/rosdistro/master/releases/fuerte.yaml fuerte

# newer distributions (Groovy, Hydro, ...) must not be listed anymore, they are being fetched from the rosdistro index.yaml instead
```
2. Go to [this gist](https://gist.github.com/Meltwin/0317ae7481c94da7fd66c3eea8d40740) and click **Raw**. Replace **\<enter base.yaml address\>** in the file by the address of the raw file.
4. Save the file (require sudo rights).

### 1.2 - Initializing rosdep
```shell
sudo rosdep init
rosdep update
```

## 2 - Installation

### 2.1 - Create a catkin Workspace

Place yourself in the working folder of your choice. Create a ROS Workspace:
```shell
mkdir ./noetic_ws
cd ./noetic_ws
```

Downloading ROS1 source code.
```shell
rosinstall_generator desktop --rosdistro noetic --deps --tar > noetic-desktop.rosinstall
mkdir ./src
vcs import --input noetic-desktop.rosinstall ./src
```

#### 2.1.1 - Resolving Dependencies

Install all the dependencies with rosdep:
```shell
rosdep install --from-paths ./src --ignore-packages-from-source --rosdistro noetic -y
```

If you are on another distro than classical Ubuntu/Kubuntu/Xubuntu (e.g. Manjaro, Mint, ...), 
add ```--distro=ubuntu:jammy``` at the end of the precendent command:

#### 2.1.2 - Building the catkin Workspace

##### 2.1.2.A Building from the source

Let's build the sources. Run the following commands.
```shell
./src/catkin/bin/catkin_make_isolated -DCMAKE_BUILD_TYPE=Release
```
It's likely that you will have some errors while building the sources. In the next subsections, 
you will have the needed steps to resolve the issue.

Don't worry about the warning and notes (especially about deprecated BOOST_PRAGMA_MESSAGE ), only focus on the errors.

##### 2.1.2.B - Fixing *rosconsole* error

The second error that you will likely encounter is linked to rosconsole and log4cxx. It means that you are likely to have log4cxx 11 or 12.
To fix that you need to follow the changes suggested on [this PR](https://github.com/ros/rosconsole/pull/54) 
and especially [this commit](https://github.com/ros/rosconsole/pull/54/commits/9f930c007dd40aa7ede771b8859b529e024d7bfb).
Here is a quick summary of what to do:
- Open **src/rosconsole/src/rosconsole/impl/rosconsole_log4cxx.cpp** and replace its content by [this version of the file](https://raw.githubusercontent.com/ros/rosconsole/9f930c007dd40aa7ede771b8859b529e024d7bfb/src/rosconsole/impl/rosconsole_log4cxx.cpp)
- Same thing with **src/rosconsole/test/thread_test.cpp** with [this version](https://raw.githubusercontent.com/ros/rosconsole/9f930c007dd40aa7ede771b8859b529e024d7bfb/test/thread_test.cpp)
- Same thing with **src/rosconsole/test/utest.cpp** with [this version](https://raw.githubusercontent.com/ros/rosconsole/9f930c007dd40aa7ede771b8859b529e024d7bfb/test/utest.cpp)

##### 2.1.2.C - Fixing *std::share_mutex* error

The third error that you will likely encounter is a C++ compilation version. It's a problem where the packages wants 
to be compiled with C++11 and use objects from C++17. 

The solution that I found was to change in every CMakeLists.txt of the packages that generate this errors the C++ version 
to use.

You can use [this small python script](https://gist.github.com/Meltwin/1ee35296d2bb86fee19d639580e3c91f) to do that for you. 
Just place it at the root of your ROS workspace and run

```shell
python3 change_cpp.py
```

##### 2.1.2.D - Installing noetic

##### Guide installation

If you want to install noetic in the working folder, run the installation with:

```shell
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
```

##### Opt folder installation (require sudo privileges)

If you want to install noetic in */opt/ros/noetic* you can follow these steps:

1. Run ```sudo mkdir /opt/ros/noetic```
2. Run ```sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/noetic```
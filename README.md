# gazebo_ignition
Repo for using gazebo ignition

Tutorial using gazebo ignition in bumperbot workspace

1. extract the bumperbot_ws.zip file
2. open new terminal and cd to bumperbot_ws folder
3. type "sudo rosdep init" to initialize rosdep tool in this workspace, then update it with "rosdep update"
4. type "rosdep install --from-paths src -y --ignore-src" to scan all the workspace and aslo installing all the missing package
5. we need to instal some python libraries. First, install pip by "sudo apt-get install python3-pip"
6. then type "pip install transforms3d"
7. instal 3 zip files in this link "https://drive.google.com/drive/folders/1_sOZwuD99MJytBb0A0Ic6MiQ2FX6UNmH?usp=sharing" and extract them.
8. copy the extracted files into bumperbot_ws/src/bumperbot_description
9. since we add 3 new files in bumperbot_description package, we need to install them.
10. first we need to declare them in the CMakeLists.txt file. Open CMakeLists.txt file in VsCode.
11. write "models worlds photos" alongside with the "meshes urdf rviz" in row 9. You can find the code in this repo.

        

12. still in the bumperbot_description, open launch/gazebo.launch.py
13. lets declare a new argument below the "model_arg"

        world_name_arg = DeclareLaunchArgument(name="world_name", default_value="empty")
    
    world_name is the parameter that contains the name of our world, either empty, small_house, and small_warehouse. if we didnt set the world_name value, then by default it will open the empty.world.
    The argument that we've just created only contain the name of our world. in order to simulate it in gazebo, we have to provide the full path to the definition of the world, which is adding extension .world.
14. so, lets create a new variable

        world_path = PathJoinSubstitution([
        bumperbot_description,
        "worlds",
        PythonExpression(expression=["'", LaunchConfiguration("world_name"), "'", "+ '.world'"])
        ])

    we also need to import the "PathJoinSubstitution" and "PythonExpression"class from "launch.substitutions" package
    The code above is used to take the name from world_name and then append it with the .world extension.


16. Change "gazebo = IncludeLaunchDescription()" into this code format
        
            gazebo = IncludeLaunchDescription(
                PythonLaunchDescriptionSource([os.path.join(
                    get_package_share_directory("ros_gz_sim"), "launch"), "/gz_sim.launch.py"]),
                launch_arguments={
                    "gz_args": PythonExpression(["'", world_path, " -v 4 -r'"])
                }.items()    
             )
    we have just reformatted the launch_argument to dictionary, and we adding the full directory of the world that we want to start.

17. before being able to simulate our robot, we need to set an environmental variable.
18. to do so, lets create new variable and modify the gazebo_resource_path.

            model_path = str(Path(bumperbot_description).parent.resolve())
            model_path += pathsep + os.path.join(bumperbot_description, "models")

            gazebo_resource_path = SetEnvironmentVariable(
                "GZ_SIM_RESOURCE_PATH", model_path
                )
so this change is about sending out "str(Path(bumperbot_description).parent.resolve())" to global so that we can append it to a new path. Since we are appending a new path, we can import a path separator.

        from os import pathsep

for the moment, we've just added a new path to the first path. The new path here is simply the directory to the models folder.

20. to complete it, we need to declare the new launch argument. so we have to add "world_name_arg" in the LaunchDescription().

            return LaunchDescription([
                model_arg,
                world_name_arg,
                gazebo_resource_path,
                robot_state_publisher_node,
                gazebo,
                gz_spawn_entity,
                gz_ros2_bridge
            ])
21. for the moment, we have finished the configuration gazebo.launch.py file.
22. Before we start to simulate it, we have to build the workspace by opening a new terminal, cd to bumperbot_ws, and colcon build.
23. after that, dont forget to setup it by (. install/setup.bash)
24. then, just write this code to start the gazebo simulation.

            ros2 launch bumperbot_description gazebo.launch.py world_name:=small_house
25. We can change the world by changing the name of the world in the world_name:= parameter. If we didnt add the world name, then by default it will open the empty.world
    
                    
            



    



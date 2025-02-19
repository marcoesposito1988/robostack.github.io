# Contributing

Many thanks for taking the time to read this and for contributing to RoboStack!

This project is in early stages and we are looking for contributors to help it grow. 
The developers are on [gitter](https://gitter.im/RoboStack/Lobby) where we discuss steps forward.

We welcome all kinds of contribution -- code or non-code -- and value them
highly. We pledge to treat everyones contribution fairly and with respect and
we are here to help awesome pull requests over the finish line.

Please note we have a code of conduct, and follow it in all your interactions with the project.

We follow the [NumFOCUS code of conduct](https://numfocus.org/code-of-conduct).


# Adding new packages via pull requests
You can open a pull request that will get build automatically in our CI.

An example can be found [here](https://github.com/RoboStack/ros-noetic/pull/44). Simply add the required packages to the `vinca_*.yaml` files, where the * indicates the desired platform (linux_64, osx, win or linux_aarch64). Ideally, try to add packages to all of these platforms.

Sometimes, it may be required to patch the packages. An example of how to do so can be found in [this PR](https://github.com/RoboStack/ros-noetic/pull/32).


# Testing changes locally

```bash
# First, create a new conda environment and add the conda-forge and robostack channels:

# For ROS2:
conda create -n robostackenv python=3.10
# For ROS1:
conda create -n robostackenv python=3.9

# For both ROS1+2
conda activate robostackenv
conda config --remove channels defaults
conda config --add channels conda-forge
conda config --add channels robostack-staging

# Install some dependencies
mamba install pip conda-build anaconda-client mamba conda catkin_pkg ruamel_yaml rosdistro empy networkx requests boa

# Install vinca
pip install git+https://github.com/RoboStack/vinca.git@master --no-deps

# Clone the relevant repo
git clone https://github.com/RoboStack/ros-humble.git  # or: git clone https://github.com/RoboStack/ros-noetic.git

# Move in the newly cloned repo
cd ros-humble  # or: cd ros-noetic

# Make a copy of the relevant vinca file
cp vinca_linux_64.yaml vinca.yaml  # replace with your platform as necessary

# Now modify vinca.yaml as you please, e.g. add new packages to be built
code vinca.yaml

# Run vinca to generate the recipe; the recipes will be located in the `recipes` folder
vinca --multiple

# Build the recipe using boa:
boa build recipes -m ./.ci_support/conda_forge_pinnings.yaml -m ./conda_build_config.yaml

# You can also generate an azure pipeline locally, e.g.
vinca-azure -d recipes -t mytriggerbranch -p linux-64
# which will create a `linux.yml` file that contains the azure pipeline definition
```

# How does it work?
- The `vinca.yaml` file specifies which packages should be built. 
  - Add the desired package under `packages_select_by_deps`. This will automatically pull in all dependencies of that package, too.
  - Note that all packages that are already build in one of the channels listed under `skip_existing` will be skipped. You can also add your local channel to that list by e.g. adding `/home/ubuntu/miniconda3/conda-bld/linux-64/repodata.json`. 
  - If you want to manually skip packages, you can list them under `packages_skip_by_deps`.
  - If you set `skip_all_deps` to `True`, you will only build packages listed under `packages_select_by_deps` but none of their dependencies.
  - The `packages_remove_from_deps` list allows you to never build packages, even if they are listed as dependencies of other packages. We use it for e.g. the stage simulator which is not available in conda-forge, but is listed as one of the dependencies of the ros-simulator metapackage.
  - If you want to manually rebuild a package that already exists, you need to comment out the channels listed under `skip_existing`. You probably want to set `skip_all_deps: true`, otherwise all dependencies will be rebuilt in this case.
- If the package does not build successfully out of the box, you might need to patch it. To do so, create a new file `ros-$ROSDISTRO-$PACKAGENAME.patch` in the `patch` directory (replace `$PACKAGENAME` with the name of the package, and replace any underscores with hyphens; and replace `$ROSDISTRO` with "noetic" or "galactic"). You can also use platform specifiers to only apply the patch on a specific platform, e.g. `ros-$ROSDISTRO-$PACKAGENAME.win.patch`.
- The `robostack.yaml` and `packages-ignore.yaml` files are the equivalent of the [rosdep.yaml](http://wiki.ros.org/rosdep/rosdep.yaml) and translate ROS dependencies into conda package names (or in the case of the dependencies listed in `packages-ignore.yaml` the dependencies are ignored by specifying an empty list).

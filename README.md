# everest-core

This is the main part of EVerest containing the actual charge controller logic included in a large set of modules.

All documentation and the issue tracking can be found in our main repository here: https://github.com/EVerest/everest

### Prerequisites:

#### Hardware recommendations

It is recommended to have at least 4GB of RAM available to build EVerest.
More CPU cores will optionally boost the build process, while requiring more RAM accordingly.

#### Ubuntu 20.04
```bash
sudo apt update
sudo apt install -y python3-pip git rsync wget cmake doxygen graphviz build-essential clang-tidy cppcheck maven openjdk-11-jdk npm docker docker-compose libboost-all-dev nodejs libssl-dev libsqlite3-dev clang-format curl rfkill
```
In order to build EVerest, we need clang-format version greater than 11. To date *apt* does only install an older version, so we install version 12 manually:
```bash
sudo apt install clang-format-12
```
Then make Ubuntu use version 12:
``` bash
sudo update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-12 100
```
EVerest requires nodejs version >10.20 for node_api version 6+. as Ubuntu 20.4 only installs v10.19 including node_api version <6 by default we need to install a newer version.

Follow [**these instructions**](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions)
to install a newer version, minimum v12.

#### OpenSuse
```bash
zypper update && zypper install -y sudo shadow
zypper install -y --type pattern devel_basis
zypper install -y git rsync wget cmake doxygen graphviz clang-tools cppcheck boost-devel libboost_filesystem-devel libboost_log-devel libboost_program_options-devel libboost_system-devel libboost_thread-devel maven java-11-openjdk java-11-openjdk-devel nodejs nodejs-devel npm python3-pip gcc-c++ libopenssl-devel sqlite3-devel
```

#### Fedora 36
```bash
sudo dnf update
sudo dnf install make automake gcc gcc-c++ kernel-devel python3-pip git rsync wget cmake doxygen graphviz clang-tools-extra cppcheck maven java-11-openjdk java-11-openjdk-devel boost-devel nodejs nodejs-devel npm openssl-devel libsqlite3x-devel curl rfkill
```

### Build & Install:

It is required that you have uploaded your public [ssh key](https://www.atlassian.com/git/tutorials/git-ssh) to [github](https://github.com/settings/keys).

To install the [Everest Dependency Manager](https://github.com/EVerest/everest-dev-environment/blob/main/dependency_manager/README.md), follow these steps:

Install required python packages:
```bash
python3 -m pip install --upgrade pip setuptools wheel jstyleson jsonschema
```
Get EDM source files and change into the directory:
```bash
git clone git@github.com:EVerest/everest-dev-environment.git
cd everest-dev-environment/dependency_manager
```
Install EDM:
```bash
python3 -m pip install .
```
We need to add */home/USER/.local/bin* and *CPM_SOURCE_CACHE* to *$PATH*:
```bash
export PATH=$PATH:/home/$(whoami)/.local/bin
export CPM_SOURCE_CACHE=$HOME/.cache/CPM
```

Now setup EVerest workspace: 
```bash
cd everest-dev-environment/dependency_manager
edm --config ../everest-complete.yaml --workspace ~/checkout/everest-workspace
```

Install [ev-cli](https://github.com/EVerest/everest-utils/tree/main/ev-dev-tools):

Change the directory and install ev-cli:
```bash
cd ~/checkout/everest-workspace/everest-utils/ev-dev-tools
python3 -m pip install .
```

Now we can build EVerest!

```bash
mkdir -p ~/checkout/everest-workspace/everest-core/build
cd ~/checkout/everest-workspace/everest-core/build
cmake ..
make install
```

(Optional) In case you have more than one CPU core and more RAM availble you can use the following command to significantly speed up the build process:
```bash
cmake -j$(nproc) ..
make -j$(nproc) install
```
*$(nproc)* puts out the core count of your machine, so it is using all available CPU cores!
You can also specify any number of CPU cores you like.
 
Done!

<!--- WIP: [everest-cpp - Init Script](https://github.com/EVerest/everest-utils/tree/main/everest-cpp) -->

### Simulation

In order to test your build of Everest you can simulate the code on your local machine! Check out the different configuration files to run EVerest and the corresponding nodered flows in the [config folder](config/).

 Check out this [guide for using EVerest SIL](https://everest.github.io/tutorials/run_sil/index.html


### Troubleshoot

**1. Problem:** "make install" fails with complaining about missing header files.

**Cause:** Most probably your *clang-format* version is older than 11 and *ev-cli* is not able to generate the header files during the build process.

**Solution:** Install newer clang-format version and make Ubuntu using the new version e.g.:
```bash
sudo apt install clang-format-12
sudo update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-12 100
```
Verify clang-format version:
```bash
clang-format --version
Ubuntu clang-format version 12.0.0-3ubuntu1~20.04.4
```
To retry building EVerest delete the entire everest-core/**build** folder and recreate it. 
Start building EVerest using *cmake ..* and *make install* again.


**2. Problem:** Build speed is very slow.

**Cause:** *cmake* and *make* are only utilizing one CPU core.

**Solution:** use 
```bash
cmake -j$(nproc) .. 
```
and 
```bash
make -j$(nproc) install
```
to use all available CPU cores.
Be aware that this will need roughly an additional 1-2GB of RAM per core.
Alternatively you can also use any number between 2 and your maximum core count instead of *$(nproc)*.


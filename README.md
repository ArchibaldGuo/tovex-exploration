# TOVEX

**Gain-Preserving Topological Voxel-Sphere Exploration with Branch-Aware Target Selection**

[![ROS](https://img.shields.io/badge/ROS-Noetic-22314E.svg)](https://wiki.ros.org/noetic)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04-E95420.svg)](https://releases.ubuntu.com/20.04/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Release](https://img.shields.io/badge/release-upon%20paper%20acceptance-lightgrey.svg)](#release-status)

TOVEX is a completion-oriented autonomous exploration framework for LiDAR-equipped UAVs. It attaches unresolved volumetric evidence from an incremental ESDF to a sparse Topological Voxel-Sphere (TVS) graph, evaluates reachable graph routes with path-integrated gain, and applies branch-aware structural preferences only when their utility remains within an explicit fraction of the best currently admissible gain.

The framework is designed for confined and branch-dense environments in which rapid broad coverage may still leave short terminal passages or partially resolved connectors unexplored. It does not use global map-coordinate guidance, prescribed branch orders, ideal trajectories, or ground-truth occupancy during planning.

## Release Status

This repository is currently being prepared for public release. The source code, frozen experiment configurations, common-volume evaluator, and analysis scripts used by the associated paper will be made public upon acceptance of the manuscript.

Large ROS bags, complete point-cloud archives, and site-sensitive physical-run records are not planned for inclusion in the repository. Availability of additional research artifacts will be described in the paper's Data Availability Statement.

## Method Overview

TOVEX contains three coupled stages:

1. **Metric perception and TVS construction.** LiDAR observations update a TSDF/ESDF map. Positive-clearance samples generate voxel spheres that own nearby unresolved voxels, producing a sparse graph over navigable space.
2. **Gain-preserving exploration decisions.** The unresolved evidence encountered along each reachable graph path is accumulated with distance decay. Short-branch, branch-completion, and route-continuation candidates are admitted only above relative-gain gates; otherwise, TOVEX falls back to the base-gain maximizer.
3. **Collision-aware execution.** The accepted graph route is revalidated and passed to the local trajectory planner. Reachable re-anchoring, bounded path relay, cooldown checks, and duplicate-command suppression improve execution continuity without changing the volumetric gain definition.

The intended perception chain is:

```text
/velodyne_points -> Voxblox TSDF/ESDF -> TOVEX TVS graph -> local trajectory planner
```

The accumulated global point cloud is used for visualization, logging, and offline evaluation. It is not used as an oracle map for exploration decisions.

## Planned Public Repository Layout

```text
tovex-exploration/
├── src/
│   ├── free_exploration/       # TVS construction, graph search, and target selection
│   └── tovex_sim/              # Gazebo/PX4 integration and launch files
├── config/
│   ├── simulation/             # Frozen simulation profiles used in the paper
│   └── hardware/               # Safety-oriented UAV and UGV deployment profiles
├── evaluation/
│   └── common_volume/          # Planner-independent reachable-free evaluator
├── scripts/
│   ├── run_simulation.sh
│   ├── reproduce_tables.py
│   └── reproduce_figures.py
├── docs/
│   ├── architecture.md
│   ├── parameters.md
│   └── troubleshooting.md
├── third_party/
│   └── README.md               # Upstream versions and required external packages
├── CITATION.cff
├── LICENSE
└── README.md
```

The public release will not redistribute EDEN, GBPlanner, TARE, EGO-Planner, Voxblox, PX4, or FAST-LIO source code. Upstream revisions, installation links, and TOVEX-specific adapters or patches will be documented separately so that the ownership and license boundaries remain explicit.

## Requirements

The reference environment is expected to use:

- Ubuntu 20.04;
- ROS Noetic with catkin;
- C++14;
- PCL and Eigen3;
- OpenMP;
- Voxblox and `voxblox_ros`;
- Gazebo 11, PX4 SITL, and MAVROS for UAV simulation;
- EGO-Planner for collision-aware local trajectory generation.

FAST-LIO is used by the physical deployment profile but is not required by the core TVS target-selection module.

Exact tested revisions and installation instructions will be included in `docs/dependencies.md` before the public release.

## Build

The following commands show the intended release workflow. The repository URL will be updated when the project becomes public.

```bash
mkdir -p ~/tovex_ws/src
cd ~/tovex_ws/src
git clone https://github.com/<ORG>/tovex-exploration.git

cd ~/tovex_ws
rosdep install --from-paths src --ignore-src -r -y
catkin_make -DCMAKE_BUILD_TYPE=Release
source devel/setup.bash
```

## Simulation Quick Start

The cleaned public launch interface will provide separate commands for the simulator, autonomy stack, and visualization:

```bash
# Terminal 1: Gazebo, PX4 SITL, and simulated LiDAR
roslaunch tovex_sim maze_px4_gazebo.launch

# Terminal 2: Voxblox, TOVEX, and local trajectory planning
roslaunch tovex_sim maze_autonomy.launch

# Terminal 3: optional visualization
rviz -d $(rospack find tovex_sim)/rviz/tovex_maze.rviz
```

Headless and data-recording examples will be provided in `scripts/`. The public commands will reproduce the released configuration without requiring hard-coded global branch coordinates or a prescribed route.

## Core Configuration

The paper configuration is defined entirely through versioned ROS launch and YAML files. The principal parameter groups are:

- TVS sphere radius, center-separation factor, and edge radius;
- minimum unresolved evidence and completion threshold;
- path-gain distance decay;
- relative-gain gates for short branches, branch completion, and route continuation;
- structural route-length, degree, and heading predicates;
- cooldown, re-anchoring, progress, and command-resend checks;
- platform-specific velocity, acceleration, altitude, and clearance limits.

The frozen launch manifests are the source of truth for all numerical values. Simulation and hardware profiles are kept separate because the physical deployments use additional safety restrictions and do not enable every structural policy used in the full simulation configuration.

## Paper Reproduction

The release is planned to include:

- the frozen TOVEX configuration used for the simulation study;
- common reachable-free coverage evaluation;
- trajectory-distance, revisit, and turning metrics;
- scripts that regenerate the quantitative tables and plots from trajectory CSV files;
- parameter and environment manifests with version hashes;
- visual-validity criteria used to reject wall crossing, persistent in-place circling, and deadlocked runs.

The evaluator uses ground-truth environment geometry only offline to define a common reference volume. No ground-truth map is provided to TOVEX or any baseline during exploration.

## Physical Deployment Profiles

The simulation profile contains the complete gain-preserving structural hierarchy. The archived warehouse and garage deployments share TVS construction, node updates, path-integrated base gain, and graph search, while using reduced policy sets and platform-specific collision-aware execution for site safety. These profiles will be identified explicitly rather than presented as identical software configurations.

## Citation

If you use TOVEX, please cite the associated paper. The final bibliographic metadata and DOI will be added after publication.

```bibtex
@article{guo_tovex,
  title   = {TOVEX: Gain-Preserving Topological Voxel-Sphere Exploration with Branch-Aware Target Selection},
  author  = {Guo, Fenghe and Zhu, Xingbao and Sun, Chenyang and Zhang, Junrui and Shen, Runjie},
  journal = {Drones},
  year    = {forthcoming}
}
```

## License and Third-Party Notices

The planned TOVEX release will use the MIT License while preserving all existing copyright and license notices inherited from the `free_exploration` package. Third-party dependencies remain subject to their respective licenses and are not covered by the TOVEX license.

Before public release, every source file, model asset, configuration, and patch will be audited for license compatibility, personal paths, credentials, and private experiment records.

## Contact

Questions and reproducibility requests may be submitted through GitHub Issues after the repository becomes public. Author contact details will be added after acceptance.


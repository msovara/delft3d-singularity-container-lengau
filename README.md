## **User Notes**  

# 🌀 Delft-3D Flexible Mesh Singularity Container on LENGAU

**Author**: Mthetho Vuyo Sovara  
**Updated**: 25 February 2025 \
**Last Updated**: 05 March 2025

---

## 📖 Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Initial Setup](#initial-setup)
4. [Container Inspection & Validation](#container-inspection--validation)
5. [Executing Delft3D Tools](#executing-delft3d-tools)
6. [Data Management: Binding Host Directories](#data-management-binding-host-directories)
7. [Running Batch Jobs](#running-batch-jobs)
8. [Troubleshooting](#troubleshooting)
9. [Additional Resources](#additional-resources)

---

## 🌟 Introduction <a name="introduction"></a>
This guide provides step-by-step instructions for running **Delft3D Flexible Mesh** hydrodynamic modeling tools using **Singularity containers** on the **CHPC Lengau cluster**. Singularity ensures reproducible execution by encapsulating all dependencies in a portable container.

---

## 🛠️ Prerequisites <a name="prerequisites"></a>
- ✅ CHPC Lengau cluster account  
- ✅ Basic Linux command-line knowledge  
- ✅ Some Familiarity with Delft3D workflow  

---

## 🚀 Initial Setup <a name="initial-setup"></a>

### 1. Login and Module Configuration
```bash
ssh username@lengau.chpc.ac.za  # Replace with your username
module purge                    # Clean environment
module load chpc/singularity/3.5.3
module load chpc/earth/delft3d-container/delft3d-singularity
module list                     # Verify loaded modules
```
### 2. Verify Singularity Image Path
After loading the module, check the SINGULARITY_IMAGE environment variable:
```bash
echo $SINGULARITY_IMAGE
```
Expected Output:
Path to `.sif` file, e.g., `/apps/chpc/earth/delft3d-container/centos7_delft3d4-65936_sha256...sif`

---
## 🔍 Container Inspection & Validation <a name="container-inspection--validation"></a>

### 3. Inspect Container Content
```bash
singularity exec $SINGULARITY_IMAGE ls -l /opt/delft3d/bin/
```

### 4. Check Library Dependencies
```bash
singularity exec $SINGULARITY_IMAGE ldd /opt/delft3d/bin/d_hydro
```
- Note: All libraries should show valid paths. "Not found" errors indicate container configuration issues.

### 5. View Build Script (If Available)
```bash
singularity inspect --deffile $SINGULARITY_IMAGE
```
---

🛠️ Executing Delft3D Tools <a name="executing-delft3d-tools"></a>

### 6. Basic Execution Methods
   Interactive Shell Example
```bash
singularity shell $SINGULARITY_IMAGE
Singularity> d_hydro --help
```
Single Command Execution
```bash
singularity exec $SINGULARITY_IMAGE d_hydro --help
```
### 7. Binding Example: Running a Test Case (Internal Bind). Do not run!
```bash
singularity exec --no-home \
  -B /mnt/lustre/users/msovara:/output \
  $SINGULARITY_IMAGE \
  /opt/delft3d/bin/d_hydro \
  /opt/delft3d/examples/config.xml \
  /opt/delft3d/examples/input.mdw \
  > /home/msovara/lustre/delft-output.log 2>&1
```
Command Breakdown:

- `--no-home:` Prevents binding of the host home directory into the container.
- `-B /mnt/lustre/users/msovara:/output:`Binds the host directory to /output inside the container.
- `/opt/delft3d/bin/d_hydro`: The executable inside the container.
- `/opt/delft3d/examples/config.xml`: Path to the configuration file inside the container.
- `/opt/delft3d/examples/input.mdw`: Path to the input file inside the container.
- `> /output/delft-output.log 2>&1`: Redirects stdout and stderr to a log file on the host.
---

### 8. Delft3D Input Files for Simulation
Delft3D requires several input files to run a simulation. Below is a breakdown of the essential files and how they are used.

### 📂 Core Input Files

#### 🔹 1. Grid File (`.grd`)
Defines the computational grid, including:
- Bathymetry data
- Coordinates
- Grid cell connectivity  

**Example:** `grid.grd`

#### 🔹 2. Main Input File (`.mdf`)
Master configuration file that specifies:
- Simulation time and timestep  
- Physical and numerical parameters  
- Links to other input files  

**Example:** `flow2d3d.mdf`

#### 🔹 3. Boundary Conditions File (`.bnd`)
Defines open boundaries for:
- Water levels  
- Discharges  
- Wind forcing (if applicable)  

**Example:** `boundary.bnd`

#### 🔹 4. Initial Conditions File (`.ini`)
Sets the starting conditions for:
- Water levels  
- Velocities  
- Salinity and temperature (optional)  

**Example:** `initial.ini`

#### 🔹 5. Depth File (`.dep`)
Provides depth values at grid points (if not embedded in the `.grd` file).  
**Example:** `depth.dep`

---

🚀 Running Batch Jobs <a name="running-batch-jobs"></a>
### 9. The setup: 
#### Delft3D executable will read the config.xml file and its dependencies so ensure that paths are real and applicable.
The XML file configures a hydrodynamic simulation using Deltares' flow2D3D module. It specifies the model definition file (r17.mdf), enables remote control via DelftOnline, and sets the output directory for simulation results. The simulation is set to start with the flow2D3D module named myNameFlow.will

## XML Code

- Place the config.xml file in the appropriate directory (where the input data is located makes /input binding simpler).
- Ensure the referenced files (r17.mdf, r17.url) are available.
- Run the simulation using Deltares' software.
---
### 10. Parallel Execution Template: LENGAU Job Script:
___
📂 Data Management: Binding Host Directories <a name="data-management-binding-host-directories"></a>
Best Practices:

- Use absolute paths for binding.
- Avoid binding unnecessary directories.
- Maintain separate input/output directories.

🛠️ Troubleshooting <a name="troubleshooting"></a>
Common Issues:

- Remove error and log files from previous runs. 
- Permission Denied: Verify .sif file accessibility with ls -l $SINGULARITY_IMAGE.
- Missing Dependencies: Use ldd check from Section 4.
- XML Errors: Validate with xmllint before execution.
- Environment Conflicts: Use --cleanenv flag if needed:
  ```bash
  singularity exec --cleanenv $SINGULARITY_IMAGE ...
  ```
Additional Resources <a name="additional-resources"></a>
- How to singularity: https://wiki.chpc.ac.za/howto:singularity
- Delft3D Official Documentation: https://oss.deltares.nl/web/delft3d

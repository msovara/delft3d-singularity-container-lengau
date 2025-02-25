## **User Notes**  

# 🌀 Delft3D Flexible Mesh Singularity Container on LENGAU

**Author**: Mthetho Vuyo Sovara  
**Last Updated**: 25 February 2025  

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
### 9. Parallel Execution Template
```bash
#!/bin/bash
#PBS -N delft3d_job                # Job name
#PBS -l select=1:ncpus=32:mpiprocs=32  # Request 1 node with 32 CPUs
#PBS -l walltime=02:00:00           # Set a 2-hour time limit
#PBS -q workq                       # Submit to the 'workq' queue
#PBS -o delft3d_output.log           # Standard output log file
#PBS -e delft3d_error.log            # Standard error log file

# Load necessary modules
module load singularity

# Define variables
CONTAINER="/home/apps/chpc/earth/delft3d-singularity-container/centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif"
EXECUTABLE="/opt/delft3d/bin/run_dflow2d3d.sh"
INPUT_DIR="/home/msovara/lustre/SoftwareBuilds/delft3d_interactive_oneapi/delft3dfm-68819/examples/01_standard"
OUTPUT_DIR="/home/msovara/lustre/delft3d_output"

# Create output directory if it doesn't exist
mkdir -p $OUTPUT_DIR

# Change to working directory
cd $INPUT_DIR || exit 1

# Run Delft3D-FM inside Singularity container using MPI
mpirun -np 32 singularity exec $CONTAINER $EXECUTABLE > $OUTPUT_DIR/delft3d_run.log 2>&1
```
---

📂 Data Management: Binding Host Directories <a name="data-management-binding-host-directories"></a>
Best Practices:

- Use absolute paths for binding.
- Avoid binding unnecessary directories.
- Maintain separate input/output directories.

🛠️ Troubleshooting <a name="troubleshooting"></a>
Common Issues:

- Permission Denied: Verify .sif file accessibility with ls -l $SINGULARITY_IMAGE.
- Missing Dependencies: Use ldd check from Section 4.
- XML Errors: Validate with xmllint before execution.
- Environment Conflicts: Use --cleanenv flag if needed:
  ```bash
  singularity exec --cleanenv $SINGULARITY_IMAGE ...
  ```
Additional Resources <a name="additional-resources"></a>
- https://wiki.chpc.ac.za/howto:singularity
- Delft3D Official Documentation: https://oss.deltares.nl/web/delft3d

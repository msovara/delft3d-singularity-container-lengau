## **User Notes**  

# Running the Delft3D Flexible Mesh Singularity Container on LENGAU

**Author**: Mthetho Vuyo Sovara 
**Last Updated**: 12 February 2025

---

## Table of Contents
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

## Introduction <a name="introduction"></a>
This guide explains how to run Delft3D hydrodynamic modeling tools using Singularity containers on the CHPC Lengau cluster. Singularity allows reproducible execution by encapsulating dependencies in a portable container.

---

## Prerequisites <a name="prerequisites"></a>
- CHPC Lengau cluster account
- Basic Linux command-line knowledge
- Familiarity with Delft3D workflow

---

## Initial Setup <a name="initial-setup"></a>

### 1. Login and Module Configuration
```bash
ssh username@lengau.chpc.ac.za  # Replace with your username
module purge                    # Clean environment
module load chpc/singularity/3.5.3
module load chpc/earth/delft3d-container/delft3d-singularity
module list                     # Verify loaded modules
```

### 2. Check if the singularity image variable is correctly set:
After loading the module, verify that the SINGULARITY_IMAGE environment variable is set properly by running:
```bash
echo $SINGULARITY_IMAGE
```
Expected output: Path to .sif file like:
`/apps/chpc/earth/delft3d-container/centos7_delft3d4-65936_sha256...sif`

## Container Inspection & Validation <a name="container-inspection--validation"></a>
### 3. Inspect container content
```bash
singularity exec $SINGULARITY_IMAGE ls -l /opt/delft3d/bin/
```

### 4. Check library dependencies 
```bash
singularity exec $SINGULARITY_IMAGE ldd /opt/delft3d/bin/d_hydro
```
- **Note**: All libraries should show valid paths. "Not found" errors indicate container configuration issues.

### 5. View Build Script (If available)
```bash
singularity inspect --deffile /home/apps/chpc/earth/delft3d-singularity-container/centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb.sif
```
----

## Executing Delft3D Tools <a name="executing-delft3d-tools"></a>

### 6. Basic Execution Methods
**Interactive Shell**
```bash
singularity shell $SINGULARITY_IMAGE
Singularity> delwaq1 --help
```
**Single Command Execution**
```bash
singularity exec $SINGULARITY_IMAGE delwaq1 --help
```
### 7. Running Test case: Internal Bind
```bash
singularity exec --no-home \
  -B /mnt/lustre/users/msovara:/output \
  $SINGULARITY_IMAGE \
  /opt/delft3d/bin/d_hydro \
  /opt/delft3d/examples/config.xml \
  /opt/delft3d/examples/input.mdw \
  > /home/msovara/lustre/delft-output.log 2>&1
```
 **Explanation of the Command**
- --no-home: Prevents binding of the host home directory into the container.
- -B /mnt/lustre/users/msovara:/output: Binds the host directory /mnt/lustre/users/msovara to /output inside the container.
- /opt/delft3d/bin/d_hydro: The executable inside the container.
**- /opt/delft3d/examples/config.xml: Path to the configuration file inside the container.**
**- /opt/delft3d/examples/input.mdw: Path to the input file inside the container.**
- > /output/delft-output.log 2>&1: Redirects both stdout and stderr to a log file on the host.

### 8. Running Test Case: External Bind

## Delft3D Input Files for Simulation

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
In the `example` directory, d_flow2d3d is missing, but there is a wrapper script:
```bash
singularity exec /home/apps/chpc/earth/delft3d-singularity-container/centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif \
  /opt/delft3d/bin/run_dflow2d3d.sh
 > /mnt/lustre/users/msovara/new-delft3d-dir/output/delft-output.log 2>&1
```
**Avoiding Conflicts**
- Use --no-home to prevent automatic binding of your home directory.
- Use explicit bind paths to avoid overlapping directories.
- Ensure the container paths do not conflict with internal paths (e.g., /opt/delft3d).

## Data Management: Binding Host Directories <a name="data-management-binding-host-directories"></a>

**Multi-directory Example**:
```bash
singularity exec \
  -B /lustre/users/you/data:/mnt/data \
  -B /lustre/users/you/configs:/mnt/configs \
  $SINGULARITY_IMAGE \
  /opt/delft3d/bin/delwaq1 -i /mnt/configs/input.dat
```
**Best Practices**:
- Use absolute paths for binding
- Avoid binding unnecessary directories
- Maintain separate input/output directories

## Running Batch Jobs <a name="running-batch-jobs"></a>
### 10. Sample PBS Script (Sequential Execution)
```bash
#!/bin/bash
#PBS -N delft3d_sequential
#PBS -l select=1:ncpus=4:mem=8GB
#PBS -l walltime=02:00:00
#PBS -j oe
#PBS -o /path/to/output.log

module load chpc/earth/delft3d-container/delft3d-singularity

# Define paths
HOST_DATA="/lustre/users/yourusername/data"
SIF_IMAGE=$SINGULARITY_IMAGE

# Execute workflow
singularity exec -B $HOST_DATA:/mnt/data $SIF_IMAGE \
  /opt/delft3d/bin/delwaq1 /mnt/data/input1.dat

singularity exec -B $HOST_DATA:/mnt/data $SIF_IMAGE \
  /opt/delft3d/bin/delwaq2 /mnt/data/input2.dat
```
### 11. Parallel Execution Template
```bash
#!/bin/bash
#PBS -N delft3d_parallel
#PBS -l select=1:ncpus=8:mem=16GB
...

# Run concurrent processes
for sim in config{1..4}; do
  singularity exec -B $HOST_DATA:/mnt/data $SIF_IMAGE \
    /opt/delft3d/bin/dflowfm /mnt/data/${sim}.xml &
done
wait  # Wait for all background jobs
```
## 12. Troubleshooting <a name="troubleshooting"></a>
**Common Issues**:
- Permission Denied: Verify .sif file accessibility with ls -l $SINGULARITY_IMAGE
- Missing Dependencies: Use ldd check from Section 4
- XML Errors: Validate with xmllint before execution
- Environment Conflicts: Use --cleanenv flag if needed:
```singularity exec --cleanenv $SINGULARITY_IMAGE ...```

## 13. Additional Resources <a name="additional-resources"></a>
- https://wiki.chpc.ac.za/howto:singularity
- Delft3D Official Documentation

---

## **Additional Notes**
- If you encounter permission issues, check that the `.sif` file is accessible.
- Ensure that `/path/to/data` is correctly mapped inside the container.
- If needed, use `singularity run --cleanenv` to avoid environment conflicts.

---


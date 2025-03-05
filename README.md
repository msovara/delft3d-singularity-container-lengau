## **User Notes**  

# 🌀 Delft3D Flexible Mesh Singularity Container on LENGAU

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

```xml
# Deltares Hydrodynamic Simulation Configuration

This document describes the XML configuration file used for setting up a hydrodynamic simulation using Deltares' `flow2D3D` module.

## File Overview
- **File Name**: `config.xml`
- **Encoding**: `iso-8859-1`
- **Schema**: [Deltares Hydro Schema](http://content.oss.deltares.nl/schemas/d_hydro-1.00.xsd)
- **Created By**: Deltares (`create_config_xml.tcl`, Version 1.00)
- **Creation Date**: 06 March 2013, 18:09:37
- **File Version**: 1.00

## XML Structure

### Root Element
- `<deltaresHydro>`: The root element of the XML file.
  - Namespace: `http://schemas.deltares.nl/deltaresHydro`
  - Schema Location: `http://content.oss.deltares.nl/schemas/d_hydro-1.00.xsd`

### Documentation
- `<documentation>`: Contains metadata about the file, including creation details.

### Control Section
- `<control>`: Defines the control flow of the simulation.
  - `<sequence>`: Specifies the sequence of operations.
    - `<start>myNameFlow</start>`: Starts the simulation with the `flow2D3D` module named `myNameFlow`.

### Flow2D3D Configuration
- `<flow2D3D name="myNameFlow">`: Configures the `flow2D3D` module.
  - `<library>flow2d3d</library>`: Specifies the library to be used.
  - `<mdfFile>r17.mdf</mdfFile>`: Points to the model definition file (`r17.mdf`).

### DelftOnline Configuration
- `<delftOnline>`: Configures remote control and monitoring.
  - `<enabled>true</enabled>`: Enables DelftOnline.
  - `<urlFile>r17.url</urlFile>`: Specifies the URL file for communication.
  - `<waitOnStart>false</waitOnStart>`: Does not wait for a client connection on startup.
  - `<clientControl>true</clientControl>`: Allows client control (start, step, stop, terminate).
  - `<clientWrite>false</clientWrite>`: Prevents client from modifying data.

### Output Configuration
- `<output>`: Specifies the output directory for simulation results.
  - `<directory>output/r17</directory>`: Output files will be saved in `output/r17`.
```
## XML Code
Here is an example XML configuration:
```xml
<?xml version="1.0" encoding="iso-8859-1"?>
<deltaresHydro xmlns="http://schemas.deltares.nl/deltaresHydro"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://schemas.deltares.nl/deltaresHydro http://content.oss.deltares.nl/schemas/d_hydro-1.00.xsd">
    <documentation>
        File created by    : Deltares, create_config_xml.tcl, Version 1.00
        File creation date : 06 March 2013, 18:09:37
        File version       : 1.00
    </documentation>
    <control>
        <sequence>
            <start>myNameFlow</start>
        </sequence>
    </control>
    <flow2D3D name="myNameFlow">
        <library>flow2d3d</library>
        <mdfFile>r17.mdf</mdfFile>
    </flow2D3D>
    <delftOnline>
        <enabled>true</enabled>
        <urlFile>r17.url</urlFile>
        <waitOnStart>false</waitOnStart>
        <clientControl>true</clientControl>    <!-- client allowed to start, step, stop, terminate -->
        <clientWrite>false</clientWrite>    <!-- client allowed to modify data -->
    </delftOnline>
    <output>
        <directory>output/r17</directory>
    </output>
</deltaresHydro>
```
- Place the config.xml file in the appropriate directory (where the input data is located makes /input binding simpler).
- Ensure the referenced files (r17.mdf, r17.url) are available.
- Run the simulation using Deltares' software.
---
### 10. Parallel Execution Template: LENGAU Job Script:
```bash
#!/bin/bash
#PBS -N delft3d_job
#PBS -l select=1:ncpus=24:mpiprocs=24
#PBS -P ERTH1609
#PBS -q smp
#PBS -l walltime=00:30:00
#PBS -o /home/bgweba/lustre/TNPA_Delft3D/examples/03_flow-wave/delft3d_output/delft3d.out
#PBS -e /home/bgweba/lustre/TNPA_Delft3D/examples/03_flow-wave/delft3d_output/delft3d.err
#PBS -m abe
#PBS -M bgweba@gmail.com

# Clear any loaded modules
module purge

# Load required modules
module load chpc/singularity/3.5.3
module load chpc/openmpi/4.1.1/gcc-7.3.0
module load chpc/earth/delft3d-container/delft3d-singularity

# Define variables
SINGULARITY_IMAGE="/home/apps/chpc/earth/delft3d-singularity-container/centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif"
EXECUTABLE="/opt/delft3d/bin/run_dflow2d3d.sh"
INPUT_DIR="/home/bgweba/lustre/TNPA_Delft3D/examples/03_flow-wave"
OUTPUT_DIR="/home/bgweba/lustre/TNPA_Delft3D/examples/03_flow-wave/delft3d_output"

# Create directories
mkdir -p $OUTPUT_DIR
#mkdir -p $INPUT_DIR/r17
#mkdir -p $INPUT_DIR/input/r17  # Explicitly create /input/r17 directory

# Change to working directory
cd $INPUT_DIR || { echo "Failed to change to INPUT_DIR: $INPUT_DIR"; exit 1; }

# Run Delft3D-FM inside Singularity container using MPI
mpirun -np $nproc singularity exec \
   --bind $INPUT_DIR:/input \
   --bind $OUTPUT_DIR:/output \
   $SINGULARITY_IMAGE $EXECUTABLE > $OUTPUT_DIR/delft3d_run_flow_wave.log 2>&1
```
___
📂 Data Management: Binding Host Directories <a name="data-management-binding-host-directories"></a>
Best Practices:

- Use absolute paths for binding.
- Avoid binding unnecessary directories.
- Maintain separate input/output directories.

🛠️ Troubleshooting <a name="troubleshooting"></a>
Common Issues:

- Remove error and log files from precious runs. 
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

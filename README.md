## **User Notes**  

# Running Delft3D via Singularity/Docker Containers on Lengau Cluster

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
# Expected output: Path to .sif file like:
# /apps/chpc/earth/delft3d-container/centos7_delft3d4-65936_sha256...sif
```
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
### 7. Running Test case:
```bash
singularity exec $SINGULARITY_IMAGE \
  /opt/delft3d/bin/d_hydro \
  /opt/delft3d/examples/config.xml \
  /opt/delft3d/examples/input.mdw \
  > output.log 2>&1
```
## Data Management: Binding Host Directories <a name="data-management-binding-host-directories"></a>

### 8. Basic Directory Binding
```bash
singularity exec -B /host/path:/container/path $SINGULARITY_IMAGE delwaq1
```
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
### 9. Sample PBS Script (Sequential Execution)
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
### 10. Parallel Execution Template
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
## 11. Troubleshooting <a name="troubleshooting"></a>
**Common Issues**:
- Permission Denied: Verify .sif file accessibility with ls -l $SINGULARITY_IMAGE
- Missing Dependencies: Use ldd check from Section 4
- XML Errors: Validate with xmllint before execution
- Environment Conflicts: Use --cleanenv flag if needed:
```singularity exec --cleanenv $SINGULARITY_IMAGE ...```

## 12 Additional Resources <a name="additional-resources"></a>
- https://wiki.chpc.ac.za/howto:singularity
- Delft3D Official Documentation



### Under Construction...
To start an interactive session inside the container, run:  
```bash
singularity shell centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif
```
You will see a `Singularity>` prompt. Inside the shell, you can run:  
```bash
delwaq1 --help
```

### 11. Run a Specific Command from Outside the Container  
If you prefer to run delft3d without entering the container shell:  
```bash
singularity exec centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif delwaq1 --help
```
---

### 12. Running Delft3D examples
This should execute the d_hydro program with the .mdw and XML configuration input files and log the output into output.log. Make sure that both the input files and the executable are accessible inside the container.
```bash
singularity exec $SINGULARITY_IMAGE /opt/delft3d/bin/d_hydro /opt/delft3d/examples/config.xml /opt/delft3d/examples/input.mdw > /mnt/lustre/users/msovara/delft3d-output.log 2>&1
```
- **NOTE**: d_hydro tries to read an XML configuration file. Verify that the XML configuration file is correctly formatted. Even a small syntax issue (such as a missing tag or unclosed element) can cause this error.

### 13. Binding to a Host Directory
Scenario:
You want to bind a directory on the host, for example, /lustre/usernam/data, to a directory inside the container, say /mnt/data, so that you can access the host's data from within the container
```bash
singularity exec -B /home/username/data:/mnt/data $SINGULARITY_IMAGE /opt/delft3d/bin/delwaq1
```
Explanation:
- -B /home/username/data:/mnt/data: This binds the /home/username/data directory on your host machine to /mnt/data inside the container.
- $SINGULARITY_IMAGE: This is the path to your Singularity image (which you already have set).
- /opt/delft3d/bin/delwaq1: This is the executable file for delwaq1 within the container.

What Happens:
- The /home/username/data directory from the host system is now accessible inside the container at /mnt/data.
- When delwaq1 runs, it can read from and write to the /mnt/data directory inside the container, which actually corresponds to /home/username/data on your host system.

### 14. Example for Multiple Directories:
If you need to bind multiple directories, for example, /home/username/configs and /home/username/results, you can use the following command: 
```bash
singularity exec -B /home/username/data:/mnt/data -B /home/username/configs:/mnt/configs -B /home/username/results:/mnt/results $SINGULARITY_IMAGE /opt/delft3d/bin/delwaq1
```
In this case, /home/username/data, /home/username/configs, and /home/username/results on the host will be accessible inside the container at /mnt/data, /mnt/configs, and /mnt/results, respectively.

### 15. Running serial Delft3D jobs in Batch Mode 
Sequential Execution (Run binaries one after the other)
If the binaries need to be run one after the other, you can modify your script to include each command in sequence. 
```bash
#!/bin/bash
#PBS -N delft3d_batch_job       # Job name
#PBS -l select=1:ncpus=4:mem=8GB # Resources: 1 node, 4 CPUs, 8GB memory
#PBS -l walltime=02:00:00        # Wall time (2 hours)
#PBS -j oe                       # Combine stdout and stderr into one file
#PBS -o /home/username/job_output.log  # Output log file

# Load the required module for Singularity and the Delft3D container
module load chpc/earth/delft3d-container/delft3d-singularity

# Define the path to the Singularity image
export SINGULARITY_IMAGE="/home/username/chpc/earth/delft3d-singularity-container/centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif"

# Set the host directory to be bound (replace with actual directory paths)
HOST_DATA_DIR="/home/username/data"
CONTAINER_DATA_DIR="/mnt/data"

# Define the Delft3D binaries
DELWAQ1_BINARY="/opt/delft3d/bin/delwaq1"
DELWAQ2_BINARY="/opt/delft3d/bin/delwaq2"
DFLOWFM_BINARY="/opt/delft3d/bin/dflowfm"

# Bind the host directory to the container directory and run the binaries sequentially
singularity exec -B $HOST_DATA_DIR:$CONTAINER_DATA_DIR $SINGULARITY_IMAGE $DELWAQ1_BINARY > /home/msovara/delft3d_output.log 2>&1
singularity exec -B $HOST_DATA_DIR:$CONTAINER_DATA_DIR $SINGULARITY_IMAGE $DELWAQ2_BINARY >> /home/msovara/delft3d_output.log 2>&1
singularity exec -B $HOST_DATA_DIR:$CONTAINER_DATA_DIR $SINGULARITY_IMAGE $DFLOWFM_BINARY >> /home/msovara/delft3d_output.log 2>&1

```
Explanation:
The binaries (delwaq1, delwaq2, and dflowfm) are executed one after the other.
The output for each binary is appended to the same log file (delft3d_output.log) using >>.

### 16. Parallel Execution (Run binaries simultaneously)
If the binaries can be run in parallel, you can modify the script to execute them concurrently using background jobs (&) or GNU parallel. Here’s an example using background jobs:
```bash
#!/bin/bash
#PBS -N delft3d_batch_job       # Job name
#PBS -l select=1:ncpus=4:mem=8GB # Resources: 1 node, 4 CPUs, 8GB memory
#PBS -l walltime=02:00:00        # Wall time (2 hours)
#PBS -j oe                       # Combine stdout and stderr into one file
#PBS -o /home/username/job_output.log  # Output log file

# Load the required module for Singularity and the Delft3D container
module load chpc/earth/delft3d-container/delft3d-singularity

# Define the path to the Singularity image
export SINGULARITY_IMAGE="/home/username/chpc/earth/delft3d-singularity-container/centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif"

# Set the host directory to be bound (replace with actual directory paths)
HOST_DATA_DIR="/home/username/data"
CONTAINER_DATA_DIR="/mnt/data"

# Define the Delft3D binaries
DELWAQ1_BINARY="/opt/delft3d/bin/delwaq1"
DELWAQ2_BINARY="/opt/delft3d/bin/delwaq2"
DFLOWFM_BINARY="/opt/delft3d/bin/dflowfm"

# Run the binaries in parallel using background jobs
singularity exec -B $HOST_DATA_DIR:$CONTAINER_DATA_DIR $SINGULARITY_IMAGE $DELWAQ1_BINARY > /home/msovara/delft3d_output.log 2>&1 &
singularity exec -B $HOST_DATA_DIR:$CONTAINER_DATA_DIR $SINGULARITY_IMAGE $DELWAQ2_BINARY >> /home/msovara/delft3d_output.log 2>&1 &
singularity exec -B $HOST_DATA_DIR:$CONTAINER_DATA_DIR $SINGULARITY_IMAGE $DFLOWFM_BINARY >> /home/msovara/delft3d_output.log 2>&1 &

# Wait for all background jobs to finish
wait
```
Explanation:
The binaries are executed in parallel using the & operator to run each command in the background.
The wait command ensures that the script waits for all the background jobs to finish before it terminates.

---

## **Additional Notes**
- If you encounter permission issues, check that the `.sif` file is accessible.
- Ensure that `/path/to/data` is correctly mapped inside the container.
- If needed, use `singularity run --cleanenv` to avoid environment conflicts.

---


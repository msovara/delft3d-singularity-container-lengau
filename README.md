# Singularity & Docker Containers on Lengau  
A guide to running the Delft3D Singularity Container on Lengau. 

## **User Notes**  

### 1. Sign into Login02 on Lengau  
Before using the container, sign in and load the required modules:  
```bash
ssh username@lengau.chpc.ac.za
module purge
module load chpc/singularity/3.5.3
module load chpc/earth/delft3d-container/delft3d-singularity
module list
```

### 2. Check if the singularity image variable is correctly set:
After loading the module, verify that the SINGULARITY_IMAGE environment variable is set properly by running:
```bash
echo $SINGULARITY_IMAGE
```

### 3. Inspect the container structure
```bash
singularity exec $SINGULARITY_IMAGE ls -l /opt/delft3d/bin/
```

### 4. Test the Singularity container by running the delwaq1 command i.e. Run delwaq1 using Singularity
- After loading the module and confirming the SINGULARITY_IMAGE path is set, run the following command to execute delwaq1 inside the Singularity container:
```bash
singularity exec $SINGULARITY_IMAGE /opt/delft3d/bin/delwaq1
```
- The delwaq1 binary exists inside ```/opt/delft3d/bin/``` and has executable permissions ```(-rwxr-xr-x)```. Run wrapper script:

```bash
singularity exec $SINGULARITY_IMAGE /opt/delft3d/bin/run_delwaq1.sh
```
---

### 5. Alternatively, you can open a Shell Inside the Container  
To start an interactive session inside the container, run:  
```bash
singularity shell centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif
```
You will see a `Singularity>` prompt. Inside the shell, you can run:  
```bash
delwaq1 --help
```

### 6. Run a Specific Command from Outside the Container  
If you prefer to run delft3d without entering the container shell:  
```bash
singularity exec centos7_delft3d4-65936_sha256.d24792169bd11f937b709f6456a73289229d621464e32271533dbc2b77cfbb9b.sif delwaq1 --help
```
---

### 7. Binding to a Host Directory
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

### 8. Example for Multiple Directories:
If you need to bind multiple directories, for example, /home/username/configs and /home/username/results, you can use the following command: 
```bash
singularity exec -B /home/username/data:/mnt/data -B /home/username/configs:/mnt/configs -B /home/username/results:/mnt/results $SINGULARITY_IMAGE /opt/delft3d/bin/delwaq1
```
In this case, /home/username/data, /home/username/configs, and /home/username/results on the host will be accessible inside the container at /mnt/data, /mnt/configs, and /mnt/results, respectively.

### 9. Running serial Delft3D jobs in Batch Mode 
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

### 10. Parallel Execution (Run binaries simultaneously)
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


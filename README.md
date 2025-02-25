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

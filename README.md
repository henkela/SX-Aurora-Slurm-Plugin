# SX-Aurora-Slurm-Plugin
gres_plugin for Slurm 
It allows for scheduling whole aurora cards on nodes using gres in Slurm but not to share one aurora between jobs. 

Checkout https://github.com/henkela/SX-Aurora-Slurm-Plugin/releases for the latest release. 
This has been tested on 20.02.4.

## Getting started
### Compiling 
You should be used to compiling custom code for Slurm. Briefly:
1. Clone this repo to src/plugins/gres/aurora in your local copy of Slurm or unpack the release tarball and copy the files to that folder
2. Add ''src/plugins/gres/aurora/Makefile'' to the configure.ac file in the slurm root directory
3. rm existing configure file
4. Add ''aurora'' to the SUBDIRS variable in src/plugins/gres/aurora/Makefile.am
5. Run autoreconf
6. make && make install if this is a new Slurm-source-tree. or patch the slurm.spec file
  
I would strongly advise to build a separate slurm cluster on the aurora-nodes for testing (that's at least how I did it; mind that slurmctld and slurmdbd are needed because of the gres-usage. And don't forget to change the Ports in the slurm.conf if you have other slurmctlds running) because many things can go wrong while doing the reconfiguration which may crash the slurmctld which in turn would be "suboptimal" in a productive cluster. Once you have everything up and running with the test-cluster, you can move it to the productive environment. 

### Slurm-Configuration
1. You should have the nodes configured in your slurm.conf or an approriate include file. The node definition should look similar to
  ```bash
  GresTypes=aurora
  SelectType=select/cons_tres
  Nodename=<nodename> Gres=aurora:<count>
  ```

  If you have multiple VEs per VH and want to share them between different jobs, you have to create a shared parition like the one below
  In this case it is also recommended to define CPUs as well as memory of the nodes as shared resources like in the example below with two A100
  ```bash
  GresTypes=aurora
  SelectType=select/cons_tres
  SelectTypeParameters=CR_Core_Memory
  NoneName=vh[100-101] CPUs=80 Sockets=2 CoresPerSocket=20 ThreadsPerCore=2 RealMemory=192078 Gres=aurora:8
  PartitionName=aurora Shared=Yes Nodes=vh[100-101]
  ```

2. You gres.conf should contain at least: 
  ```bash 
  Nodename=<nodename> File=/dev/veslot[<ve slot numbers as csv>]
  ```

  So for the example with the two A100 it would be
  ```bash
  NodeName=vh0t[100-101] Name=aurora File=/dev/veslot[0,1,2,3,4,5,6,7]
  ```

3. Your cgroups.conf needs to have ContrainDevices=yes, which raises the necessity for 
4. you need cgroup_allowed_devices_file.conf to contain at the very least (but check slurm-doku on that, please)
  ```bash 
  /dev/cpu/*/* 
  ```
  
Once the configuration files are changed restart slurmctld and the slurmds. Run a little test like: 

```bash 
srun -n1 --gres=aurora:1 -p<yourpartition> env|sort 
```
  
and check for SLURM-variables being set, i.e. VE_NODE_NUMBER should also be there. 

The example below show two slurm jobs with 4 VEs each, that run on the same A100 in parallel

  ```bash
  VE_NODE_NUMBER=0,1,2,3                                       |  VE_NODE_NUMBER=0,1,2,3
  AURORA_VISIBLE_DEVICES=0,1,2,3                               |  AURORA_VISIBLE_DEVICES=0,1,2,3
  SLURM_JOBID=30                                               |  SLURM_JOBID=31
  SLURM_JOB_AURORAS=0,1,2,3                                    |  SLURM_JOB_AURORAS=4,5,6,7
  ```

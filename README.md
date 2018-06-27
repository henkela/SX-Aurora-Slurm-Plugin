# SX-Aurora-Slurm-Plugin
gres_plugin for Slurm 
It allows for scheduling whole aurora cards on nodes using gres in Slurm but not to share one aurora between jobs. 

Checkout https://github.com/henkela/SX-Aurora-Slurm-Plugin/releases for the latest release. 

## Getting started
### Compiling 
You should be used to compiling custom code for Slurm. Briefly:
1. Clone this repo to src/plugins/gres/aurora in your local copy of Slurm or unpack the release tarball and copy the files to that folder
2. Add ''src/plugins/gres/aurora/Makefile'' to the configure.ac file
3. Run ./autogen.sh 
4. Add aurora to src/plugins/gres/aurora/Makefile.am 
5. Run ./configure --prefix=<your slurm install>
6. make && make install if this is a new Slurm-source-tree. 
  
I would strongly advise to build a separate slurm cluster on the aurora-nodes for testing (that's at least how I did it; mind that slurmctld and slurmdbd are needed because of the gres-usage. And don't forget to change the Ports in the slurm.conf if you have other slurmctlds running) because many things can go wrong while doing the reconfiguration which may crash the slurmctld which in turn would be "suboptimal" in a productive cluster. Once you have everything up and running with the test-cluster, you can move it to the productive environment. 

### Slurm-Configuration
1. You should have the nodes configured in your slurm.conf or an approriate include file. The node definition should look similar to 
  ```bash
  Nodename=<nodename> Gres=aurora:<count> 
  ```
2. You gres.conf should contain at least: 
  ```bash 
  Nodename=<nodename> File=/dev/veslot[0,1] 
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

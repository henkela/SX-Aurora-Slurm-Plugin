# SX-Aurora-Slurm-Plugin
gres_plugin for Slurm 
It allows for scheduling whole aurora cards on nodes using gres in Slurm but not to share one aurora between jobs. 

Checkout https://github.com/henkela/SX-Aurora-Slurm-Plugin/releases for the latest release. 

## Getting started
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

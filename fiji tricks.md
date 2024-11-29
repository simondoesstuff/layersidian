Make it more clear when inside a slurm node.
```bash
PS1="\$SLURMD_NODENAME$ "
```
(best in `.bashrc`). Good for interactive jobs.

---
Good practice for gpu jobs to explicitly request amount of gpus.
```bash
srun -p nvidia-a100 --gres=gpu:1  ...
```
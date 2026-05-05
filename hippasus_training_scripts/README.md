## Running Training Scripts on Hippasus with a Bash Script

This workflow runs LeRobot training directly on the Hippasus GPU server using a bash script. It does not use Slurm. The training is launched inside [tmux]([url](https://en.wikipedia.org/wiki/Tmux)) so it can keep running after disconnecting from SSH.

#### 1. SSH into Hippasus
```bash
ssh jzt25@hippasus.doc.ic.ac.uk
```

Go to the script directory:

```bash
cd ~/Documents/HippasusScripts
```

#### 2. Check available GPUs

```bash
nvidia-smi
```

Hippasus has 4 GPUs. Pick a GPU with low memory usage and low utilization. For example, if GPU 0 is free, the training script should include:

```bash
export CUDA_VISIBLE_DEVICES=0 # if GPU 1 is free, set equal to 1
```

#### 3. Create the training bash script

Create the script:

```bash
nano train_pi05_1k.sh
```

Example scripts are in the hippasus_training_scripts folder.

Then, make scripts executable:

```bash
chmod +x train_pi05_1k.sh
```

Note: May need to check login by doing: 

```bash
hf auth whoami # Should output huggingface username
hf auth login # If hf auth whoami doesn't work, login
```

#### 5. Start training with tmux

Create a tmux session:

```bash
tmux new -s pi05_1k
```

Inside tmux, run the bash script and save logs:

```bash
cd ~/Documents/HippasusScripts
./train_pi05_1k.sh 2>&1 | tee pi05_1k_train.log
```

The command does two things:

```bash
./train_pi05_1k.sh
```

runs the training script.

```bash
2>&1 | tee pi05_1k_train.log
```

prints output to the terminal and saves the same output to pi05_1k_train.log.

#### 6. Confirm training started

Expected output should include lines like:

```bash
Creating dataset
Creating policy
Loading model from: lerobot/pi05_base
All keys loaded successfully!
Creating optimizer and scheduler
Start offline training on a fixed dataset
Training:   0%|          | ...
```

In a second SSH terminal, monitor GPU usage:

```bash
watch -n 1 nvidia-smi
```

The selected GPU should show a Python process and increased memory usage.

#### 7. Safely disconnect while training continues

Detach from tmux by doing Ctrl and b together, release them, then press d.

After detaching, you can safely close the SSH terminal. The training will continue running on Hippasus.

#### 8. Check progress later

SSH back into Hippasus.

Option A: reattach to the live tmux session:

```bash
tmux attach -t pi05_1k
```

Detach again with:

```bash
Ctrl-b d
```

Lowkey sus doing this. Only really need to reattach if you want to kill the training run. Option B is preferred. 

Option B: check the log without reattaching:

```bash
cd ~/Documents/HippasusScripts
tail -f pi05_1k_train.log
```

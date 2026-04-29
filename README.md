## Recording a Dataset

To record a new dataset, use the `lerobot-record` command. The flags below configure the follower/leader arms, camera streams, and dataset parameters.

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/tty.usbmodem5A7C1187061 \
  --robot.id=blue_follower \
  --robot.cameras="{wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, world: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 25}}" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/tty.usbmodem5B141114541 \
  --teleop.id=blue_leader \
  --display_data=true \
  --dataset.repo_id=justintiensmith/multicolour_block_pick_place \
  --dataset.num_episodes=50 \
  --dataset.single_task="Pick up the red block and carefully place it in the black bin" \
  --dataset.streaming_encoding=false \
  --dataset.encoder_threads=4 \
  --dataset.fps=15
```

lerobot-record --robot.type=so101_follower --robot.port=/dev/tty.usbmodem5A7C1187061 --robot.id=blue_follower --robot.cameras="{wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 25}, world: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}" --teleop.type=so101_leader --teleop.port=/dev/tty.usbmodem5AB01583971  --teleop.id=blue_leader --display_data=true --dataset.repo_id=justintiensmith/red_block_precision-multicolour_block_pick_place --dataset.num_episodes=1 --dataset.single_task="Pick up the blue block and carefully place it in the black bin" --dataset.streaming_encoding=true --dataset.encoder_threads=4 --dataset.fps=25 --resume=true --dataset.root=./data/multicolour_block_pick_place_2


Terminal 1: Policy Server
First, SSH into the remote machine. If you need to share the port on your local device, use the port-forwarding option:

```bash
# Standard SSH
ssh jzt25@devi.doc.ic.ac.uk

```bash
# OR SSH with port forwarding
ssh -L 8080:localhost:8080 jzt25@devi.doc.ic.ac.uk

Once connected, set up your environment variables, activate the Conda environment, and start the server:

```bash
export HF_HOME=/vol/dissolve/justin/hf_cache
export TRANSFORMERS_CACHE=/vol/dissolve/justin/hf_cache
export HF_DATASETS_CACHE=/vol/dissolve/justin/hf_cache
export PIP_CACHE_DIR=/vol/dissolve/justin/pip_cache

export HF_TOKEN=$"HF_TOKEN"

export TORCHINDUCTOR_DISABLE=1
export TORCH_COMPILE_DISABLE=1
export CUDA_LAUNCH_BLOCKING=1

source /vol/dissolve/justin/miniforge3/etc/profile.d/conda.sh
conda activate /vol/dissolve/justin/miniforge3/envs/lerobot

python -m lerobot.async_inference.policy_server --host=127.0.0.1 --port=8080

Terminal 2:

python -m lerobot.async_inference.robot_client     --server_address=127.0.0.1:8080     --robot.type=so101_follower     --robot.port=/dev/tty.usbmodem5A7C1187061     --robot.id=blue_follower_arm     --robot.cameras="{wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, world: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}"     --task="Pick up the red block and carefully place it in the black bin"     --policy_type=pi05     --pretrained_name_or_path=justintiensmith/pi05_30k_multicolour_block_pick_place     --policy_device=cuda     --actions_per_chunk=50     --chunk_size_threshold=0.5     --aggregate_fn_name=weighted_average  

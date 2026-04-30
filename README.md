# LeRobot Training & Inference Notes

This repository contains scripts and documentation for training and running inference on SO-101 robot arms. Training scripts are located in the /training_scripts folder. 

More documentation can be found here: https://huggingface.co/docs/lerobot

Matt's models and datasets: https://huggingface.co/mattpidden

My models and datasets: https://huggingface.co/justintiensmith

## Robot Port Reference

| Robot Role | Color | Port Address |
| :--- | :--- | :--- |
| **Leader** | Blue | `/dev/tty.usbmodem5AB01583971` |
| **Follower** | Blue | `/dev/tty.usbmodem5A7C1187061` |
| **Leader** | Orange | `/dev/tty.usbmodem5B141114541` |
| **Follower** | Orange | `/dev/tty.usbmodem5B141129191` |

## Calibrating the Robot

```bash
# Follower Calibration (blue)
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem5A7C1187061 \
    --robot.id=blue_follower

# Leader Calibration (blue)
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=/dev/tty.usbmodem5AB01583971 \ 
    --teleop.id=blue_leader 
```

## Datasets

#### Recording a Dataset
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
  --dataset.fps=25
```

#### Downloading Dataset:

```bash
hf download justintiensmith/consistent-block-pick-and-place-into-basket-large \
  --repo-type dataset \
  --local-dir ./data/consistent-block-pick-and-place-into-basket-large 
```

#### Deleting Episodes from a Dataset
```bash
Deleting episodes from the dataset:

lerobot-edit-dataset --repo_id=justintiensmith/consistent-block-pick-and-place-into-basket-large --operation.type=delete_episodes --operation.episode_indices="[159, 160, 161, 162, 163, 164]" --root=./data/consistent-block-pick-and-place-into-basket-large --push_to_hub=true
```

## Running Inference
Running inference requires setting up a policy server and a robot client across two separate terminals.

#### Terminal 1: Policy Server
First, SSH into the remote machine. If you need to share the port on your local device, use the port-forwarding option:

```bash
# Standard SSH
ssh jzt25@devi.doc.ic.ac.uk

# OR SSH with port forwarding
ssh -L 8080:localhost:8080 jzt25@devi.doc.ic.ac.uk
```

Once connected, set up your environment variables, activate the Conda environment (using the pi05 environment in this example), and start the server:

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
conda activate /vol/dissolve/justin/miniforge3/envs/lerobot_pi05

python -m lerobot.async_inference.policy_server --host=127.0.0.1 --port=8080
```

#### Terminal 2: Robot Client
In a new terminal, activate the relevant lerobot environment locally and then run the robot client to connect to the policy server and execute the task:

```bash
python -m lerobot.async_inference.robot_client \
    --server_address=127.0.0.1:8080 \
    --robot.type=so101_follower \
    --robot.port=/dev/tty.usbmodem5B141129191 \
    --robot.id=orange_follower \
    --robot.cameras="{wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 25}, world: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}" \
    --task="Pick up the red block and carefully place it in the black bin" \
    --policy_type=pi05 \
    --pretrained_name_or_path=mattpidden/pi05_5k_precision-multicolour_block_pick_place \
    --policy_device=cuda \
    --actions_per_chunk=50 \
    --chunk_size_threshold=0.5 \
    --aggregate_fn_name=weighted_average
```

## Human in the Loop (HIL)

#### Dataset Collection
Example HIL dataset collection for SmolVLA.

```bash
python examples/hil/hil_data_collection.py \
    --rtc.enabled=true \
    --rtc.execution_horizon=20 \
    --rtc.max_guidance_weight=5.0 \
    --rtc.prefix_attention_schedule=LINEAR \
    --robot.type=so101_follower \
   --robot.port=/dev/tty.usbmodem5B141129191 --robot.cameras="{camera1: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, camera2: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/tty.usbmodem5AB01583971 \
  --teleop.id=blue_leader \
    --policy.path=mattpidden/smolvla_10k_precision-multicolour_block_pick_place \
    --dataset.repo_id=justintiensmith/hil-smolvla-test-dataset \
    --dataset.single_task="Pick up the red block and carefully place it in the black bin" \
    --dataset.fps=25 \
    --dataset.episode_time_s=1000 \
    --dataset.num_episodes=25 \
    --interpolation_multiplier=3
```

#### Controls
```bash
    SPACE  - Pause policy
    c      - Take control
    p      - Resume policy after pause/correction
    →      - End episode
    ESC    - Stop and push to hub
```

## Other Helpful Commands:

```bash
lerobot-find-cameras opencv

lerobot --help
```

## Helpful Resources:

Clothes folding blog: https://huggingface.co/spaces/lerobot/robot-folding

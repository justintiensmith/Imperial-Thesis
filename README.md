Recording a dataset command flags:

lerobot-record --robot.type=so101_follower --robot.port=/dev/tty.usbmodem5A7C1187061 --robot.id=blue_follower_arm --robot.cameras="{wrist: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30}, world: {type: opencv, index_or_path: 1, width: 640, height: 480, fps: 30}}" --teleop.type=so101_leader --teleop.port=/dev/tty.usbmodem5B141114541 --teleop.id=orange_leader_arm --display_data=true --dataset.repo_id=justintiensmith/multicolour_block_pick_place --dataset.num_episodes=50 --dataset.single_task="Pick up the red block and carefully place it in the black bin" --dataset.streaming_encoding=false --dataset.encoder_threads=4 --dataset.fps=15

Running Inference:

Terminal 1:

ssh jzt25@devi.doc.ic.ac.uk

or to share port on local device:

ssh -L 8080:localhost:8080 jzt25@devi.doc.ic.ac.uk

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

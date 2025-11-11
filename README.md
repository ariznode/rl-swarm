# RL Swarm

RL Swarm is a peer-to-peer system for reinforcement learning. It allows you to train models collaboratively with others in the swarm, leveraging their collective intelligence. It is open source and permissionless, meaning you can run it on a consumer laptop at home or on a powerful GPU in the cloud. You can also connect your model to the Gensyn Testnet to receive an on-chain identity that tracks your progress over time.

## Requirements

**Recommended Hardware**

- 8 core CPU, 32 GB RAM.



OR

- CUDA devices (officially supported):
    - RTX 3090
    - RTX 4090
    - RTX 5090
    - A100
    - H100

## Instructions

### Run the Swarm

#### 1. Install Dependencies

```sh
sudo apt update && apt upgrade -y
sudo apt install screen curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
sudo apt install python3 python3-pip python3-venv python3-dev -y
sudo apt update

apt update
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
node -v

npm install -g yarn
yarn -v
```

#### 2. Clone this repo

```sh
git clone https://github.com/gensyn-ai/rl-swarm
```

#### 2. Running Swarm Inside Screen

Create screen session

```sh
screen -S swarm
```

#### Run Swarm node

```sh
cd rl-swarm
python3 -m venv .venv
. .venv/bin/activate
bash ./run_rl_swarm.sh
```

### Login

<p align="left">
  <img src="https://drive.google.com/uc?export=view&id=1MH_CGdYOXEovDo6quBUIw_z53TwxKRTS" width="600" alt="Waiting Login">
</p>
After your terminal showing like this you need to detach screen by pressing `CTRL A + D`

and you need to tunneling http://localhost:3000/ by enter this command
```sh
sudo npm install -g localtunnel
curl https://loca.lt/mytunnelpassword
lt --port 3000
```
you will get url to login and your ip ass tunnel password

After success login attach swarm screen

```sh
screen -r swarm
```
Wait untill you see

<p align="left">
  <img src="https://drive.google.com/uc?export=view&id=13Ah-NvXGgjFvo_YwGPA-RDnK-up8_rcl" width="600" alt="N">
</p>

During setup, you'll be asked if you'd like to participate in the **1. Hugging Face** **2. Model** **. AI Prediction Market**.

1. N then ENTER
2. Choose your Model
   Models:
   - Gensyn/Qwen2.5-0.5B-Instruct
   - Qwen/Qwen3-0.6B
   - nvidia/AceInstruct-1.5B
   - dnotitia/Smoothie-Qwen3-1.7B
   - Gensyn/Qwen2.5-1.5B-Instruct

   If you want to use default mode press ENTER
3. Y then ENTER (to join Judge)

After this setup you can see your node - [Gensyn Testnet Dashboard](https://dashboard.gensyn.ai/).

### Back up Node

If you are Termius user you can back up swarm.pem with sftp

<p align="left">
  <img src="https://drive.google.com/uc?export=view&id=1pn6OXJ0c-0GbaDYTMOKfC1X_bZV2-4M8" width="600" height="400" alt="Gambar dari Google Drive">
</p>

### Key Commands

1. Attach screen to check node logs
```sh
screen -d -r swarm
```
2. Activate virtual environment
```sh
rm -rf .venv && git pull && python3 -m venv .venv && source .venv/bin/activate
```
3. Start node
```sh
pkill -f rl-swarm || true && bash run_rl_swarm.sh
```

## Node Update

Run in swarm directory

```bash
cd && cd-rl-swarm
```

```bash
git stash && git pull
```

```bash
sudo rm rgym_exp/src/manager.py
```

```bash
nano rgym_exp/src/manager.py
```

Copy all of this
```bash
import os
os.environ["TOKENIZERS_PARALLELISM"] = "false"
import time
from collections import defaultdict
import subprocess
import re
import logging
logging.getLogger("hivemind").setLevel(logging.CRITICAL)

from genrl.blockchain import SwarmCoordinator
from genrl.communication import Communication
from genrl.communication.hivemind.hivemind_backend import HivemindBackend
from genrl.data import DataManager
from genrl.game import BaseGameManager
from genrl.game.game_manager import DefaultGameManagerMixin
from genrl.logging_utils.global_defs import get_logger
from genrl.logging_utils.system_utils import get_system_info
from genrl.rewards import RewardManager
from genrl.roles import RoleManager
from genrl.state import GameState
from genrl.trainer import TrainerModule
from huggingface_hub import login, whoami
from hivemind import DHT

from rgym_exp.src.utils.name_utils import get_name_from_peer_id
from rgym_exp.src.prg_module import PRGModule


class SwarmGameManager(BaseGameManager, DefaultGameManagerMixin):
    """GameManager that orchestrates a game using a SwarmCoordinator."""

    def __init__(
        self,
        coordinator: SwarmCoordinator,
        max_stage: int,
        max_round: int,
        game_state: GameState,
        reward_manager: RewardManager,
        trainer: TrainerModule,
        data_manager: DataManager,
        communication: Communication,
        role_manager: RoleManager | None = None,
        run_mode: str = "train",
        log_dir: str = "logs",
        hf_token: str | None = None,
        hf_push_frequency: int = 20,
        **kwargs,
    ):
        super().__init__(
            max_stage=max_stage,
            max_round=max_round,
            game_state=game_state,
            reward_manager=reward_manager,
            trainer=trainer,
            data_manager=data_manager,
            communication=communication,
            role_manager=role_manager,
            run_mode=run_mode,
        )

        assert isinstance(self.communication, HivemindBackend)
        self.train_timeout = 60 * 60 * 24 * 31  # 1 month

        # Logging Setup
        self.peer_id = self.communication.get_id()
        self.state.peer_id = self.peer_id
        self.animal_name = get_name_from_peer_id(self.peer_id, True)

        # Register peer_id and get current round from the chain
        self.coordinator = coordinator
        self.coordinator.register_peer(self.peer_id)
        round, _ = self.coordinator.get_round_and_stage()
        self.state.round = round

        self.communication.step_ = (
            self.state.round
        )  # initialize communication module to contract's round

        # enable push to HF if token was provided
        self.hf_token = hf_token
        if self.hf_token not in [None, "None"]:
            self._configure_hf_hub(hf_push_frequency)

        get_logger().info(
            f"🐱 Hello 🐈 [{get_name_from_peer_id(self.peer_id)}] 🦮 [{self.peer_id}]!"
        )
        get_logger().info(f"bootnodes: {kwargs.get('bootnodes', [])}")
        get_logger().info(f"Using Model: {self.trainer.model.config.name_or_path}")

        with open(os.path.join(log_dir, f"system_info.txt"), "w") as f:
            f.write(get_system_info())

        self.batched_signals = 0.0
        self.time_since_submit = time.time()  # seconds
        self.submit_period = 3.0  # hours
        self.submitted_this_round = False

        # PRG Game
        self.prg_module = PRGModule(log_dir, **kwargs)
        self.prg_game = self.prg_module.prg_game
        
        # Store bootnodes for reconnection
        self.bootnodes = kwargs.get('bootnodes', [])

    def _get_total_rewards_by_agent(self):
        rewards_by_agent = defaultdict(int)
        for stage in range(self.state.stage):
            rewards = self.rewards[stage]
            for agent_id, agent_rewards in rewards.items():
                for batch_id, batch_rewards in agent_rewards.items():
                    tot = 0
                    for generation_rewards in batch_rewards:
                        tot += sum(generation_rewards)
                    rewards_by_agent[agent_id] += tot

        return rewards_by_agent

    def _get_my_rewards(self, signal_by_agent):
        if len(signal_by_agent) == 0:
            return 0
        if self.peer_id in signal_by_agent:
            my_signal = signal_by_agent[self.peer_id]
        else:
            my_signal = 0
        my_signal = (my_signal + 1) * (my_signal > 0) + my_signal * (my_signal <= 0)
        return my_signal

    def _try_submit_to_chain(self, signal_by_agent):
        elapsed_time_hours = (time.time() - self.time_since_submit) / 3600
        if elapsed_time_hours > self.submit_period:
            try:
                self.coordinator.submit_reward(
                    self.state.round, 0, int(self.batched_signals), self.peer_id
                )
                self.batched_signals = 0.0
                if len(signal_by_agent) > 0:
                    max_agent, max_signal = max(
                        signal_by_agent.items(), key=lambda x: x[1]
                    )
                else:  # if we have no signal_by_agents, just submit ourselves.
                    max_agent = self.peer_id

                self.coordinator.submit_winners(
                    self.state.round, [max_agent], self.peer_id
                )
                self.time_since_submit = time.time()
                self.submitted_this_round = True
            except Exception as e:
                get_logger().debug(str(e))

    def _hook_after_rewards_updated(self):
        try:
            signal_by_agent = self._get_total_rewards_by_agent()
            self.batched_signals += self._get_my_rewards(signal_by_agent)
        except Exception as e:
            # If signal_by_agent is empty, we just submit ourself as winner according to logic in _try_submit_to_chain
            get_logger().debug(f"Error getting total rewards by agent: {e}")
            signal_by_agent = {}
        self._try_submit_to_chain(signal_by_agent)

    def _hook_after_round_advanced(self):
        try:
            if self.prg_game:
                prg_history_dict = self.prg_module.prg_history_dict
                results_dict = self.trainer.play_prg_game_logits(prg_history_dict)
                self.prg_module.play_prg_game(results_dict, self.peer_id)
        except Exception as e:
            get_logger().info(f"Error playing PRG game, continuing with the next round")

        self._save_to_hf()

        # Try to submit to chain again if necessary, but don't update our signal twice
        if not self.submitted_this_round:
            try:
                signal_by_agent = self._get_total_rewards_by_agent()
            except Exception as e:
                get_logger().debug(f"Error getting total rewards by agent: {e}")
                signal_by_agent = {}
            self._try_submit_to_chain(signal_by_agent)

        # Reset flag for next round
        self.submitted_this_round = False

        # Block until swarm round advances
        self.agent_block()

    def _hook_after_game(self):
        self._save_to_hf()

    def _configure_hf_hub(self, hf_push_frequency):
        username = whoami(token=self.hf_token)["name"]
        model_name = self.trainer.model.config.name_or_path.split("/")[-1]
        model_name += "-Gensyn-Swarm"
        model_name += f"-{self.animal_name}"
        self.trainer.args.hub_model_id = f"{username}/{model_name}"
        self.hf_push_frequency = hf_push_frequency
        get_logger().info("Logging into Hugging Face Hub...")
        login(self.hf_token)

    def _save_to_hf(self):
        if (
            self.hf_token not in [None, "None"]
            and self.state.round % self.hf_push_frequency == 0
        ):
            get_logger().info(f"pushing model to huggingface")
            try:
                repo_id = self.trainer.args.hub_model_id

                self.trainer.model.push_to_hub(
                    repo_id=repo_id,
                    token=self.hf_token,
                    commit_message=f"rl-swarm: round {self.state.round}, agent {self.animal_name}",
                    tags=[
                        "rl-swarm",
                        "genrl-swarm",
                        "grpo",
                        "gensyn",
                        f"I am {self.animal_name}",
                    ],
                )
            except Exception:
                get_logger().exception(
                    "Failed to push model to the Hugging Face Hub. When you conclude training please try manually pushing it yourself using the instructions here: https://huggingface.co/docs/hub/en/models-uploading",
                    stack_info=True,
                )

    def find_existing_p2pd(self):
        """Try to find existing p2pd ports"""
        try:
            result = subprocess.run(['ss', '-tlpn'], capture_output=True, text=True)
            output = result.stdout
            
            # Look for p2pd in the output
            if 'p2pd' in output:
                # Extract the ports
                tcp_match = re.search(r'.*:(\d+).*p2pd.*tcp', output)
                udp_match = re.search(r'.*:(\d+).*p2pd.*udp', output)
                
                if tcp_match and udp_match:
                    return [
                        f"/ip4/0.0.0.0/tcp/{tcp_match.group(1)}",
                        f"/ip4/0.0.0.0/udp/{udp_match.group(1)}/quic"
                    ]
            return None
        except:
            return None

    def agent_block(
        self, check_interval=5.0, log_timeout=10.0, max_check_interval=60.0 * 15
    ):
        start_time = time.monotonic()
        fetch_log_time = start_time
        check_backoff = check_interval
        reconnect_attempts = 0
        max_reconnect_attempts = 3
        
        # Store initial configuration
        initial_peers = self.communication.dht.initial_peers
        
        while time.monotonic() - start_time < self.train_timeout:
            curr_time = time.monotonic()
            try:
                _ = self.communication.dht.get_visible_maddrs(latest=True)
                reconnect_attempts = 0
            except Exception as e:
                get_logger().warning(f"P2PD connection lost at {time.strftime('%Y-%m-%d %H:%M:%S')}")
                get_logger().warning(f"Error details: {str(e)}")
                
                if reconnect_attempts < max_reconnect_attempts:
                    try:
                        # Check for existing p2pd first
                        existing_maddrs = self.find_existing_p2pd()
                        
                        if existing_maddrs:
                            get_logger().info(f"Found existing p2pd ports, attempting to connect...")
                            host_maddrs = existing_maddrs
                            client_mode = True
                            start = False
                        else:
                            get_logger().info(f"No existing p2pd found, creating new instance...")
                            host_maddrs = ["/ip4/0.0.0.0/tcp/0"]
                            client_mode = False
                            start = True
                        
                        # Try to reconnect
                        get_logger().info(f"Reconnection attempt {reconnect_attempts + 1}/{max_reconnect_attempts}")
                        new_dht = DHT(
                            start=start,
                            host_maddrs=host_maddrs,
                            initial_peers=initial_peers + self.bootnodes,
                            client_mode=client_mode,
                            use_ipfs=False
                        )
                        
                        time.sleep(5)
                        self.communication.dht = new_dht
                        get_logger().info("Successfully created new DHT connection")
                    except Exception as reinit_error:
                        get_logger().warning(f"Connection attempt {reconnect_attempts + 1} failed: {reinit_error}")
                        reconnect_attempts += 1
                        if reconnect_attempts < max_reconnect_attempts:
                            get_logger().info(f"Retrying in {check_interval} seconds...")
                            time.sleep(check_interval)
                            continue
                
                if reconnect_attempts >= max_reconnect_attempts:
                    get_logger().warning("Max reconnection attempts reached, continuing without DHT...")
                    self.state.round += 1
                    return

            # Retrieve current round and stage
            try:
                round_num, stage = self.coordinator.get_round_and_stage()
            except Exception as e:
                if curr_time - fetch_log_time > log_timeout:
                    get_logger().debug(
                        f"Could not fetch round and stage: {e}. Next check in {check_interval}s."
                    )
                    fetch_log_time = curr_time
                time.sleep(check_interval)
                continue

            if round_num >= self.state.round:
                get_logger().info(f"🐝 Joining round: {round_num}")
                check_backoff = check_interval
                self.state.round = round_num
                return
            else:
                get_logger().info(
                    f"Already finished round: {round_num}. Next check in {check_backoff}s."
                )
                time.sleep(check_backoff)
                check_backoff = min(check_backoff * 2, max_check_interval)

            if round_num == self.max_round - 1:
                return

        get_logger().info("Training timed out!")

```

Then Ctrl + O then Enter then Ctrl + x

then start your node

```bash
bash run_rl_swarm.sh
```

That's All

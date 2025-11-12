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

If you are inside screen detach your screen first by pressing `ctrl a + d` then

```bash
pkill screen
```

Create new screen session

```bash
screen -S gensyn
```

If not inside screen you can skip this step

### update to code zero

```bash
cd && cd-rl-swarm
```

then update your node

```bash
rm -rf .venv && git stash && git pull && python3 -m venv .venv && source .venv/bin/activate && bash run_rl_swarm.sh
```

For model use default model

That's All

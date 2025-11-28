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
apt update && apt upgrade -y
apt install npm screen curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
apt install python3 python3-pip python3-venv python3-dev -y
```

```sh
apt update
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
node -v
```

```sh
npm install -g yarn
yarn -v
```

#### 2. Clone this repo

```sh
git clone https://github.com/gensyn-ai/rl-swarm && cd rl-swarm
```

#### 2. Running Swarm Inside Screen

Create screen session

```sh
screen -S gensyn
```

#### Run Swarm node

```sh
python3 -m venv .venv
. .venv/bin/activate
bash run_rl_swarm.sh
```

### Login

<p align="left">
  <img src="https://drive.google.com/uc?export=view&id=1MH_CGdYOXEovDo6quBUIw_z53TwxKRTS" width="600" alt="Waiting Login">
</p>
After your terminal showing like this you need to detach screen by pressing `CTRL A + D`

and you need to tunneling http://localhost:3000/ by enter this command
```sh
npm install -g localtunnel
curl https://loca.lt/mytunnelpassword
lt --port 3000
```
you will get url to login and your ip ass tunnel password

After success login attach swarm screen

```sh
screen -r gensyn
```
Wait untill you see

<p align="left">
  <img src="https://drive.google.com/uc?export=view&id=13Ah-NvXGgjFvo_YwGPA-RDnK-up8_rcl" width="600" alt="N">
</p>

During setup, you'll be asked if you'd like to participate in the **1. Hugging Face** **2. Model** **. AI Prediction Market**.

1. N then ENTER
2. Choose your Model,   If you want to use default mode press ENTER
   Models:
   - Qwen/Qwen2.5-Coder-0.5B-Instruct
   - Qwen/Qwen2.5-Coder-1.5B-Instruct

   If you want to use default mode press ENTER

After this setup you can see your node - [Gensyn Testnet Dashboard](https://dashboard.gensyn.ai/).

### Back up Node

If you are Termius user you can back up swarm.pem with sftp

<p align="left">
  <img src="https://drive.google.com/uc?export=view&id=1pn6OXJ0c-0GbaDYTMOKfC1X_bZV2-4M8" width="600" height="400" alt="Gambar dari Google Drive">
</p>

### Key Commands

1. Attach screen to check node logs
```sh
screen -d -r gensysn
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

If you are inside screen detach your screen first by pressing `ctrl a + d`

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

# Gswarm Monitor Bot

Make sure to install all dependesies, and create your own tg bot with @botfather and your eoa address.

What is token bot?

- after you create your own bot you'll see your bot token

What is chat id?

- you can get chat id @userinfobot on telegram

what is eoa?

- you can find your eoa : https://dashboard.gensyn.ai/?application=RLSwarm and login using email method, and then you will see green 0xaddress

#### Gswarm installation

```bash
screen -S gswarm
```

Copy this command box one by one

```bash
curl -L go.dev/dl/go1.22.4.linux-amd64.tar.gz | tar -xzf - -C /usr/local
```

```bash
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bash_profile
```

```bash
source ~/.bash_profile
```

```bash
go version
```

```bash
go install github.com/Deep-Commit/gswarm/cmd/gswarm@latest
```

```bash
sed -i 's/0xFaD7C5e93f28257429569B854151A1B8DCD404c2/0x7745a8FE4b8D2D2c3BB103F8dCae822746F35Da0/g' $(which gswarm)
```

```bash
gswarm
```

and enter your :
- Bot token
- Chat ID
- EOA address

That's All

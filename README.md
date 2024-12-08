# Whisper Video Subtitle Generator

A Python tool for automatically generating and adding subtitles to videos using OpenAI's Whisper model. This tool is particularly optimized for Japanese content with optional English translation capabilities.

## Features

- 🎯 Speech recognition using OpenAI's Whisper model
- 🔤 Support for both Japanese and English subtitles
- 🔄 Translation capabilities from Japanese to English
- 📝 SRT file generation
- 🎬 Subtitle embedding into videos
- 🎨 Customizable subtitle styling
- 💪 Efficient processing of long videos
- 🚀 GPU acceleration support

## Requirements

- Docker
- NVIDIA GPU (recommended)
- CUDA support
- Ubuntu/WSL2 environment

## Installation

### 1. Docker Setup

```bash
# Remove old Docker versions
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do 
    sudo apt remove $pkg
done

# Install Docker
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker packages
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
wsl --shutdown

# Making Docker Commands Usable Without Sudo
sudo groupadd docker
sudo usermod -aG docker rui
wsl --shutdown

# Setting Docker to Start Automatically
sudo visudo
# Add the following to the end of the file:
docker ALL=(ALL)  NOPASSWD: /usr/sbin/service docker start
sudo nano ~/.bashrc
# Add the following to the end of the file:
if [[ $(service docker status | awk '{print $4}') = "not" ]]; then
sudo service docker start > /dev/null
fi
source ~/.bashrc
```

### 2. NVIDIA Docker Setup

```bash
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### 3. Create Docker Container

```bash
docker pull nvcr.io/nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04
docker run -it --gpus all nvcr.io/nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04
```

### 4. Environment Setup

```bash
# Install required packages
apt update && apt full-upgrade -y
apt install git wget nano ffmpeg -y

# Install Miniconda
cd ~ && mkdir tmp && cd tmp
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh

# Start the docker container
cd ..
rm -rf tmp

exit

docker container ls -a

docker start <container id>

docker exec -it <container id> /bin/bash
```

### 5. Create Conda Environment

```yaml
# subtitle-generator.yml
name: subtitle-generator
channels:
  - conda-forge
  - pytorch
  - nvidia
  - defaults
dependencies:
  - python=3.9
  - pip
  - cudatoolkit=11.8
  - tiktoken
  - pillow
  - tqdm
  - srt
  - moviepy
  - pip:
    - openai-whisper
    - torch
    - torchvision
    - torchaudio
```

```bash
mkdir subtitle_generator
cd subtitle_generator
nano subtitle-generator.yml

conda env create -f subtitle-generator.yml
conda activate subtitle-generator
```

## Usage

### Generate Subtitles

```bash
# Generate Japanese subtitles
python subtitle-generator.py generate --input video.mp4 --output_srt japanese.srt --model large-v3

# Generate English subtitles (translated from Japanese)
python subtitle-generator.py generate --input video.mp4 --output_srt english.srt --model large-v3 --translate
```

### Add Subtitles to Video

```bash
# Add Japanese subtitles
python subtitle-generator.py add --input video.mp4 --output_video video_jp_sub.mp4 --input_srt japanese.srt

# Add English subtitles
python subtitle-generator.py add --input video.mp4 --output_video video_en_sub.mp4 --input_srt english.srt
```

### Available Whisper Models

- `tiny`: Smallest and fastest (39M parameters)
- `base`: Better accuracy than tiny (74M parameters)
- `small`: Good balance of speed/accuracy (244M parameters)
- `medium`: Higher accuracy (769M parameters)
- `large`: Most accurate (1550M parameters)
- `large-v3`: Latest version with improvements

English-only models (faster for English content):
- `tiny.en`
- `base.en`
- `small.en`
- `medium.en`

## File Management

### Copy Files to Container

```bash
docker cp "/path/to/video.mp4" container_name:/root/subtitle_generator/
```

### Copy Files from Container

```bash
docker cp container_name:/root/subtitle_generator/output.mp4 "/path/to/destination/"
```

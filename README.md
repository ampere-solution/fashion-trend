# Fashion Trend Analysis- Multimodal
## User Interface
This is a real time fashion analytics platform - YOLOv11 detects people in video, a VLM (Qwen 2.5-VL) classifies each person’s gender/age/hair/clothing, and K-means extracts clothing colors.  A Llama-32.-3B LLM answers fashion questions over the live tracking data via RAG.
The Flask/SocketIO UI shows the video feed with gender-colored labels, live person analysis, breakdown charts, and a chat assistant.  Optimized for Arm64 Ampere processor, auto-loads models and auto-starts a demo video.

<img width="2091" height="1094" alt="image-20260504-170533.png" src="image-20260504-170533.png" />

## Architectural Diagram
This demo runs a multi-threaded Flask/SocketIO pipeline where a video reader feed YOLOv11 via an asynchronous queue, with separate workers for VLM inference.  The VLM layer spawns 5 parallel llama-server subprocesses, each pinned to 16 CPU cores via task set, classifying gender/age/hair/clothing through  layer of pipeline. K-Means extracts clothing colors.  All feed to RAG context. 
<img width="2091" height="1094" alt="image-20260504-173944.png" src="image-20260504-173944.png" />

## What This Demo Shows
This demo shows real-time fashion analytics on video: as people walk through the frame, YOLOv11 detects and tracks them while a VLM classifies each person’s gender, age group, hair length, clothing styles, and clothing colors - displayed as live labels over the video, and aggregated into breakdown charts.
Alongside, a Fashion AI Assistant chat panel lets you ask natural-language questions about what’s been seen, powered by a local Llama-3.2-3B LLM with RAG context built from the live tracking data.  The demo has demo mode auto-cycles 15 shuffled fashion questions, and a system monitor shows CPU/memory and per-model status (YOLO/LLM/VLM) - all running on a single Arm64 Ampere server with 5 parallel VLM instances.

## Target Audience
- Retail and Fashion Industry
   - Store operations managers - analyze foot traffic demographics to optimize staffing, product placement
   - Visual merchandiser - track which clothing colors/styles customers actually wear vs. what’s displayed 
   - Fashion buyers and merchandise planners- validate trend assumptions with real world data on what demographics wear 
   - Marketing teams - measure campaign impact by tracking demographics shifts in store visitors after promotions.
- Shopping Mall and Commercial Estate
   - Mall operators - understand visitor demographics per zone for leasing decision.
- Smart City and Public Sector
   - Event organizers- crowd composition at festivals, concerts, exhibitions for safety planning 
   - Tourism - visitor profile analysis at landmarks and attractions
- AI/ML Infrastructure Buyers 
   - Edge AI integration- system integrators building on-premise vision analytics
   - Enterprise architect - evaluate local LLM + VLM deployment for vertical applications on Ampere processors.
- Security and Loss Prevention
   - Retail security teams - descriptive suspect search
   - Asset tracking - backpack/handbag/suitcase detection for lost-item recovery or unattended bag alerts
- Developers and Technical Evaluators
   - Computer vision engineering- reference architecture for multi-model pipelines (YOLO + VLM + LLM)
   - Solution architects - demo to show clients what’s achievable with open source models on Arm64 processors.

## Key Message - What are we trying to convince of?
**Core message**

A single Arm64 CPU server runs production grade, multi models, real time AI vision, LLM workloads locally, observably with out a GPU.
- Ampere processors are the right CPUs platforms for AI inference
   - This demo exists to prove Ampere can host the workloads people assume require GPUs.
   - The whole stack is tuned for Ampere.
- You don’t need GPUs for production AI vision workloads
   - A single Arm64 CPU server run YOLOv11, multiple VLM instances, and LLM simultaneously in real time
   - Q816 quantization delivers GPU class inference economics on Ampere hardware.
   - Rethink your AI infrastructure budget - CPU inference is viable today
- Multi-modal AI pipelines are practical, not just research demos
   - 3 different model families (detector + VLM + LLM) coordinate in one application with sub-second latency
   - Proves that the “compound AI system” pattern works 
   - Stop thinking ofAI as one model per use case - orchestrate multiple specialized models.
- Local/on-premise AI is fully capable 
   - Everything runs in docker container
   - Local inference is no longer a compromise vs. cloud APIs.
- Vision language models unlock structured insight from raw video
   - VLM extracts descriptive attributes that classical CV can’t do
   - Show how to make VLM outputs reliable in production.
- RAG works on live operational data, not just documents
   - LLM answers fashion questions grounded in real-time tracking state, not a static knowledge base.
   - Show how to engineer context windows for live data.
- Open source models are enterprise ready
   - Llama-3.2-3B, Qwen2.5-VL-3B, and YOLOv11 are all open source and can run locally
   - Competitive with proprietary APIs for vertical use cases at a fraction of the cost
   - Open models + smart engineering  > closed API dependency.
- Real time fashion/retail analytics is a solved problem now
   - Demographics, clothing trends, and natural language store intelligence - all from existing camera feeds
   - No specialized hardware, no model training.
   - Retail decision makers can deploy this class of analytics now.


## Proof Points - How does It Show This?
- No GPUs needed for production AI vision
   - Visible proof - 3 models status cards all show green simultaneously while video plays smoothly at 23.97 fps
   - Architectural proof - docker-compose.yaml pins CPUs to docker container running 5 VLM instances, LLM, and YOLO - no Nvidia runtime, no CUDA dependency anywhere.
   - Quantization proof:  llama.cpp base image and Q8R16 GGUF models are Ampere proprietary quant that only runs on Arm64 hardware.
- Multi modal AI pipelines are practical
   - Visible proof - video stream shows YOLO box labels, VLM gender/age tags updating live, and LLM answering chat questions. All three run concurrently 
   - Latency proof - LLM card show TTFT, token/s, etc
- Local/on-premise - no cloud required 
   - Visible proof - application runs locally in docker containers 
   - Config proof - zero API key environment variables.  use in-process llama-cpp-python
- RAG works on live operational data
   - Visible proof - rag context endpoint expose the live context feed to the LLM
   - Interactive proof - RAG setting panel toggle on/off and watch LLM answers change accordingly.
- Open source models are enterprise ready
   - Visible proof - model dropdown list open source GGUFs model option exists.
   - Answer quality proof - LLM responds to fashion queries are coherent, grounded in live data, returned in seconds. 
- Real time fashion/retail analytics is a solved problem now
   - Visible proof - clothing colors, gender, age group, clothing styles breakdowns updated live as people enter and leave the frames.
   - Generality proof:  works on any input video in /app/videos
 
## Running the Demo
**Recommended Resources**
- Minimum: 96 cores. Recommended: 96+ cores
- Minimum: 64GB RAM. Recommended: 64+ GB RAM
- Minimum: 20GB disk space. Recommended: 100+ GB (multiple models, large models)

**Software Stack**
- Base image: amperecomputingai/llama.cpp:3.4.2-ampereone
- Python: 3.x
- Build tool: build-essential, cmake

Backend:
- Flask: 3.1.0
- Flask-SocketIO: 5.4.1
- simple-websocket: 1.1.0
- requests: 2.31.0
- putils: 5.9.0

AI:
- YOLOv11n: 8.3.0
- ByteTrack: 0.5.2
- OpenCV: 4.9.0
- llama-cpp-python: 0.3.16: Llama-3.2-3B-Instrucyt-Q8R16 fashion Q&A
- llama-server: Qwen2.5-VL-3B-Q8R16 person classification

FrontEnd:
- chart.js: 4.4.1
- SocketIO client: 4.7.0
- Vanilla HTM/CSS

**Demo Deployment**
- Install docker 
- Git clone from the repo: GitHub - ampere-solution/fashion-trend 
- Download the Ampere optimized model from Hugging Face.
   - Llama-3.2-3B-Instruct-Q8R16.gguf
   - qwen-2.5-vl-3b-instruct-Q8R16.gguf
   - mmproj-qwen-2.5-vl-3b-instruct-Q8_0.gguf
- Place the model inside the models/ directory.
- Place the videos inside the videos/ directory.
- docker-compose.yaml:
```yaml
services:
  app:
    cpuset: "0-95"
    image: tinguyen2024/fashion-trend-analysis:v1.1a
    #build:
    #  context: .
    #  dockerfile: Dockerfile
    ports:
      - "8000:8000"
    volumes:
      - ./videos:/app/videos:ro
      - ./models:/app/models:ro
      - tracking-data:/data
    environment:
      - DB_PATH=/data/tracking.db
      - VIDEOS_DIR=/app/videos
      - MODELS_DIR=/app/models
      - LLM_THREADS=16
      - LLM_THREADS_BATCH=16
      - LLM_CTX=100000
      - LLM_BATCH=2048
      - VLM_INSTANCES=4
      - VLM_CPUSET_0=0-15
      - VLM_THREADS_0=16
      - VLM_CPUSET_1=16-31
      - VLM_THREADS_1=16
      - VLM_CPUSET_2=32-47
      - VLM_THREADS_2=16
      - VLM_CPUSET_3=48-63
      - VLM_THREADS_3=16
      - VLM_CPUSET_4=64-79
      - VLM_THREADS_4=16
      - VLM_CTX=1024
      - VLM_INTERVAL=0
      - VLM_PORT=8012
      - AUTO_LOAD_LLM=Llama-3.2-3B-Instruct-Q8R16.gguf
      - AUTO_LOAD_VLM=qwen-2.5-vl-3b-instruct-Q8R16.gguf
      - AUTO_START_VIDEO=westfield-mall.mp4
    restart: unless-stopped

  grafana:
    image: grafana/grafana:11.4.0
    ports:
      - "4000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
      - ./grafana/dashboards:/var/lib/grafana/dashboards:ro
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
      - GF_INSTALL_PLUGINS=simpod-json-datasource
      - GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH=/var/lib/grafana/dashboards/overview.json
    depends_on:
      - app
    restart: unless-stopped

volumes:
  tracking-data:
  grafana-data:
```
- Run the setup script:  ./start_app.sh.  The script will pull the demo docker image from docker hub, setup the environments neccessary for this demo.
- Open the demo at http://< your_ip_address >:8000

**Demo Talking Points**
- This is a real time fashion analytics platform - it watches a video feed and, person by person, tells you their gender, age group, hair length, what they’re wearing and the colors - all running locally on Arm64 Ampere processors.  Three AI models are running simultaneously: YOLOv11 for detection and tracking, a vision language model (VLM) for person analysis, and a local LLM you can chat with about what it is seeing.
- Live video feed - each person gets #track_id + gender, color-coded:  pink for female, blue for male, and gray for unknown. Labels glide smoothly because we apply exponentially moving average smoothing on positions - no jitter even as people move.  Feed is capped at 23.97fps for consistent, broadcast-smooth playback.
- The AI person analysis show the 5 currently-visible people.  Each row is the VLM’s read on that person:  gender, age, hair, clothing description.
- Three architecture cards with 3-state status (not load, idle, actively inferring).  You can literally watch YOLO inference time in milliseconds, LLM TTFT, and VLM throughput tick in real time.
- Gender is hard from a single video crop, so we built a multi-layer classification pipeline.  The VLM is prompted to look at face, body shape, hair, and clothing cues.  The we ran clothing based overrides (a dress or skirt is definitive), facial-hair overrides, etc.
- To keep up in real time, we run 5 parallel VLM server instances, each pinned to its own 16 CPU cores via taskset.  Worker threads dedupe so no two instances analyze the same person. This makes per person VLM analysis feasible at video speed.
- You can ask local LLM questions about the scene - i.e: how many women in the last 5 minutes, what colors are the most common? and answered from the live RAG context built from tracking data. 

**Stop the Demo**
- Graceful stop
```bash
# stop_app.sh
$ docker compose stop
```
- Remove the demo
```bash
$ docker compose down
```


--------------------
# fashion-trend-docker
### Getting Started
1. **Download** the Ampere optimized model from [Hugging Face](https://huggingface.co).
2. - Llama-3.2-3B-Instruct-Q8R16.gguf
   - qwen-2.5-vl-3b-instruct-Q8R16.gguf
   - mmproj-qwen-2.5-vl-3b-instruct-Q8_0.gguf
3. **Place** the model inside the `models/` directory.
4. **Place** the videos inside the `videos/` directory.
5. **Run** the setup script:
   ```bash
   ./start_app.sh

**The script** will pull the demo docker image from docker hub, setup the environments neccessary for this demo.

**Open** the demo at http://< your_ip_address >:8000

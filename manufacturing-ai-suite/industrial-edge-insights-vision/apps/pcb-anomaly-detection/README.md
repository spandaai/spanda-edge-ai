
---

# Get Started

* **Time to Complete:** 30 minutes
* **Programming Language:** Python 3

---

## Prerequisites

* [System Requirements](docs/user-guide/system-requirements.md)

### Install Host Prerequisites

Before running any scripts, ensure your host system has `jq` installed.
`jq` is required by shell scripts to parse and manipulate JSON files (such as `payload.json`).

```bash
sudo apt update
sudo apt install jq -y
```

If you skip this, commands like `./sample_start.sh` will fail with:

```
jq: command not found
```

---

## Setup the Application

> The following instructions assume Docker engine is setup in the host system.

1. **Clone the repository and change into directory**

   ```sh
   git clone git@github.com:spandaai/spanda-edge-ai.git
   cd spanda-edge-ai/manufacturing-ai-suite/industrial-edge-insights-vision/
   ```

2. **Set app specific environment variable file**

   ```sh
   cp .env_pcb_anomaly_detection .env
   ```

3. **Find and set `HOST_IP` in `.env`**

   * **On Windows** (Command Prompt):

     ```cmd
     ipconfig
     ```

     Use the **Wireless LAN adapter Wi-Fi IPv4 address** — unless your PC is physically connected via Ethernet.

   * **On Linux/macOS**:

     ```bash
     hostname -I
     ```

4. **Edit the `.env` file**

   ```sh
   HOST_IP=<HOST_IP>   # IP address of server where DLStreamer Pipeline Server is running.

   MR_PSQL_PASSWORD=       # PostgreSQL service & client adapter e.g. intel1234
   MR_MINIO_ACCESS_KEY=    # MinIO service & client access key e.g. intel1234
   MR_MINIO_SECRET_KEY=    # MinIO service & client secret key e.g. intel1234

   MTX_WEBRTCICESERVERS2_0_USERNAME=<username>   # WebRTC credentials e.g. intel1234
   MTX_WEBRTCICESERVERS2_0_PASSWORD=<password>

   # Change MR_URL default from:
   # MR_URL=<PROTOCOL>://<MR_IP>:32002
   # to:
   MR_URL=http://<HOST_IP>:32002

   # Application directory
   SAMPLE_APP=pcb-anomaly-detection
   ```

5. **Install prerequisites**

   ```sh
   ./setup.sh
   ```

   This sets up application prerequisites, downloads artifacts, sets executable permissions, etc.
   Downloaded resource directories are made available to the application via volume mounting in docker compose file automatically.

---

## Deploy the Application

6. **Bring up the application**

   ```sh
   docker compose up -d
   ```

7. **Fetch list of loaded pipelines**

   ```sh
   ./sample_list.sh
   ```

   Example Output:

   ```sh
   Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
   Running sample app: pcb-anomaly-detection
   Checking status of dlstreamer-pipeline-server...
   Server reachable. HTTP Status Code: 200
   Loaded pipelines:
   [
       {
           "description": "DL Streamer Pipeline Server pipeline",
           "name": "user_defined_pipelines",
           "parameters": {
               "properties": {
                   "classification-properties": {
                       "element": {
                           "format": "element-properties",
                           "name": "classification"
                       }
                   }
               },
               "type": "object"
           },
           "type": "GStreamer",
           "version": "pcb_anomaly_detection"
       }
   ]
   ```

8. **Start the sample application**

   ```sh
   ./sample_start.sh -p pcb_anomaly_detection
   ```

   Example Output:

   ```sh
   Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
   Running sample app: pcb-anomaly-detection
   Checking status of dlstreamer-pipeline-server...
   Server reachable. HTTP Status Code: 200
   Loading payload from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/apps/pcb-anomaly-detection/payload.json
   Payload loaded successfully.
   Starting pipeline: pcb_anomaly_detection
   Launching pipeline: pcb_anomaly_detection
   Extracting payload for pipeline: pcb_anomaly_detection
   Found 1 payload(s) for pipeline: pcb_anomaly_detection
   Payload for pipeline 'pcb_anomaly_detection' {"source":{"uri":"file:///home/pipeline-server/resources/videos/anomalib_pcb_test.avi","type":"uri"},"destination":{"frame":{"type":"webrtc","peer-id":"anomaly"}},"parameters":{"classification-properties":{"model":"/home/pipeline-server/resources/models/pcb-anomaly-detection/deployment/Anomaly classification/model/model.xml","device":"CPU"}}}
   Posting payload to REST server at http://10.223.23.156:8080/pipelines/user_defined_pipelines/pcb_anomaly_detection
   Payload for pipeline 'pcb_anomaly_detection' posted successfully. Response: "f0c0b5aa5d4911f0bca7023bb629a486"
   ```

   > This starts the pipeline. View the inference stream on WebRTC:
   > `http://<HOST_IP>:8889/anomaly/`

9. **Get pipeline status**

   ```sh
   ./sample_status.sh
   ```

   Example Output:

   ```sh
   [
       {
           "avg_fps": 24.123323428597942,
           "elapsed_time": 9.865960359573364,
           "id": "f0c0b5aa5d4911f0bca7023bb629a486",
           "message": "",
           "start_time": 1752123260.5558383,
           "state": "RUNNING"
       }
   ]
   ```

10. **Stop pipeline instance**

    ```sh
    ./sample_stop.sh
    ```

    Example Output:

    ```sh
    No pipelines specified. Stopping all pipeline instances
    Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
    Running sample app: pcb-anomaly-detection
    Checking status of dlstreamer-pipeline-server...
    Server reachable. HTTP Status Code: 200
    Instance list fetched successfully. HTTP Status Code: 200
    Found 1 running pipeline instances.
    Stopping pipeline instance with ID: f0c0b5aa5d4911f0bca7023bb629a486
    Pipeline instance with ID 'f0c0b5aa5d4911f0bca7023bb629a486' stopped successfully. Response: {
        "avg_fps": 26.487679514091333,
        "elapsed_time": 25.634552478790283,
        "id": "f0c0b5aa5d4911f0bca7023bb629a486",
        "message": "",
        "start_time": 1752123260.5558383,
        "state": "RUNNING"
    }
    ```

    Stop specific instance:

    ```sh
    ./sample_stop.sh --id f0c0b5aa5d4911f0bca7023bb629a486
    ```

11. **Bring down the application**

    ```sh
    docker compose down -v
    ```

---

## Model Not Found (Fix)

If model is not found, unzip manually inside the container:

1. Enter container:

   ```bash
   docker exec -it --user root dlstreamer-pipeline-server bash
   ```

2. Install unzip:

   ```bash
   apt-get update && apt-get install -y unzip
   ```

3. Unzip model:

   ```bash
   cd /home/pipeline-server/resources/models/pcb-anomaly-detection
   unzip pcb_anomaly_detection.zip
   ```

4. Exit & restart:

   ```bash
   exit
   docker restart dlstreamer-pipeline-server
   ```

---

how to swap the video source depending on whether the user has:

* a **local file** (on their laptop/PC),
* a **USB webcam**, or
* a **remote RTSP stream** (e.g., from a Raspberry Pi camera).

Right now the default `payload.json` hardcodes the test video at:

```json
"uri": "file:///home/pipeline-server/resources/videos/anomalib_pcb_test.avi"
```

---

## Changing the Video Source after running the command **./setup.sh**

By default, the pipeline runs on the sample video (`anomalib_pcb_test.avi`).
You can change the **`payload.json`** file to use a different source:

### 1. Local video file

If your video file is on the laptop/host machine, first copy it into the container’s mounted `resources/videos` folder:

```bash
cp /path/to/your/local_video.mp4 manufacturing-ai-suite/industrial-edge-insights-vision/resources/videos/
```

Then edit `payload.json`:

```json
"source": {
    "uri": "file:///home/pipeline-server/resources/videos/local_video.mp4",
    "type": "uri"
}
```

---

### 2. USB webcam (plugged into host machine)

Find your webcam device:

```bash
ls /dev/video*
```

Usually it’s `/dev/video0`. Then set:

```json
"source": {
    "uri": "v4l2:///dev/video0",
    "type": "uri"
}
```

---

### 3. RTSP stream (e.g. Raspberry Pi camera)

If you’re streaming from a Raspberry Pi using RTSP, set the URI to your Pi’s stream address:

```json
"source": {
    "uri": "rtsp://<raspberry_pi_ip>:8554/live",
    "type": "uri"
}
```

> ⚠️ Make sure your laptop and Raspberry Pi are on the **same network**, and replace `<raspberry_pi_ip>` with the Pi’s IP address.

---

### Example complete payload.json with RTSP stream

```json
[
    {
        "pipeline": "pcb_anomaly_detection",
        "payload": {
            "source": {
                "uri": "rtsp://192.168.1.50:8554/live",
                "type": "uri"
            },
            "destination": {
                "frame": {
                    "type": "webrtc",
                    "peer-id": "anomaly"
                }
            },
            "parameters": {
                "classification-properties": {
                    "model": "/home/pipeline-server/resources/models/pcb-anomaly-detection/deployment/Anomaly classification/model/model.xml",
                    "device": "CPU"
                }
            }
        }
    }
]
```

## Further Reading

* [Helm based deployment](docs/user-guide/how-to-deploy-using-helm-charts.md)
* [MLOps using Model Registry](docs/user-guide/how-to-enable-mlops.md)
* [Run multiple AI pipelines](docs/user-guide/how-to-run-multiple-ai-pipelines.md)
* [Publish frames to S3](docs/user-guide/how-to-run-store-frames-in-s3.md)
* [View telemetry data](docs/user-guide/how-to-view-telemetry-data.md)
* [Publish metadata to OPCUA](docs/user-guide/how-to-use-opcua-publisher.md)

---

## Troubleshooting

* [Troubleshooting Guide](docs/user-guide/troubleshooting-guide.md)

---

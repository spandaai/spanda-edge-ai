# Get Started

-   **Time to Complete:** 30 minutes
-   **Programming Language:**  Python 3

## Prerequisites

- [System Requirements](docs/user-guide/system-requirements.md)

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

## Setup the application
> Note that the following instructions assume Docker engine is setup in the host system.

1. Clone the **edge-ai-suites** repository and change into industrial-edge-insights-vision directory. The directory contains the utility scripts required in the instructions that follows.
    ```sh
    git clone git@github.com:spandaai/spanda-edge-ai.git
    cd spanda-edge-ai/manufacturing-ai-suite/industrial-edge-insights-vision/
    ```
2.  Set app specific environment variable file
    ```sh
    cp .env_worker_safety_gear_detection .env

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
    ```    

4.  Edit the HOST_IP and other environment variables in `.env` file as follows
    ```sh
    HOST_IP=<HOST_IP>   # IP address of server where DLStreamer Pipeline Server is running.

    MR_PSQL_PASSWORD=  #PostgreSQL service & client adapter e.g. intel1234

    MR_MINIO_ACCESS_KEY=   # MinIO service & client access key e.g. intel1234
    MR_MINIO_SECRET_KEY=   # MinIO service & client secret key e.g. intel1234
    
    MTX_WEBRTCICESERVERS2_0_USERNAME=<username>  # WebRTC credentials e.g. intel1234
    MTX_WEBRTCICESERVERS2_0_PASSWORD=<password>

    # Change MR_URL default from:
    # MR_URL=<PROTOCOL>://<MR_IP>:32002
    # to:
    MR_URL=http://<HOST_IP>:32002

    # application directory
    SAMPLE_APP=worker-safety-gear-detection
    ```
5.  Install pre-requisites. Run with sudo if needed.
    ```sh
    ./setup.sh
    ```
    This sets up application pre-requisites, download artifacts, sets executable permissions for scripts etc. Downloaded resource directories are made available to the application via volume mounting in docker compose file automatically.

## Deploy the Application

6.  Bring up the application
    ```sh
    docker compose up -d
    ```
7.  Fetch the list of pipeline loaded available to launch
    ```sh
    ./sample_list.sh
    ```
    This lists the pipeline loaded in DL Streamer Pipeline Server.
    
    Example Output:

    ```sh
    # Example output for Worker Safety Gear Detection
    Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
    Running sample app: worker-safety-gear-detection
    Checking status of dlstreamer-pipeline-server...
    Server reachable. HTTP Status Code: 200
    Loaded pipelines:
    [
        ...
        {
            "description": "DL Streamer Pipeline Server pipeline",
            "name": "user_defined_pipelines",
            "parameters": {
            "properties": {
                "detection-properties": {
                    "element": {
                        "format": "element-properties",
                        "name": "detection"
                    }
                }
            },
            "type": "object"
            },
            "type": "GStreamer",
            "version": "worker_safety_gear_detection"
        }
        ...
    ]
    ```
8.  Start the sample application with a pipeline.
    ```sh
    ./sample_start.sh -p worker_safety_gear_detection
    ```
    This command would look for the payload for the pipeline specified in `-p` argument above, inside the `payload.json` file and launch the a pipeline instance in DLStreamer Pipeline Server. Refer to the table, to learn about different options available. 
    
    Output:

    ```sh
    # Example output for Worker Safety Gear Detection
    Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
    Running sample app: worker-safety-gear-detection
    Checking status of dlstreamer-pipeline-server...
    Server reachable. HTTP Status Code: 200
    Loading payload from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/apps/worker-safety-gear-detection/payload.json
    Payload loaded successfully.
    Starting pipeline: worker_safety_gear_detection
    Launching pipeline: worker_safety_gear_detection
    Extracting payload for pipeline: worker_safety_gear_detection
    Found 1 payload(s) for pipeline: worker_safety_gear_detection
    Payload for pipeline 'worker_safety_gear_detection' {"source":{"uri":"file:///home/pipeline-server/resources/videos/Safety_Full_Hat_and_Vest.avi","type":"uri"},"destination":{"frame":{"type":"webrtc","peer-id":"worker_safety"}},"parameters":{"detection-properties":{"model":"/home/pipeline-server/resources/models/worker-safety-gear-detection/deployment/Detection/model/model.xml","device":"CPU"}}}
    Posting payload to REST server at http://10.223.23.156:8080/pipelines/user_defined_pipelines/worker_safety_gear_detection
    Payload for pipeline 'worker_safety_gear_detection' posted successfully. Response: "784b87b45d1511f08ab0da88aa49c01e"
    ```
    NOTE: This would start the pipeline. We can view the inference stream on WebRTC by opening a browser and navigating to http://<HOST_IP>:8889/worker_safety/
    
9.  Get status of pipeline instance(s) running.
    ```sh
    ./sample_status.sh
    ```
    This command lists status of pipeline instances launched during the lifetime of sample application.
    
    Output:
    ```sh
    # Example output for Worker Safety Gear Detection
    Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
    Running sample app: worker-safety-gear-detection
    [
    {
        "avg_fps": 30.036955894826452,
        "elapsed_time": 3.096184492111206,
        "id": "784b87b45d1511f08ab0da88aa49c01e",
        "message": "",
        "start_time": 1752100724.3075056,
        "state": "RUNNING"
    }
    ]
    ```
10.  Stop pipeline instance.
    ```sh
    ./sample_stop.sh
    ```
    This command will stop all instances that are currently in `RUNNING` state and respond with the last status.
    
    Output:
    ```sh
    # Example output for Worker Safety Gear Detection
    No pipelines specified. Stopping all pipeline instances
    Environment variables loaded from [WORKDIR]/manufacturing-ai-suite/industrial-edge-insights-vision/.env
    Running sample app: worker-safety-gear-detection
    Checking status of dlstreamer-pipeline-server...
    Server reachable. HTTP Status Code: 200
    Instance list fetched successfully. HTTP Status Code: 200
    Found 1 running pipeline instances.
    Stopping pipeline instance with ID: 784b87b45d1511f08ab0da88aa49c01e
    Pipeline instance with ID '784b87b45d1511f08ab0da88aa49c01e' stopped successfully. Response: {
        "avg_fps": 29.985911953641363,
        "elapsed_time": 37.45091152191162,
        "id": "784b87b45d1511f08ab0da88aa49c01e",
        "message": "",
        "start_time": 1752100724.3075056,
        "state": "RUNNING"
    }
    ```
    If you wish to stop a specific instance, you can provide it with an `--id` argument to the command.    
    For example, `./sample_stop.sh --id 784b87b45d1511f08ab0da88aa49c01e`

11. Bring down the application
    ```sh
    docker compose down -v
    ```
    This will bring down the services in the application and remove any volumes.

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
   cd /home/pipeline-server/resources/models/worker-safety-gear-detection
   unzip worker_safety_gear_detection.zip
   ```

4. Exit & restart:

   ```bash
   exit
   docker restart dlstreamer-pipeline-server
   ```

---

---

how to swap the video source depending on whether the user has:

* a **local file** (on their laptop/PC),
* a **USB webcam**, or
* a **remote RTSP stream** (e.g., from a Raspberry Pi camera).

Right now the default `payload.json` hardcodes the test video at:

```json
"uri": "file:///home/pipeline-server/resources/videos/Safety_Full_Hat_and_Vest.avi"
```

---

## Changing the Video Source after running the command **./setup.sh**

By default, the pipeline runs on the sample video (`Safety_Full_Hat_and_Vest.avi`).
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
        "pipeline": "worker_safety_gear_detection",
        "payload": {
            "source": {
                "uri": "rtsp://192.168.1.50:8554/live",
                "type": "uri"
            },
            "destination": {
                "frame": {
                    "type": "webrtc",
                    "peer-id": "worker_safety"
                }
            },
            "parameters": {
                "classification-properties": {
                    "model": "/home/pipeline-server/resources/models/models/worker-safety-gear-detection/deployment/Detection/model/model.xml",
                    "device": "CPU"
                }
            }
        }
    }
]
```


## Further Reading
- [Helm based deployment](docs/user-guide/how-to-deploy-using-helm-charts.md)
- [MLOps using Model Registry](docs/user-guide/how-to-enable-mlops.md)
- [Run multiple AI pipelines](docs/user-guide/how-to-run-multiple-ai-pipelines.md)
- [Publish frames to S3 storage pipelines](docs/user-guide/how-to-run-store-frames-in-s3.md)
- [View telemetry data in Open Telemetry](docs/user-guide/how-to-view-telemetry-data.md)
- [Publish metadata to OPCUA](docs/user-guide/how-to-use-opcua-publisher.md)

## Troubleshooting
- [Troubleshooting Guide](docs/user-guide/troubleshooting-guide.md)

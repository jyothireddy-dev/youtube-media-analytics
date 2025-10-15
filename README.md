# my-n8n-project
Automation workflows with n8n , No-Code-Architects toolkit and MiniO
# YouTube Media Analytics Pipeline with NCA Toolkit, n8n, Docker, and MinIO

## Project Description
This project automates the analysis of YouTube videos by downloading videos from provided URLs, transcribing the audio into text, summarizing content, and generating highlight clips. It leverages the No-Code Architects Toolkit API for media processing, n8n for workflow automation, Docker for containerization, and MinIO for S3-compatible local storage.

## Features
- Batch processing of YouTube video URLs
- Video/audio download with NCA Toolkit's yt-dlp based downloader
- Audio transcription using NCA Toolkit
- Automated transcription summarization workflow via n8n
- Generation of short video clips to highlight key points
- Storage of media and outputs on local MinIO bucket
- Fully containerized with Docker for easy deployment

## Prerequisites
- Docker installed and running on your system
- Git for version control
- Basic knowledge of Docker Compose (recommended)
- MinIO running as S3-compatible storage
- Access to NCA Toolkit API with a valid API key
- n8n workflow automation platform setup

## Installation and Setup

### 1. Clone the Project Repository
https://github.com/jyothireddy-dev/YouTube-pipeline.git

### 2. Setup MinIO (Local S3 Storage)
Run MinIO using Docker:
docker run -d -p 9000:9000 -p 9001:9001 --name minio
-e MINIO_ROOT_USER=admin -e MINIO_ROOT_PASSWORD=yourpassword
-v ${PWD}/minio-data:/data quay.io/minio/minio server /data --console-address ":9001"
- Access the MinIO console at http://localhost:9001 with the above credentials.
- Create a bucket named `nca-toolkit`.
- Note your access key and secret key.

### 3. Build and Run No-Code Architects Toolkit
- Set environment variables in a `.env` file or export in your shell:
API_KEY=your_api_key_here
S3_ENDPOINT_URL=http://host.docker.internal:9000
S3_ACCESS_KEY=admin
S3_SECRET_KEY=yourpassword
S3_BUCKET_NAME=nca-toolkit
S3_REGION=None
LOCAL_STORAGE_PATH=/tmp
MAX_QUEUE_LENGTH=10
GUNICORN_WORKERS=4
GUNICORN_TIMEOUT=300

- Build the toolkit Docker image and run:
docker build -t no-code-architects-toolkit .
docker run -d -p 8080:8080 --env-file .env --name nca-toolkit no-code-architects-toolkit


### 4. Run n8n Automation Platform
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n

- Access the n8n editor at http://localhost:5678

## Usage
- Use n8n to create workflows which:
  - Take a list of YouTube URLs (input webhook or manual set)
  - Call NCA Toolkit `/v1/BETA/media/download` to download videos
  - Transcribe audio using `/v1/media/transcribe`
  - Summarize transcription text via custom function or AI integration
  - Generate short clips using `/v1/video/cut`
  - Save or upload results to MinIO bucket for easy access
- The workflow can be triggered manually or scheduled.

## Example n8n Workflow Components
- Manual Trigger or Webhook (input URLs)
- HTTP Request nodes calling NCA Toolkit API endpoints
- Function node for summarization logic
- File handling nodes for output saving
- MinIO integration for storage

## Project Structure
- `Dockerfile` for NCA Toolkit image
- `.env` sample for configuration
- `minio-data/` local directory for MinIO persistent data
- n8n workflows exported JSON (recommended to keep in repo)

## Contributing
Contributions welcome! Please submit pull requests and issues on the GitHub repository.

## License
This project is licensed under the GNU General Public License v2.0 (GPL-2.0).

---

Running and Testing the Workflow
1. Import the Workflow
Open your n8n editor at http://localhost:5678

Click the workflow menu (top right) → Import from File

Select the JSON workflow file provided

2. Configure API Key
Edit every HTTP Request node that calls NCA Toolkit

Replace "your_api_key_here" in the header with your real API key

3. Start the Workflow
Activate the workflow by toggling the "Active" switch near the top

The workflow waits for incoming webhook requests at /webhook/youtube-urls

4. Testing with Sample Data
Use a tool like curl, Postman, or your browser to send a POST request

Example curl command:

bash
curl -X POST http://localhost:5678/webhook/youtube-urls \
  -H "Content-Type: application/json" \
  -d '{"url": "https://www.youtube.com/watch?v=example_video_id"}'
The workflow runs step-by-step, downloading, transcribing, summarizing, and generating clips

5. Review Outputs
Check node execution results in n8n UI for each step’s response

Outputs from clip generation and transcripts will be as per your NCA Toolkit storage setup (e.g., MinIO or local paths)



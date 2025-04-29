# Hybrid-Cloud-Face-Identifier-System

A serverless, hybrid-cloud pipeline for detecting and matching faces in video streams using OpenFaaS functions, AWS S3, and DynamoDB (with optional Ceph).

## Overview
This project demonstrates a scalable, event-driven architecture for video-based face recognition. Incoming videos are ingested from S3 (or Ceph), processed by OpenFaaS functions to extract and compare face embeddings, and results are stored back in S3 and DynamoDB.

## Key Features
- **Serverless Processing**: Face-recognition logic encapsulated in OpenFaaS Python functions.
- **Hybrid Storage**: Ingest from AWS S3 or any S3-compatible Ceph endpoint; output CSV to S3.
- **Metadata Tracking**: Store person identifiers and timestamps in DynamoDB.
- **REST API**: Optional HTTP wrapper (Flask + Waitress) for on-demand inference.
- **Tooling**:
  - `monitorInputBucket.py`: Polls input buckets and triggers functions.
  - `generateResultsUtil.py`: Validates results against ground-truth.
  - `workload.py`: Simulates bulk video uploads for load testing.

## Prerequisites
- Python 3.9+
- Docker and OpenFaaS CLI for function deployment.
- AWS account with:
  - S3 buckets (input/output)
  - DynamoDB table
  - IAM credentials with S3 & DynamoDB permissions
- (Optional) Ceph for private S3-compatible workloads.

## Installation
```bash
git clone https://github.com/hemanth1999k/Hybrid-Cloud-Face-Identifier-System.git
cd Hybrid-Cloud-Face-Identifier-System
pip install -r requirements.txt
```

## Configuration
Copy the example environment file and edit credentials and paths:
```bash
cp .env.example .env
```
Populate `.env` with your settings:
```ini
AWS_REGION=us-west-2
ACCESS_KEY_ID=<YOUR_AWS_KEY>
SECRET_ACCESS_KEY=<YOUR_AWS_SECRET>
S3_INPUT_BUCKET=<INPUT_BUCKET_NAME>
S3_OUTPUT_BUCKET=<OUTPUT_BUCKET_NAME>
TABLE_NAME=<DYNAMODB_TABLE>
INPUT_LOCAL_STORAGE_DIR=./tmp/videos
INPUT_FRAME_STORAGE_DIR=./tmp/frames
OUTPUT_FILE_DIRECTORY=./tmp/results
# Optional Ceph settings
CEPH_ACCESSKEY_ID=<CEPH_KEY>
CEPH_SECRETKEY_ID=<CEPH_SECRET>
CEPH_ENDPOINT_URL=<CEPH_ENDPOINT_URL>
```

## Deployment
Deploy the face-recognition function:
```bash
faas-cli deploy -f face-recognition.yml
```
Deploy the HTTP wrapper (optional):
```bash
faas-cli deploy -f face-recognition-http.yml
```

## Usage
Seed DynamoDB:
```bash
python populateDynamoDbUtil.py
```
Start the bucket monitor:
```bash
python monitorInputBucket.py
```
Invoke via HTTP API:
```bash
curl -X POST http://127.0.0.1:8080/function/face-recognition-http \
     -H "Content-Type: application/json" \
     -d '{"bucket":"my-input-bucket","key":"video.mp4"}'
```
Validate results:
```bash
python generateResultsUtil.py
```
Simulate load:
```bash
python workload.py --num-videos 20
```

## Project Structure
```
.
├── face-recognition/           # OpenFaaS function (handler, utils, encodings)
├── face-recognition-http/      # HTTP wrapper function (Flask + Waitress)
├── face-recognition.yml        # OpenFaaS config
├── face-recognition-http.yml   # HTTP wrapper config
├── monitorInputBucket.py       # Polls buckets & invokes functions
├── populateDynamoDbUtil.py     # Seed DynamoDB with sample data
├── generateResultsUtil.py      # Compare outputs vs. ground truth
├── workload.py                 # Load-generation tool
├── student_data.json           # Sample metadata records
├── mapping/                    # Ground-truth mapping files
└── requirements.txt            # Python dependencies
```

## Contributing
1. Fork the repository  
2. Create a feature branch (`git checkout -b feature/YourFeature`)  
3. Commit your changes (`git commit -m "Add feature"`)  
4. Push to your branch (`git push origin feature/YourFeature`)  
5. Open a pull request  

Please follow existing code style and include tests for new functionality.

## License
This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

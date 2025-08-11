# 🎬 Stream Transcoder -  Video Transcoding System

A scalable, cloud-native video transcoding system that automatically converts uploaded videos into multiple quality levels.

## 🏗️ Architecture Overview

Stream Transcoder is built with a microservices architecture designed for scalability and reliability:

### Core Components

1. **🐳 HLS Transcoder Service** (`transcoder-container/`)

   - Dockerized FFmpeg-based transcoding service
   - Converts videos to multiple quality streams
   - Generates multiple quality levels (360p, 480p, 720p)
   - Automatically uploads processed content to S3

2. **⚡ API Service**

   - Node.js/TypeScript API server
   - SQS message consumer
   - ECS
   - Docker container orchestration

### Step-by-Step Process

1. **Video Upload**: User uploads video to S3 bucket
2. **Event Trigger**: S3 triggers notification to SQS queue
3. **Message Processing**: API service polls SQS for new messages
4. **Container Launch**: API spawns Docker container with video key
5. **Transcoding**: FFmpeg converts video to multiple HLS qualities
6. **Upload**: Processed video files uploaded back to S3
7. **Cleanup**: SQS message deleted

## 🎯 Key Features

### 🔧 Quality Levels

| Quality | Resolution |
| ------- | ---------- |
| 360p    | 480x360    |
| 480p    | 858x480    |
| 720p    | 1280×720   |

### 🚀 Scalability Features

- **Containerized Processing**: Isolated, scalable transcoding jobs
- **Queue-based Architecture**: Handle high volumes with SQS
- **Stateless Design**: Easy horizontal scaling
- **Cloud-native**: AWS S3, SQS integration, AWS ECS

## 🚀 Quick Start

### Prerequisites

- Docker
- Node.js 18+
- AWS Account (S3, SQS access, ECS)
- FFmpeg (included in Docker image)

## 📝 Configuration

### Environment Variables

**Transcoder** (`.env`):

```env

AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_aws_access_key_id
AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key


BUCKET_NAME=your-input-bucket-name
KEY=path/to/your/video/file.mp4
UPLOAD_BUCKET_NAME=your-output-bucket-name
```

**API Service** (`.env`):

```env

AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_aws_access_key_id
AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key


SQS_QUEUE_URL=https://sqs.us-east-1.amazonaws.com/123456789012/your-queue-name

ECS_TASK_DEFINITION=arn:aws:ecs:us-east-1:123456789012:task-definition/your-task-definition
ECS_CLUSTER_ARN=arn:aws:ecs:us-east-1:123456789012:cluster/your-cluster
ECS_SECURITY_GROUP=sg-1234567890abcdef0
ECS_SUBNET_1=subnet-1234567890abcdef0
ECS_SUBNET_2=subnet-1234567890abcdef1
ECS_SUBNET_3=subnet-1234567890abcdef2

BUCKET_NAME=your-input-bucket-name
KEY=path/to/your/video/file.mp4
UPLOAD_BUCKET_NAME=your-output-bucket-name
```


## System Overview

### Introduction
Momento is a cloud-native photo and video management application built on AWS infrastructure. The system enables users to upload, store, and search their media files through intelligent tagging and metadata organization. This document outlines the core architectural components and their interactions.

### Core Components

#### User's Browser

The browser serves as the client interface where users interact with the application. It handles the upload form presentation, file selection, and initial upload requests. The browser communicates with the backend server through HTTP requests and displays upload progress and results to the user. All authentication tokens are managed client-side, and the browser never directly communicates with S3 or DynamoDB.

#### Upload Server/Backend API

The upload server is the security and orchestration layer of the system. It receives all upload requests from browsers, performs authentication and authorization checks, validates file types and sizes, and handles the actual communication with AWS services. This server mediates all interactions between users and cloud storage, providing a controlled entry point where security policies are enforced. The server also coordinates tag generation and metadata storage operations.

#### S3 Bucket
AWS S3 provides the object storage layer for all media files. The bucket is organized with a folder structure that isolates user data: momento-uploads/user-{userId}/photos/ and momento-uploads/user-{userId}/videos/. Files are stored with encryption at rest, and access is controlled through IAM policies that the upload server uses. The bucket never accepts direct uploads from browsers, ensuring all files pass through validation.

#### DynamoDB

DynamoDB serves as the metadata database storing information about uploaded files and their associated tags. The schema uses userId as the partition key and uploadId as the sort key, enabling efficient queries for all files belonging to a specific user. Each record contains file metadata like filename, file type, S3 location, upload timestamp, and an array of tags. This design supports the search functionality where users can find files based on tags, dates, or other attributes.

#### IAM Roles and Policies

IAM provides the security framework that ties all components together. The upload server operates with an IAM role that has specific permissions to write to S3 and DynamoDB. These permissions are scoped to only the necessary actions: the server can write to S3 but only within the appropriate user folders, and it can write to DynamoDB but cannot delete records. This implements least-privilege access where each component has exactly the permissions it needs and no more.


#### Architecture Rationale

The architecture prioritizes security and learning value over simplicity. By routing all uploads through a server rather than using direct browser-to-S3 uploads, the system demonstrates enterprise security patterns where validation and control happen server-side. The single shared bucket with folder-based isolation follows multi-tenant SaaS patterns used in production systems. The separation between file storage in S3 and metadata storage in DynamoDB enables flexible querying without scanning all files. Each architectural choice reflects real-world cloud security considerations that would apply in consulting scenarios.





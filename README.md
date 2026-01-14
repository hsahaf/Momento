# Momento - Cloud Photo & Video Management Platform
An intelligent, searchable archive where every memory is tagged, backed up, and ready to surface at a moment's notice.

## Overview
Momento is a secure, cloud-native photo and video management application built to demonstrate enterprise-grade cloud architecture and security practices. This project showcases practical implementation of AWS security controls, serverless architecture patterns, and defense-in-depth security principles.

### Why This Project?
This project was designed to solve a real personal problem while demonstrating cloud security expertise. I needed a centralized system to organize photos across multiple devices without vendor lock-in, and I wanted intelligent search capabilities through automated tagging. Rather than using existing solutions, I am building this from scratch to showcase security-focused cloud architecture skills relevant to cloud security consulting roles.

### Architecture Highlights
The application uses a server-mediated upload architecture where files flow from the browser through a validation server before reaching S3 storage. This design prioritizes security over convenience, enabling comprehensive server-side validation, malware scanning, and content inspection before any files reach storage. All user data is isolated through folder-based organization within a shared S3 bucket, following multi-tenant security best practices used in enterprise SaaS applications.

Metadata and tags are stored in DynamoDB with a schema optimized for user-specific queries. The system uses a hybrid approach to tag generation: simple metadata like timestamps and file properties are extracted synchronously during upload, while computationally expensive AI features like object recognition run asynchronously. This architecture demonstrates understanding of when to defer work for system scalability.

### Security Implementation
Security is embedded throughout every architectural decision. IAM policies enforce least-privilege access with role-based permissions scoped to specific S3 folders. The upload server validates all file types, sizes, and content before accepting uploads. All S3 buckets use encryption at rest, and access logging is enabled through CloudTrail for audit trails. Authentication is required before any file access, and the system is designed with defense-in-depth principles where multiple security layers protect user data.

#### Technology Stack
Cloud Platform: AWS (S3, DynamoDB, IAM, CloudTrail, GuardDuty)

Backend: Node.js with AWS SDK

Frontend: HTML, JavaScript, CSS

Security: Server-side validation, encrypted storage, IAM policies, audit logging

### Key Learning Outcomes
This project demonstrates practical experience with cloud security architecture patterns, secure file upload and storage workflows, IAM policy design and least-privilege principles, NoSQL database schema design for multi-tenant applications, and asynchronous processing architectures. The work emphasizes security-first thinking and systematic evaluation of architectural trade-offs.


#### Documentation
The docs/ folder contains detailed architectural decisions, security considerations, database schema design, and API specifications. Each major decision includes rationale and trade-off analysis, demonstrating the thought process behind technical choices.
What Makes This Different
Unlike tutorial projects that focus on getting features working quickly, Momento prioritizes understanding security implications and architectural trade-offs. Every decision is documented with "why" not just "what," and the server-mediated upload flow was specifically chosen over simpler alternatives to maximize learning and demonstrate security expertise. This approach mirrors real-world cloud security consulting where understanding trade-offs and articulating security decisions is more valuable than just implementing features.


Note to Reviewers: This is an active development project. The repository structure and documentation will evolve as implementation progresses. Current focus is on establishing secure foundations before adding advanced features.

## Security Design
### Security Philosophy
Momento implements defense-in-depth security where multiple layers protect user data and system integrity. The architecture assumes that any single security control might fail and therefore employs overlapping protections. Security is not added as an afterthought but is embedded in every architectural decision from the initial design phase.

### Authentication and Authorization
#### User Authentication
All requests to the upload server require valid authentication credentials before any operations are performed. The server validates user identity at the entry point, ensuring that only authenticated users can initiate uploads or retrieve their files. Authentication tokens are verified server-side rather than trusting client-side claims, preventing users from impersonating others or accessing data they don't own.
#### Authorization Enforcement
After authentication, the server enforces authorization by ensuring users can only access their own data. When a user uploads a file, the server determines the S3 path based on the authenticated user's ID, not from any user-provided input. When retrieving files, database queries are scoped to the user's partition key. This prevents path traversal attacks where a user might try to access another user's folder by manipulating request parameters.

### Server-Side Validation
#### File Type Validation
The upload server validates file types by examining both the file extension and the actual file content rather than trusting user-provided MIME types. This prevents malicious files from being disguised as images or videos. Only whitelisted file types are accepted, following a default-deny approach where everything is rejected unless explicitly allowed.
#### File Size Limits
The server enforces maximum file size limits to prevent denial-of-service attacks where an attacker attempts to exhaust storage resources or bandwidth. Size checks happen before the server begins uploading to S3, ensuring resources aren't wasted on oversized files. These limits are configurable based on the deployment environment's capacity.
#### Malware Scanning
The upload pipeline includes malware scanning before files reach permanent storage. This protects both the system and other users from malicious content. Files that fail scanning are rejected and never written to S3. The scanning happens server-side where security controls can't be bypassed by a compromised client.

### IAM Security Model
#### Least Privilege Access
The upload server operates with an IAM role that has the minimum permissions necessary to function. The role can write to S3 but only to paths matching the pattern momento-uploads/user-*/, preventing accidental or malicious writes to other S3 locations. The role can write to DynamoDB but cannot delete records, providing an audit trail that can't be erased. The role cannot modify IAM policies or access other AWS services, limiting the blast radius if the server is compromised.
#### Folder-Based Isolation
User data isolation in S3 is enforced through IAM policies that scope permissions to specific folder prefixes. Each user's files are stored under momento-uploads/user-{userId}/, and the server's IAM policy ensures it can only write to paths where the userId matches the authenticated user. This provides logical separation within a shared bucket, following multi-tenant security patterns.
#### No Browser Access to AWS Credentials
AWS access keys never reach the browser. The server mediates all AWS interactions using its IAM role, and browsers only communicate with the server over HTTPS. This prevents credential theft, ensures validation can't be bypassed, and maintains centralized security control. When browsers need to download files, the server generates time-limited presigned URLs that expire after a short period, further limiting exposure.

### Data Protection
#### Encryption at Rest
All files in S3 are encrypted at rest using AWS-managed encryption keys. This protects data if physical storage media is compromised or improperly disposed of. DynamoDB tables also use encryption at rest, ensuring metadata receives the same protection as the files themselves.
#### Encryption in Transit
All communication between the browser and upload server uses HTTPS with TLS encryption. Communication between the server and AWS services uses AWS SDK's built-in TLS encryption. This prevents eavesdropping and man-in-the-middle attacks on data as it moves through the network.

### Audit and Monitoring
#### CloudTrail Logging
AWS CloudTrail logs all API calls made to AWS services, creating an immutable audit trail of who accessed what and when. This includes S3 object access, DynamoDB queries, and IAM role assumptions. The logs are stored in a separate secure location and can be used for forensic analysis if a security incident occurs.
#### Access Logging
S3 bucket access logging records all requests made to the bucket, including successful and failed attempts. This provides visibility into access patterns and can help identify unusual activity like repeated failed access attempts or unexpected access from unusual IP addresses.

### Security Trade-offs
#### Complexity vs. Control
The server-mediated upload architecture is more complex than direct browser-to-S3 uploads using presigned URLs, but it provides complete control over validation and security enforcement. This complexity is intentional, demonstrating that security often requires accepting architectural complexity in exchange for stronger security guarantees.

#### Performance vs. Validation
Server-side validation adds latency to the upload process compared to direct uploads, but this trade-off is acceptable because it ensures no files bypass security controls. The system prioritizes security over performance, reflecting enterprise priorities where data protection outweighs speed.

#### User Experience vs. Security
Users must authenticate before uploading files, and uploads take slightly longer due to server-side processing. These friction points are deliberate security controls that prevent anonymous uploads and ensure validation occurs. The design accepts that good security sometimes creates minor inconveniences in exchange for significantly better protection.

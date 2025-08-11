# CodePush Server

Self-hosted CodePush server for React Native over-the-air updates. Replaces Microsoft App Center CodePush with your own infrastructure.

## 🏗️ Architecture

### Components
- **API Server**: Node.js/Express server handling management and acquisition APIs
- **Storage**: AWS S3 for update packages, AWS Redis for caching and metrics
- **Authentication**: GitHub OAuth (production) or debug mode (testing)
- **CLI**: Command-line tool for developers (`code-push-standalone`)

### Data Flow
```
React Native App → CodePush CLI → CodePush Server → AWS S3
                                      ↓
                              AWS Redis Cluster
```

## 🚀 Quick Setup

### Prerequisites
- Docker and Docker Compose
- AWS S3 bucket
- AWS Redis cluster (ElastiCache)
- AWS credentials (or IAM role for EC2)
- GitHub OAuth app (for production authentication)

### 1. Clone and Configure
```bash
git clone <this-repo>
cd code-push-server

# For QA environment
cp .env.qa.example .env.qa
# Edit .env.qa with your configuration

# For Production environment  
cp .env.prod.example .env.prod
# Edit .env.prod with your configuration
```

### 2. Start Server
```bash
# QA environment
docker-compose --env-file .env.qa up --build

# Production environment
docker-compose --env-file .env.prod up --build
```

### 3. For Developers
See the complete **[Developer Guide](./DEVELOPER_GUIDE.md)** for step-by-step instructions on:
- Account registration and authentication
- Creating and managing apps
- Releasing updates
- Integration with React Native projects
- Troubleshooting common issues

## 🌍 Environment Configuration

### QA Environment
- **Purpose**: Testing and development
- **Authentication**: Debug mode enabled (no OAuth required)
- **Access**: Internal only (VPN required)
- **URL**: `http://codepush.qa.crabi.net:3000`

### Production Environment
- **Purpose**: Live app updates
- **Authentication**: GitHub OAuth required
- **Access**: Internal only (VPN required)
- **URL**: `http://codepush.prod.crabi.net:3000`

## 🔧 Key Configuration Options

| Variable | QA | Production | Description |
|----------|----|-----------| ------------|
| `NODE_ENV` | development | production | Runtime environment |
| `DEBUG_DISABLE_AUTH` | true | false | Skip OAuth for testing |
| `ENABLE_ACCOUNT_REGISTRATION` | true | false | Allow new accounts |
| `AWS_BUCKET_NAME` | crabi-qa-code-push-server | crabi-prod-code-push-server | S3 bucket |
| `REDIS_HOST` | cache.9eczd4.0001.use1.cache.amazonaws.com | cache.9eczd4.0001.use1.cache.amazonaws.com | Redis cluster |

## 🔐 Security Features

### AWS Integration
- **IAM Roles**: EC2 instances use IAM roles (no hardcoded AWS keys)
- **Private Subnets**: Server only accessible via bastion host/VPN
- **S3 Bucket Policies**: Restrict access to specific IAM roles

### Authentication
- **Production**: GitHub OAuth required for all operations
- **QA**: Debug mode available for testing (can be disabled)
- **Access Keys**: Generated per user, can be revoked

### Network Security
- **Internal Only**: No public internet access
- **VPN Required**: All access through corporate VPN
- **Bastion Host**: SSH access via jump host only

## 📚 API Compatibility

This server provides REST APIs compatible with Microsoft's CodePush service:

- **Management API**: `/apps`, `/deployments`, `/releases`
- **Acquisition API**: `/updateCheck`, `/reportStatus`  
- **Authentication**: `/auth/login`, `/auth/register`

## 🛠️ Development & Deployment

### Local Development
```bash
# Use local Redis and AWS credentials
cp .env.example .env
# Edit .env with local configuration
docker-compose --profile local up --build
```

### Production Deployment
- Deployed on AWS EC2 with Docker Compose
- Uses existing AWS ElastiCache Redis cluster
- Automatic SSL and DNS via Route53
- Infrastructure managed via AWS CDK

## 🔍 Monitoring & Troubleshooting

### Health Checks
```bash
# Check server status
curl http://YOUR_SERVER_URL/authenticated
# Expected: Authentication error (security working!)

# Check containers
docker-compose ps

# View logs
docker-compose logs api
```

### Common Issues
- **CLI Connection**: Ensure `CODE_PUSH_SERVER_URL` environment variable is set
- **Authentication**: Verify GitHub OAuth configuration
- **AWS Access**: Check IAM roles and S3 permissions
- **Network**: Ensure VPN connection for internal access

## 📖 Documentation

- **[Developer Guide](./DEVELOPER_GUIDE.md)**: Complete workflow for developers
- **[API Documentation](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/api)**: Microsoft CodePush API reference
- **Infrastructure**: See `aws-infra` repository for deployment details

## 🚨 Important Notes

### For Developers
- **Always set**: `export CODE_PUSH_SERVER_URL=http://codepush.qa.crabi.net:3000`
- **VPN Required**: Must be on corporate VPN to access server
- **Start Here**: Read the [Developer Guide](./DEVELOPER_GUIDE.md) first

### For DevOps
- **Environment Files**: Never commit `.env.qa` or `.env.prod` to git
- **AWS Credentials**: Use IAM roles, not access keys in production
- **Monitoring**: Check Docker container health and AWS resource usage

---

**Need help?** Check the [Developer Guide](./DEVELOPER_GUIDE.md) for detailed instructions and troubleshooting.
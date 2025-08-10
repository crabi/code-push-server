# CodePush Server

Self-hosted CodePush server for React Native over-the-air updates. Replaces Microsoft App Center CodePush with your own infrastructure.

## 🚀 Getting Started

### Prerequisites
- Docker and Docker Compose
- AWS S3 bucket
- AWS Redis cluster (ElastiCache)
- AWS credentials (or IAM role for EC2)
- GitHub OAuth app (for production authentication)

### 1. Clone and Setup
```bash
git clone <this-repo>
cd code-push-server
```

### 2. Configure Environment

Choose your environment template and configure:

**For QA Environment:**
```bash
cp .env.qa.example .env.qa
# No AWS keys needed - EC2 uses IAM role for S3 access
# Debug mode enabled for testing
```

**For Production Environment:**
```bash
cp .env.prod.example .env.prod
# No AWS keys needed - EC2 uses IAM role for S3 access
# Configure GitHub OAuth credentials for authentication
```

**Key Configuration:**
- **AWS S3**: Automatic access via EC2 IAM role (no keys required)
- **AWS Redis**: Uses existing Redis cluster at `cache.redis.cache.amazonaws.com`
- **Authentication**: Debug mode (QA) or GitHub OAuth (Production)

### 3. Start Server

```bash
# QA environment (uses AWS Redis cluster)
docker-compose --env-file .env.qa up --build

# Production environment (uses AWS Redis cluster)
docker-compose --env-file .env.prod up --build
```

Server will be available at your configured URL.

### 4. Environment Configuration

**QA Environment:**
- Uses `.env.qa` file (from `.env.qa.example`)
- Connects to AWS Redis cluster: `cache.redis.cache.amazonaws.com`
- Debug mode enabled for testing
- IAM role provides AWS S3 access (no hardcoded keys)

**Production Environment:**
- Uses `.env.prod` file (from `.env.prod.example`) 
- Connects to AWS Redis cluster: `cache.redis.cache.amazonaws.com`
- OAuth authentication required, no debug mode
- IAM role provides AWS S3 access (no hardcoded keys)

## 📱 Using with React Native Apps

### Install CLI
The CLI is built into this project:
```bash
# From the code-push-server/cli directory
cd cli && npm install -g .
# Or use npx directly from your React Native project
# npx code-push-standalone ...
```

### Update package.json Scripts
Replace `appcenter` commands with `code-push-standalone` and point to your server:
```json
{
  "scripts": {
    "codepush:qa": "code-push-standalone release-react ios --server YOUR_SERVER_URL --app YourApp --deploymentName Staging",
    "codepush:prod": "code-push-standalone release-react ios --server YOUR_SERVER_URL --app YourApp --deploymentName Production"
  }
}
```

Replace `YOUR_SERVER_URL` with your actual server URL.

### Update React Native App Configuration
Point your React Native app to your CodePush server URL:

**iOS (Info.plist):**
```xml
<key>CodePushServerURL</key>
<string>YOUR_SERVER_URL</string>
```

**Android (strings.xml):**
```xml
<string moduleConfig="true" name="CodePushServerUrl">YOUR_SERVER_URL</string>
```

Replace `YOUR_SERVER_URL` with your actual server URL.

### Complete Developer Workflow

1. **Register/Login:**
```bash
code-push-standalone register --server YOUR_SERVER_URL
# or
code-push-standalone login --server YOUR_SERVER_URL
```

2. **Create Apps:**
```bash
code-push-standalone app add MyApp-iOS --server YOUR_SERVER_URL
code-push-standalone app add MyApp-Android --server YOUR_SERVER_URL
```

3. **Release Updates:**
```bash
# iOS
code-push-standalone release-react MyApp-iOS ios --server YOUR_SERVER_URL

# Android
code-push-standalone release-react MyApp-Android android --server YOUR_SERVER_URL
```

4. **Manage Deployments:**
```bash
# List deployments
code-push-standalone deployment ls MyApp-iOS --server YOUR_SERVER_URL

# View history
code-push-standalone deployment history MyApp-iOS Staging --server YOUR_SERVER_URL

# Rollback if needed
code-push-standalone rollback MyApp-iOS Staging --server YOUR_SERVER_URL
```

## 🌍 Multi-Environment Setup

### Simple Environment Management
```bash
# QA Environment
cp .env.example .env.qa
# Edit .env.qa with QA AWS credentials
docker-compose --env-file .env.qa up

# Production Environment
cp .env.example .env.prod
# Edit .env.prod with production AWS credentials
NODE_ENV=production docker-compose --env-file .env.prod up

# Check status
docker-compose ps
```

### Key Environment Differences

| Setting | QA | Production |
|---------|----|-----------|
| `NODE_ENV` | development | production |
| `DEBUG_DISABLE_AUTH` | true | false |
| `AWS_BUCKET_NAME` | your-qa-bucket | your-prod-bucket |
| OAuth | Optional | Required |

## 🔧 Configuration Options

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `AWS_BUCKET_NAME` | ✅ | - | S3 bucket for storing updates |
| `AWS_ACCESS_KEY_ID` | ✅ | - | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | ✅ | - | AWS secret key |
| `AWS_REGION` | ❌ | us-east-1 | AWS region |
| `GITHUB_CLIENT_ID` | ✅* | - | GitHub OAuth client ID |
| `GITHUB_CLIENT_SECRET` | ✅* | - | GitHub OAuth client secret |
| `DEBUG_DISABLE_AUTH` | ❌ | false | Disable auth for testing |
| `DEBUG_USER_ID` | ❌ | - | User ID for debug mode |
| `LOGGING` | ❌ | true | Enable request logging |
| `ENABLE_PACKAGE_DIFFING` | ❌ | true | Enable delta updates |
| `ENABLE_ACCOUNT_REGISTRATION` | ❌ | true | Allow new account creation |

*Required for production use

### GitHub OAuth Setup

1. Go to: https://github.com/settings/developers
2. Click "New OAuth App"
3. Fill in:
   - **Application name**: `CodePush Server`
   - **Homepage URL**: `http://your-server:3000`
   - **Authorization callback URL**: `http://your-server:3000/auth/callback/github`
4. Copy Client ID and Client Secret to `.env`

## 🏗️ Architecture

### Components
- **API Server**: Node.js/Express server handling management and acquisition
- **Storage**: AWS S3 for update packages, AWS Redis for caching/metrics
- **Authentication**: GitHub/Microsoft OAuth (or debug mode for testing)
- **CLI**: Command-line tool for developers

### Architecture
```
React Native App → CodePush CLI → CodePush Server → AWS S3
                                      ↓
                              AWS Redis Cluster
```

## 🔒 Security Notes

### ⚠️ **CRITICAL: Environment Files**
- **NEVER commit environment files to git** (they may contain OAuth secrets!)
- Files like `.env.qa`, `.env.prod` are in `.gitignore`
- Only commit `.env.qa.example`, `.env.prod.example` as templates
- **AWS Security**: EC2 uses IAM roles - no AWS keys in environment files

### Other Security Best Practices
- ✅ **IAM Roles**: EC2 instances use IAM roles (no hardcoded AWS keys)
- ✅ **Private Subnet**: Server only accessible via bastion/VPN
- ✅ **S3 Bucket Policies**: Restrict access to specific IAM roles
- ✅ **OAuth Authentication**: Required for production (GitHub)
- Disable debug auth in production (`DEBUG_DISABLE_AUTH=false`)
- Rotate AWS keys regularly
- Use separate AWS accounts for QA/Prod if possible



## 🔍 Troubleshooting

### Server won't start
- Check AWS credentials are valid
- Ensure S3 bucket exists and is accessible
- Verify Docker is running
- Check logs: `docker-compose logs api`

### CLI authentication fails
- Ensure GitHub OAuth is properly configured
- Check callback URLs match exactly
- Try clearing CLI cache: `code-push-standalone logout`
- For testing, ensure `DEBUG_DISABLE_AUTH=true` is set

### Updates not deploying
- Verify app and deployment names match exactly
- Check S3 bucket permissions
- Review server logs: `docker-compose logs api`
- Ensure React Native app is configured with correct server URL

## 🧪 Testing Your Setup

### Quick Test Commands
```bash
# 1. Start server
docker-compose --env-file .env.qa up -d

# 2. Test server is running
curl YOUR_SERVER_URL/account
# Should return authentication error (this is correct!)

# 3. Check containers
docker-compose ps

# 4. Test CLI
code-push-standalone --version
code-push-standalone app list --server YOUR_SERVER_URL

# 5. Stop server
docker-compose down
```

### Expected Results
- **Server up**: `curl` returns authentication error (security working!)
- **CLI test**: Shows "not logged in" error (correct behavior)
- **Server down**: `curl` shows "Connection refused"
- **Container**: Should show `api` running (connects to AWS Redis)



## 📚 API Documentation

The server exposes REST APIs compatible with the original CodePush service:

- **Management API**: `/apps`, `/deployments`, `/releases`
- **Acquisition API**: `/updateCheck`, `/reportStatus`
- **Authentication**: `/auth/login`, `/auth/register`

For detailed API documentation, see the original [CodePush REST API docs](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/api).
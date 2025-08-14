# CodePush Server - Developer Guide

Complete step-by-step guide for developers to use the CodePush server.

## 🚀 Quick Start Workflow

### Prerequisites
- Be on **VPN** for internal server access
- Node.js installed on your machine

### Step 1: Install the CLI
```bash
# Install globally from the project
cd code-push-server/cli
npm install -g .

# Or install directly with npm (if published)
npm install -g code-push-standalone

# Verify installation
code-push-standalone --version
```

### Step 2: Set Server URL
```bash
# ⚠️ CRITICAL: The CLI needs this environment variable
export CODE_PUSH_SERVER_URL=https://codepush.qa.crabi.net
```

### Step 3: Register Your Account
```bash
code-push-standalone register
```

**If you're a new user:**
- Browser opens to GitHub OAuth registration page
- Click "GitHub" button to authenticate
- Copy the **access key** from the success page
- Paste into CLI prompt when asked

**If you're already registered:**
- Browser will show "You are already registered with the service using this authentication provider"
- Cancel the CLI registration process (Ctrl+C)
- Run login instead: `code-push-standalone login`
- Browser opens to GitHub OAuth login page
- Click "GitHub" button to authenticate  
- Copy the **access key** from the success page
- Paste into CLI prompt when asked

### Step 4: Verify Registration
```bash
code-push-standalone whoami
# Expected: your-email@crabi.com (GitHub)
```

### Step 5: Create Your Apps
```bash
# Create separate apps for iOS and Android
code-push-standalone app add MyApp-iOS
code-push-standalone app add MyApp-Android

# Verify apps were created
code-push-standalone app list
```

### Step 6: Release Updates
```bash
# iOS Release
code-push-standalone release-react MyApp-iOS ios

# Android Release  
code-push-standalone release-react MyApp-Android android
```

## 📱 Integrating with Your React Native Project

### 1. Install CLI 
```bash
# Option 1: Install from the code-push-server project
cd code-push-server/cli
npm install -g .

# Option 2: Install from npm (if published)
npm install -g code-push-standalone

# Verify installation
code-push-standalone --version
```

### 2. Update package.json Scripts
Replace your existing `appcenter` scripts:

```json
{
  "scripts": {
    "codepush:qa": "code-push-standalone release-react MyApp-iOS ios --server https://codepush.qa.crabi.net --deploymentName Staging",
    "codepush:prod": "code-push-standalone release-react MyApp-iOS ios --server https://codepush.qa.crabi.net --deploymentName Production"
  }
}
```

### 3. Update React Native App Configuration

**iOS (Info.plist):**
```xml
<key>CodePushServerURL</key>
<string>https://codepush.qa.crabi.net</string>
```

**Android (strings.xml):**
```xml
<string moduleConfig="true" name="CodePushServerUrl">https://codepush.qa.crabi.net</string>
```

## 🔧 Essential CLI Commands

### App Management
```bash
# List all your apps
code-push-standalone app list

# Create new app
code-push-standalone app add MyNewApp

# Remove app (careful!)
code-push-standalone app remove MyApp
```

### Deployment Management
```bash
# List deployments for an app
code-push-standalone deployment ls MyApp-iOS

# View deployment history
code-push-standalone deployment history MyApp-iOS Staging

# Clear deployment (remove all releases)
code-push-standalone deployment clear MyApp-iOS Staging

# Rollback to previous release
code-push-standalone rollback MyApp-iOS Staging
```

### Release Management
```bash
# Standard release
code-push-standalone release-react MyApp-iOS ios

# Release with specific deployment
code-push-standalone release-react MyApp-iOS ios --deploymentName Staging

# Release with description
code-push-standalone release-react MyApp-iOS ios --description "Bug fixes and improvements"

# Mandatory update
code-push-standalone release-react MyApp-iOS ios --mandatory

# Target specific app version
code-push-standalone release-react MyApp-iOS ios --targetBinaryVersion "1.2.0"
```

### Account Management
```bash
# Check who you're logged in as
code-push-standalone whoami

# Login (if you need to re-authenticate)
code-push-standalone login

# Logout
code-push-standalone logout
```

## 🛠️ Development Workflow

### Daily Development Process

1. **Morning Setup:**
```bash
# Set server URL (add to your .bashrc/.zshrc for persistence)
export CODE_PUSH_SERVER_URL=https://codepush.qa.crabi.net

# Verify you're logged in
code-push-standalone whoami
```

2. **Making Updates:**
```bash
# After making changes to your React Native app
npm run codepush:qa  # Push to staging first

# Test the update on staging, then push to production
npm run codepush:prod
```

3. **Checking Status:**
```bash
# See what's deployed
code-push-standalone deployment ls MyApp-iOS

# Check deployment history
code-push-standalone deployment history MyApp-iOS Production
```

### Team Workflow

**For New Team Members:**
1. Get VPN access
2. Run the Quick Start Workflow above
3. Ask team lead to add you as collaborator to existing apps

**For Existing Apps:**
```bash
# List apps you have access to
code-push-standalone app list

# If you don't see expected apps, ask team lead to add you:
# code-push-standalone collaborator add MyApp-iOS your-email@crabi.com
```

## 🚨 Troubleshooting

### Common Issues

**❌ `getaddrinfo ENOTFOUND undefined`**
```bash
# Fix: Set the server URL environment variable
export CODE_PUSH_SERVER_URL=https://codepush.qa.crabi.net
```

**❌ "You are not currently logged in"**
```bash
# Fix: Login or register
code-push-standalone login
# or
code-push-standalone register
```

**❌ "You are already registered with the service using this authentication provider"**
```bash
# This appears during registration if you already have an account
# Fix: Cancel registration (Ctrl+C) and login instead
code-push-standalone login
# Then follow the browser OAuth flow to get your access key
```

**❌ "Cannot connect to server"**
- Ensure you're on VPN
- Check server status with team
- Verify URL is correct

**❌ "App does not exist"**
- Check app name spelling: `code-push-standalone app list`
- Ensure you have access to the app
- Ask team lead to add you as collaborator

### Getting Help

**Check CLI version:**
```bash
code-push-standalone --version
```

**View command help:**
```bash
code-push-standalone --help
code-push-standalone release-react --help
```

**Debug mode (verbose output):**
```bash
code-push-standalone app list --debug
```

## 🔐 Security & Best Practices

### Do's ✅
- Always test releases on **Staging** before **Production**
- Use meaningful descriptions for releases
- Keep your access key secure
- Use mandatory updates for critical fixes
- Target specific app versions when needed

### Don'ts ❌
- Don't share your access key
- Don't push directly to Production without testing
- Don't delete apps without team confirmation
- Don't commit access keys to git

### Environment URLs
- **QA/Testing**: `https://codepush.qa.crabi.net`
- **Production**: `https://codepush.prod.crabi.net` (when available)

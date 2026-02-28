---
name: Multi-Platform Deployment
description: This skill should be used when the user asks to "deploy application", "deploy to production", "release app", "deploy to AWS", "deploy to Vercel", "deploy to Kubernetes", "iOS deployment", "Android deployment", "deploy smart contract", "web3 deployment", "deploy to multiple platforms", or needs guidance on deployment strategies across web, mobile, and blockchain platforms.
version: 1.0.0
---

# Multi-Platform Deployment

Comprehensive deployment guidance for web applications, mobile apps, and blockchain smart contracts.

## Web Application Deployment

### Static Site Deployment

**Vercel:**
```bash
# Install CLI
npm i -g vercel

# Deploy
vercel --prod

# With environment variables
vercel --prod -e DATABASE_URL=@database-url
```

**Netlify:**
```bash
# Install CLI
npm i -g netlify-cli

# Deploy
netlify deploy --prod --dir=dist
```

**AWS S3 + CloudFront:**
```bash
# Sync to S3
aws s3 sync dist/ s3://my-bucket --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id $DIST_ID \
  --paths "/*"
```

### Container Deployment

**Docker Build & Push:**
```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

```bash
# Build and push
docker build -t myapp:latest .
docker tag myapp:latest registry.example.com/myapp:latest
docker push registry.example.com/myapp:latest
```

### Kubernetes Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: registry.example.com/myapp:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Rolling update
kubectl set image deployment/myapp myapp=registry.example.com/myapp:v2

# Rollback
kubectl rollout undo deployment/myapp
```

## Mobile App Deployment

### iOS App Store

**Prerequisites:**
- Apple Developer account ($99/year)
- App Store Connect access
- Distribution certificate and provisioning profile

**Deployment Steps:**
1. Archive app in Xcode
2. Validate archive
3. Upload to App Store Connect
4. Submit for review
5. Release upon approval

**Fastlane Automation:**
```ruby
# Fastfile
default_platform(:ios)

platform :ios do
  lane :release do
    increment_build_number
    build_app(scheme: "MyApp")
    upload_to_app_store(
      skip_metadata: true,
      skip_screenshots: true
    )
  end
end
```

```bash
# Run deployment
fastlane release
```

### Google Play Store

**Prerequisites:**
- Google Play Console account ($25 one-time)
- App signing key
- Service account for API access

**Fastlane for Android:**
```ruby
# Fastfile
default_platform(:android)

platform :android do
  lane :release do
    gradle(task: "bundleRelease")
    upload_to_play_store(
      track: "production",
      aab: "app/build/outputs/bundle/release/app-release.aab"
    )
  end
end
```

### Cross-Platform (Expo/EAS)

```bash
# Configure EAS
eas build:configure

# Build for both platforms
eas build --platform all --profile production

# Submit to stores
eas submit --platform ios
eas submit --platform android
```

## Blockchain Deployment

### Ethereum Mainnet

```javascript
// deploy.js (Hardhat)
async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying with:", deployer.address);

  const Contract = await ethers.getContractFactory("MyContract");
  const contract = await Contract.deploy(constructorArg1, constructorArg2);
  await contract.waitForDeployment();

  console.log("Contract deployed to:", await contract.getAddress());

  // Verify on Etherscan
  await hre.run("verify:verify", {
    address: await contract.getAddress(),
    constructorArguments: [constructorArg1, constructorArg2],
  });
}
```

**Foundry Deployment:**
```bash
# Deploy
forge create src/MyContract.sol:MyContract \
  --rpc-url $RPC_URL \
  --private-key $PRIVATE_KEY \
  --constructor-args arg1 arg2

# Verify
forge verify-contract $ADDRESS src/MyContract.sol:MyContract \
  --chain mainnet \
  --etherscan-api-key $ETHERSCAN_KEY
```

### Multi-Chain Deployment

```javascript
// Deploy to multiple chains
const chains = {
  ethereum: { rpc: process.env.ETH_RPC, chainId: 1 },
  polygon: { rpc: process.env.POLYGON_RPC, chainId: 137 },
  arbitrum: { rpc: process.env.ARB_RPC, chainId: 42161 },
  base: { rpc: process.env.BASE_RPC, chainId: 8453 }
};

async function deployToAllChains() {
  const deployments = {};

  for (const [name, config] of Object.entries(chains)) {
    const provider = new ethers.JsonRpcProvider(config.rpc);
    const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);

    const factory = new ethers.ContractFactory(abi, bytecode, wallet);
    const contract = await factory.deploy();
    await contract.waitForDeployment();

    deployments[name] = await contract.getAddress();
    console.log(`Deployed to ${name}: ${deployments[name]}`);
  }

  return deployments;
}
```

## Deployment Strategies

### Blue-Green Deployment

```
┌─────────────┐     ┌─────────────┐
│   Router    │     │   Router    │
└──────┬──────┘     └──────┬──────┘
       │                   │
   ┌───┴───┐           ┌───┴───┐
   ▼       ▼           ▼       ▼
┌─────┐ ┌─────┐     ┌─────┐ ┌─────┐
│Blue │ │Green│  →  │Blue │ │Green│
│ ✓   │ │     │     │     │ │  ✓  │
└─────┘ └─────┘     └─────┘ └─────┘
```

### Canary Deployment

```yaml
# Kubernetes canary with Istio
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp
  http:
  - route:
    - destination:
        host: myapp
        subset: stable
      weight: 90
    - destination:
        host: myapp
        subset: canary
      weight: 10
```

### Rolling Update

```yaml
# Kubernetes rolling update
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

## Environment Management

### Environment Variables

```bash
# Development
cp .env.example .env.development

# Staging
cp .env.example .env.staging

# Production (use secrets manager)
aws secretsmanager get-secret-value --secret-id prod/myapp
```

### Feature Flags

```javascript
// LaunchDarkly example
const ldclient = LaunchDarkly.init(process.env.LD_SDK_KEY);

const showNewFeature = await ldclient.variation(
  'new-feature',
  { key: user.id },
  false
);
```

## Deployment Checklist

### Pre-Deployment

- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] Database migrations ready
- [ ] Environment variables configured
- [ ] Rollback plan documented
- [ ] Monitoring alerts configured

### Post-Deployment

- [ ] Health checks passing
- [ ] Smoke tests executed
- [ ] Performance metrics normal
- [ ] Error rates acceptable
- [ ] Notify stakeholders

## Additional Resources

### Reference Files

- **`references/cloud-providers.md`** - Provider-specific deployment guides
- **`references/deployment-scripts.md`** - Reusable deployment scripts

### Example Files

- **`examples/k8s-deployment.yaml`** - Complete Kubernetes manifests
- **`examples/github-deploy-workflow.yml`** - CI/CD deployment workflow

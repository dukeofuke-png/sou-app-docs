# AWS Deployment Guide (SOU Song Browser & Materials Server)

## Architecture Overview
- Frontend (React Song Browser): Host static build on S3 with CloudFront CDN.
- Backend (Materials Server): Deploy Express app as AWS Lambda behind API Gateway, using `serverless-http` wrapper added in `materials-server/server.js`.
- Data: Pre-generated JSON (`songs_app_export_sanitized.json`) bundled by frontend for now. Future: serve `/songs` from Lambda (cached) or store in S3.
- PDFs: Avoid exposing local absolute Google Drive paths. Either (A) proxy Google Drive via Lambda using file IDs, or (B) upload PDFs to a private S3 bucket and serve via signed URLs.

## 1. Pre-Deployment Steps
1. Sanitize paths to remove local machine details:
   ```bash
   node scripts/sanitize_paths.js
   # creates sou-song-browser/src/data/songs_app_export_sanitized.json
   ```
2. Confirm frontend now imports sanitized data (already updated in `src/App.js`).
3. (Optional) Remove absolute path fields entirely from any committed JSON.

## 2. Build Frontend
```bash
cd sou-song-browser
npm install
npm run build
```
Output: `sou-song-browser/build/`.

## 3. Create S3 Bucket
- Name: `sou-song-browser-prod` (globally unique)
- Block public access: OFF (temporarily) + add bucket policy for CloudFront later.
- Upload contents of `build/` (maintain folder structure).

## 4. Create CloudFront Distribution
- Origin: S3 bucket.
- Default Root Object: `index.html`.
- Error Responses: Map 403/404 → `/index.html` (SPA routing).
- Enable compression.
- (Later) Add custom domain + ACM cert.

## 5. Deploy Materials Server as Lambda
### Option A: Manual Zip
```bash
cd materials-server
npm ci --only=production
zip -r materials-server.zip . -x "*.git*" "node_modules/.bin/*"
```
Upload zip to new Lambda function:
- Runtime: Node.js 18+
- Handler: `server.js` (Lambda will look for `exports.handler` we added)
- Memory: 256MB (adjust later)
- Timeout: 10s

### Option B: AWS SAM (future)
Use a `template.yaml` and `sam build/deploy` for infra-as-code.

## 6. Configure API Gateway
- Create REST API → Integrate Lambda.
- Resources:
  - `/health` → Lambda proxy
  - `/pdf/{fileId}` → Lambda proxy
- Enable CORS (Allowed Origins: your CloudFront domain).
- Deploy Stage: `prod`.
- Note Invoke URL: `https://<api-id>.execute-api.<region>.amazonaws.com/prod`.

## 7. Environment Variables (Lambda)
Set in Lambda configuration:
- `NODE_ENV=production`
- `FRONTEND_URL=https://<your-cloudfront-domain>`
- (Future) `MATERIALS_PATH` if needed for drive integration.

## 8. Point Frontend to Backend
In CloudFront (or re-build frontend with env var):
Set `REACT_APP_MATERIALS_URL=https://<api-id>.execute-api.<region>.amazonaws.com/prod` prior to build OR use runtime injection via HTML `<script>` pattern (advanced).
Rebuild if variable changed.

## 9. Test Endpoints
```bash
curl https://<api-id>.execute-api.<region>.amazonaws.com/prod/health
```
Expect JSON: `{ "status": "ok", ... }`.

## 10. Handling PDFs Securely (Next)
Current data references local absolute paths. Choose a strategy:
A. **Google Drive IDs**: Store only file IDs in JSON (e.g., `songSheetDriveId`) then Lambda fetches `https://drive.google.com/uc?export=download&id=<id>`.
B. **S3 Transfer**: Bulk upload PDFs to S3 under `pdf/` and build stable keys (use `songSheetUrlToken` from sanitized JSON). Serve via CloudFront or signed S3 URLs.
Recommendation: Start with Drive IDs if you already have them; sanitize and remove any user paths.

## 11. Future Enhancements
- Add `/songs` endpoint with mtime caching (move JSON file to S3 → Lambda downloads & caches in memory).
- Add auth for admin routes using a separate Lambda (or same Express app with session disabled; prefer stateless JWT).
- Infrastructure as code: migrate manual steps to SAM or CDK.

## 12. Quick Rollback Plan
- Keep previous zip versions in S3 (versioned bucket) or store zip artifact in a release tag.
- Redeploy previous Lambda zip if new release fails.
- In CloudFront, invalidate only changed paths or full distribution if caching issues arise.

## 13. Monitoring & Logs
- Use CloudWatch Logs for Lambda function (check cold starts, errors).
- Set alarms on `5XX` API Gateway metrics.

## 14. Security Notes
- Never commit `.env` files or credentials.
- Remove absolute local filesystem paths from any public artifact.
- Consider WAF for public endpoints later.

---
Need script to convert absolute PDF paths → Google Drive IDs or S3 keys? Ask and we can add one next.

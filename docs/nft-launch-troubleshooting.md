# NFT Launch 404 Troubleshooting

Use this checklist to debug "404 Not Found" errors when launching or minting NFTs for FarLore on Base Sepolia. Work through the sections in order so you can pinpoint whether the issue is routing, metadata delivery, or contract access.

## 1) Confirm the route exists
- **Frontend routing:** Make sure the mint/launch page is registered. In frameworks like Next.js, verify `pages/mint.tsx` or an equivalent route exists and the build output is deployed.
- **API path:** If the frontend calls `/api/mint` (or similar), check the server or serverless function is deployed at that exact path. Mismatched paths (e.g., `/mint`) commonly return 404s.
- **Logs:** Tail deployment logs for routing errors. On Vercel, use `vercel logs --since 10m`. On self-hosted setups, check reverse proxy logs (Nginx, Caddy) for 404 entries.

## 2) Verify metadata availability
- **Metadata URL:** The contract’s `baseURI` should resolve. Open the full token URI (e.g., `https://<cdn>/metadata/1.json`) in a browser; a 404 here means the storage location is wrong or the file is missing.
- **Pinning/hosting:** If using IPFS, ensure the CID is pinned and accessible via a gateway. For S3/Cloudflare R2, confirm the bucket/object key matches the contract URI path and that public read permissions are enabled.
- **Content negotiation:** Double-check that the metadata JSON has `Content-Type: application/json`; some CDNs return 404 when extensions are missing.

## 3) Contract deployment sanity checks
- **Network:** Confirm the app and wallet are pointed to **Base Sepolia**. A mismatched chain ID leads to calls against the wrong RPC endpoint and 404-like errors from backend relays.
- **Address:** Ensure the frontend and backend both use the deployed contract address. Re-deployments without config updates are a frequent cause of failed mints.
- **ABI alignment:** If you updated the contract ABI, redeploy the backend build so it matches the latest interface. Otherwise, function lookups can fail upstream and bubble up as 404s.

## 4) API handler validation
- **HTTP method:** Verify the client uses the method your handler expects. For example, a POST-only `/api/mint` will 404 if called with GET in some frameworks.
- **Auth/wallet gate:** If the route enforces connected wallets or signatures, validate the guard clause before throwing a 404. Return a 401/403 instead to avoid masking authorization problems.
- **Health check:** Add a lightweight GET `/api/health` endpoint during incidents to confirm the deployment is reachable and not misconfigured at the host level.

## 5) Contract read test
Use a read-only call to confirm the contract is reachable on Base Sepolia (replace the address and RPC URL as needed):

```bash
cast call <CONTRACT_ADDRESS> "name()(string)" --rpc-url https://sepolia.base.org
```

If this fails, the issue is with the deployment or network, not the frontend route.

## 6) Post-incident hardening
- Add integration tests that hit the mint API and assert a 200 + transaction payload.
- Include a smoke test for the metadata URL in CI to detect missing objects before deploys.
- Document the exact API paths and sample payloads in the runbook so future releases don’t regress.

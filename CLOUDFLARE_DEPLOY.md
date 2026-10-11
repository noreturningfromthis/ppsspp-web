# Cloudflare Pages deployment

The validation workflow builds the pinned PPSSPP-WASM submodule and packages the Angular shell with the emulator runtime. It only saves a GitHub Actions artifact; it has no Cloudflare credentials or deployment job.

## Before creating the public site

Wait for the `ci-validation` workflow run to succeed, then download its `cloudflare-pages-site-<commit SHA>` artifact from that run. Extract the artifact so the `index.html` file is at the root of the extracted directory. The package contains the emulator runtime and its license/source notices, but never the game image.

Create a Cloudflare Pages **Direct Upload** project using Wrangler. Direct Upload is the right fit for this custom Emscripten build; Cloudflare cannot convert a Direct Upload project to a Git-integrated project later. Set `ci-validation` as the production branch so the explicit deployment command below publishes to the production URL.

In PowerShell, from the directory containing the extracted site folder:

```powershell
npx wrangler@4 login
npx wrangler@4 pages project create ctw-psp --production-branch ci-validation
npx wrangler@4 pages deploy .\cloudflare-site --project-name ctw-psp --branch ci-validation
```

Use the same project name in both commands. If `ctw-psp` is already taken in Cloudflare, choose another lowercase name and replace it in both places. Wrangler opens Cloudflare's sign-in flow; do not put an API token in this repository. After deployment, Cloudflare displays the public `https://<project-name>.pages.dev` URL.

## Hosting limits and game delivery

The package validation rejects any individual asset larger than Cloudflare Pages' 25 MiB limit and rejects recognized game-image/archive extensions. If the PPSSPP runtime's `.data` asset exceeds that limit, do not deploy this package to Pages; use a separately validated delivery plan (Cloudflare recommends R2 for larger files) and update the runtime asset URL and CORS policy first.

This deployment only hosts the website and emulator runtime. A visitor still needs game data made available through a lawful arrangement. The site cannot access a game file on the developer's PC, and this package does not include Chinatown Wars.

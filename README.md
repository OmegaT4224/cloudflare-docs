# Cloudflare Docs

**[View the docs →](https://developers.cloudflare.com/)**

## Why Cloudflare Docs is open source

Our documentation is open source so that we can stay connected with our community and quickly implement feedback. Whether you have opened an issue to provide feedback or contributed your own content, we thank you for helping us maintain quality documentation.

If you have any feedback for our documentation or are interested in contributing, please refer to our [contribution guidelines.](https://github.com/cloudflare/cloudflare-docs/blob/production/CONTRIBUTING.md)

## Setup

You must have a recent version of Node.js (22+) installed. You may use [Volta](https://github.com/volta-cli/volta), a Node version manager, to install the latest version of Node and `npm`, which is a package manager that is included with `node`'s installation.

```sh
$ curl https://get.volta.sh | bash
$ volta install node@22
```

Install the Node.js dependencies for this project using npm or another package manager:

```sh
$ npm install
```

## Development

When making changes to the site, including any content changes, you may run a local development server by running the following command:

```sh
$ npm run dev
```

This spawns a server that will be accessible via `http://localhost:1111` in your browser. Additionally, any changes made within the project – including `content/**` changes – will automatically reload your browser tab(s), allowing you to instantly preview your changes.

## Deployment

Our docs are deployed using [Cloudflare Pages](https://pages.cloudflare.com). Every commit pushed to production will automatically deploy to [developers.cloudflare.com](https://developers.cloudflare.com), and any pull requests opened will have a corresponding staging URL available in the pull request comments.

## For Cloudflare employees

To get write access to this repo, please reach out to the **Developer Docs** room in chat.

## License and Legal Notices

Except as otherwise noted, Cloudflare and any contributors grant you a license to the Cloudflare Developer Documentation and other content in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode), see the [LICENSE file](https://github.com/cloudflare/cloudflare-docs/blob/production/LICENSE), and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the [LICENSE-CODE file](https://github.com/cloudflare/cloudflare-docs/blob/production/LICENSE-CODE).

Cloudflare products and services referenced in the documentation may be either trademarks or registered trademarks of Cloudflare in the United States and/or other countries. The licenses for this project do not grant you rights to use any Cloudflare names, logos, or trademarks. Cloudflare's general trademark guidelines can be found at [https://www.cloudflare.com/trademark/](https://www.cloudflare.com/trademark/).
Cloudflare and any contributors reserve all other rights, whether under their respective copyrights, patents, or trademarks, whether by implication, estoppel, or otherwise.

Please note that we may use AI tools to help us review technical documentation, pull requests and other issues submitted to our public GitHub page in order to identify and correct mistakes and other inconsistencies in our developer documentation. Please refrain from sharing any personal information in your submissions.


## 🧞 Commands

All commands are run from the root of the project, from a terminal:

| Command                   | Action                                      |
|:--------------------------|:--------------------------------------------|
| `npm install`             | Installs dependencies                       |
| `npm run dev`             | Starts local dev server at `localhost:1111` |
| `npx astro build`         | Build your production site to `./dist/`     |
| `npm run astro -- --help` | Get help using the Astro CLI                |

## 👀 Want to learn more?

Check out [Starlight’s docs](https://starlight.astro.build/), read [the Astro documentation](https://docs.astro.build), or jump into the [Astro Discord server](https://astro.build/chat).
#!/bin/bash

# === AXIOM-AF CERT REGISTRATION SCRIPT ===
# Purpose: Fetch cert, update DNS, register with ReflectChain (no cleanup)

# === CONFIGURATION ===
USERNAME="ALC-ROOT-1010-1111-XCOV∞"
NFT_ID="10101111"
CERT_URL="https://your.r2.link/certificates/FD_AXIOM_VALIDATOR_CERT.pem"
CERT_PATH="/tmp/FD_AXIOM_VALIDATOR_CERT.pem"
CLOUDFLARE_API_TOKEN="YOUR_CLOUDFLARE_API_TOKEN"
ZONE_ID="YOUR_CLOUDFLARE_ZONE_ID"
RECORD_ID="YOUR_DNS_RECORD_ID"
REFLECTCHAIN_ENDPOINT="https://reflect.axiom.af/validator/update"
GITHUB_REPO="github.com/YOUR_USERNAME/floating-dragon-genesis"
GIT_BRANCH="main"

# === STEP 1: DOWNLOAD CERT ===
echo "[AXIOM] 📥 Downloading validator certificate from R2..."
curl -sSL "$CERT_URL" -o "$CERT_PATH" || {
  echo "[AXIOM] ❌ Failed to download cert"; exit 1;
}
echo "[AXIOM] ✅ Certificate downloaded: $CERT_PATH"

# === STEP 2: UPDATE CLOUDFLARE DNS TXT RECORD ===
CERT_CONTENT=$(base64 "$CERT_PATH" | tr -d '\n')
echo "[AXIOM] 🌐 Updating Cloudflare DNS record..."

curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data "{\"type\":\"TXT\",\"name\":\"_axiom-cert\",\"content\":\"$CERT_CONTENT\",\"ttl\":120,\"proxied\":false}" \
  | jq '.'

# === STEP 3: REGISTER INTO REFLECTCHAIN OR LOCAL CHAIN ===
echo "[AXIOM] 🔐 Registering cert on ReflectChain..."
curl -s -X POST "$REFLECTCHAIN_ENDPOINT" \
  -H "Content-Type: application/json" \
  -d "{
    \"uid\": \"$USERNAME\",
    \"nft_id\": \"$NFT_ID\",
    \"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",
    \"cert_b64\": \"$CERT_CONTENT\",
    \"cert_url\": \"$CERT_URL\"
  }" | jq '.'

# === STEP 4 (OPTIONAL): GIT PUSH TO AXIOM REPO ===
echo "[AXIOM] 🪪 Committing cert to GitHub repo..."
gh repo clone "$GITHUB_REPO"
cd floating-dragon-genesis
mkdir -p certs
cp "$CERT_PATH" ./certs/
git checkout -b cert-registry
git add ./certs/$(basename "$CERT_PATH")
git commit -m "🔐 Register cert for $USERNAME"
git push --set-upstream origin cert-registry

echo "[AXIOM] ✅ GitHub push complete"

# === FINAL ===
echo "[AXIOM] ✅ Certificate integration complete."
echo "📌 File stored at: $CERT_PATH"
echo "🌐 DNS Record: _axiom-cert TXT"
echo "🧬 ReflectChain Registered"

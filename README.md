# Cloudflare Gateway Pi-hole Scripts (CGPS)

![Cloudflare Gateway Analytics screenshot showing a thousand blocked DNS requests](.github/images/gateway_analytics.png)

Cloudflare Gateway allows you to create custom rules to filter HTTP, DNS, and network traffic based on your firewall policies. This is a collection of scripts that can be used to get a similar experience as if you were using Pi-hole, but with Cloudflare Gateway - so no servers to maintain or need to buy a Raspberry Pi!

## About the individual scripts

- `cf_list_delete.js` - Deletes all lists created by CGPS from Cloudflare Gateway. This is useful for subsequent runs.
- `cf_list_create.js` - Takes a blocklist.txt file containing domains and creates lists in Cloudflare Gateway
- `cf_gateway_rule_create.js` - Creates a Cloudflare Gateway rule to block all traffic if it matches the lists created by CGPS.
- `cf_gateway_rule_delete.js` - Deletes the Cloudflare Gateway rule created by CGPS. Useful for subsequent runs.
- `download_lists.js` - Initiates blocklist and whitelist download.

## Features

- Support for basic hosts files
- Full support for domain lists
- Automatically cleans up filter lists: removes duplicates, invalid domains, comments and more
- Works **fully unattended**
- **Allowlist support**, allowing you to prevent false positives and breakage by forcing trusted domains to always be unblocked.
- Experimental **SNI-based filtering** that works independently of DNS settings, preventing unauthorized or malicious DNS changes from bypassing the filter.
- Optional health check: Sends a ping request ensuring continuous monitoring and alerting for the workflow execution, or messages a Discord webhook with progress.

## Usage

### Prerequisites

1. Node.js installed on your machine
2. Cloudflare [Zero Trust](https://one.dash.cloudflare.com/) account - the Free plan is enough. Use the Cloudflare [documentation](https://developers.cloudflare.com/cloudflare-one/) for details.
3. Cloudflare email, API **token** with Zero Trust read and edit permissions, and account ID. See [here](https://github.com/mrrfv/cloudflare-gateway-pihole-scripts/blob/main/extended_guide.md#cloudflare_api_token) for more information about how to create the token.
4. A file containing the domains you want to block - **max 300,000 domains for the free plan** - in the working directory named `blocklist.txt`. Mullvad provides awesome [DNS blocklists](https://github.com/mullvad/dns-blocklists) that work well with this project. A script that downloads recommended blocklists, `download_lists.js`, is included.
5. Optional: You can whitelist domains by putting them in a file `allowlist.txt`. You can also use the `get_recomended_whitelist.sh` Bash script to get the recommended whitelists.
6. Optional: A Discord (or similar) webhook URL to send notifications to.

### Running locally

1. Clone this repository.
2. Run `npm install` to install dependencies.
3. Copy `.env.example` to `.env` and fill in the values.
4. If you haven't downloaded any filters yourself, run the `node download_lists.js` command to download recommended filter lists (OISD Small and AdAway; about 50 000 domains).
5. Run `node cf_list_create.js` to create the lists in Cloudflare Gateway. This will take a while.
6. Run `node cf_gateway_rule_create.js` to create the firewall rule in Cloudflare Gateway.
7. Profit! Time is money after all. You can update the lists by repeating steps 4, 5 and 6.

### Running in GitHub Actions

These scripts can be run using GitHub Actions so your filters will be automatically updated and pushed to Cloudflare Gateway. This is useful if you are using a frequently updated blocklist.

Please note that:
- GitHub Actions wasn't intended to be used for this purpose, therefore the local options are recommended.
- the GitHub Action downloads the recommended blocklists and whitelist by default. You can change this behavior by setting Actions variables.

1. Create a new empty, private repository. Forking or public repositories are discouraged, but supported - although the script never leaks your API keys and GitHub Actions secrets are automatically redacted from the logs, it's better to be safe than sorry. There is **no need to use the "Sync fork" button** if you're doing that! The GitHub Action downloads the latest code regardless of what's in your forked repository.
2. Create the following GitHub Actions secrets in your repository settings:
   - `CLOUDFLARE_API_TOKEN`: Your Cloudflare API Token with Zero Trust read and edit permissions
   - `CLOUDFLARE_ACCOUNT_ID`: Your Cloudflare account ID
   - `CLOUDFLARE_LIST_ITEM_LIMIT`: The maximum number of blocked domains allowed for your Cloudflare Zero Trust plan. Default to 300,000. Optional if you are using the free plan.
   - `PING_URL`: /Optional/ The HTTP(S) URL to ping (using curl) after the GitHub Action has successfully updated your filters. Useful for monitoring.
   - `DISCORD_WEBHOOK_URL`: /Optional/ The Discord (or similar) webhook URL to send notifications to. Good for monitoring as well.
3. Create the following GitHub Actions variables in your repository settings if you desire:
   - `ALLOWLIST_URLS`: Uses your own allowlists. One URL per line. Recommended allowlists will be used if this variable is not provided.
   - `BLOCKLIST_URLS`: Uses your own blocklists. One URL per line. Recommended blocklists will be used if this variable is not provided.
   - `BLOCK_PAGE_ENABLED`: Enable showing block page if host is blocked.
4. Create a new file in the repository named `.github/workflows/main.yml` with the contents of `auto_update_github_action.yml` found in this repository. The default settings will update your filters every week at 3 AM UTC. You can change this by editing the `schedule` property.
5. Enable GitHub Actions in your repository settings.

### DNS setup for Cloudflare Gateway

1. Go to your Cloudflare Zero Trust dashboard, and navigate to Networks -> Resolvers & Proxies -> DNS locations.
2. Click on the default location or create one if it doesn't exist.
3. Configure your router or device based on the provided DNS addresses.

Alternatively, you can install the Cloudflare WARP client and log in to Zero Trust. This method proxies your traffic over Cloudflare servers, meaning it works similarly to a commercial VPN. You need to do this if you want to use the SNI-based filtering feature, as it requires Cloudflare to inspect your raw traffic (HTTPS remains encrypted if "TLS decryption" is disabled).

### Malware blocking

The default filter lists are only optimized for ad & tracker blocking because Cloudflare Zero Trust itself comes with much more advanced security features. It's recommended that you create your own Cloudflare Gateway firewall policies that leverage those features on top of CGPS.

### Dry runs

To see if e.g. your filter lists are valid without actually changing anything in your Cloudflare account, you can set the `DRY_RUN` environment variable to 1, either in `.env` or the regular way. This will only print info such as the lists that would be created or the amount of duplicate domains to the console.

**Warning:** This currently only works for `cf_list_create.js`.

<!-- markdownlint-disable-next-line MD026 -->
## Why not...

### Pi-hole or Adguard Home?

- Complex setup to get it working outside your home
- Requires a Raspberry Pi

### NextDNS?

- DNS filtering is disabled after 300,000 queries per month on the free plan

### Cloudflare Gateway?

- Requires a valid payment card or PayPal account
- Limit of 300k domains on the free plan

### a hosts file?

- Potential performance issues, especially on [Windows](https://github.com/StevenBlack/hosts/issues/93)
- No filter updates
- Doesn't work for your mobile device
- No statistics on how many domains you've blocked

## License

MIT License. See `LICENSE` for more information.

Sourav's notes

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TheWebDexter Tech Protection Setup Guide</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; line-height: 1.6; color: #24292e; max-width: 900px; margin: 0 auto; padding: 20px; }
        h1 { border-bottom: 1px solid #eaecef; padding-bottom: 0.3em; margin-bottom: 24px; }
        h2 { margin-top: 24px; margin-bottom: 16px; font-weight: 600; line-height: 1.25; border-bottom: 1px solid #eaecef; padding-bottom: 0.3em; }
        h3 { margin-top: 24px; margin-bottom: 16px; font-weight: 600; font-size: 1.25em; }
        code { padding: 0.2em 0.4em; margin: 0; font-size: 85%; background-color: #f6f8fa; border-radius: 6px; font-family: ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, "Liberation Mono", monospace; }
        pre { background-color: #f6f8fa; padding: 16px; overflow: auto; line-height: 1.45; border-radius: 6px; }
        pre code { background-color: transparent; padding: 0; }
        .warning { background-color: #fff8c5; padding: 16px; border-left: 5px solid #e36209; margin-bottom: 16px; }
        .note { background-color: #f1f8ff; padding: 16px; border-left: 5px solid #0366d6; margin-bottom: 16px; }
        table { border-collapse: collapse; width: 100%; margin-bottom: 16px; }
        table th, table td { border: 1px solid #dfe2e5; padding: 6px 13px; }
        table tr:nth-child(2n) { background-color: #f6f8fa; }
        a { color: #0366d6; text-decoration: none; }
        a:hover { text-decoration: underline; }
    </style>
</head>
<body>

<h1>🚀 TheWebDexter Tech Protection List: Deployment Guide</h1>
<p>This guide details how to deploy a custom, high-performance ad-blocking and security filter (300,000+ domains) to Cloudflare Gateway using GitHub Actions.</p>

<div class="note">
    <strong>Goal:</strong> Automate the blocking of ads, trackers, malware, and gambling sites for corporate/startup environments using a single consolidated list named <em>"TheWebDexter Tech Protection List"</em>.
</div>

<h2>Step 1: Prerequisites</h2>
<ul>
    <li>A <strong>GitHub Account</strong>.</li>
    <li>A <strong>Cloudflare Account</strong> (Free Zero Trust plan).</li>
</ul>

<h2>Step 2: Cloudflare Configuration</h2>
<p>You need to retrieve two credentials from your Cloudflare Zero Trust Dashboard.</p>

<h3>1. Get Account ID</h3>
<ol>
    <li>Log in to the <a href="https://one.dash.cloudflare.com/" target="_blank">Cloudflare Zero Trust Dashboard</a>.</li>
    <li>Copy your <strong>Account ID</strong> from the URL bar (the string immediately after <code>dash.cloudflare.com/</code>).</li>
    <li><em>Example:</em> <code>191b417c62b998c8963b4abf52917d36</code></li>
</ol>

<h3>2. Create API Token</h3>
<ol>
    <li>Go to <a href="https://dash.cloudflare.com/profile/api-tokens" target="_blank">User API Tokens</a>.</li>
    <li>Click <strong>Create Token</strong> &rarr; <strong>Custom Token</strong>.</li>
    <li><strong>Permissions:</strong> Account &rarr; Zero Trust &rarr; Edit.</li>
    <li><strong>Account Resources:</strong> Include &rarr; <strong>All accounts</strong>.</li>
    <li>Click <strong>Create Token</strong> and copy the secret key.</li>
</ol>

<h2>Step 3: GitHub Repository Setup</h2>
<div class="warning">
    <strong>CRITICAL:</strong> You must distinguish between <strong>Secrets</strong> (hidden credentials) and <strong>Variables</strong> (configuration settings). Placing these in the wrong tab will cause the deployment to fail.
</div>

<h3>1. Add Secrets (Credentials)</h3>
<p>Go to <strong>Settings</strong> &rarr; <strong>Secrets and variables</strong> &rarr; <strong>Actions</strong> &rarr; <strong>Secrets</strong> tab.</p>
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Value</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>CLOUDFLARE_ACCOUNT_ID</code></td>
            <td>(Your Account ID from Step 2.1)</td>
        </tr>
        <tr>
            <td><code>CLOUDFLARE_API_TOKEN</code></td>
            <td>(Your API Token from Step 2.2)</td>
        </tr>
    </tbody>
</table>

<h3>2. Add Variables (Configuration)</h3>
<p>Go to <strong>Settings</strong> &rarr; <strong>Secrets and variables</strong> &rarr; <strong>Actions</strong> &rarr; <strong>Variables</strong> tab.</p>
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Value</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>CLOUDFLARE_LIST_ITEM_LIMIT</code></td>
            <td><code>300000</code></td>
            <td><strong>Required.</strong> Forces the script to create 1 single list instead of 300 chunks.</td>
        </tr>
        <tr>
            <td><code>BLOCKLIST_URLS</code></td>
            <td>(See below)</td>
            <td>List of blocklist source URLs.</td>
        </tr>
        <tr>
            <td><code>ALLOWLIST_URLS</code></td>
            <td><code>https://raw.githubusercontent.com/anudeepND/whitelist/master/domains/whitelist.txt</code></td>
            <td>Prevents breaking popular services.</td>
        </tr>
    </tbody>
</table>

<h4>Recommended Blocklists (Paste into BLOCKLIST_URLS)</h4>
<pre><code>https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/pro.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/tif.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/gambling.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/doh-vpn-proxy-bypass.txt</code></pre>

<h2>Step 4: Create the Deployment Workflow</h2>
<p>Create a file in your repository at <code>.github/workflows/update_lists.yml</code> with the following content. This script includes a special step to rename the list to <strong>"TheWebDexter Tech Protection List"</strong>.</p>

<pre><code>name: Update Filter Lists

on:
  schedule:
    - cron: '0 3 * * 1' # Runs every Monday at 3:00 AM
  workflow_dispatch:    # Allows manual triggering
  push:
    branches:
      - main
    paths:
      - '.github/workflows/update_lists.yml'

jobs:
  cgps:
    name: Update Cloudflare Gateway
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        run: npm ci

      # --- CUSTOM NAME CONFIGURATION ---
      - name: Set Custom List Name
        run: |
          grep -rl "CGPS List" . | xargs sed -i 's/CGPS List/TheWebDexter Tech Protection List/g'
          grep -rl "CGPS" . | xargs sed -i 's/CGPS/TheWebDexter/g'

      - name: Download Lists
        run: npm run download:blocklist
        env:
          BLOCKLIST_URLS: ${{ vars.BLOCKLIST_URLS }}

      - name: Download Allowlists
        run: npm run download:allowlist
        env:
          ALLOWLIST_URLS: ${{ vars.ALLOWLIST_URLS }}

      - name: Update Cloudflare
        run: npm run cloudflare-create
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          CLOUDFLARE_LIST_ITEM_LIMIT: ${{ vars.CLOUDFLARE_LIST_ITEM_LIMIT }}</code></pre>

<h2>Step 5: Run & Verify</h2>
<ol>
    <li>Go to the <strong>Actions</strong> tab in GitHub.</li>
    <li>Select <strong>Update Filter Lists</strong> on the left.</li>
    <li>Click <strong>Run workflow</strong>.</li>
    <li>Wait for the green checkmark.</li>
    <li><strong>Success Check:</strong> Log in to Cloudflare > <strong>My Team</strong> > <strong>Lists</strong>. You should see a single list named: <br>
    <code>TheWebDexter Tech Protection List - Chunk 1</code> containing ~300,000 domains.</li>
</ol>

<h2>Step 6: Connect Devices</h2>
<p>To use the protection, configure your devices or router to use your Cloudflare Gateway DNS location.</p>
<ul>
    <li><strong>Windows/Mac:</strong> Install the Cloudflare WARP Client and sign in to your Zero Trust team.</li>
    <li><strong>Routers/Browsers:</strong> Use the dedicated <strong>DNS over HTTPS (DoH)</strong> URL found in Cloudflare under <em>Gateway &rarr; DNS Locations</em>.</li>
</ul>

</body>
</html>

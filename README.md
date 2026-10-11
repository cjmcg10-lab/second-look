# Second Look

Search used listings, flip through their photos, and see how far each one is below the price of buying the same thing new.

- **worker.js** is the whole app: the screen you see and the private helper that talks to eBay.
- **wrangler.toml** tells Cloudflare how to run it.

Your keys never go in these files. They are stored in Cloudflare as secrets.

---

## Launch steps

Do Part 1 first. eBay takes about one business day to approve new accounts, and you can do Parts 2 and 3 while you wait. Until the eBay key is added, the app runs with sample listings.

### Part 1: Get a free eBay key

1. Go to **developer.ebay.com** and tap **Join**.
2. Fill in the form and submit it. eBay asks for a business email. If you have a work or custom-domain address, use it.
3. Wait for the approval email. eBay says about one business day.
4. Sign in and open **Application Keys** (developer.ebay.com/my/keys).
5. Type an application title, such as `Second Look`, and create a keyset under **Production** (not Sandbox).
6. eBay keeps a new keyset switched off until you answer a privacy question. Next to your App ID, tap **Notifications**, then choose **Marketplace Account Deletion**.
7. Slide **Not persisting eBay data** to On. Tap **Confirm**, pick the reason that fits (the app does not store eBay user data), and tap **Submit**.
8. Go back to **Application Keys**. Copy two values from the Production row and keep them somewhere private:
   - **App ID (Client ID)**
   - **Cert ID (Client Secret)**

If eBay rejects the sign-up: use the "My account registration was rejected" option on eBay's **Developer Account Support** page. This has been happening to some new sign-ups in 2026.

### Part 2: Put the files on GitHub

1. Go to **github.com**, tap **+**, then **New repository**.
2. Name it `second-look`. Choose **Private**. Tap **Create repository**.
3. Tap **uploading an existing file**.
4. Choose `worker.js`, `wrangler.toml` and `README.md`.
5. Tap **Commit changes**.

### Part 3: Put it online with Cloudflare

1. Go to **dash.cloudflare.com/sign-up** and make a free account. No card is needed.
2. Open **Workers & Pages** and tap **Create**.
3. Next to **Import a repository**, tap **Get started**.
4. Connect your GitHub account and allow access to the `second-look` repository. Select it.
5. Set the project name to exactly `second-look`. Leave **Build command** empty. Leave **Deploy command** as `npx wrangler deploy`.
6. Tap **Deploy** and wait for it to finish.
7. Open the address Cloudflare shows. It looks like `https://second-look.YOUR-NAME.workers.dev`. You should see the app with sample listings.
8. In Cloudflare, open the `second-look` Worker, then **Settings**.
9. Under **Variables and Secrets**, tap **Add**. Choose type **Secret**.
   - Variable name `EBAY_CLIENT_ID`, value: your **App ID**.
10. Tap **Add variable**. Choose type **Secret** again.
    - Variable name `EBAY_CLIENT_SECRET`, value: your **Cert ID**.
11. Tap **Deploy**.
12. Reload the app. Tap **Setup**. "Live eBay listings" should say **Connected**.

If it says **Not working**, the message under it tells you why. The usual causes are Sandbox keys instead of Production keys, or Part 1 step 7 not being finished.

### Part 4: Add it to your iPhone

1. Open the app's address in **Safari**.
2. Tap **Share**, then **Add to Home Screen**.
3. Tap **Add**. It now opens like any other app.

---

## Optional: more price sources

Add these the same way as the eBay secrets (Part 3, steps 8 to 11).

| Secret name | What it adds | Free allowance |
| --- | --- | --- |
| `SERPAPI_KEY` | A "Check Google Shopping" button on each listing | 250 checks a month, from serpapi.com |
| `BESTBUY_KEY` | Best Buy's current price when you open a listing | Free, but Best Buy approves keys by application |

Store prices by barcode need no key. They come from UPCitemdb's free plan, which allows about 100 lookups a day and is shared with other apps, so it sometimes runs out.

## How the new price is worked out

1. The app identifies the product by eBay's catalog number, then barcode, then brand and model number. If none exist, it falls back to the wording of the title.
2. It collects brand-new listings of that product, drops prices far from the middle (empty boxes, scalpers), and takes the low-typical price.
3. When you open a listing, it also looks up store prices by barcode and ignores any older than 120 days.
4. If sources agree within 15%, the match is marked as confirmed by two sources. If they disagree, you see the range.
5. The discount always uses the lowest source, so savings are never overstated.

Labels:

- **Exact match**: the product was identified by catalog number, barcode or model number.
- **Similar item**: matched on the title only. Treat it as a rough guide. The sticker is outlined instead of solid.
- **No new price**: nothing matched closely enough, so no discount is shown.

You can tap **Fix the new price** on any listing. The app remembers your price for that product on that device.

## Staying free

| Service | Free allowance | What the app uses |
| --- | --- | --- |
| eBay | 5,000 calls a day | 1 per search page, 1 to 2 per listing priced |
| Cloudflare Workers | 100,000 requests a day | A handful per search |
| GitHub | Free private repositories | Three small files |

New prices are remembered for a day to save calls. If eBay's allowance runs out, the app says so and works again the next day.

## Good to know

- Anyone who has the app's address can use it, and that uses your eBay allowance. Keep the address to yourself.
- Facebook Marketplace, Craigslist, OfferUp, Mercari and Poshmark do not offer free listing access. The app opens your search on those sites in Safari instead.
- Saved searches are checked when you open the app. There are no push notifications.
- Saved searches, your ZIP code and price fixes are stored on the device. The Home Screen app and Safari keep separate copies.

## Updating the app

Replace `worker.js` in the GitHub repository with a newer copy. Cloudflare redeploys by itself within a minute or two.

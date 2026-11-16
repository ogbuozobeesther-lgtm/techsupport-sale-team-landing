# TechSupport CRM product page (Esther's version)

A product discovery and enquiry page for TechSupport, a laptop and technology seller in Nigeria. Visitors can browse laptops, get a suggestion based on how they will use the laptop, compare specifications and send an enquiry. There is no cart and no checkout. Every enquiry goes to a Make.com scenario as a lead, so Make can pass it on to a CRM, a Google Sheet, an email alert or anything else.

This is a second design of the TechSupport product page. It does the same job as the first one (`techsupport-product-page`) and sends the same data, but it looks different and uses its own Make webhook.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole page: layout, styles, product data, webhook setting and form code |
| `README.md` | This guide |

Nothing needs installing. There is no build step. Open `index.html` in a browser and it works.

## What is on the page

1. Header with Home, Products, About Us, Contact and a Get a Quote button. On a phone it folds into a menu button.
2. Hero with the main headline and a "Not sure where to start?" picker. The visitor taps what they will mostly use the laptop for (for example Meetings and travel) and the page shows the laptop we suggest, with its specs, price and an Enquire button.
3. The lineup: one wide row per laptop with a drawing, category, description, four key specs, sample price and an **Enquire About This Laptop** button.
4. A side by side comparison table.
5. Why Choose TechSupport: four numbered reasons on a dark band.
6. About Us with a three step "how buying works" list.
7. Contact section with the enquiry form. A dark strip on top of the form always shows which laptop the enquiry is for.
8. Footer.

## Design

| Part | Choice |
|---|---|
| Background | Warm paper `#f5f1e8` with white cards `#fffdf8` |
| Text | Near black ink `#1a1712`, muted brown grey `#5c554a` |
| Accent | Burnt orange `#c8431a` for main buttons and highlights |
| Headings | Bricolage Grotesque |
| Body text | Instrument Sans |
| Labels and spec names | JetBrains Mono, small capitals |
| Signature details | Thick ink borders with a hard offset shadow on the picker and the form, product rows instead of cards |

All colours live at the top of the `<style>` block under `:root`. Change one there and it changes everywhere.

The fonts load from Google Fonts. If there is no internet, the page falls back to the system font and still works.

## How an enquiry travels

1. A visitor clicks any Enquire button. The page scrolls to the form, fills in the product and briefly highlights it.
2. They fill in their details and click **Send Enquiry**. The page checks the required fields.
3. The page sends one JSON object straight to the Make webhook with a `POST` request.
4. The success message appears only when Make replies that it accepted the data. If Make does not answer or says no, the visitor sees an error and can try again.

The webhook this page uses:

```
https://hook.eu1.make.com/n7oj8udg1kxky1bvvafn3v6u44yt2bc1
```

Make receives these exact field names:

```json
{
  "Customer Name": "Ada Test",
  "Email": "ada@example.com",
  "Phone": "0803 000 0000",
  "Product Interested In": "Dell Latitude",
  "Budget": "₦400,000 to ₦600,000",
  "Customer Message": "Need 3 for the office",
  "Lead Source": "Product Page",
  "Submission Date": "2026-11-16T10:14:23.413Z"
}
```

| Field | Required | Where it comes from |
|---|---|---|
| Customer Name | Yes | Name box |
| Email | Yes | Email box, must look like `name@example.com` |
| Phone | Yes | Phone box, 7 to 20 digits, spaces, `+`, `-` or brackets |
| Product Interested In | Yes | Product dropdown, or "Not sure yet, please advise" |
| Budget | No | Budget dropdown, empty text if not chosen |
| Customer Message | No | Message box, empty text if blank |
| Lead Source | Always sent | Always `Product Page` |
| Submission Date | Always sent | The visitor's send time in ISO 8601 format (UTC), which Make reads as a date. Nigeria is UTC+1 |

## Built in safety checks

- Required fields are checked before anything is sent. Each problem shows a message under its field.
- The Send button locks while a request is running, so double clicks send only one enquiry.
- Sending the exact same enquiry twice in a row is blocked with a message.
- A hidden "Website" box catches spam bots. People never see it. If it is filled in, the page shows success but sends nothing.
- If Make takes longer than 15 seconds, the request stops and the visitor sees a connection error.
- All product text is escaped before it goes on the page, so typing HTML into the data block cannot break the layout.

## Editing the page

Open `index.html` and search for `EDITABLE CONTENT`. It is near the bottom of the file. Everything you would want to change is in that one block:

- `makeWebhookUrl`: the Make webhook that receives enquiries.
- `contact`: the email, phone and country shown on the page. These are placeholders now.
- `priceNote`: the line under each price saying it is a sample.
- `budgetOptions`: the choices in the Budget dropdown.
- `needs`: the buttons in the hero picker. Each one has a `label`, the `product` id it suggests and a short `why` sentence.
- `products`: one `{ ... }` block per laptop, with `id`, `name`, `category`, `description`, `processor`, `ram`, `storage`, `screen` and `price`.

Write `price` as a plain number, like `650000`. The page shows it as ₦650,000.

The product rows, the comparison table, the hero picker and the form's product dropdown all update from this block. To add a laptop, copy a block, give it a new `id` and change the details. To remove one, delete its block and any `needs` entry that points to its `id`. A need that points to a missing laptop is skipped quietly.

### Product photos

Each product has `image: ""`, which shows a sample laptop drawing in the colour set by `illustration` (`silver`, `graphite` or `black`). When you have real photos, upload them somewhere public and put the link in `image`, for example `image: "https://your-site.com/photos/hp-elitebook.png"`. Photos with a transparent or light background look best on the sand coloured panels.

## Changing the webhook

Replace the address in `makeWebhookUrl` with the one from your Make Custom Webhook module. If you set it to `""`, the form shows a configuration error and does not pretend the enquiry was sent.

The webhook address is visible to anyone who views the page source. That is normal for a simple form like this. If you ever start getting junk leads, add a filter in Make (for example, drop bundles with no email) or create a new webhook and update the address here.

## Setting up the Make scenario

1. In Make, create a new scenario and add **Webhooks > Custom webhook** as the first module.
2. Create a webhook (or pick the existing one ending in `...yt2bc1`) and copy its address into `makeWebhookUrl` if it is different.
3. Click **Run once** so the webhook is listening.
4. Send a test enquiry from the page (see below). Make should show "Successfully determined".
5. Add your next modules, for example **Google Sheets > Add a row** or a CRM module, and map the eight fields above into the right columns.
6. Turn the scenario on so it runs for every enquiry, not just test runs.

If you change the field names in the form later, open the webhook module and click **Redetermine data structure**, then send one more test.

## Testing the form

1. In Make, open the scenario with the Custom Webhook and click **Run once** so it is listening.
2. Double-click `index.html` to open it in your browser, or open the deployed page.
3. Click **Enquire About This Laptop** on any row, fill in the form and send it.
4. The bundle appears in Make. Check that all eight fields arrived with the right values.

Things worth checking:

- Leave Name, Email or Phone empty and click Send. You should see a message under each field.
- Type a bad email such as `ada@` and click Send. You should see the email message.
- Click Send several times quickly. Only one enquiry should reach Make.
- Pick a different need in the hero picker and click its Enquire button. The form should show that laptop.
- Try the page on a phone. The menu collapses into a button at the top right and the product rows stack.

## If something goes wrong

- "The enquiry service did not accept your enquiry": Make answered with an error. Check that the scenario is on, or that you clicked Run once, and that the webhook address is correct.
- "Your enquiry was not sent because the connection failed": no internet, or Make did not answer within 15 seconds. Try again.
- "Configuration error": `makeWebhookUrl` is empty. Put the webhook address back.
- The bundle arrives in Make but fields are missing: click **Redetermine data structure** in the webhook module and send one more test.

## Publishing the page

The page is a single file, so any static host works. With Vercel, Netlify or GitHub Pages, upload the folder and the host serves `index.html` as the home page. No settings or environment variables are needed.

## Before going live

- Replace the sample specs and prices with real ones.
- Replace the placeholder email and phone number.
- Add real product photos.
- Turn the Make scenario on so enquiries are processed without clicking Run once.

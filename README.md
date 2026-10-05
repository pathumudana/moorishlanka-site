# Moorish Lanka website — v1

Two static pages, sharing one stylesheet:

- `index.html` — moorishlanka.com (licensing advisory, consulting, MPN partner status, contact)
- `academy.html` — intended for academy.moorishlanka.com (courses, Koko payment section, enrolment)
- `styles.css` — shared design system

This is a static MVP: no backend, no real payments yet, no live MPN badge. It's built to deploy today and to be extended module‑by‑module.

## 1. Deploy to Azure (Static Web Apps)

Azure Static Web Apps is the fastest path for a static site like this, it's free at this scale, gives you free TLS, and supports custom domains and subdomains out of the box.

1. Push these files to a GitHub repo (e.g. `moorishlanka-site`).
2. In the Azure Portal: **Create a resource → Static Web App**.
   - Subscription: your existing Azure subscription.
   - Plan type: **Free**.
   - Deployment source: GitHub → select the repo/branch.
   - Build details: app location `/`, no build step needed (plain HTML/CSS).
3. Azure creates a GitHub Actions workflow automatically and deploys on every push.
4. Once deployed, go to the Static Web App's **Custom domains** blade:
   - Add `moorishlanka.com` (and `www.moorishlanka.com`), verify via the TXT/CNAME record Azure gives you, add it at your domain registrar.
5. For the academy subdomain, you have two clean options:
   - **Same Static Web App, same deployment**: add a custom domain `academy.moorishlanka.com` pointing at the same app, and have your hosting serve `academy.html` as that domain's `index.html` (rename the file to `index.html` inside an `academy/` folder and add a route rule in `staticwebapp.config.json`, or simplest for now: deploy it as a *second* Static Web App — see below).
   - **Separate Static Web App** (recommended while this is simple): create a second Static Web App from the same repo (or a second repo) containing only `academy.html` (renamed `index.html`) + `styles.css`, and attach `academy.moorishlanka.com` to it. This keeps the Academy's payment code isolated from the main consulting site, which matters once Koko goes live.

Either way, at your domain registrar you'll add a CNAME for `academy` pointing at that Static Web App's default hostname, same as the root domain.

## 2. Microsoft Partner (MPN) section

The partner panel on `index.html` is a placeholder. Once you have:
- Your MPN/Partner ID,
- Confirmed designations (e.g. Solutions Partner for Security),
- The official partner logo from Partner Center's marketing resources,

replace the "pending" text and swap in the real badge image. Microsoft's partner agreement has specific rules about how the logo can be displayed — check Partner Center's brand guidelines before publishing it.

## 3. Koko payments — what's real vs. placeholder here

The Academy page's Koko section is **UI only**. To actually take a payment or a Koko installment plan, you need a small server-side piece, because payment provider API keys must never sit in a static page a visitor's browser can read. The clean way to add this on Azure:

1. Add **Azure Functions** (consumption plan is cheap/free at this scale) linked to the Static Web App — Azure Static Web Apps has first-class support for this ("Static Web Apps + managed Functions API").
2. The Function holds your Koko merchant credentials (in Azure Key Vault or Function app settings, never in the HTML/JS) and:
   - Creates a Koko checkout session / installment plan when "Enrol with Koko" is clicked,
   - Receives Koko's webhook/callback to confirm payment,
   - Updates your records (even a simple Azure Table or Dataverse row is enough at this size).
3. The static page just calls your Function's endpoint (e.g. `/api/create-koko-session`) and redirects the browser to whatever checkout URL Koko returns.

I can build this Function (and the matching front-end wiring) once you have Koko's merchant/API documentation and test credentials — happy to do that as the next step.

## 4. Suggested next steps, in order

1. Get the site live on both domains with placeholder content (this package).
2. Swap in your real MPN ID/badge once Partner Center issues it.
3. Wire the contact and enrolment forms to something that actually sends you the submission (Azure Function + email, or a form service like Formspree as a quick interim).
4. Build the Koko Azure Function and connect the "Pay with Koko" button.
5. Split `index.html`/`academy.html` into multiple pages per section if the content grows (e.g. a dedicated `/licensing`, `/consulting`, `/partner` page each).

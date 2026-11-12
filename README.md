# Thirsty Dreamer — Netlify deployment package

This is the client website with a secure Netlify Function that sends contact-form inquiries through Resend. The Resend API key is read only by the server-side function and is never included in browser JavaScript.

## Required before email delivery can work

You must supply a **complete, valid Resend API key** and a sender address on a domain verified in Resend. The API token previously pasted into chat was truncated (`re_...`) and cannot be used. Do not paste API keys into chat or commit them to Git.

## Netlify setup

1. Deploy this project folder to Netlify (Git-connected deployment is recommended; Netlify's manual deploy can be used for a static preview, but serverless functions must be included in a supported deploy).
2. In Netlify → Site configuration → Environment variables, add:
   - `RESEND_API_KEY` — the full Resend API key (keep secret).
   - `CONTACT_TO_EMAIL` — `amrittiwari305@gmail.com` (change to the client's preferred inbox if needed).
   - `CONTACT_FROM_EMAIL` — e.g. `Website Inquiries <inquiries@client-domain.com>`, using a domain verified in Resend.
3. In Resend, add a domain controlled by the client and publish the exact DNS records Resend supplies. Wait until it is verified.
4. Trigger a fresh deploy after setting the variables.
5. Submit a real test inquiry and confirm the message arrives in the receiving inbox (also check spam/junk).
6. Add the client's custom domain in Netlify → Domain management and apply the exact DNS records Netlify displays at the domain registrar. Wait for HTTPS provisioning.

## Local testing

1. Install Node.js 20+ and the Netlify CLI (`npm install -g netlify-cli`).
2. Copy `.env.example` to `.env` and replace the placeholders locally. Never commit `.env`.
3. Run `npm run dev` from the project root.
4. Open the local URL printed by Netlify CLI and submit the contact form.

Email delivery requires a valid Resend API key and a verified sender domain. Without those, the function deliberately returns a clear configuration error rather than falsely reporting success.

## Contact form behavior

- The browser posts to `/.netlify/functions/contact`.
- The server validates the name, email, phone, inquiry type, message, and consent.
- User-provided content is HTML-escaped before it is included in the email.
- The visitor's email is set as `reply_to`, so replying to the notification goes to the visitor.
- If Supabase is configured, a secondary private archive copy is attempted after email delivery; email delivery does not depend on Supabase.

## Security

- Never place `RESEND_API_KEY` in `index.html`, `js/site.js`, or any browser-delivered file.
- Never commit `.env`, `.env.local`, or real API credentials.
- Use Netlify environment variables in production.


## Static contact-form email (EmailJS)

This build sends the contact form directly from the browser using EmailJS. It does not require Vercel Functions, Netlify Functions, or a custom Node.js backend.

1. Create an EmailJS account and connect the mailbox/service that will send the inquiry.
2. Create an email template. Set the template's **To email** to `massi@motherdrinks.co` for the client's production site. For your own testing, temporarily set it to `amrittiwari305@gmail.com`.
3. Use these template variables in the EmailJS template:
   - `{{from_name}}` — visitor name
   - `{{reply_to}}` — visitor email (set Reply-To to this)
   - `{{phone}}` — visitor phone
   - `{{inquiry_type}}` — inquiry category
   - `{{message}}` — visitor message
   - `{{consent}}` — consent status
4. Open `emailjs-config.js` and replace `YOUR_EMAILJS_SERVICE_ID`, `YOUR_EMAILJS_TEMPLATE_ID`, and `YOUR_EMAILJS_PUBLIC_KEY` with the identifiers from EmailJS → Account / Email Services / Email Templates.
5. Upload the static website files to any static host. No backend function is required.

EmailJS's browser Public Key is designed for client-side use; do not put a private access token or secret key in this file. Restrict allowed origins in EmailJS where available, enable CAPTCHA/spam protection if available, and note that EmailJS free-plan sending quotas apply. The recipient is controlled in the EmailJS template dashboard; `toEmail` is included as a template parameter for convenience but the template's To field should be configured explicitly.

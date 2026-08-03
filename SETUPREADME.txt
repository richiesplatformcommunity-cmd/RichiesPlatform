SEGMENTATION FORM PAGE — SETUP NOTES
=====================================
File: index.html (single self-contained file — HTML, CSS and JS all inline)

This is Stage 1 of the funnel described in the Business System Development
Plan: the first page traffic lands on from Link in Bio / Online Ads. It
segments each visitor into Business Owner / Freelancer / Student, collects
their name + email, and routes them to the matching Stage 2 landing page.

Before going live, edit the CONFIG block near the bottom of index.html
(search for "CONFIG — edit these before going live") and the two tracking
snippets in <head>:

1. GA4 (Google Analytics 4)
   - Near the top of <head>, replace every "G-XXXXXXXXXX" with your real
     GA4 Measurement ID (Admin > Data Streams > your web stream).
   - A "generate_lead" event fires automatically on successful submit,
     tagged with the chosen segment (business_owner / freelancer / student).

2. Meta Pixel
   - In <head>, replace "0000000000000000" (appears twice — once in the
     script, once in the <noscript> fallback) with your real Pixel ID from
     Meta Events Manager.
   - A "Lead" event fires automatically on successful submit, tagged with
     the chosen segment.

3. Leads database (Google Sheets)
   - Set CONFIG.SHEETS_WEBHOOK_URL to a Google Apps Script Web App URL
     (or a Zapier/Make catch hook) that appends each submission to your
     Sheet. Apps Script is the free native option — deploy a script bound
     to your Sheet as a Web App ("Execute as: Me", "Who has access:
     Anyone"), then paste the deployment URL in.
   - The payload sent is: fullName, email, segment, source, submittedAt.
   - This call uses mode:"no-cors" and never blocks the redirect, so a
     slow or misconfigured webhook won't strand a visitor on this page —
     rely on the Stage 1 follow-up system (per the dev plan) to catch
     anyone the webhook call fails to reach.

4. Landing pages (Stage 2 routing)
   - Set the three URLs in CONFIG.LANDING_PAGES to your real Business
     Owner / Freelancer / Student landing pages.
   - The visitor's name, email and segment are passed along as URL query
     params (?name=...&email=...&segment=...) so the landing page /
     payment step can prefill or personalize.

5. Branding
   - Update the "Privacy Policy" link's href.

BRAND SYSTEM (Richies Platform, per the brand guideline PDF)
==============================================================
This version is built on the actual Richies Platform brand book, not a
generic placeholder theme:

- Colors: only the 4 approved brand colors are used anywhere in the file
  — Dark Blue #072463, Pink #cb4473, Light Azure #e0faff, Light Neon Pink
  #ffecfc. Dark mode reuses Dark Blue as the page background (rather than
  inventing a new near-black) and brightens Pink slightly (#E8709B) so it
  still passes contrast on navy.
- Logo: the header uses the real Auxiliary Logo mark, extracted directly
  from the brand PDF and re-composited with its correct transparency
  (the raw PDF export had its alpha channel stripped — this file has the
  corrected version baked in as an inline base64 image, so there's no
  separate asset to manage). The wordmark next to it follows the
  guideline's two-tone treatment: "Richies" in Dark Blue, "Platform" in
  Pink.
- The pink underline under "BEFORE WE BEGIN" mirrors the recurring
  heading-underline motif used throughout the brand guideline's section
  titles.
- Fonts: the guideline specifies "Gatwick" (headings) and "Canvas Sans"
  (body/UI), which are licensed brand fonts — we don't have the font
  files. Poppins and Work Sans stand in as close open-source matches for
  now. Every font-family rule in the CSS already lists Gatwick / Canvas
  Sans FIRST, so as soon as you add the real fonts (see the @font-face
  comment block in <head>) they take over automatically with no other
  changes needed.
- Segment cards deliberately don't invent new accent colors per segment
  (the brand guideline restricts the palette to the 4 colors above) —
  icon tiles alternate between Light Azure and Light Neon Pink, and the
  single "selected" accent color is always Pink, matching how the
  guideline itself uses pink for checkmarks and highlights throughout.

Everything else (copy, layout, validation, honeypot spam trap) is ready
to use as-is. Tested responsive from 390px mobile up, and in both light
and dark OS color schemes.


STAGE 2 — THE 3 LANDING PAGES
=====================================
Files: business-owners.html, freelancers.html, get-started.html
(single self-contained files, same pattern as index.html)

These are the three Stage 2 segmented landing pages from the Business
System Development Plan — each one collects the commitment fee via
Paystack, then hands the visitor off to the centralised booking page.
Filenames match the placeholder paths already set in index.html's
CONFIG.LANDING_PAGES, so if you host all four files at the same domain
root, you only need to update the domain there, not the paths.

  Segment           File                  Fee      Deliverable framing
  ----------------  --------------------  -------  --------------------------------
  Business Owner    business-owners.html  ₦25,000  Digital Marketing Audit + Competitor Report
  Freelancer        freelancers.html      ₦20,000  Niche Selection Blueprint + Pricing Calculator
  Student/Beginner  get-started.html      ₦15,000  Career Roadmap + Resume Review

Each page reads ?name=&email=&segment= from the URL (as sent by
index.html) to show a personalized greeting and prefill the checkout
email — visiting the file directly with no query string still works
fine, the greeting just stays hidden.

Before going live, edit the CONFIG block near the bottom of each of the
3 files (search for "CONFIG — edit these before going live"), and the
GA4 / Meta Pixel snippets in each <head>:

1. GA4 and Meta Pixel
   - Use the exact same Measurement ID / Pixel ID as index.html across
     all 3 landing pages, so the whole funnel rolls up under one
     property/ad account.
   - Events fired automatically: view_item / ViewContent on load,
     begin_checkout / InitiateCheckout when "Pay" is clicked,
     purchase / Purchase once Paystack confirms payment client-side.
     All are tagged with segment, value (the NGN fee) and currency.

2. Paystack
   - Set CONFIG.PAYSTACK_PUBLIC_KEY in EACH of the 3 files to your real
     Paystack PUBLIC key (starts with pk_live_ or pk_test_). The public
     key is meant to be used client-side — never put your secret key
     (sk_...) in this file.
   - Amounts are already set per page (₦25,000 / ₦20,000 / ₦15,000) via
     CONFIG.AMOUNT_NGN — the script converts to kobo automatically.
   - IMPORTANT: the on-page "payment succeeded" callback fires as soon
     as Paystack's popup reports success in the visitor's browser. That's
     enough to redirect them and fire your ad-tracking events, but it is
     NOT proof of payment by itself (a visitor could in theory close the
     tab before the callback runs, or the network could drop). Before
     treating someone as paid, confirm the transaction server-side too —
     either enable a Paystack webhook, or use System.io's native Paystack
     integration on the booking page as your source of truth.

3. Central booking page (Stage 2.2 of the dev plan)
   - You don't have this link yet — CONFIG.CENTRAL_BOOKING_URL is set to
     a placeholder ("https://your-systeme-domain.com/book-a-call") in
     all 3 files. Update all three once the System.io booking page exists.
   - Name, email, segment and the Paystack payment reference are passed
     along as URL query params so the booking page can prefill or
     cross-reference the payment.

4. Branding
   - Update the "Privacy Policy" link's href in each file's footer.
   - The refund-policy line in each FAQ section is marked
     "[Placeholder — set your real refund policy here before launch.]"
     — replace with your actual policy before going live.

5. Founder photo ("Who you're talking to" section)
   - All 3 pages have a section introducing Lawrence Martins Ekpen, since
     he's taking every consultation personally — this sits right before
     the checkout card, where a trust signal does the most good.
   - The photo you shared came through as a pasted image, not a file, so
     I couldn't embed it directly (same as the real logo file earlier —
     I can only bake in files I can actually open on disk). Each page
     currently shows a placeholder "LM" initials badge in place of a
     photo.
   - To swap it in: send the photo as an actual file upload (not pasted
     inline), and I'll re-embed it the same way I did the brand logo —
     or if you're doing it yourself, replace the
     <div class="founder-photo" aria-hidden="true">LM</div>
     line in each of the 3 files with:
     <div class="founder-photo"><img src="lawrence.jpg" alt="Lawrence Martins Ekpen"></div>
     (drop lawrence.jpg next to the HTML files, or point src at a hosted
     URL).
   - The bio text is short by design — a landing page needs a trust
     signal, not a full resume. It's worded slightly differently on each
     page (audit/growth systems for Business Owners, positioning/pricing
     mentorship for Freelancers, AlphaTech Con/ALX Ascend for Students) to
     match what that segment cares about most, but all 3 point back to
     the same core facts: founder of Richies Platform, product marketing
     & growth specialist, AlphaTech Con facilitator.

Same brand system as index.html throughout (see the BRAND SYSTEM notes
above) — same colors, logo, fonts, and Gatwick/Canvas Sans font-swap
path. Tested responsive from mobile up, and in both light and dark OS
color schemes, including the checkout validation and dark-mode states.


COMMUNITY MEMBERSHIP PAGE (standalone — separate from the 4 files above)
=====================================
Files: community.html, community-thank-you.html

This is a different funnel from everything above. The other 4 files are
the Business System Development Plan's Stage 1→2 flow (segmentation form
→ 3 audience landing pages, each selling a one-off consultation). This
page instead sells Richies Platform community MEMBERSHIP directly —
built from the "Richies Platform 2.0 Community Onboarding" content and
LinkedIn description you sent. It's a standalone, self-contained page,
not linked from index.html or the 3 landing pages — traffic reaches it
directly via referral links, WhatsApp status, paid ads, or SEO, per what
you described.

Flow: community.html (sell the community, pick Freelancer or Business
Developer track, collect name/phone/email, pay via Paystack) →
community-thank-you.html (confirmation + link to the WhatsApp waiting
group).

Before going live, edit the CONFIG block near the bottom of each file:

1. GA4 and Meta Pixel — same IDs as your other 4 pages, so everything
   rolls up under one property/ad account. community.html fires
   view_item / begin_checkout / purchase (tagged "track": freelancer or
   business_developer); community-thank-you.html just fires a page view
   for retargeting-exclusion / thank-you-audience purposes — no extra
   purchase event there, since that already fired on community.html.

2. Membership fee — you said this isn't set yet, so
   CONFIG.AMOUNT_NGN is 0 by default. While it's 0, the price shows "TBA"
   and the Pay button is disabled ("Membership Fee Coming Soon") instead
   of silently accepting a fake amount. As soon as you put a real number
   in AMOUNT_NGN, the whole checkout activates automatically — no other
   changes needed.

   Open question I couldn't resolve from what you sent: is this a
   ONE-TIME fee or a RECURRING membership (monthly/termly)? The page is
   currently wired for a one-time Paystack charge. If it should renew,
   this needs to be rebuilt on Paystack's subscription Plans API instead
   — flag it and I'll switch it over. There's a placeholder note about
   this in the FAQ section too.

3. Paystack — same PUBLIC key pattern as the 3 landing pages
   (CONFIG.PAYSTACK_PUBLIC_KEY, pk_live_/pk_test_, never the secret key).
   Same caveat as before: the on-page "success" callback is good enough
   for the redirect and ad-tracking events, but confirm payment
   server-side too (Paystack webhook) before treating someone as a paid
   member.

4. Lead capture — Make.com, not Google Sheets this time
   Set CONFIG.MAKE_WEBHOOK_URL to a Make.com scenario's "Custom Webhook"
   trigger URL (in Make: add a Webhooks > Custom Webhook module, copy its
   address). The lead (name, phone, email, track) is sent here BEFORE
   Paystack opens, so you keep the contact even if someone abandons
   checkout — same fire-and-forget pattern as the Sheets webhook on
   index.html (mode: "no-cors", never blocks the page).

5. WhatsApp waiting group — set in community-thank-you.html's CONFIG
   block: CONFIG.WHATSAPP_GROUP_URL, a chat.whatsapp.com invite link.
   Placeholder for now since you don't have it yet.

6. Founder name — still needs your confirmation
   This page uses "Lawrence Martins Ekpen" (same as the 3 landing pages),
   but the LinkedIn posts you shared tag him as "Martins Lawrence" as the
   facilitator. I haven't heard back on which is correct — let me know
   and I'll fix it everywhere in one pass rather than you having to
   find every instance across 6 files.

7. Founder photo — same limitation as before: the image with the photo +
   wordmark lockup came through as a pasted image, not a file, so it
   isn't something I can open and embed. The "Who you're learning from"
   section uses the same "LM" placeholder badge as the 3 landing pages.
   Send just the headshot as an actual file attachment (not the full
   deck) and it'll get embedded the same way the brand logo was.

Content note: the curriculum details (weekly structure, performance
metrics, attendance rules, community group names) are reproduced from
what you pasted, condensed into scannable sections rather than full
paragraphs — a sales page needs to be skimmable. Nothing was invented;
if anything reads off, it's worth a proofread pass against your source
deck before launch.

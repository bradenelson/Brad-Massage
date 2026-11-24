# Boyfriend Massage Parlor — live booking edition

A mobile-friendly joke spa website with real Google Calendar availability, confirmed event creation, and optional automatic email via Resend.

## Deployment

1. Create a private Google Calendar dedicated to these appointments (or use your main calendar). Note: only events on the selected calendar are checked for conflicts; to avoid clashes with other calendars, mirror busy events onto it or use your main calendar.
2. In Google Cloud, enable the Google Calendar API, create a service account, and download its JSON key. Share the selected Google Calendar with the service account email and grant **Make changes to events** permission. If your Google account prevents external sharing, use an eligible account or a different integration.
3. Deploy this entire folder as a Netlify site (Git deployment recommended). **Do not use Netlify Drop for live booking:** the site requires Netlify Functions and environment variables.
4. In Netlify → Site configuration → Environment variables, set:
   - `GOOGLE_SERVICE_ACCOUNT_EMAIL` = `client_email` from the JSON key
   - `GOOGLE_PRIVATE_KEY` = `private_key` from the JSON key (including BEGIN/END markers; literal `\\n` supported)
   - `GOOGLE_CALENDAR_ID` = calendar ID from Google Calendar settings
   - `BOOKING_TIMEZONE` = `America/Los_Angeles` (default)
5. To enable email notifications, create a Resend account, verify a sending domain, then set:
   - `RESEND_API_KEY` = Resend API key
   - `RESEND_FROM_EMAIL` = verified sender address (e.g. Spa <bookings@yourdomain.com>)
   - `NOTIFY_EMAIL` = your email address
6. Trigger a fresh deployment after changing environment variables. Visit the website, select a date and massage, and test a booking.

## Booking behavior

- Live slots are shown from **5 PM–9 PM Pacific**, in 30-minute increments, up to 45 days ahead; appointments must start at least 30 minutes from now.
- Busy times are read from Google Calendar; booked appointments are created in that calendar.
- If email delivery fails, the appointment still exists and the client is told to share the ticket.
- The frontend never stores Google credentials or email API keys. They stay in Netlify server environment variables.

## Important limitations

- Google Calendar's free/busy check followed by event insertion is **not atomic**: two near-simultaneous bookings could theoretically overlap. For a one-person private site this risk is small; a production-grade system needs transactional locking.
- No access code is required. Anyone who can reach the public site or its endpoints can view free slots and book appointments. Consider host-level access protection or rate limiting.
- Only one calendar is checked. Use a calendar that reflects your actual availability.
- No SMS is included; notifications are email only.
- This is a deployable implementation, not an already-connected live service. Until Google credentials are configured, the booking interface intentionally fails closed.

## Local development

Run `npm install`, then `npx netlify dev` with environment variables configured. The page and `/.netlify/functions/*` endpoints will be served together.

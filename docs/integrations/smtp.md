# Email / SMTP

SMTP is optional. The app runs fine without it. Turn it on if you want any of these:

- **Password resets** (the `Forgot password?` link on the login screen).
- **User invites** (the admin invites someone by email instead of handing them a link).
- **Weekly summary emails** (opt-in per user).
- **App-specific notifications**: expiration digest (CookTrace), recipe-share notifications (CookTrace).

Without SMTP, all of the above either fall back to admin-generated links or silently skip.

## Where the settings live

All three apps expose SMTP at `Settings → Email` as its own section. The form is byte-for-byte the same across the three apps. All three call the same `POST /api/app-config` (per-field save) and `POST /api/app-config/test-email` (test send) endpoints server-side.

!!! note "If you are on NutriTrace 1.0.3 or earlier"
    The SMTP fields lived inside `Settings → Authentication` before the layout was aligned with CookTrace and LiftTrace. Everything below still applies. Only the parent screen differs.

## Field-by-field

| Field         | `app_config` key | Notes                                                                                     |
| ------------- | ---------------- | ----------------------------------------------------------------------------------------- |
| SMTP Host     | `smtp_host`      | e.g. `smtp.fastmail.com`, `smtp.gmail.com`, `smtp.postmarkapp.com`.                       |
| Port          | `smtp_port`      | Defaults to `587`. Use `465` for implicit TLS, `2525` if your provider offers it.         |
| TLS           | `smtp_secure`    | Boolean toggle. Turn on for port 465 (implicit TLS). Off for 587 (STARTTLS is auto).      |
| Username      | `smtp_user`      | Usually the mailbox address. Provider-specific for API-backed transports (see below).      |
| Password      | `smtp_pass`      | See "The Change button" below.                                                            |
| From address  | `smtp_from`      | Can be a bare address (`noreply@example.com`) or `Name <addr@example.com>` display form.  |

### The Change button (password field)

The password field never round-trips a real value to the browser. The server redacts stored passwords on GET and returns `••••••••` as a placeholder. When the form sees that placeholder, it locks the input read-only and swaps the reveal button for a **Change** button. Tap Change to blank the field, then type a new password. If you Save without touching the password field (still showing `••••••••`), the stored password is left alone.

This is the same reason the "reveal password" eye icon disappears once a password is stored: there is nothing meaningful to reveal, because the app doesn't have the plaintext to show you.

## Send Test

The Send Test button opens a small dialog that asks where to send the test. It defaults to your account's own email address, but you can type anything.

The test is a real, branded HTML email (same wrapper as the password reset and invite templates), not a bare SMTP handshake. If it lands in the recipient's inbox, the whole delivery pipeline works, styling included. If you get a green banner but no email, check the recipient's spam folder before assuming SMTP is broken.

The dialog runs against the currently-typed form values, not the last-saved ones. So you can try `smtp.example.com` and hit Test without saving; if it fails, the save is safe to skip.

## Env-lock

Set any of `SMTP_HOST / SMTP_PORT / SMTP_SECURE / SMTP_USER / SMTP_PASS / SMTP_FROM` in the environment (or via `*_FILE` Docker secrets), and the Settings form freezes with a lock banner. All fields become read-only until the env vars are removed and the server is restarted.

```env
SMTP_HOST=smtp.fastmail.com
SMTP_PORT=587
SMTP_USER=you@example.com
SMTP_PASS_FILE=/run/secrets/smtp_pass
SMTP_FROM=CookTrace <noreply@example.com>
```

Env vars take priority over anything already in `app_config`, and the seed runs on every boot.

## Provider quirks

**Gmail.** SMTP requires an [App Password](https://support.google.com/accounts/answer/185833), not your Google account password. Enable 2FA on the account first, then generate an App Password and use that as `SMTP_PASS`. Host `smtp.gmail.com`, port `587`, TLS off (STARTTLS auto).

**Postmark.** Use your Server API token as *both* `SMTP_USER` and `SMTP_PASS`. Host `smtp.postmarkapp.com`, port `587`. The From address must be a verified sender in the Postmark dashboard.

**Mailgun.** Use SMTP credentials (found under `Sending → Domain Settings → SMTP Credentials`), not your Mailgun API key. Host `smtp.mailgun.org` or `smtp.eu.mailgun.org`, port `587`.

**AWS SES.** Generate SMTP credentials from the SES console (they're different from your AWS access keys). Host `email-smtp.<region>.amazonaws.com`, port `587`.

**Local test relay.** [MailHog](https://github.com/mailhog/MailHog) or [Mailpit](https://github.com/axllent/mailpit) at `smtp.local:1025` (no TLS, no auth) is the easy way to inspect templates without sending real mail.

## Troubleshooting

If Send Test returns "SMTP test failed":

- Check host + port + TLS combo. Port 465 needs TLS on; 587 wants TLS off (STARTTLS is auto-negotiated).
- Confirm the account isn't locked / rate-limited by the provider (Gmail is particularly stingy).
- Check `docker compose logs` for the underlying nodemailer error; the UI shows only the short form.

## Related

- [Push notifications](notifications.md)

# Two-step sign-in

Two-step sign-in asks for a code from an authenticator app after your password. NutriTrace has it from its next release; the other Trace apps don't have it yet.

It's for accounts that sign in with a password. If you sign in with single sign-on (OIDC), your identity provider handles two-step sign-in, and NutriTrace says so in Settings > Profile. API tokens, MCP tokens, sessions you're already signed in with, saved sign-ins with biometric unlock, and the Android app's Use Locally mode (which has no accounts) are not affected.

An account with two-step sign-in on (or one that must set it up because it's required) is never linked to a single sign-on account by its email alone (Auto-Link): sign in with your password and code, then link the provider yourself in Settings > Profile > Linked Accounts. A provider already linked to an account is trusted to do its own two-step sign-in.

## Set it up

1. Open **Settings > Profile > Two-Step Sign-In** and choose **Set Up Two-Step Sign-In**.
2. Enter your password and choose **Continue**.
3. Scan the QR code with an authenticator app (Aegis, 2FAS, Google Authenticator, 1Password, Bitwarden, and others). Can't scan it, for example on the phone the app is on? Choose **Copy Key** and enter the key in the app instead: time-based, 6 digits, every 30 seconds.
4. Type the 6-digit code the app shows and choose **Turn On**. Two-step sign-in is only on once a right code has been entered.
5. **Save your recovery codes.** You get ten, shown once. Each signs you in once if you lose your phone. **Copy Codes** (or **Download** in a browser), keep them somewhere safe like a password manager, then choose **I Saved My Codes**.

The card then shows **On**, when it was turned on, and how many recovery codes are left. **New Recovery Codes** replaces them all (your password and a code needed). **Turn Off** needs your password and a code from the app or a recovery code.

## Signing in

After your username and password, NutriTrace asks for the **Code** from your authenticator app. **Use a Recovery Code** takes one of your saved codes instead.

- A code is accepted for its 30 seconds and one step either side, for a phone clock that's a little off.
- A code works once. The same code, or an older one, is refused afterwards.
- Five wrong codes send you back to the password, and an account gets five wrong codes per 15 minutes (an address ten), so a code can't be guessed.
- The Android app asks for the code in the same way, when you connect to a server and in **Settings > Server Connection**.

**Forgot Password** changes your password, but never signs you in past the code: after the reset, sign in with the new password and your code.

## Locked out { #locked-out }

Getting back in never takes SQL, and always touches one account only. Try these in order.

### 1. A recovery code

On the sign-in page, after your password, choose **Use a Recovery Code**. Then make new codes, or set up two-step sign-in again on your new phone, in Settings > Profile.

### 2. Ask an admin

An admin opens **Settings > Users** and chooses the **Reset Two-Step Sign-In** button next to the person. The admin enters their own password (and their own code, when they use two-step sign-in). An admin can't reset their own this way; use a recovery code, another admin, or step 3.

The person then signs in with their password alone, and can set two-step sign-in up again.

### 3. The server's owner

With `RECOVERY_TOKEN` set in the server's environment (16 characters or more, for example `openssl rand -hex 24`), open **Locked Out?** on the sign-in page, enter the token and the username under **Reset Two-Step Sign-In for One Account**, and choose **Reset Two-Step Sign-In**. Nothing else changes. (The same box can still disable user management, which deletes every account: that's not needed here.)

Or, on the server:

```bash
docker exec -it nutritrace npm run reset-2fa -- <username>
```

Run it without a username to list the accounts that have two-step sign-in on. It works whatever the state of `RECOVERY_TOKEN`.

Every reset, every recovery code used, and every time two-step sign-in is turned on or off is written to the server log, without codes or secrets. When the server sends email ([Email / SMTP](../integrations/smtp.md)), the person is emailed too.

## Require it for everyone

An admin can turn on **Settings > Users > Require Two-Step Sign-In for Everyone**. Anyone who signs in with a password and hasn't set it up is taken through setup right after their password, the next time they sign in, and saves their recovery codes before the app opens. People accepting an invite set it up straight away. Single sign-on accounts aren't affected.

## How secrets are kept

- Each authenticator secret is encrypted (AES-256-GCM, tied to its account) with a key of its own, never the sign-in secret `JWT_SECRET`. Changing `JWT_SECRET` doesn't affect two-step sign-in.
- The key is `two-step.key`, made at random next to the database (`/data/db` in the standard compose file) the first time someone sets up two-step sign-in, readable by the server only. Nothing to set up, and it survives restarts and updates. To keep it outside the data folder, set `TWO_STEP_KEY` (or `TWO_STEP_KEY_FILE`, see [Docker secrets](../self-hosting/secrets.md)).
- Recovery codes are stored only as hashes.
- A full backup holds the encrypted secrets and the hashes, never the key. Back up `two-step.key` with the database folder.
- `TWO_STEP_KEY` must be 32 characters or more (`openssl rand -hex 32`); a shorter one isn't used, and the log says so.

If the key changes or is lost (a different `TWO_STEP_KEY`, a backup restored on another server), nobody is locked out: codes from authenticator apps can't be checked, and the sign-in page and Settings say so plainly. Recovery codes still work, and the steps under [Locked out](#locked-out) reset it.

## Older apps

An Android app or a cached web page from before two-step sign-in can't enter a code. Signing in to an account with two-step sign-in on from one says "This account uses two-step sign-in. Update NutriTrace (or reload this page) to enter your code." and doesn't sign in.

## Related

- [Local users, invites, roles](local-users.md)
- [SSO-only mode and recovery](sso-only.md)
- [Session lifetime and password policy](sessions.md)

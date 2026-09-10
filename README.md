# Purdue SSO

A Violentmonkey userscript that automates Purdue sign-in: username, password, authenticator codes, and the “Stay signed in?” prompt.

## Install and configure

1. Install [Violentmonkey](https://violentmonkey.github.io/get-it/).
2. Open [purdue-sso.user.js](https://raw.githubusercontent.com/pmxi/purdue-sso/main/purdue-sso.user.js) and install it. Alternatively, create a new script in Violentmonkey and paste the file's contents.
3. Open the installed script in Violentmonkey's editor. Fill in the three empty fields near the top:

   ```js
   const config = {
     username: 'your-career-account',
     password: 'your-password',
     totp_uri: 'your-existing-otpauth-uri',
   };
   ```

4. Save, then visit your Purdue service and begin its normal login flow.

The repository ships with **empty credentials** and does nothing until configured. Edit only the installed browser copy. Do not commit your configured copy or share an extension export containing it. Installing a newer copy may replace your edits; preserve your configuration privately before updating.

`totp_uri` must be an existing `otpauth://totp/...` enrollment URI containing the authenticator secret. A six-digit code is not the secret. The script does not enroll an authenticator or extract secrets from an authenticator app. SHA-1, SHA-256, SHA-512, 6–8 digits, and custom periods are supported.

For Firefox private windows, allow Violentmonkey under **Extensions and Themes → Violentmonkey → Run in Private Windows → Allow**. See [Firefox's instructions](https://support.mozilla.org/en-US/kb/extensions-private-browsing).

## Behavior

- Runs on `sso.purdue.edu`, `idp.purdue.edu`, and `login.microsoftonline.com`, over HTTPS and in the top-level page only.
- On Microsoft, requires Purdue's tenant, the configured account, or a recent continuation of that login. A different detected account prevents automation.
- Fills username/password, chooses “Use a verification code” instead of app approval, and generates a TOTP locally with Web Crypto.
- Checks “Don't show this again” and chooses **Yes** on “Stay signed in?”.
- Excludes hidden login fields, preserves conflicting user-entered values, stops on detected sign-in errors, and prevents duplicate submissions.
- Waits for a fresh authenticator code when the current code has fewer than five seconds left.
- Stops after three minutes on a single page. The Violentmonkey menu offers **pause/enable** and **retry sign-in**; retry clears the per-tab submission guard.

The bare `https://sso.purdue.edu/` address is not a login entry point. Start from the Purdue service you want to use, such as [Brightspace](https://purdue.brightspace.com).

## Credential handling

The installed copy stores the password and authenticator secret together in source code. Anyone who can read that copy or its exports can read both. This reduces the separation normally provided by two-factor authentication.

There are no external libraries, telemetry, or network requests made by the script itself. It fills the normal Purdue/Microsoft login forms. It uses Violentmonkey's isolated content context; its session-storage markers contain timestamps, not credentials. See the official [metadata](https://violentmonkey.github.io/api/metadata-block/) and [API](https://violentmonkey.github.io/api/gm/) documentation.

## Development

Requires Node.js 20 or newer. No dependencies to install.

```sh
npm test
npm run check
```

Tests use synthetic credentials and public TOTP vectors. They cover TOTP generation, account guards, repeated submissions, input handling, the off-screen password field on Microsoft's username screen, and the stay-signed-in checkbox/Yes sequence. DOM tests simulate the forms; they do not sign in to a real account.

The password/MFA flow was exercised against Purdue's live Microsoft login in September 2026. The later username and stay-signed-in fixes have regression coverage. Login pages can change; this is an unofficial project and is not affiliated with Purdue or Microsoft.

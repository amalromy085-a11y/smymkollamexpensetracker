# Security Policy

## Project: Yuvadeepti SMYM Kollam Expense Tracker

---

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.x     | ✅ Actively supported |
| < 1.0   | ❌ Not supported   |

---

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please **do not open a public GitHub issue**.

Instead, report it privately by:

- 📧 **Email:** amalromy085@gmail.com
- 🔒 **GitHub Private Disclosure:** Use GitHub's [Security Advisory](../../security/advisories/new) feature

We will acknowledge your report within **48 hours** and aim to release a fix within **7 days** for critical issues.

---

## Security Architecture

### Authentication

- Passwords are **hashed client-side** using a deterministic hash function before comparison
- No plaintext passwords are stored anywhere in the application
- Session data is stored in `sessionStorage` (cleared when the browser tab closes)
- There is **no persistent login token** stored in `localStorage`

> ⚠️ **Production Recommendation:** For a production deployment, replace the client-side hash with **bcrypt** on a secure backend server. Client-side hashing alone should not be the sole authentication mechanism for sensitive systems.

### Firebase Firestore

- All transaction data is stored in **Google Firebase Firestore**
- Firebase API keys in this project are **not secret** — they are intended to be public (this is by Firebase design)
- Access control is enforced via **Firestore Security Rules**, not the API key
- Recommended Firestore Security Rules for this project:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /transactions/{doc} {
      allow read, write: if true;
    }
  }
}
```

> ⚠️ **Production Recommendation:** Tighten Firestore rules to require authenticated users (Firebase Authentication) before allowing reads/writes. The current open rule is acceptable only because login is handled at the application level.

### Input Validation

- All form inputs (date, amount, description, category, type) are validated before submission
- Amounts must be positive numbers
- Date fields use native HTML `<input type="date">` to prevent malformed input
- Description and category fields are plain text — no HTML rendering is applied to user input, preventing XSS via innerHTML injection in table cells

### Data Export

- CSV export is handled entirely **client-side** using the Blob API
- No data is sent to any third-party server during export
- Exported files are generated in-memory and downloaded directly

---

## Known Limitations

| Limitation | Risk Level | Mitigation |
|---|---|---|
| Client-side password hashing | Medium | Acceptable for internal org use; upgrade to server-side bcrypt for public apps |
| Open Firestore rules | Medium | Restrict via Firebase Auth for higher security needs |
| No rate limiting on login | Low | No server involved; brute force is browser-local only |
| Single HTML file deployment | Low | No build pipeline — keep file access restricted |
| No HTTPS enforcement | Medium | Always serve over HTTPS (GitHub Pages, Firebase Hosting enforce this automatically) |

---

## Recommended Security Hardening (Production)

If you plan to deploy this beyond internal use, consider the following upgrades:

1. **Add Firebase Authentication**
   - Replace the hardcoded user list with Firebase Auth (email/password or Google Sign-In)
   - Update Firestore rules to `allow read, write: if request.auth != null;`

2. **Move to a Backend**
   - Use Node.js + Express with bcrypt for password hashing
   - Use JWT tokens for session management
   - Store credentials in MongoDB or a secure database — never in the frontend code

3. **Enable Firebase App Check**
   - Prevents unauthorized apps from accessing your Firestore database
   - Enable in Firebase Console → App Check

4. **Set Content Security Policy (CSP) headers**
   - If hosted on a server, add the following HTTP header:
   ```
   Content-Security-Policy: default-src 'self'; script-src 'self' https://www.gstatic.com https://cdnjs.cloudflare.com https://fonts.googleapis.com; style-src 'self' https://fonts.googleapis.com; connect-src 'self' https://*.firebaseio.com https://*.googleapis.com;
   ```

5. **Rotate Firebase Config if Compromised**
   - If you suspect your Firebase project has been misused, go to Firebase Console → Project Settings → Delete and recreate the web app to get a new `appId`
   - Restrict your Firebase API key to specific HTTP referrers in Google Cloud Console → APIs & Services → Credentials

---

## Dependency Security

| Dependency | Version | Source | Purpose |
|---|---|---|---|
| Firebase JS SDK | 10.12.0 | gstatic.com (Google CDN) | Firestore database |
| Chart.js | 4.4.1 | cdnjs.cloudflare.com | Charts and graphs |
| Poppins / Playfair Display | Latest | fonts.googleapis.com | Typography |

- All dependencies are loaded from **trusted CDNs** (Google, Cloudflare)
- No npm packages or local dependencies — no supply chain risk from `node_modules`
- Dependency versions are pinned to avoid unexpected breaking changes

---

## Hosting Recommendations

| Platform | HTTPS | Free | Recommended |
|---|---|---|---|
| GitHub Pages | ✅ | ✅ | ✅ Yes |
| Firebase Hosting | ✅ | ✅ | ✅ Yes |
| Netlify | ✅ | ✅ | ✅ Yes |
| Vercel | ✅ | ✅ | ✅ Yes |
| Plain HTTP server | ❌ | – | ❌ No |

> Always host over **HTTPS**. Serving over plain HTTP exposes Firebase credentials and session data to network interception.

---

## Responsible Disclosure

We follow a **responsible disclosure** policy. If you find a vulnerability:

1. Report it privately (see above)
2. Give us reasonable time to fix it before public disclosure
3. We will credit you in the release notes if you wish

Thank you for helping keep Yuvadeepti SMYM Kollam Expense Tracker secure. 🙏

---

*Last updated: April 2026*
*Maintained by: Yuvadeepti SMYM Kollam*

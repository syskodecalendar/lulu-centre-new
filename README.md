# Lulu Centre — managed website

This version retains the existing cinematic scrolling design and adds a larger logo, a leasing enquiry popup and a password-protected content manager. The website is one page with multiple sections; every section is listed in the admin editor. Text, headings, counters, image paths and descriptions, links, video source/poster, SEO description/title and decorative background images are editable. Changes go live when the admin clicks Save Changes. Adding entirely new pages or changing the layout/animations still requires development.

## Start locally

Requires Node.js 22.13+ (Node.js 24 recommended).

```bash
npm ci
cp .env.example .env
npm run setup-admin
npm start
```

The setup command asks you to choose a username and a password (at least 12 characters). There is no default password. Open http://localhost:3000 for the website and http://localhost:3000/admin/ for the admin area. You can change the password inside the admin area. Sessions expire after eight hours and end when the server restarts.

## Adjust the logo height

The home-page header logo has increased from 68 px to **80 px** on desktop and from 57 px to **66 px** on mobile.

In the admin area, choose **Logo & email settings**, change the desktop/mobile heights, and click **Save Changes**. Width follows the image proportions automatically.

For a manual source change, the variables near the bottom of `styles.css` are:

```css
:root {
  --header-logo-height: 80px;         /* Top navigation logo */
  --header-logo-height-mobile: 66px;
}
```

For example, change the header value to `90px` to increase its height. With the backend running, saved admin settings override these CSS defaults, so use the admin settings to change the active website. Header sizes are limited to 140 px in the editor; check the page after large changes.

## Leasing enquiries and email

A second floating button sits above WhatsApp on the right. It opens an accessible dialog with name, brand/company, email, phone, requirements and contact consent. The dialog supports Escape, keyboard focus and mobile scrolling.

The initial recipient is **leasing@lulucentre.com**, taken from the website's existing contact details. Change it in **Logo & email settings** if another address should receive enquiries.

Configure the server's `.env` file with your real mail provider settings:

```dotenv
SMTP_HOST=your-provider-smtp-host
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your-mail-account
SMTP_PASS=your-mail-account-or-app-password
SMTP_FROM=your-mail-account
```

For SMTP port 465, use `SMTP_SECURE=true`. Port 587 requires STARTTLS. Keep the `.env` file on the server and outside source control. Restart after changing it. The sender must be authorized by your email provider; visitor emails are used as Reply-To rather than as the sender.

**Email credentials are not included or configured in this package.** Real email delivery must be checked after supplying the real SMTP details. The backend was tested with a local SMTP server. Valid enquiries are saved in the admin inbox before email is attempted. Email status is `sent`, `pending` (SMTP missing), or `failed` (SMTP error). If mail fails, configure/fix SMTP and click **Retry email** on the enquiry. An SMTP acceptance does not guarantee placement in the recipient's inbox.

## Edit website content

1. Open `/admin/` and sign in.
2. Choose a section in the left-hand menu. The home section is named **Home & welcome**; the property, experience, floors, retail, facilities, walkthrough, access, future and contact sections follow.
3. Edit wording, counter values or links. For images/video, upload a replacement or enter a relative `assets/` / `uploads/` path or an HTTPS URL. Image captions, alt text and link descriptions are separate fields.
4. Click **Save Changes**, then open **View website**.

Uploads accept PNG, JPEG, WebP, GIF, MP4 and WebM, up to 50 MB each. SVG and executable files are not accepted. The existing walkthrough video is included. Large new videos can be placed on an HTTPS media host and linked in the editor. Edit both the video source and the walkthrough button's link when replacing the walkthrough video. Replacing the logo in the header, footer and preloader requires updating each matching image field.

Content and enquiry saves use SQLite. If two admins/tabs save the same version, the second save is rejected to protect the first change; reload and reapply the change. Uploaded media does not publish until its field is saved. Layout, styling and animations are maintained in the original code.

## Production hosting

**This is now a Node.js application. Dragging this folder to static Netlify hosting will not run the admin or enquiry backend.** Deploy it to a VPS, Node-enabled cPanel, or a Node/container host with a persistent disk. You can alternatively keep the frontend on Netlify only after adding separately hosted backend/API routing; that split deployment is not included here.

1. Upload the project and install with `npm ci --omit=dev`.
2. Copy `.env.example` to `.env` and supply your real settings.
3. Set `NODE_ENV=production`, `APP_ORIGIN=https://your-real-domain` (no trailing slash), and an absolute `DATA_DIR` on a persistent disk.
4. Run `npm run setup-admin` interactively on the server.
5. Start the app under your hosting process manager. Route the public HTTPS website to port 3000 (or your chosen `PORT`). If there is exactly one trusted reverse proxy, set `TRUST_PROXY=1` and prevent direct external access to the Node port.
6. Confirm `/health` returns `{"ok":true}`, then test admin login, content saving, file uploading and a real leasing enquiry from a phone and desktop.

HTTPS is required for production login cookies. Keep the entire `DATA_DIR` durable across restarts, deployments and container replacement. It holds `site.sqlite`, SQLite journal files, `admin.json` and uploaded media. Do not expose it as a public folder. Back up the whole directory using a consistent SQLite backup/snapshot or while the service is stopped. Do not run multiple application instances against the same SQLite file; use one instance for this implementation. Rate limits and admin sessions are held in the running process.

The included Dockerfile uses Node.js 24. Mount a persistent volume at `/app/data`, set the environment variables, run the admin setup inside the container with an interactive terminal, and put the container behind HTTPS. Give the container's `node` user write permission on the volume.

## Validation

```bash
npm test
```

Integration tests cover authentication, origin checks, CSRF enforcement, content persistence, conflict handling, HTML escaping, image upload validation, enquiry storage, duplicate submission protection, SMTP delivery and password changes. The tests use temporary accounts and a local mail server; they do not send real email.

## Included files

- `index.html`, `styles.css`, `script.js`: existing site with the updated logo sizing.
- `leasing.js`: popup interactions and enquiry submission.
- `admin/`: password-protected admin UI and content editor.
- `server.js`, `backend/content.js`: authenticated API, server-rendered content, uploads, SQLite and email.
- `scripts/setup-admin.js`: securely provision/reset the admin account.
- `.env.example`, `Dockerfile`, `package-lock.json`: installation and deployment configuration.

No hosting account, domain, real admin password or SMTP credentials are configured. Publishing and real inbox delivery require the production setup above.

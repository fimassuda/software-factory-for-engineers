# Notification providers for Cadence: Knock vs Novu vs MagicBell vs Courier

Checked: 2026-11-11. All claims link to the vendor's own docs, pricing page, GitHub repo, or npm/package source. Package versions come from `npm view` on the same date. Where a page did not answer the question, this file says "not stated" or "unverified" rather than guessing. Pricing pages change often; figures are quoted as the pages read today.

Cadence context: React Router 7 framework mode (SSR, Node), SQLite + Drizzle, shadcn/ui + Tailwind. Event-driven notifications only. Roles: Student, Instructor, Platform Admin, Team Admin. Local dev switches between seeded users with a DevUI panel. "Push" = browser Web Push now, native mobile maybe later.

## Package status (npm, 2026-11-11)

| Provider | Current packages | Deprecated / legacy |
| --- | --- | --- |
| Knock | `@knocklabs/react` 0.15.1 (MIT), `@knocklabs/client` 0.23.1 (MIT), `@knocklabs/node` 1.41.1 (Apache-2.0), `@knocklabs/cli`; mobile: `@knocklabs/react-native` 0.13.1, `@knocklabs/expo` 0.10.1 | `@knocklabs/react-notification-feed` 0.8.15 (last modified 2025-06; not flagged deprecated on npm, but the [React docs](https://docs.knock.app/in-app-ui/react/overview) point its users to a migration guide) |
| Novu | `@novu/react` 3.19.2, `@novu/js` 3.19.2, `@novu/api` 3.19.1 (server), `@novu/framework` 2.14.0; mobile: `@novu/react-native` 3.19.0 | `@novu/node` (npm: "March 20, 2025 marks the end of support... migrate to the new @novu/api"), `@novu/notification-center` (npm: "April 3, 2025 marks the end of support... migrate to the new @novu/react or @novu/js") |
| MagicBell | `@magicbell/react` 1.14.0, `magicbell-js` 1.8.0 (both modified 2026-07) | `@magicbell/core` flagged deprecated on npm. `@magicbell/magicbell-react` 11.5.1 and `@magicbell/react-headless` 5.5.1 are not flagged, but the current [React docs](https://www.magicbell.com/docs/libraries/magicbell-react) document only `@magicbell/react`; treat the older pair as legacy (inference, not stated) |
| Courier | `@trycourier/courier-react` 9.3.5 (MIT, React 18+), `@trycourier/courier-react-17`, `@trycourier/courier-js` 3.7.1, `@trycourier/courier` 10.0.0 (server, Apache-2.0); mobile: `@trycourier/courier-react-native` 6.0.9 | `@trycourier/react-inbox` and `@trycourier/react-provider` (npm: "Moved to @trycourier/courier-react"). `@trycourier/react-preferences` 7.4.8 is not flagged but belongs to the old v7 line |

---

## 1. Knock

**In-app inbox**
- `@knocklabs/react` ships `NotificationIconButton` (bell + unread/unseen badge), `NotificationFeedPopover`, and `NotificationFeed` (full-page list) ([React overview](https://docs.knock.app/in-app-ui/react/overview)).
- Restyling: import `@knocklabs/react/dist/index.css` and override `--rnf-*` CSS variables, or drop the stylesheet entirely and target the unique `rnf-` classes; `colorMode="dark"` on `KnockFeedProvider`; `renderItem` on `NotificationFeed` replaces the whole cell; `NotificationCell` accepts `children` and `avatar` ([feed docs](https://docs.knock.app/in-app-ui/react/feed)). No `classNames` prop is documented, so Tailwind utilities can only reach the default UI via global CSS against `rnf-*` classes, or via `renderItem`.
- Headless: `useKnockClient`, `useNotifications(client, feedChannelId)`, `useNotificationStore(feed)` returning `items` and `metadata.unread_count`/`unseen_count`; you must call `markAsSeen`/`markAllAsSeen` yourself ([custom UI](https://docs.knock.app/in-app-ui/react/custom-notifications-ui)). This makes a fully shadcn-built inbox straightforward.
- Real-time: websocket to Knock's managed real-time service ([feeds overview](https://docs.knock.app/in-app-ui/feeds/overview)). Statuses: unread, read, seen, archived. Feed retention "unlimited" on all plans ([pricing](https://knock.app/pricing)).

**Email**
- Bring your own ESP: SES, Cloudflare Email, MailerSend, Mailgun, Mailjet, Mailtrap, Mandrill, Postmark, Resend, SendGrid, SMTP, SparkPost; plus a "Knock (testing only)" provider and sandbox mode ([email overview](https://docs.knock.app/integrations/email/overview)).
- Unsubscribe: for workflows marked commercial, Knock "adds required unsubscribe headers" and exposes `vars.commercial_unsubscribe_url` for 1-click unsubscribe, which sets `commercial_subscribed=false` ([commercial unsubscribe](https://docs.knock.app/preferences/commercial-unsubscribe)). Exact header names (`List-Unsubscribe`/`-Post`) not named on the page. For transactional workflows (most of Cadence), the per-user preference link is the hosted preference center via signed URL ([preferences overview](https://docs.knock.app/preferences/overview)).

**Web push**
- "Knock does not have a first-party web push channel." The documented route is FCM: `firebase-messaging-sw.js` service worker, VAPID key from Firebase, `getToken`, then `knock.user.setChannelData(fcmChannelId, { tokens: [token] })` ([FCM web push tutorial](https://docs.knock.app/tutorials/sending-web-push-notifications-with-fcm)). No Knock permission-UX helper documented.
- Push providers: APNs, FCM, Expo, OneSignal, Amazon SNS ([push overview](https://docs.knock.app/integrations/push/overview)). Native later = add APNs/FCM/Expo channels; FCM web tokens and FCM Android tokens use the same provider.

**Preferences**
- Per-user `PreferenceSet` with `channel_types`, `channels`, `categories`, `workflows`; unset = `true` (opted in); environment-level default set; per-tenant defaults; preference conditions; workflow "Override recipient preferences" toggle for must-send messages ([preferences overview](https://docs.knock.app/preferences/overview)). Category x channel-type grid maps directly to "notification type on channel".
- UI: hosted no-code preference page (signed URL, `default` set only) ([preferences overview](https://docs.knock.app/preferences/overview)); no prebuilt React preference component; docs show building one with `knockClient.user.getPreferences()`/`setPreferences()` ([React preferences](https://docs.knock.app/in-app-ui/react/preferences)). A headless preferences hooks page exists at `/in-app-ui/react/headless/preferences` (listed in [React overview](https://docs.knock.app/in-app-ui/react/overview); not read in detail).

**Triggering and identity**
- Node: `@knocklabs/node`. Recipients can be pre-identified IDs or inline objects that are "guaranteed to be identified before the workflow is triggered" (upsert on trigger) ([trigger API](https://docs.knock.app/send-notifications/triggering-workflows/api)).
- Idempotency: optional `Idempotency-Key` header, 24 h window, mismatched params error ([trigger API](https://docs.knock.app/send-notifications/triggering-workflows/api)).
- Client auth: public API key + optional "enhanced security mode" requiring an RS256 JWT with `sub` = user id, signed locally with `signUserToken` from `@knocklabs/node` (no network call); "Production environments should enable enhanced security mode" because otherwise the public key "can act as any user" ([security](https://docs.knock.app/in-app-ui/security-and-authentication)). DevUI fit: sign a token per seeded user in the root loader; switching user = new `user.id` + token on the provider.

**Dev/test**
- Production + development (+ custom) environments, each with own users and versioned content; commit and promote dev -> prod ([environments](https://docs.knock.app/concepts/environments)). Knock CLI (`@knocklabs/cli`) pulls/pushes workflows as local files and promotes commits ([CLI](https://docs.knock.app/cli)). No local emulator/fake server documented.

**Cost and licence** ([pricing](https://knock.app/pricing))
- Free: $0, 10,000 messages/month, unlimited seats and environments, 30-day logs, unlimited feed retention, preferences included; not hard-capped (Knock contacts you). Knock branding not removable.
- Starter: $250/month, 50,000 messages, then $0.005/message; branding removal.
- Closed-source SaaS; client SDK monorepo is MIT ([knocklabs/javascript](https://github.com/knocklabs/javascript)). Not self-hostable. Whether each channel delivery counts as a "message" is not defined on the pricing page.

---

## 2. Novu

**In-app inbox**
- `@novu/react` `<Inbox />` composed of `Bell`, `Notifications`, `InboxContent`, `Preferences`; tabs, filters, snooze, localization ([inbox overview](https://docs.novu.co/platform/inbox/overview)). Requires React 18+ (19 supported).
- Restyling: `appearance` prop with `baseTheme` (built-in `dark`), `variables` (e.g. `colorPrimary`, `colorBackground`, `borderRadius`, mapped to `--nv-*` CSS vars), `elements` (each key takes a style object, a class string - Tailwind explicitly supported - or a callback with runtime context), `icons`, `animations` ([styling](https://docs.novu.co/platform/inbox/configuration/styling)). Render props: `renderNotification`, `renderSubject`, `renderBody`, `renderBell` ([React SDK](https://docs.novu.co/platform/sdks/react)). This is the closest of the four to "pass Tailwind classes per element", which fits shadcn tokens well.
- Headless: `useNotifications` and other hooks under `NovuProvider` ([useNotifications](https://docs.novu.co/platform/sdks/react/hooks/use-notifications)); the full hook list was not verified.
- Real-time: websocket (`socketUrl`, e.g. `wss://eu.socket.novu.co`) ([production guide](https://docs.novu.co/platform/inbox/prepare-for-production), [React SDK](https://docs.novu.co/platform/sdks/react)).

**Email**
- Bring your own ESP (SendGrid, SES, Resend, Postmark, Brevo, Mailgun, Mailjet, SparkPost, Mandrill, MailerSend, Mailtrap, SMTP, others); a "demo email integration" for testing, limits not stated ([email](https://docs.novu.co/platform/integrations/email)).
- Unsubscribe: "By default, Novu does not add any unsubscribe links to the email." You build your own tokenised unsubscribe page and call the preferences API; `List-Unsubscribe` not mentioned (provider `_passthrough` headers may work, unverified) ([email](https://docs.novu.co/platform/integrations/email)). No hosted preference page found.

**Web push**
- Push providers: FCM, Expo, APNs, OneSignal, Push Webhook, Pusher Beams, Pushpad; tokens stored per subscriber per integration via `subscribers.credentials.update` (replaces the token array, max 100 per channel) ([push](https://docs.novu.co/platform/integrations/push)). No native VAPID/Web Push provider documented.
- Browser routes: Pushpad ("supports sending notifications to all major browsers via FCM, Mozilla autopush") ([Pushpad](https://docs.novu.co/platform/integrations/push/pushpad)), FCM web, OneSignal, or Push Webhook (Novu calls your endpoint, you send with your own VAPID keys; inference from the provider name, not read in detail). No permission-UX helper.
- Native later: APNs/FCM/Expo integrations exist; `@novu/react-native` is an inbox SDK.

**Preferences**
- Layers, most specific wins per channel: workflow defaults in code -> dashboard workflow defaults -> subscriber global -> subscriber per-workflow; "critical" workflows hide from preferences and ignore subscriber choices ([preferences](https://docs.novu.co/platform/concepts/preferences)). Model is workflow x channel (no separate category concept on that page; tags/grouping not verified).
- UI: the `<Inbox />` includes a Preferences view and a standalone `Preferences` component; available in self-hosted too ([self-hosted vs cloud](https://docs.novu.co/community/self-hosted-and-novu-cloud)).

**Triggering and identity**
- Node: `@novu/api` (`novu.trigger({ workflowId, to: { subscriberId, email, ... }, payload })`); `to` accepts inline subscriber objects (max 100 recipients) ([trigger](https://docs.novu.co/api-reference/events/trigger-event)); "Novu upserts subscriber details on every trigger" ([subscribers](https://docs.novu.co/platform/concepts/subscribers)).
- Idempotency: `idempotency-key` header, but "not enabled for all organizations" (contact support); `transactionId` dedup is checked during processing, not at the API boundary ([trigger](https://docs.novu.co/api-reference/events/trigger-event)).
- Client auth: `applicationIdentifier` + `subscriberId`; HMAC mode requires `subscriberHash = HMAC-SHA256(secretKey, subscriberId)` computed server-side, toggled per environment on the Novu In-App integration; without it new subscribers can be created from the client ([production guide](https://docs.novu.co/platform/inbox/prepare-for-production), [inbox overview](https://docs.novu.co/platform/inbox/overview)). DevUI fit: compute the hash per seeded user in the loader.

**Dev/test**
- Development + Production by default; custom environments only on Team/Enterprise; "Publish changes" promotes dev -> prod ([environments](https://docs.novu.co/platform/developer/environments)). Code-first `@novu/framework` with `npx novu dev` for local workflow development; search snippets of Novu docs say it now opens the dashboard in a Local mode replacing the old Local Studio, but the pages disagree and this was not read directly (unverified; see [Studio docs](https://docs.novu.co/framework/studio)). Self-hosting is also a full local option (MongoDB, Redis, S3-compatible storage) ([self-hosting](https://docs.novu.co/community/self-hosting-novu/overview)).

**Cost and licence**
- Free: $0, 10,000 workflow runs/month, 5 workflows, 2 environments, 3 team members, 24 h activity-feed (log) retention ([pricing](https://novu.co/pricing)); inbox feed retention 30 days ([limits](https://docs.novu.co/platform/developer/limits)). Branding removal disabled on free ([production guide](https://docs.novu.co/platform/inbox/prepare-for-production)).
- Pro: from $30/month, 30,000+ runs, 7-day log retention, branding removed; overage $1.20 per 1K runs. Team: from $250/month, 250,000+ runs ([pricing](https://novu.co/pricing)).
- Open core: core MIT, `/enterprise` folder under commercial licence ([novuhq/novu](https://github.com/novuhq/novu)). Self-hosted lacks snooze, custom environments, >2 environments, branding removal ([self-hosted vs cloud](https://docs.novu.co/community/self-hosted-and-novu-cloud)). The pricing page frames supported self-hosting as Enterprise. Note: the 5-workflow cap on free is tight if every event type (refund decided, lesson published, ...) is its own workflow.

---

## 3. MagicBell

**In-app inbox**
- `@magicbell/react`: `ContextProvider` (prop `token` = user JWT), `FloatingInbox`, `Inbox`, `InlineNotification`, `ToastContainer`, `UserPreferences`, `WebPushButton`, Slack buttons ([React docs](https://www.magicbell.com/docs/libraries/magicbell-react)).
- Restyling: per-component stylesheets, overridable class names, `--magicbell-*` CSS custom properties, `className` on several components, `unstyled` on buttons; component slots `ItemComponent`, `ButtonComponent`, `InboxComponent` ([React docs](https://www.magicbell.com/docs/libraries/magicbell-react)).
- Headless: no hooks documented for `@magicbell/react`; `magicbell-js` is the API client ([JS client](https://www.magicbell.com/docs/libraries/magicbell-js)). Legacy `@magicbell/react-headless` has hooks but belongs to the older line.
- Real-time: docs say "persistent connection" without naming it ([in-app](https://www.magicbell.com/docs/channels/in_app)); the `@magicbell/react` 1.14.0 package depends on `reconnecting-websocket` (package.json), and the API exposes an Ably `connection_id` ([API reference](https://www.magicbell.com/docs/api/reference)). So: websocket, transport details unverified.

**Email**
- Providers: SendGrid, Mailgun, Amazon SES, SMTP, Ping Email ([email channel](https://www.magicbell.com/docs/channels/email)). No Resend or Postmark integration listed (SMTP could cover them). Templates and unsubscribe/`List-Unsubscribe` not documented on that page.

**Web push (strongest of the four)**
- First-party Web Push channel using your own VAPID `public_key`/`private_key`; tokens are standard `PushSubscription` `endpoint` + `keys.p256dh`/`keys.auth`, managed at `/channels/web_push/tokens` ([web push channel](https://www.magicbell.com/docs/channels/web_push), [API reference](https://www.magicbell.com/docs/api/reference)).
- Client: `WebPushButton` handles subscribe flow; service worker must `importScripts('https://assets.magicbell.io/web-push-notifications/sw.js')`, default path `/sw.js`; props `unsupportedLabel` and `requestInstallLabel` ("needs to be installed as a PWA (iOS requirement)") ([React docs](https://www.magicbell.com/docs/libraries/magicbell-react)).
- Native later: APNs, FCM, Expo under `/channels/mobile_push/...` ([API reference](https://www.magicbell.com/docs/api/reference)). No `@magicbell/react-native` on npm.

**Preferences**
- Category x channel: `GET/PUT /channels/user_preferences`, categories each with a list of `{ channel, enabled }` ([API reference](https://www.magicbell.com/docs/api/reference)); `UserPreferences` React component "lets users toggle notification channels per category" ([React docs](https://www.magicbell.com/docs/libraries/magicbell-react)). Defaults for unset preferences and project-level defaults: not documented (unverified). No hosted preference page found.

**Triggering and identity**
- Node: `magicbell-js` (project client). Sends are "broadcasts" with inline `recipients` (`external_id`, `email`, names, `custom_attributes`) and `category`/`topic` ([API reference](https://www.magicbell.com/docs/api/reference)); users are upserted by `external_id` (immutable once set) or email; external id wins ([user primitive](https://www.magicbell.com/docs/primitive/user)). There is no workflow engine in the Knock/Novu sense; routing per channel is via category + preferences.
- Idempotency key: not found in the docs read (unverified).
- Client auth: always a user JWT, HS256, signed locally with the project secret, claims `user_external_id`/`user_email` + `api_key` ([user JWT](https://www.magicbell.com/docs/api/authentication/user)). DevUI fit: sign per seeded user in the loader, no network call.

**Dev/test**
- No environments concept; docs say to create separate projects for staging/production ([quick start](https://www.magicbell.com/docs/quick-start)). Free plan includes 1 project, so dev/prod separation needs Startup ($99 per extra project) ([pricing](https://www.magicbell.com/pricing)). No test mode or local emulator found. A web push tester exists ([web push channel](https://www.magicbell.com/docs/channels/web_push)).

**Cost and licence** ([pricing](https://www.magicbell.com/pricing))
- Builder: $0, 1,000 deliveries/month, 1 project, unlimited team members and users. Deliveries count per channel per user (in-app + email to one user = 2).
- Startup: $249/month, 50,000 deliveries, 5 projects, $0.0025/extra delivery.
- Proprietary SaaS. Even the JS SDK repo uses a custom "LICENSE AGREEMENT" (limited, non-transferable; modifications assigned to MagicBell) ([magicbell-js LICENSE](https://github.com/magicbell/magicbell-js/blob/main/LICENSE)). Not self-hostable.

---

## 4. Courier

**In-app inbox**
- `@trycourier/courier-react` (React 18+): `CourierInbox`, `CourierInboxPopupMenu`, `CourierToast`, `CourierPreferences`; `useCourier()` hook ([React SDK](https://www.courier.com/docs/sdk-libraries/courier-react-web)). Underneath are Web Components (`@trycourier/courier-ui-inbox`) from the MIT [courier-web](https://github.com/trycourier/courier-web) monorepo.
- Restyling: `CourierInboxTheme` object (colors, fonts, radius, shadows) with `lightTheme`/`darkTheme` and `mode: "light" | "dark" | "system"` ([customize](https://www.courier.com/docs/in-app/customize-the-inbox.md)); slot renderers `renderHeader`, `renderListItem`, `renderEmptyState`, `renderLoadingState`, `renderErrorState`, `renderPaginationItem`, `renderMenuButton` ([custom inbox](https://www.courier.com/docs/in-app/build-a-custom-inbox.md)). No CSS-variable or className API documented.
- Package-source caveat (verified in `@trycourier/courier-react` 9.3.5 `dist/index.mjs`): each renderer's JSX is mounted via `createRoot()` into a detached container (`reactNodeToHTMLElement`), i.e. a separate React root. React context from the app (router, theme, i18n providers) does not flow into custom list items. `courier-ui-inbox` 2.7.4 also uses `attachShadow({ mode: "open" })` for some sub-elements (unread badge, option menu), so global Tailwind does not reach those. Restyling to shadcn is therefore theme-object based, or headless.
- Headless: `useCourier().inbox` with `registerFeeds`, `listenForUpdates`, `load`, `readMessage`, `archiveMessage`, `fetchNextPageOfMessages`, `totalUnreadCount` ([custom inbox](https://www.courier.com/docs/in-app/build-a-custom-inbox.md)).
- Real-time: `listenForUpdates()`; the `@trycourier/courier-js` source opens `new WebSocket` to `wss://realtime.courier.io` (package source). Messages persist "until the user reads or archives them"; no retention window stated ([inbox](https://www.courier.com/docs/in-app/inbox.md)).

**Email**
- Bring your own ESP: Resend, SES, Postmark, SendGrid, SMTP, Mailgun, Mandrill, SparkPost, Mailjet, MailerSend, and others ([email](https://www.courier.com/docs/external-integrations/email/intro)). Test environment ships a testing email provider that only delivers to the signup address ([workspaces/environments](https://www.courier.com/docs/workspaces/overview#environments-and-api-keys)).
- Unsubscribe: hosted per-user preference page, commonly linked from email footers ([preference center](https://www.courier.com/docs/in-app/preference-center.md)). `List-Unsubscribe` header behaviour not documented in the pages read (unverified).

**Web push**
- Push providers: APNs, FCM, Expo, OneSignal, Airship, SNS, Pusher Beams, MagicBell, and others; no native VAPID Web Push documented ([push overview](https://www.courier.com/docs/external-integrations/push/intro-to-push)). The FCM provider page covers Android/iOS only and does not mention browsers ([FCM](https://www.courier.com/docs/platform/channels/push/firebase-fcm/)). A Courier guide routes browser push through OneSignal ([Node push guide](https://www.courier.com/guides/nodejs-send-push-notifications)). Web push via FCM web tokens is plausible but not documented by Courier (unverified).
- Native later: strongest mobile story (native iOS/Android/React Native/Flutter SDKs that register APNs/FCM tokens) ([push overview](https://www.courier.com/docs/external-integrations/push/intro-to-push)).

**Preferences**
- Subscription topics with default `OPTED_IN` / `OPTED_OUT` / `REQUIRED`; tenant defaults; per-topic channel choice (`custom_routing`) when channel selection is enabled ([preferences](https://www.courier.com/docs/platform/preferences/preferences-overview)). Topic x channel matches Cadence's need.
- UI: hosted per-user preference page plus embeddable `CourierPreferences` (React, Vue, Angular, Web Components, mobile), "fully themeable" ([preference center](https://www.courier.com/docs/in-app/preference-center.md)).

**Triggering and identity**
- Node: `@trycourier/courier` v7+ (`client.send.message({ message: { to, template, data } })`); `to` accepts `user_id`, `email`, `data`, `tenant_id`, `preferences` inline; whether inline fields update the stored profile is not stated ([send API](https://www.courier.com/docs/api-reference/send/send-a-message)). Sends to a bare email skip preferences ([email](https://www.courier.com/docs/external-integrations/email/intro)), so always send by `user_id`.
- Idempotency: `Idempotency-Key` on `/send`, default 25 h, configurable up to 1 year via `x-idempotency-expiration` ([send API](https://www.courier.com/docs/api-reference/send/send-a-message)).
- Client auth: always a JWT minted by your backend via `POST /auth/issue-token` (scope `user_id:<id> inbox:read:messages inbox:write:events`, set `expires_in` or it never expires); client calls `signIn({ userId, jwt })` ([authentication](https://www.courier.com/docs/platform/inbox/authentication)). No local-signing option documented, so every token is a network round-trip (cache per user). DevUI fit: `signOut` + `signIn` on user switch.

**Dev/test**
- Each workspace has Production and Test environments with separate templates, integrations, keys, logs; draft keys render unpublished templates; mock keys (no downstream delivery) are Enterprise-only ([workspaces/environments](https://www.courier.com/docs/workspaces/overview#environments-and-api-keys)). CLI and MCP server exist ([pricing](https://www.courier.com/pricing)). No local emulator.

**Cost and licence** ([pricing](https://www.courier.com/pricing))
- Developer: free, 10,000 sends/month. Business: $0.005/send pay-as-you-go. "No per-seat or channel add-on fees."
- Not stated on the page: what counts as a send, overage behaviour on free, log retention, whether Inbox/Preferences are on the free plan (unverified).
- Proprietary SaaS; client SDKs MIT, server SDK Apache-2.0 (npm). Not self-hostable.

---

## Comparison table

| Criterion | Knock | Novu | MagicBell | Courier |
| --- | --- | --- | --- | --- |
| React inbox components | Bell button, popover, full-page feed (`@knocklabs/react`) | `<Inbox />` + `Bell`, `Notifications`, `InboxContent`, `Preferences` (`@novu/react`) | `FloatingInbox`, `Inbox`, toasts (`@magicbell/react`) | `CourierInbox`, popup menu, toasts (`@trycourier/courier-react`, Web Components underneath) |
| Restyling | `--rnf-*` CSS vars, replace CSS via `rnf-` classes, `renderItem` | `appearance`: vars, per-element Tailwind class strings or callbacks, render props | `--magicbell-*` vars, `className`, slot components | Theme object, light/dark/system, slot renderers (separate React roots; partial shadow DOM) |
| Headless | Yes, hooks (`useNotifications`, `useNotificationStore`) | Yes, hooks (`useNotifications`, ...) | Only via `magicbell-js` (no documented hooks in current React SDK) | Yes, `useCourier().inbox` |
| Real-time | Websocket | Websocket | Websocket (package dep; docs vague) | Websocket (`wss://realtime.courier.io`, package source) |
| Email | BYO ESP incl. Resend, Postmark, SES; test provider | BYO ESP incl. Resend, Postmark, SES; demo provider | SendGrid, Mailgun, SES, SMTP, Ping Email (no Resend/Postmark integration) | BYO ESP incl. Resend, Postmark, SES; test provider in Test env |
| Unsubscribe | Commercial 1-click + headers; hosted pref page link | Build your own; none added by default | Not documented | Hosted pref page link; headers unverified |
| Web push | Not first-party; via FCM web (or OneSignal) | Not first-party; via Pushpad, FCM, OneSignal, Push Webhook | First-party VAPID Web Push + `WebPushButton` + SW import | Not documented first-party; via OneSignal (FCM web unverified) |
| Native push later | APNs, FCM, Expo, OneSignal, SNS; RN/Expo SDKs | APNs, FCM, Expo, OneSignal; RN inbox SDK | APNs, FCM, Expo | APNs, FCM, Expo + native mobile SDKs |
| Preference model | Channel type / channel / category / workflow; env + tenant defaults; conditions | Workflow x channel; workflow defaults (code or dashboard); critical flag | Category x channel; defaults undocumented | Topic x channel; OPTED_IN/OUT/REQUIRED; tenant defaults |
| Preference UI | Hosted page; build React UI yourself (client methods/hooks) | Built into Inbox + standalone component | `UserPreferences` component | Hosted page + `CourierPreferences` component |
| Server SDK / identity | `@knocklabs/node`; inline identify on trigger | `@novu/api`; upsert on every trigger | `magicbell-js`; upsert by `external_id` in broadcast recipients | `@trycourier/courier`; inline `to`, profile update semantics unstated |
| Client auth | Optional RS256 user token, signed locally | Optional HMAC `subscriberHash`, computed locally | Mandatory HS256 user JWT, signed locally | Mandatory JWT via `POST /auth/issue-token` (network call) |
| Idempotency | `Idempotency-Key`, 24 h | `idempotency-key`, may need enabling by support | Not found | `Idempotency-Key`, 25 h default, up to 1 y |
| Environments / test | Dev + prod + custom (all plans), CLI, commit/promote | Dev + prod on free; custom on Team+; `npx novu dev`; self-host | Separate projects (1 on free) | Prod + Test per workspace; mock keys Enterprise |
| Free tier | 10k messages/mo, unlimited seats/envs, 30-day logs, unlimited feed retention | 10k runs/mo, 5 workflows, 3 seats, 2 envs, 30-day inbox, 1-day logs | 1,000 deliveries/mo (per channel), 1 project | 10k sends/mo; retention/feature gating not stated |
| Next tier | Starter $250/mo, 50k msgs, $0.005/msg | Pro from $30/mo, 30k runs, $1.20/1k | Startup $249/mo, 50k deliveries | Business $0.005/send PAYG |
| Open source | No (SDKs MIT) | Open core: MIT + commercial `/enterprise`; self-hostable | No (SDK under proprietary licence) | No (SDKs MIT) |

## Web push summary

- **Who supports it:** Only MagicBell has a first-party, standards-based Web Push (VAPID) channel and a React subscribe helper (`WebPushButton`, with built-in "unsupported" and "install as PWA" states) ([React docs](https://www.magicbell.com/docs/libraries/magicbell-react), [web push channel](https://www.magicbell.com/docs/channels/web_push)). Knock states it has no first-party web push channel and documents FCM web ([tutorial](https://docs.knock.app/tutorials/sending-web-push-notifications-with-fcm)). Novu offers Pushpad, FCM, OneSignal, or Push Webhook ([push](https://docs.novu.co/platform/integrations/push)). Courier documents no browser path beyond OneSignal ([guide](https://www.courier.com/guides/nodejs-send-push-notifications)).
- **What is needed in Cadence regardless of provider:** a web app manifest (`display: standalone`), a root-scoped service worker (`/sw.js` or `/firebase-messaging-sw.js` served from `public/`), VAPID keys (your own for MagicBell or Push Webhook; Firebase-issued for FCM), a permission request triggered from a user gesture, and storing the resulting subscription/token against the user in the provider. With FCM routes Cadence also takes the Firebase JS SDK and a Firebase project.
- **iOS limits:** Web Push works only for Home Screen web apps on iOS/iPadOS 16.4+, with a manifest `display` of `standalone` or `fullscreen`, and permission must be requested "in response to direct user interaction"; it is the same W3C Web Push and needs no Apple Developer Program membership ([WebKit](https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/)). Students on iPhone in Safari tabs will not get push; Cadence needs an "Add to Home Screen" prompt. EU availability on iOS 17.4+: conflicting secondary statements; Apple's DMA pages read today did not address Home Screen web apps, so this is unverified.
- **Path to native:** Web Push subscriptions do not carry over to native apps; native needs APNs/FCM device tokens registered from the app. All four model this as extra channels/tokens on the same user, so preferences and workflows carry over. FCM-based web push (Knock, Novu, Courier) shares one provider integration with Android later; MagicBell keeps web push and mobile push as separate channels under one user. Courier has the most complete native SDKs; Knock and Novu have React Native/Expo packages.

## Recommended shortlist

**1. Knock (primary).**
- Preference model is the best fit: category x channel-type grid, environment defaults, per-tenant defaults (useful later for Team Admin), and an override switch for must-send messages ([preferences](https://docs.knock.app/preferences/overview)).
- Headless hooks make a pixel-matched shadcn/Tailwind bell and inbox page realistic without fighting a component's CSS; the prebuilt UI is a fallback ([custom UI](https://docs.knock.app/in-app-ui/react/custom-notifications-ui)).
- Identity fits the DevUI flow: inline identify on trigger, user tokens signed locally in the RR7 loader (no network hop per user switch), `Idempotency-Key` on triggers ([trigger API](https://docs.knock.app/send-notifications/triggering-workflows/api), [security](https://docs.knock.app/in-app-ui/security-and-authentication)).
- Free tier covers dev and demo: 10k messages, unlimited environments and seats, dev/prod promotion, CLI-managed workflows in git ([pricing](https://knock.app/pricing), [CLI](https://docs.knock.app/cli)). Resend/Postmark/SES all supported.
- Cost: web push via FCM (Firebase project + SW), and the preference center UI must be built by hand.

**2. Novu (alternative, if open source / self-hosting or turnkey UI matters more).**
- Only open-source option (MIT core), self-hostable, which also allows a fully local dev stack ([repo](https://github.com/novuhq/novu)).
- Best out-of-box restyling for shadcn: per-element Tailwind class strings, CSS variables, render props, plus a built-in preferences UI inside the inbox ([styling](https://docs.novu.co/platform/inbox/configuration/styling)).
- HMAC identity is local and simple; upsert on trigger ([production guide](https://docs.novu.co/platform/inbox/prepare-for-production)).
- Weaker on: 5-workflow free cap, idempotency may need support to enable, no default unsubscribe handling, web push only via third parties.

Not shortlisted: **MagicBell** has the best web push, but only 1,000 deliveries/month and 1 project on free, no environments, no documented preference defaults, limited ESP list, and a proprietary SDK licence. **Courier** has strong preferences and native mobile, but its React renderers mount in separate React roots with partial shadow DOM (harder to make look like shadcn), client tokens need an API call per user, browser push is undocumented, and the free plan's feature gating is not stated.

## Open risks

- **Web push on Knock/Novu depends on FCM (or Pushpad/OneSignal)**: an extra vendor, Firebase SDK on the client, and Firebase-managed VAPID keys. If first-party web push becomes the priority, MagicBell or a self-sent Web Push (e.g. Novu Push Webhook) is the alternative; validate Push Webhook in a spike.
- **Usage counting**: neither Knock nor Courier define on the pricing page whether each channel delivery counts as one message/send; MagicBell explicitly counts per channel. Verify before relying on free-tier headroom for demos.
- **Knock free plan branding** cannot be removed; acceptable for dev/demo, not for launch without Starter ($250/month).
- **Novu free plan**: 5 workflows and 1-day log retention make debugging and event coverage tight; Pro ($30/month) lifts some of this.
- **Security in dev**: Knock enhanced security and Novu HMAC are opt-in; without them any client can read any user's feed with the public key. Turn them on in every environment from day one so the DevUI path matches production.
- **Hosted preference center vs in-app**: Knock's hosted page only covers the `default` preference set, not per-tenant sets ([preferences](https://docs.knock.app/preferences/overview)); a Team Admin per-tenant UI would be custom.
- **iOS**: web push reaches only installed Home Screen web apps on iOS 16.4+; EU status unverified from Apple primary sources.
- **Unverified items**: Knock headless preference hook names; Novu full hook list and tag grouping; MagicBell idempotency, preference defaults, retention, test mode; Courier `List-Unsubscribe`, FCM web support, free-plan gating of Inbox/Preferences, and log retention.

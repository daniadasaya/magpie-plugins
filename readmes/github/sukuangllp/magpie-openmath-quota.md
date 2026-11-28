# Open Math for Magpie

An unofficial Magpie plugin for Open Math model discovery, inference and native weekly quota cards. Use one or more Open Math accounts through the same plugin.

## Install

```sh
magpie plugin add github:sukuangllp/magpie-openmath-quota
magpie plugin login openmath-quota
```

For each account, enter:

- **Model API key** — for `GET /v1/models` and inference.
- **Platform API key** — optional; for official weekly usage, separate from the Model API key.
- **Team UUID** — leave blank to detect a single team from the Platform API key. If the key lists multiple teams, enter the UUID of the team whose quota you want to see.

Sign in again from Magpie to add another account. Each account has its own two keys, model set and quota card.
To rename an account, open **Providers → Open Math** and click its name in the **Accounts** list.

Choose the discovered `openmath-quota/<model>` models in Magpie. Existing custom `openmath` provider settings are not required. You can then use the models through your usual Magpie-connected agents.

```sh
magpie models
magpie quota refresh openmath-quota
```

A GitHub plugin can synchronize and install on your other computers. Sign in on each computer, or explicitly enable Magpie's Sign-ins sync if you want it to carry credentials.

## Behavior and privacy

- Uses `https://os.openmathmodel.org/v1/models` with the account's Model API key.
- Uses Chat Completions through Magpie's `@ai-sdk/openai-compatible` adapter, with the Model API key.
- Uses `POST https://os.openmathmodel.org/v1/org_model_usage` with the Platform API key and `{ "team": "<your-team-id>" }`.
- Shows only the official weekly USD quota: used/remaining, native progress colors and reset time at minute precision. The 5-hour window is not shown.
- Carries model context windows, input modalities, attachments and configurable reasoning efforts returned by Open Math.
- Stores no key, token, proxy address or team UUID in this public repository. Credentials are supplied by each user and saved by Magpie's sign-in manager.
- No other Open Math provider is modified during installation.

## Tests

Node.js 22 or newer is required. No third-party test dependencies are needed:

```sh
npm test
```

Published under the MIT license. For changes to the official plugin catalog, follow Magpie's separate npm/community publication process; this repository's `magpie-plugin` topic makes it discoverable through Magpie's GitHub plugin search.

# orca-traditional-chinese

Traditional Chinese (zh-TW) language pack for [Orca](https://github.com/stablyai/orca).

## Status

Full coverage of Orca's translatable UI catalog:

- **13,581 / 13,581** translatable strings translated (100%)
- Settings, sidebars, editor, terminal, GitHub/GitLab/Linear/Jira integrations,
  onboarding, mobile companion app, dashboard, system tray, and application menu
- Human-reviewed Taiwan Traditional Chinese — not a mechanical conversion of
  the Simplified Chinese catalog
- Resynced against `stablyai/orca` `main` (`f737f34`, 2026-09-02): 1,082 keys
  added for UI that landed since the last sync, and 30 keys dropped for UI that
  upstream removed
- 2 further keys are inline CSS animation styles, not translatable prose
  (see [Known limitations](#known-limitations)); they are excluded and fall back
  to English automatically with no user-facing impact

## Installation

Orca discovers language packs through its plugin system (git marketplaces):

1. Settings → Plugins → **Add marketplace source** with:
   - Git URL: `https://github.com/a-lang/orca-traditional-chinese.git`
   - Git ref: `main`
2. The **Traditional Chinese Language Pack** listing appears in the
   marketplace browser — install it.
3. Select **繁體中文（台灣）** from Settings → Appearance → Language.

The marketplace index (`orca-marketplace.json`) lists the pack at
`a-lang.traditional-chinese`, pinned to the `main` ref.

## How this pack was built

The English source (`en.json` from `stablyai/orca`,
`src/renderer/src/i18n/locales/en.json`) was extracted, split into batches by
UI namespace, and translated with an LLM-assisted, multi-pass process:

1. Flatten the English catalog into `path -> string` pairs, excluding keys
   under the plugin-protected namespace (`auto.components.settings.plugin*` —
   matched case-insensitively, so `PluginConsentDialog.*` and friends are
   covered too, minus the exact paths exempted in
   `plugin-translatable-chrome.ts`) — 180 protected keys stay in English by
   design.
2. Translate in batches grouped by component/namespace, with a shared
   zh-TW glossary (儲存庫 / 終端 / 外掛程式 / 預設 / 設定 …), placeholders
   preserved verbatim, and brand names / code literals kept untranslated
   (Claude, Codex, GitHub, Git, Markdown, `orca status`, etc.).
3. Terminology pass with the l10n-tw glossary tooling: banned simplified-Chinese
   terms (软件 / 服务器 / 设置 / 加载 …) are auto-replaced, and scan-only terms
   are adjudicated per namespace.
4. Cross-batch consistency pass: reconciled terminology that drifted between
   independently translated batches.
5. Validated against the same rules Orca's plugin loader enforces at runtime
   (`parsePluginLanguagePackArtifact`): max 20,000 entries, max depth 16, no
   dangerous/unsafe keys, no protected paths, no string over 8,192 chars, and
   `{{placeholder}}` sets preserved per key.

Every translated string was reviewed against the English source; no string
was left identical to English except where that is the correct choice (brand
names, technical terms, code/CLI literals, keyboard shortcuts).

## Known limitations

A few upstream i18n design constraints can't be fixed from the translation
side alone — flagging them here for visibility:

- **Positional placeholder pluralization**: some English strings compose a
  sentence from an English verb/noun injected via a positional placeholder
  (e.g. `"{{value0}} PR #{{value1}}?"` where `{{value0}}` is `close`/`reopen`
  in English). Since Chinese grammar differs from English, these preserve the
  placeholders and produce the closest natural phrasing possible.
- **180 plugin-protected keys** under `auto.components.settings.plugin*`
  (plugin trust, safety, and consent copy) are intentionally left in English —
  plugin language packs cannot override them by design.
- **2 inline CSS animation styles** (`review.animated.visual.*.styles.*`)
  exceed the 8,192-char per-string artifact limit and are not translatable
  prose; they are excluded from the pack and fall back to English.
- The language selector shows **zh-TW — a-lang.traditional-chinese** because
  upstream renders plugin packs as `{locale} — {pluginKey}`; a native display
  name (e.g. 中文（繁體）) is requested upstream in
  [#13031](https://github.com/stablyai/orca/issues/13031), with an open pull
  request in [#13140](https://github.com/stablyai/orca/pull/13140).

  **Do not add `displayName` to `orca-plugin.json` until that PR ships.**
  `pluginLanguagePackContributionSchema` is currently `.strict()`, so an
  unrecognised key fails manifest validation for the whole plugin — the pack
  would stop loading entirely rather than degrade to the old picker label.

## Contributing

Corrections and improvements welcome — please open a PR against
`locales/zh-TW.json`, keeping the existing key structure and the style
conventions above.

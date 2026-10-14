
---

## ❤️ Support This Project

If you enjoy Master Notify Dashboard Edition and would like to support its development, consider buying me a coffee! ☕

Your support helps me maintain the project, fix bugs, and develop new features.

[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support_My_Work-FF5E5B?style=for-the-badge&logo=kofi&logoColor=white)](https://ko-fi.com/xsas1337)

Every contribution is appreciated. Thank you for your support! ❤️

# 🔔 Master Notify – Dashboard Edition

**One central notification script for Home Assistant, with optional dashboard history.**

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Script%20Blueprint-41BDF5?logo=homeassistant&logoColor=white)](https://www.home-assistant.io/)
[![GitHub](https://img.shields.io/badge/GitHub-Source-181717?logo=github)](https://github.com/xsasx/Master-Notify-Dashboard-Edition)
[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support%20this%20fork-FF5E5B?logo=kofi&logoColor=white)](https://ko-fi.com/xsas1337)

This is a community fork of [Master Notify by FourtyNos](https://github.com/FourtyNos/master_notify). It keeps the centralized notification approach and adds **optional history of the two most recent notifications**, useful for Home Assistant dashboards.

> **Status:** The fork's blueprint was imported successfully into Home Assistant, and mobile push delivery plus writing the latest-notification helper were tested. The previous-notification helper, Alexa, TTS, critical alerts, and volume restoration have not yet been independently verified for this release.

## ✨ Features

- 📱 Send mobile push notifications to configurable **Admin** and **Family** device groups.
- 🔊 Send announcements to configured **Alexa notify entities**.
- 🗣️ Use a configured **TTS provider** with media players.
- 🚨 Request critical push notifications (subject to device/platform support).
- 🎚️ Configure normal and critical announcement volume, with an attempt to restore previous media-player volumes.
- 🕘 **Optional dashboard history:** save the latest and previous notification in two Home Assistant `input_text` helpers.
- 🧩 Use the same central script from many automations.

## 📥 Install the blueprint

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fxsasx%2FMaster-Notify-Dashboard-Edition%2Fblob%2Fmain%2FMaster_Notify.yaml)

1. Click **Import Blueprint** above, or open **Settings → Automations & scenes → Blueprints → Import Blueprint** in Home Assistant.
2. Paste this URL if importing manually:

   `https://github.com/xsasx/Master-Notify-Dashboard-Edition/blob/main/Master_Notify.yaml`

3. Confirm the import.
4. Open **Settings → Automations & scenes → Scripts**, create a script **from the imported blueprint**, and select your devices and optional TTS settings.
5. Save the script. Note its entity ID (for example, `script.master_notify`). Your actual ID may differ.

**Requirements:** Home Assistant **2025.12.0 or newer**, as specified by the blueprint. Mobile notifications require the Home Assistant Companion App and correctly registered mobile devices. Alexa and TTS features require the corresponding working integrations and entities.

## ⚙️ Blueprint configuration

| Option | Purpose |
|---|---|
| **Enable dashboard history** | Optional; disabled by default. Requires the two helpers below. |
| **Admin User Devices** | Home Assistant mobile-app devices receiving Admin notifications. |
| **Family User Devices** | Home Assistant mobile-app devices receiving Family notifications. |
| **Normal volume** | Default media-player volume for non-critical announcements (0.0–1.0). |
| **Critical volume** | Media-player volume for critical announcements (0.0–1.0). |
| **alexa devices** | Alexa notification entities used by your Alexa integration. |
| **tts provider** | TTS entity such as `tts.google_translate`, if available. |
| **Google and other TTS Devices** | Media players for TTS playback. |

You can use mobile push without configuring Alexa or TTS. The dashboard history is **not required** for ordinary notifications.

## 🕘 Optional dashboard history

If you want the latest two notifications displayed in Home Assistant, create **two Text helpers** first:

**Settings → Devices & services → Helpers → Create helper → Text**

| Suggested helper name | Required entity ID | Maximum length |
|---|---|---|
| Master Notify – Latest Notification | `input_text.master_notify_letzte_meldung` | 255 |
| Master Notify – Previous Notification | `input_text.master_notify_vorletzte_meldung` | 255 |

The entity IDs must match **exactly**. If Home Assistant creates different IDs, edit the helper entity IDs before enabling history. Set the maximum length to **255 characters** for each helper.

Alternatively, create them in YAML (only if you manage `input_text` in YAML):

```yaml
input_text:
  master_notify_letzte_meldung:
    name: Master Notify – Latest Notification
    max: 255
  master_notify_vorletzte_meldung:
    name: Master Notify – Previous Notification
    max: 255
```

**Do not add a second `input_text:` root key if one already exists**; merge the entries into your existing configuration instead. Restart Home Assistant if you create the helpers through YAML.

Then edit the script created from the blueprint and enable **Enable dashboard history**.

When the script runs, it copies the former latest notification to the previous helper and writes a new entry with date, time, title, and message to the latest helper. Each entry is truncated to 255 characters. The two helpers store the **last two calls**, not a permanent event archive. The blueprint uses `continue_on_error` for the history writes; if helpers are missing, notifications may still work, but history will not be recorded correctly.

### Example dashboard card

Add this **Markdown card** to any Home Assistant dashboard:

```yaml
type: markdown
title: 🔔 Master Notify History
content: >-
  **Latest notification**

  {{ states('input_text.master_notify_letzte_meldung') }}

  **Previous notification**

  {{ states('input_text.master_notify_vorletzte_meldung') }}
```

## 🚀 Send a notification from an automation

Once your script is saved, use its actual entity ID. For example:

```yaml
action: script.master_notify
data:
  title: "🔔 Home Assistant"
  message: "The notification blueprint is working!"
  group_admins_enable: true
  group_family_enable: false
  alexa_enabled: false
  google_enabled: false
  critical_enabled: false
```

For a family notification, set `group_family_enable: true`. For Alexa or TTS announcements, configure the appropriate devices in the script and set `alexa_enabled` or `google_enabled` to `true` for the call.

### Critical notification example

```yaml
action: script.master_notify
data:
  title: "🚨 Important alert"
  message: "Please check Home Assistant."
  group_admins_enable: true
  group_family_enable: false
  alexa_enabled: false
  google_enabled: false
  critical_enabled: true
```

Critical push behavior depends on the mobile platform, its permissions, and notification configuration. Test on your own devices before relying on it for emergencies.

## 🧪 Testing

1. Save a script from the blueprint with at least one Admin mobile device.
2. In **Developer tools → Actions**, run the first example, using your script's actual entity ID.
3. Confirm that the mobile push arrives.
4. If history is enabled, check the two `input_text` entities in **Developer tools → States**.
5. Run a second distinct notification and verify that the first moves to the previous-notification helper.
6. Test Alexa, TTS, critical delivery, and volume restoration separately if you plan to use them.

## 🛠️ Troubleshooting

**No push received:** Check that the selected mobile device is registered, notifications are permitted, and the Admin/Family flag for the call is enabled.

**History is empty or `unknown`:** Check that both helpers exist with the exact entity IDs above, and that **Enable dashboard history** is enabled in the script's blueprint configuration.

**Wrong script name:** Replace `script.master_notify` in the examples with the entity ID of the script you actually created.

**No Alexa/TTS announcement:** Verify the integration, notify entities, TTS provider, media players, and corresponding enabled flags. These integrations may have platform-specific requirements.

**Updating:** Replace or re-import the blueprint from this repository and check the existing script's blueprint configuration. Back up your Home Assistant configuration before making changes.

## 🙏 Credits and support

- **Original project:** [FourtyNos/master_notify](https://github.com/FourtyNos/master_notify) — original blueprint and centralized notification logic.
- **Dashboard Edition fork:** [xsasx/Master-Notify-Dashboard-Edition](https://github.com/xsasx/Master-Notify-Dashboard-Edition) — optional dashboard history and fork maintenance.
- ☕ [Support this fork on Ko-fi](https://ko-fi.com/xsas1337).

This fork is community-maintained and is not an official Home Assistant integration. Please retain attribution to the original project when redistributing modified versions.

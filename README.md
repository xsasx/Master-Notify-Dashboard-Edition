
---

## ❤️ Support This Project

If you enjoy Master Notify Dashboard Edition and would like to support its development, consider buying me a coffee! ☕

Your support helps me maintain the project, fix bugs, and develop new features.

[![Ko-fi](https://img.shields.io/badge/Ko--fi-Support_My_Work-FF5E5B?style=for-the-badge&logo=kofi&logoColor=white)](https://ko-fi.com/xsas1337)

Every contribution is appreciated. Thank you for your support! ❤️


# 🔔 Master Notify – Dashboard Edition

An enhanced Home Assistant notification blueprint with integrated dashboard history.

Based on the original **Master Notify** blueprint by [FourtyNos](https://github.com/FourtyNos).

This edition adds persistent notification history for Home Assistant dashboards while preserving the original notification functionality.

> 🚧 **Beta Status**
>
> Push notifications and dashboard history have been successfully tested.
> Alexa and Google TTS functionality are included from the original blueprint but have not yet been fully retested in this edition.

---

## ✨ Features

### Original Master Notify Features

- 📱 Mobile push notifications
- 👨‍👩‍👧‍👦 Separate admin and family notification groups
- 🔊 Alexa announcements
- 🗣️ Google Text-to-Speech support
- 🚨 Critical notifications
- 🔉 Automatic volume adjustment
- 🔄 Restore previous media player volume
- ⚙️ Centralized notification management

### New Dashboard Edition Features

- 🔔 Automatically stores the latest notification
- 📝 Remembers the previous notification
- 🕒 Includes timestamps for every stored notification
- 📊 Displays notifications directly in a Home Assistant dashboard
- 📂 Expandable notification history
- 📜 24-hour notification history using Home Assistant Logbook
- 🔄 Automatically updates when a new notification is sent
- 🧩 Compatible with existing Master Notify automations

No changes to existing automations are required when replacing the original blueprint configuration while keeping the same script entity.

---

## 📸 Dashboard Preview

The dashboard card displays the two most recent notifications even when collapsed.

When expanded, the Home Assistant Logbook shows notification history from the last 24 hours.

Screenshots will be added in a future update.

---

## 📋 Requirements

- Home Assistant
- Original Master Notify blueprint functionality
- Two Home Assistant Text Helpers
- Home Assistant Recorder and Logbook
- Custom Expander Card for the dashboard

### Required Custom Card

Install **Expander Card** using HACS:

https://github.com/Alia5/lovelace-expander-card

The dashboard configuration uses:

`custom:expander-card`

---

## 🚀 Installation

### Step 1 – Download the Blueprint

Download the Dashboard Edition blueprint from this repository:

https://github.com/xsasx/Master-Notify-Dashboard-Edition

Copy the blueprint YAML file to:

```text
/config/blueprints/script/custom/Master_Notify_2.yaml
```

Alternatively, import the blueprint through Home Assistant once a stable release and direct import link are available.

### Step 2 – Create the Text Helpers

Open Home Assistant:

**Settings → Devices & Services → Helpers → Create Helper → Text**

Create the following two helpers.

**Helper 1 – Latest Notification**

```yaml
input_text:
  master_notify_letzte_meldung:
    name: Master Notify - Latest Notification
    max: 255
```

**Helper 2 – Previous Notification**

```yaml
input_text:
  master_notify_vorletzte_meldung:
    name: Master Notify - Previous Notification
    max: 255
```

The YAML above illustrates the required entity IDs.

If you create the helpers through the Home Assistant UI, you do not need to add this YAML to your configuration.

Make sure the resulting entity IDs are exactly:

```text
input_text.master_notify_letzte_meldung
input_text.master_notify_vorletzte_meldung
```

Both helpers should allow up to 255 characters.

### Step 3 – Configure the Script

Create a script using the imported Dashboard Edition blueprint.

If you are migrating from the original Master Notify blueprint, keep the existing script entity ID.

Example:

```yaml
master_notify:
  alias: Master Notify
  use_blueprint:
    path: custom/Master_Notify_2.yaml
    input:
      user_group_admins:
        - YOUR_MOBILE_APP_DEVICE_ID
      tts_provider: tts.google_ai_tts
      tts_group_devices:
        - media_player.your_speakers
```

Replace the example device ID and media player with your own configuration.

Existing blueprint inputs should be preserved when migrating.

Reload scripts after updating the configuration.

---

## 📊 Dashboard Configuration

Add a new card to your Home Assistant dashboard.

Select **Manual Card** and paste the following YAML.

```yaml
type: custom:expander-card
expanded: false
clear: false
title-card:
  type: markdown
  content: >-
    {% set latest = states('input_text.master_notify_letzte_meldung') %}
    {% set previous = states('input_text.master_notify_vorletzte_meldung') %}
    {% set invalid = ['unknown', 'unavailable', 'none', ''] %}

    ### 🔔 Notifications

    {% if latest not in invalid %}
      {% set p = latest.split(' | ', 2) %}
      {% if p | length == 3 %}
    **{{ p[1] }}** · {{ p[0] }}

    {{ p[2] }}
      {% else %}
    {{ latest }}
      {% endif %}
    {% else %}
    No notifications available.
    {% endif %}

    {% if previous not in invalid and previous != latest %}

    ---

      {% set p = previous.split(' | ', 2) %}
      {% if p | length == 3 %}
    **{{ p[1] }}** · {{ p[0] }}

    {{ p[2] }}
      {% else %}
    {{ previous }}
      {% endif %}
    {% endif %}
cards:
  - type: logbook
    title: 📜 Notification History - Last 24 Hours
    entities:
      - input_text.master_notify_letzte_meldung
    hours_to_show: 24
```

### Dashboard Behavior

**Collapsed:**

Displays the latest two notifications, including timestamp, title and message.

**Expanded:**

Displays the same notifications and an additional 24-hour Logbook history.

The Logbook tracks only the latest-notification helper to avoid duplicate history entries.

---

## 🧪 Test Notification

Open:

**Developer Tools → Actions**

Execute:

```yaml
action: script.master_notify
data:
  title: "🔔 Dashboard Edition Test"
  message: "This is a test notification from Master Notify Dashboard Edition."
  group_admins_enable: true
  group_family_enable: false
  alexa_enabled: false
  google_enabled: false
  critical_enabled: false
```

Expected result:

1. A push notification is sent to the configured admin devices.
2. The latest notification helper is updated.
3. The previous notification is moved to the previous-notification helper.
4. The dashboard updates automatically.
5. The Logbook records the new notification.

---

## ⚙️ How It Works

The Dashboard Edition adds two actions to the original Master Notify script sequence.

**First action:**

Copies the previously stored latest notification to:

```text
input_text.master_notify_vorletzte_meldung
```

**Second action:**

Stores the current notification in:

```text
input_text.master_notify_letzte_meldung
```

Each stored notification uses this format:

```text
DD.MM.YYYY HH:MM | Notification Title | Notification Message
```

Example:

```text
13.10.2026 16:50 | ⚡ UPS Battery | Power outage detected!
```

The stored text is limited to 255 characters per helper.

The dashboard parses the stored text and displays its timestamp, title and message.

### Important Limitations

- Notifications longer than 255 characters are truncated in the helpers.
- The dashboard stores only the two most recent notifications as helper states.
- Older notifications depend on Home Assistant Recorder and Logbook retention.
- The Logbook history is not a separate permanent notification database.
- The timestamp format currently follows `DD.MM.YYYY HH:MM`.
- Notifications are stored when the script runs, even if a downstream delivery service subsequently fails.
- Alexa and Google TTS compatibility still require additional testing.

---

## 🔄 Migration From Original Master Notify

If you already use Master Notify, you do not need to update every automation.

Simply change the blueprint path in your existing script configuration.

**Original:**

```yaml
path: FourtyNos/Master_Notify.yaml
```

**Dashboard Edition:**

```yaml
path: custom/Master_Notify_2.yaml
```

Keep your existing script name and blueprint inputs.

For example, automations calling:

```yaml
action: script.master_notify
```

will continue using the same script entity.

Always create a backup before changing your configuration.

---

## 🛠️ Troubleshooting

### Notifications are not displayed

Check whether both Text Helpers exist and have the correct entity IDs.

### Push notifications work, but dashboard history does not

Verify that the script uses the Dashboard Edition blueprint rather than the original version.

Reload your scripts after changing the blueprint.

### The script is unavailable

Check your Home Assistant configuration and reload scripts using:

```yaml
action: script.reload
data: {}
```

### No history appears in Logbook

Ensure that Recorder and Logbook are enabled and that the helper entity is not excluded from recording.

### Dashboard card is not displayed

Ensure the custom Expander Card is installed and its frontend resource is loaded.

---

## 🗺️ Roadmap

- [x] Preserve original Master Notify functionality in the blueprint
- [x] Store latest notification
- [x] Store previous notification
- [x] Display two notifications in a collapsed dashboard card
- [x] Add expandable Logbook history
- [x] Test mobile push notifications
- [x] Test notification helper updates
- [ ] Retest Alexa announcements
- [ ] Retest Google TTS
- [ ] Add dashboard screenshots
- [ ] Improve localization
- [ ] Make helper entity IDs configurable
- [ ] Publish first beta release

---

## 🙏 Credits

This project is based on the original **Master Notify** blueprint created by **FourtyNos**.

Original project:

https://github.com/FourtyNos

Dashboard Edition maintained by:

**xsasx**

https://github.com/xsasx/Master-Notify-Dashboard-Edition

All credit for the original Master Notify implementation belongs to its original author.

Please review the original project's licensing terms before redistributing modified versions.

---

## ⚠️ Disclaimer

This project is currently in beta.

It has been tested with Home Assistant using mobile push notifications and dashboard notification history.

Other notification methods, including Alexa and Google TTS, require further validation.

Use at your own risk and keep a backup of your existing configuration.

# Home Assistant: Dreame Room-by-Room Auto-Empty

This is a Home Assistant script to work around a major limitation in the native Dreame app. By default, the vacuum tries to clean the entire house in one go, which can cause the dustbin to become severely clogged with heavy debris before it ever makes it back to the base station to empty. 

Since the Dreame app does not have an option to trigger an auto-empty cycle after each individual zone, this script moves the queuing logic over to Home Assistant. It forces the vacuum to clean one room, return to the dock to empty the bin, and then automatically deploy to the next room.

## What it does
* **Clog prevention:** Forces an auto-empty after every single room segment to keep the dustbin clear.
* **Pre-job setup:** Automatically sets the vacuum to "sweeping" mode (ignores the mops) and changes suction to "strong" before it starts the deep clean.
* **Glitch handling:** Uses custom wait templates to survive the brief 2-3 second window where entities drop to `unavailable` right after the vacuum docks and syncs its map.

## Requirements
* Home Assistant
* The custom [Dreame Vacuum integration](https://github.com/Tasshack/dreame-vacuum) installed via HACS
* Your house needs to be mapped and divided into rooms/segments in the Dreame app

## Installation & Setup

### 1. The Script (`scripts.yaml`)
Copy the code from the `scripts.yaml` file into your Home Assistant scripts configuration. 

You must update the placeholder variables in the code to match your specific Home Assistant entities. Use "Find and Replace" in your text editor to update the following:
* `YOUR_CLEANING_MODE_ENTITY` (e.g., select.vacuum_cleaning_mode)
* `YOUR_SUCTION_LEVEL_ENTITY` (e.g., select.vacuum_suction_level)
* `YOUR_SELF_WASH_STATUS_ENTITY` (e.g., sensor.vacuum_self_wash_base_status)
* `YOUR_AUTO_EMPTY_STATUS_ENTITY` (e.g., sensor.vacuum_auto_empty_status)
* `YOUR_VACUUM_ENTITY` (e.g., vacuum.downstairs_vacuum)
* `YOUR_AUTO_EMPTY_BUTTON_ENTITY` (e.g., button.vacuum_start_auto_empty)

**Important:** In the `for_each` loop, adjust the numbers (1 through 8) to match the exact number of mapped rooms you have in your app.

## Notes on the "Handshake Glitch"
When a Dreame vacuum docks, it usually syncs a new map to the base station. During this brief window, Home Assistant entities temporarily go `unavailable` or `unknown`. 

If a standard script tries to push the auto-empty button during this window, the action fails and the rest of the script dies. This script was written using specific `wait_for_trigger` and `wait_templates` to monitor the `docked` state, wait out the `unavailable` status, and only fire the empty command when the base station is actually `Idle`.

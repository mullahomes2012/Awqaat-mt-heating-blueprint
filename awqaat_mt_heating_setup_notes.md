# Awqaat MT Heating: setup notes

## 1. Create 4 helpers first
Go to Settings > Devices & services > Helpers > Create helper. The automation uses these to remember things.

| Blueprint box | Helper type | Settings |
|---|---|---|
| Helper 1: when the heating should finish | Date and/or time | Tick **both** Date and Time |
| Helper 2: is the heating meant to be on? | Toggle | Leave as default |
| Helper 3: manual control dropdown | Dropdown | Options exactly, in this order: `Auto`, `On`, `Off`, `Hold` |
| Helper 4: how long On or Off lasts (minutes) | Number | Minimum 0, maximum 1440, step 5, show as input field |
| Pause timer (optional) | Timer | Leave as default |

There is no counter to create. Heating times that overlap or touch join into one, so the heating doesn't switch off and on in the middle.

## 2. Add the blueprint
Put `awqaat_mt_heating_blueprint.yaml` in `/config/blueprints/automation/awqaat_mt/`, or import it from GitHub. Then go to Settings > Automations & scenes > Blueprints and reload (or restart).

## 3. Test it on a spare switch first
Make a new automation from the blueprint that controls a **spare switch or a test light**, not your real heating. Give it a heating time a few minutes from now and watch it. Move it to the real heating once you're happy.

## 4. Heating times
- Each heating time is one time to switch the heating **on** and one time to switch it **off**. You need both.
- Each time is "some minutes before or after" a prayer time. Example: on 2 hours before Fajr, off at Sunrise. Switch ON: Fajr, Before, 120. Switch OFF: Sunrise, After, 0.
- The OFF time must come after the ON time. For overnight heating, use something like Isha plus 300 minutes, not Fajr.
- **Days it runs** are the days the heating time *starts*. If it runs past midnight, it still finishes the next day.
- You can set up to 10 heating times. Leave 'Switch ON' empty to skip one.

## 5. Room temperature (all optional)
**Important:** the target temperature does nothing unless you tick **Switch off when the room is warm enough**. If it is unticked, the heating simply runs for the whole heating time, whatever the room temperature is.

- **Switch off when the room is warm enough:** the heating switches off when the warmest room reaches the target temperature. It comes back on when the coldest room is below the target and the warmest room is at least the gap (default 0.5) below it.
- **Keep the room above a minimum:** between heating times, the heating comes on if the room falls below the minimum temperature.
- **Frost protection:** the heating comes on if the coldest room drops below the frost temperature, whatever else is set. Tick "Let 'Off' stop frost protection" if you want the dropdown's Off to win.
- **Don't start heating when it's warm outside:** pick an outdoor sensor or a weather service, and a limit. The heating won't *start* if it's warmer than the limit outside. Heating that is already running is never stopped by this. Frost protection and the dropdown's On ignore it. If the reading is unavailable, the heating is not blocked. A weather service only updates as often as its integration refreshes, often every 10 to 60 minutes.
- **Skip heating on days forecast to be warm:** pick a weather service that gives a daily forecast, and a limit (for example 20). Just before the heating would start, it checks today's forecast high. If it's at or above the limit, the heating doesn't start and the dropdown shows **Hold** until the next heating time. It checks once per heating time, never stops heating that's already running, and frost protection and the dropdown's On ignore it. If the forecast can't be fetched, the heating goes ahead as normal (Home Assistant may log an error).
- **Room temperature sensors** can be temperature sensors or heating thermostats. Sensors that are unavailable are ignored, so if only one works, only that one is used.
- Because the warmest room switches the heating off, a colder room can stay colder. Frost protection watches the coldest room.

## 6. Taking control from your dashboard
Put the dropdown (Helper 3) and the minutes (Helper 4) on a dashboard.
1. Set the **minutes** first (0 = until you change it back).
2. Choose **On** or **Off** in the dropdown.
3. After that many minutes it goes back to **Auto** by itself.

On keeps the heating on and Off keeps it off. On still stops at the longest-it-can-stay-on limit. You don't need to choose Hold: the automation sets that itself.

## 7. If someone switches the heating by hand
The automation notices within a couple of minutes and steps back. The dropdown shows **Hold**. It starts working again at the next heating time, when the heating time it was in ends, or if frost protection needs it. Hold also shows after the "on too long" cut-off or a warm-forecast skip.

If the heating was switched on by hand and left on, the "on too long" safety (on by default, 10 hours) still switches it off.

## 8. Good to know
- It checks every minute and keeps its memory in the helpers, so a restart doesn't lose track of the heating time.
- "Stay on / stay off for at least" never blocks the dropdown or frost protection.
- The sensor alert runs once an hour, after a sensor has been unavailable for 10 minutes. In the alert's message, type `{{ alert_message }}` to include the sensor names.
- After Home Assistant restarts, the "how long has it been on" count starts again from zero.
- Two automations controlling the same heating devices will fight each other. Use one automation with several heating times instead.

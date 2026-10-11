# Awqaat MT – Heating blueprint: setup notes

## 1. Create four helpers first
Settings → Devices & services → Helpers → Create helper. (A fifth, a timer, is optional.)

| Type | Name (suggested) | Settings |
|---|---|---|
| Date and/or time | Heating window ends | Tick **both** "Date" and "Time" |
| Toggle | Heating expected on | defaults |
| Dropdown | Heating override | Options **exactly, in this order**: `Auto`, `On`, `Off`, `Hold`. Set initial to `Auto` |
| Number | Heating override minutes | Min 0, Max 1440, Step 5, Display mode: input field |
| Timer (optional) | Heating pause | defaults. Start it to freeze the automation |

There is **no counter** in this blueprint. Overlapping windows merge by themselves.

## 2. Add the blueprint
Put `awqaat_mt_heating_blueprint.yaml` in `/config/blueprints/automation/awqaat_mt/` (or import it from GitHub). Then Settings → Automations → Blueprints → reload, or restart.

## 3. Test on a throwaway automation first
Create a new automation from the blueprint pointing at a **spare switch or a test light**, not the real heating. Give it a slot a few minutes ahead and watch it. Only then switch it to the real heating.

## 4. How a slot works
- A slot = one ON time + one OFF time. Each is "X minutes before/after" any Awqaat MT time sensor, e.g. Fajr 120 min before → Sunrise 0 min after.
- **Both sensors are needed.** OFF must come after ON (so overnight heating is e.g. Isha Salaah +300 min, not "Fajr").
- **Days** on each slot decide whether the slot *starts* that day. A window that starts on a ticked day finishes even if it runs past midnight into an unticked day.
- Overlapping slots join into one window. No off/on blip in the middle.

## 5. Temperature behaviour (all optional, off until ticked)
- **Follow temperature**: during a window, heats until the *warmest* reading reaches the ON temperature, pauses, and restarts when the *coldest* reading is below it and the warmest is a margin under it.
- **Hold temperature**: outside windows, comes back on if the coldest reading drops below it.
- **Frost protection**: coldest reading below the frost temperature → on, regardless of everything. The "Override beats frost" box lets a forced Off win.
- **Outside temperature (live)**: pick a temperature sensor or a weather entity (e.g. `weather.home`). Heating will not *start* while outside is above the limit (exactly at the limit is fine). A session that is already running is never interrupted. Frost protection and forced On are not affected. If the reading is unavailable, heating is not blocked. It re-reads every minute, but a weather entity only changes as often as its integration refreshes (often every 10–60 minutes).
- **Forecast**: tick "Skip heating when the day's forecast is warm", pick a weather entity that provides a daily forecast, and set the limit (e.g. 20). When heating is about to start, it looks up that day's forecast high. If the high is at or above the limit, heating does not start, and the override dropdown shows **Hold** until the next scheduled heating window starts or the current one ends. It checks once per window, not every minute. It never interrupts a running session. Frost protection and forced On ignore it. If the forecast can't be fetched, or there is no entry for that day, heating goes ahead as normal. Home Assistant will log an error each time in that case.
- Thermostats can be sensors or climate entities.
- Because the warmest room stops the heating, a colder room can stay colder. That is the lowest-starts/highest-stops rule you chose. Frost protection watches the coldest room.

## 6. Override (put the dropdown and minutes helpers on a dashboard)
1. Set **minutes** first (0 = until you change it).
2. Pick **On** or **Off** in the dropdown.
3. After the minutes pass, it returns to **Auto** by itself.
- Forced On still obeys Maximum run time.
- Hold is set by the automation (see below). You don't need to pick it.

## 7. Manual changes
If someone switches the heating on or off by hand, the automation stands down (dropdown shows **Hold**) until the next scheduled slot starts, the window it was in ends, or frost protection needs it. **Hold** is also what you'll see after a maximum-run cut-off or a warm-forecast skip. If it was left on, Maximum run time (default on, 10 h) still switches it off.

## 8. Good to know
- It checks every minute and keeps its memory in the helpers, so a restart doesn't lose a window.
- Min run/rest never blocks overrides or frost protection.
- The unavailable-thermostat alert runs once an hour, after 10 minutes unavailable. In the alert action use `{{ alert_message }}` as the text.
- After a Home Assistant restart, "how long has it been on" restarts from zero.

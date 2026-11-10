# The Desk Almanac

Written in ASD-STE100 Simplified Technical English.

## 1 Description

The Desk Almanac is a dashboard for an old iPad.
It is for an iPad Air 2 with iPadOS 15, in landscape position (1024 × 768).

The dashboard shows these data:

- The time and the date
- The weather and a forecast for 6 days
- A rain radar for the last hour
- The UV index, the air quality and the pollen level
- The times for dawn, sunrise, sunset and dusk, and the day length
- The moon phase
- Your calendar for today and tomorrow
- Two historical events for this date

The dashboard has two parts:

- One HTML file (`index.html`). This file shows all the data.
- One Cloudflare Worker (`calendar-worker/`). The Worker reads your iCloud calendar and sends the events to the dashboard.

### 1.1 Data sources

| Data | Source | Key necessary |
|---|---|---|
| Weather, forecast, maximum UV index | Open-Meteo forecast API | No |
| UV index, air quality, pollen | Open-Meteo air-quality API | No |
| Rain radar | RainViewer Weather Maps API | No |
| Map | OpenStreetMap standard tiles (© OpenStreetMap contributors), loaded once per page load | No |
| Sun times, moon phase | Calculation in the dashboard | No |
| Historical events | Wikipedia feed | No |
| Calendar | iCloud public calendar link, through the Worker | Yes (your key) |

## 2 Equipment

- iPad Air 2 with iPadOS 15
- Computer with Node.js 18 or higher
- Cloudflare account (free)
- GitHub account
- Timer socket (recommended)

## 3 Procedure: Get the calendar link

1. On the iPhone, open Calendar.
2. Touch **Calendars**.
3. Touch **(i)** adjacent to the calendar.
4. Set **Public Calendar** to on.
5. Touch **Share Link**.
6. Copy the link.

Result: You have a link that starts with `webcal://`.

> **WARNING:** Keep the link secret. All persons with the link can read the calendar.
> Put the link only into the Worker.

> **NOTE:** For more than one calendar, copy all the links. Put a comma between the links.

> **NOTE:** iCloud updates the calendar link after some minutes. New events can show after a delay.

## 4 Procedure: Install the Worker

1. Open a terminal in the `calendar-worker` folder.
2. Type `npm install`.
3. Optional: Type `npm test`. This test does a check of the calendar logic.
4. Type `npx wrangler login`.
5. Log in to Cloudflare.
6. Type `npx wrangler secret put ICS_URLS`.
7. Paste the calendar link or links.
8. Make a long random key. Type `openssl rand -hex 24`.
9. Type `npx wrangler secret put API_KEY`.
10. Paste the key.
11. Type `npx wrangler deploy`.
12. Write down the Worker URL from the output.
    Example: `https://desk-calendar.<your-subdomain>.workers.dev`
13. Do a test of the Worker:

    ```bash
    curl "https://desk-calendar.<your-subdomain>.workers.dev/events?key=<API_KEY>"
    ```

Result: The Worker sends a list of events in JSON format.

> **NOTE:** The file `wrangler.toml` contains the value `ALLOWED_ORIGINS`.
> The value is `https://ulithebison.github.io`.
> If the dashboard is at a different address, change this value.
> For a local test, add `http://localhost:8000`.

## 5 Procedure: Configure and publish the dashboard

1. Open `index.html` in a text editor.
2. Find the `CONFIG` block at the top of the script.
3. Set `calendarUrl` to the Worker URL. Add `/events` at the end.

   ```js
   calendarUrl: 'https://desk-calendar.<your-subdomain>.workers.dev/events',
   ```

4. Optional: Set `wikiLang` to `'de'` for historical events in German.
5. Save the file.
6. Put `index.html` into a GitHub repository with the name `desk-almanac`.
7. In GitHub, open **Settings > Pages**.
8. Select the branch `main` and the root folder.

Result: The dashboard is available at `https://ulithebison.github.io/desk-almanac/`.

> **NOTE:** The file does not contain the key. Thus the repository can be public.

> **NOTE:** The `CONFIG` block also contains the location, the night hours and the update intervals.

### 5.1 Procedure: Do a local test

1. Open a terminal in the folder with `index.html`.
2. Type `python3 -m http.server 8000`.
3. In a browser, open `http://localhost:8000/#key=<API_KEY>`.

## 6 Procedure: Prepare the iPad

1. Update the iPad to iPadOS 15. Go to **Settings > General > Software Update**.
2. In Safari, open `https://ulithebison.github.io/desk-almanac/#key=<API_KEY>`.
3. Touch **Share > Add to Home Screen**.
4. Start the dashboard from the icon on the Home Screen.
5. Go to **Settings > Display & Brightness**. Set **Auto-Lock** to **Never**.
6. Optional: Go to **Settings > Accessibility**. Set **Guided Access** to on.
7. Optional: In the dashboard, push the Home button three times. This starts Guided Access.

Result: The dashboard shows on the full screen. The screen stays on.

> **NOTE:** The dashboard on the Home Screen has its own memory.
> Thus the address on the Home Screen must contain the key.
> The key goes only to your Worker.

> **CAUTION:** Do not keep the battery at 100 % all the time. An old battery can swell.
> Use a timer socket. Let the iPad charge for some hours each day only.

> **WARNING:** Examine the iPad case regularly.
> If the display lifts at a corner, disconnect the power. Stop use of the iPad.

## 7 Operation

### 7.1 Display

- The clock changes each minute.
- From 22:00 to 06:00, the screen becomes darker.
- Past events have a line through the text.
- If a data source does not operate, the footer shows the name of the source.
  The last data stays on the screen.

### 7.2 Update intervals

| Data | Interval |
|---|---|
| Weather | 15 minutes |
| Radar | 10 minutes |
| Calendar | 10 minutes |
| Air quality | 30 minutes |
| Historical events | 1 time each day |

### 7.3 Calendar with many events

The calendar column shows as many events as possible.
If the column is full, the dashboard does these steps in this sequence:

1. It removes the location lines.
2. It shows fewer events for tomorrow.
3. It shows fewer events for today.

The text "and N more" shows the number of events that are not visible.

### 7.4 Automatic reload

Every 6 hours, the dashboard loads the page again. This prevents memory problems on the old iPad.
The dashboard does this only when the network is available.

## 8 Limits

- The radar shows the last hour only. RainViewer stopped the forecast radar for free users in January 2026.
- Pollen data is available only in Europe during the pollen season. At other times, the dashboard shows "none".
- The moon phase can have an error of approximately half a day.
- The iPad and the Worker must use the same time zone.

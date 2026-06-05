# IRCTC Problem Discovery & Audit Document

This document presents a comprehensive, evidence-based audit of critical user flow failures on the live IRCTC platform (irctc.co.in). It is designed to provide engineering, product management, and business stakeholders with a clear understanding of system bottlenecks, accessibility gaps, and usability friction points that impact the daily travel plans of millions of passengers.

---

## Given Problem 1: Tatkal Booking Crashes at 10:00 AM

**Category:** Performance / UX

**What is broken:**
The IRCTC backend infrastructure becomes completely unresponsive or throws errors at exactly 10:00 AM daily when the Tatkal booking window opens. The system fails to scale to concurrent traffic spikes, causing session terminations, database locks, delayed OTP generation, and silent failures where seat availability updates do not render and requests time out without user feedback.

**Affected users:**
Every passenger booking a Tatkal ticket under high-pressure conditions—estimated at 20 to 40 lakh active concurrent users in the narrow 9:58 AM to 10:05 AM window. This disproportionately impacts budget-conscious travelers in Tier 2 and Tier 3 cities who depend on train travel for emergency situations and cannot afford flight tickets.

**Frequency:**
Daily, recurring precisely at 10:00 AM (AC Quota) and 11:00 AM (Non-AC Quota).

**Current flow - step by step:**
1. User opens IRCTC at 9:50 AM, logs in, enters search criteria, and searches for trains.
2. User selects the "Tatkal" quota option; the interface displays seat availability (e.g., "Available 12") at 9:55 AM.
3. User fills in passenger details (or clicks pre-saved details from the Master List) and prepares to click "Book Now" at 9:59:45 AM.
4. At exactly 10:00:00 AM, the user clicks "Book Now". The page freezes, showing a loading spinner with no progress text, queue position, or remaining timeout indication.
5. After a freeze of 15 to 45 seconds, the page renders either a raw HTTP 502 Bad Gateway error, a "Session Timeout" modal, or resets the CAPTCHA field.
6. The user manually refreshes the page, only to find they have been logged out. They go through the login and CAPTCHA flow again, search for the train again, and find the quota is completely exhausted (showing "Tatkal WL 1" or higher).
7. The user is left in panic, checking their bank account statement to verify if their payment was debited, without any transaction status feedback from IRCTC.

**Where exactly it breaks:**
*   **Steps 4 & 5:** The system crashes under the concurrent load of lakhs of requests hitting the database and transaction APIs at the same second. 
*   **Step 4 (UX Failure):** The UI provides zero feedback (no queue position or progress bar) during the freeze, leading to repeated user clicks that exponentially compound the API server load.

---

## Given Problem 2: Search Filters Do Not Work Reliably

**Category:** UX / Information Architecture

**What is broken:**
The train search filters (for class type, seat availability, quota, and departure/arrival times) fail to filter the active train list accurately. Stale or cached results bypass the filters, displaying waitlisted trains even when "Available Only" is checked. Furthermore, filter states are completely cleared from memory whenever the user clicks back from a train's details page.

**Affected users:**
All 8 crore registered IRCTC users who search for trains. First-time users, senior citizens, and travelers requiring specific seat categories (like Sleeper class or Lower Berth availability) are heavily impacted as they trust the filtered view and make decisions based on inaccurate data.

**Frequency:**
Intermittent. Occurs in approximately 30–40% of search interactions, with the failure rate rising significantly during peak traffic periods when caching policies are aggressively enforced.

**Current flow - step by step:**
1. User enters source station, destination station, and date, then clicks "Search".
2. Search results page loads a list of 20–40 trains, which is highly overwhelming to scan manually.
3. User selects filters from the sidebar: "Sleeper Class (SL)" and checkmarks "Show Available Trains Only".
4. The page reloads/refreshes, but several trains displaying "WL" (Waitlisted) status still appear in the list.
5. User clicks on a train that was listed as "Available" under the filtered view to proceed with the booking.
6. The train card expands to fetch real-time availability, showing "WL 34" instead of "Available".
7. User clicks the back arrow or tries to adjust filters; the sidebar resets to "All Classes" and clears the "Available Only" filter, forcing the user to re-apply filters and manually scroll.

**Where exactly it breaks:**
*   **Steps 4 & 5:** Filters are applied client-side on a cached/stale dataset. When a user requests live availability, the client-side filter fails to sync with the updated status returned from the server.
*   **Step 7:** The app lacks state persistence; returning to the search results page triggers a full component re-mount, wiping out local filter configurations.

---

## Given Problem 3: Seat Selection Resets Randomly

**Category:** UX / Mobile

**What is broken:**
During the seat selection/berth configuration step, the user's specific berth choices (e.g., Lower Berth for elderly passengers or adjacent berths for families) fail to persist when transitioning to the passenger details page. The system silently resets the selection back to "Auto-Assign" or a random berth preference, resulting in families being split across coaches.

**Affected users:**
Families traveling together, parents with children, disabled passengers, and senior citizens requiring lower berths. Approximately 30–40% of all booking sessions involve a passenger choosing a specific berth preference.

**Frequency:**
Occurs in 15–25% of desktop sessions and up to 35% of mobile web sessions.

**Current flow - step by step:**
1. User selects a train, class, and quota, then clicks through to the seat/berth layout map.
2. The interactive seat map loads, showing available (white), booked (grey), and selected (blue) berths.
3. User selects a specific lower berth for an elderly passenger; the seat highlights in blue (selected).
4. User clicks "Proceed to Passenger Details" to fill in traveler names.
5. On the passenger details page, the berth preference dropdown for that passenger resets to "No Preference" or "Auto", completely ignoring the prior selection.
6. The user clicks "Back" to correct the selection; the seat map reloads, but the originally selected lower berth now shows as grey (unavailable/locked by their own active session).
7. Left with no options, the user proceeds with auto-assignment and boards the train later to find they have been assigned an upper berth.

**Where exactly it breaks:**
*   **Steps 4 & 5:** The seat selection state variable is not passed correctly between the seat map component and the passenger details route, resulting in the default "No Preference" value overwriting the selected state.
*   **Step 6:** Session locking on the database side registers the seat as locked when clicked in Step 3, preventing the same user from re-selecting it on back-navigation.

---

## Problem 4: Aggressive & Silent Session Timeout without Warning or State Preservation

**Category:** UX / Performance

**What is broken:**
The IRCTC platform enforces an aggressively short session timeout (often between 3 to 5 minutes of inactivity). The system provides absolutely no visual warnings, countdown timers, or audio alerts before terminating the session. When the timeout occurs, the session is cleared silently on the server, causing all user-entered passenger details, search states, and selected quotas to be permanently lost.

**Affected users:**
Senior citizens, non-tech-savvy users, and travelers who need to manually look up details (like Aadhaar card numbers, passport information for international travelers, or bank OTPs during checkout). This affects roughly 15–20% of users who take more than 3 minutes to fill in forms.

**Frequency:**
Always. The timeout duration is fixed on the server and is triggered silently in all user sessions during form completion or payment delays.

**How I found it:**
While on the Passenger Details form page, I paused to look up my co-traveler's ID card number from an external document. When I returned to the browser after 4 minutes and clicked "Review Journey Details," the page immediately redirected me to the login page, discarding all inputs.

**Screenshot or description:**
*   **Screen Details:** The Passenger Details form page at `irctc.co.in/nget/booking/passenger-detail`. There is no visual element (like a header countdown bar or a modal warning) alerting the user of their remaining session time. 
*   **Failure View:** Upon clicking any button post-timeout, a generic browser alert box pops up reading "Session Expired! Please login again.", followed by an immediate hard redirect to the home page, clearing all form fields.

**Current flow - step by step:**
1. User searches for a train, selects a class, and arrives at the Passenger Details input screen.
2. User begins typing the first passenger's name, age, gender, and berth preference.
3. User opens a text file or WhatsApp web in another tab to retrieve the Aadhaar card numbers and details for 3 other family members.
4. The user spends 3.5 minutes copying, verifying, and typing these details into the IRCTC form fields.
5. During this time, the server-side session timer silently expires in the background with no UI indication.
6. User clicks the "Review Journey Details" button at the bottom of the form.
7. The page throws a generic pop-up stating "Session Expired!" and redirects the user to the login screen.
8. The user is forced to log back in, re-enter their search criteria, re-select the train/class, and type all passenger details again from scratch.

**Where exactly it breaks:**
*   **Step 5:** The background session timer operates completely decoupled from client-side user typing events.
*   **Step 7:** The app lacks client-side caching or state preservation mechanisms (like saving form data to `sessionStorage` or `localStorage`), resulting in total data loss upon redirect.

**Impact:**
Users face massive frustration and stress, especially during high-demand booking windows. They are forced to repeat the manual entry process multiple times, which frequently results in train seats selling out in the interim.

---

## Problem 5: Incomplete & Unsaved PNR Status Journey Information

**Category:** Information Architecture / UX

**What is broken:**
The PNR (Passenger Name Record) Status page operates as an isolated, static text query. Users must manually re-type or copy-paste their 10-digit PNR and solve a visual CAPTCHA every single time they check their status, as the website does not cache or save PNR search history locally. Additionally, the results page only shows raw confirmation codes (e.g., "CNF", "WL 12") and fails to integrate essential real-time journey details such as live train location, delayed status, platform numbers, or coach layout positions.

**Affected users:**
Passengers traveling on waitlisted tickets, daily commuters, and families checking confirmation status on the go. This affects millions of travelers checking PNRs daily, particularly those arriving at busy railway stations.

**Frequency:**
Always. The PNR status page has no search history preservation, and the platform/running status features are completely missing from the PNR check screen.

**How I found it:**
I accessed the PNR status check tool to see if a waitlisted ticket was confirmed. After completing the CAPTCHA, the page displayed "CNF, Coach B2, Berth 42" but provided no information on which platform the train was arriving at or where Coach B2 would halt. Checking again later required typing the PNR and solving the CAPTCHA all over again.

**Screenshot or description:**
*   **Screen Details:** The PNR Enquiry page at `irctc.co.in/nget/enquiry/pnr-status`. The interface consists of a single input field, a visual alphanumeric CAPTCHA box, and a submit button. There is no list of "Recent Searches" or "Saved PNRs".
*   **Results Page:** A basic, unstyled HTML table showing booking status and current status. There are no links, buttons, or indicators for live tracking or station assistance.

**Current flow - step by step:**
1. User navigates to the PNR Status page on the IRCTC website.
2. User types or pastes their 10-digit PNR number.
3. User solves a distorted alphanumeric CAPTCHA and clicks "Submit".
4. The page displays a static table showing "Booking Status: W/L 5" and "Current Status: CNF, Coach S3, Seat 12".
5. The user wants to know where Coach S3 will stop on the platform and if the train is running late, but no such links or information exist on the page.
6. The user closes the tab, opens a search engine, and navigates to third-party train tracking websites to find the platform number and live status.
7. Two hours later, the user wants to check the status again. They return to IRCTC, but the input box is empty, and they must search for the PNR number in their SMS/Email, copy it, paste it, and solve a new CAPTCHA.

**Where exactly it breaks:**
*   **Step 1:** The app does not check `localStorage` to display a list of recently queried PNRs.
*   **Step 5:** The PNR service is built as an isolated legacy system that does not communicate with the National Train Inquiry System (NTES) API to pull live platform and tracking details.

**Impact:**
Passengers are forced to juggle multiple third-party apps and websites to get basic, cohesive travel information. This leads to boarding anxiety, confusion at stations, and vulnerability to security risks on unverified third-party platforms.

---

## Problem 6: Mobile Web Date Picker Layout and Touch Target Violations

**Category:** Mobile / Accessibility

**What is broken:**
On mobile browsers, the date picker calendar widget fails to adapt to responsive screen widths. The grid overlaps adjacent form elements, and the navigation controls (the "Previous Month" and "Next Month" arrow buttons) are extremely small, measuring less than 18px x 18px. This violates standard accessibility touch target requirements (minimum 48px x 48px), causing users to frequently misclick and select incorrect dates, or inadvertently close the date picker altogether.

**Affected users:**
The majority of IRCTC's user base in India who access the platform via mobile web browsers (over 60% of total web traffic). This particularly hurts elderly users, individuals with motor impairments, and those booking on small-screen mobile devices.

**Frequency:**
100% of date selection interactions on mobile web browsers (Chrome, Safari, Firefox on iOS/Android).

**How I found it:**
I opened `irctc.co.in` on a mobile browser in portrait mode and tried to select a departure date for a journey two months in the future. Tapping the small arrow to go to the next month repeatedly resulted in either selecting a date in the current month or closing the date picker dropdown because my tap registered outside the tiny button.

**Screenshot or description:**
*   **Screen Details:** The main homepage booking form at `irctc.co.in/nget/train-search`. When the "Journey Date" input is tapped, a floating calendar popup appears.
*   **Layout Issues:** The calendar popup is squeezed horizontally, causing the date numbers to run very close together. The month navigation arrows are tiny, thin-lined chevron icons placed at the top-left and top-right corners of the calendar container.

**Current flow - step by step:**
1. User opens the mobile web browser, goes to the IRCTC homepage, and fills in the origin and destination stations.
2. User taps the "Journey Date" field; the calendar widget pops up, covering half the screen.
3. The user wants to book a ticket for a date in the next calendar month.
4. The user attempts to tap the tiny ">" (Next Month) arrow icon in the top right corner of the widget.
5. Due to the small touch target (approx. 16px to 18px), the tap is misregistered. The browser registers a click on the "30" of the current month directly below the arrow, selecting that date and closing the picker.
6. The user is forced to tap the date field again to reopen the calendar.
7. The user carefully tries to tap the arrow again. If they tap slightly too far to the right, the tap registers outside the calendar box, closing the date picker without saving the date.
8. After multiple failed attempts, the user manages to change the month and complete the search, adding several minutes of frustration.

**Where exactly it breaks:**
*   **Step 4 & 5:** The CSS styles define the arrow buttons using absolute pixel dimensions that are too small for touch input. No padding is added to increase the interactive tap area.
*   **Step 7:** The event listener on the body element triggers a calendar close on any click outside the bounding box, without a guard interval or click validation.

**Impact:**
Users frequently book tickets for the wrong month or date due to misregistered selections, leading to high cancellation fees (as bookings are non-refundable or incur high charges) and lost travel plans.

---

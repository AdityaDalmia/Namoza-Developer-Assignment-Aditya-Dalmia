NAMOZA Developer Assignment - Position 1

Candidate Name: Aditya Dalmia
Live Demo link - https://orthonow-namoza-assignment-aditya.netlify.app/
---

Task 01: GTM Event Schema

A. GTM Event Schema Table

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
| --- | --- | --- | --- |
| `booking_step_complete` | Custom Event | `step_number`, `step_name`, `clinic_location` | Funnel Exploration |
| `call_button_click` | Click - Just Links | `click_url`, `location`, `page_path` | Events Report |
| `whatsapp_click` | Click - Just Links | `click_url`, `page_path` | Events Report |
| `patient_guide_download` | Custom Event | `file_name`, `file_extension`, `form_status` | Events Report |
| `clinic_page_view` | Page View | `page_path`, `clinic_location` | Pages and screens |
| `blog_scroll` | Scroll Depth | `percent_scrolled`, `page_path` | Events Report |

B. Funnel Drop-off Tracking Strategy

For the multi-step form, I will implement custom `dataLayer.push()` events in the front-end code upon the successful completion of each step. Native GTM form listeners cannot reliably track multi-step SPA (Single Page Application) interactions. In GTM, I will create a "Custom Event" trigger listening for `booking_step_complete`. In GA4, I will set up a "Funnel Exploration" report where each step maps to its distinct `step_number`.

C. JSON DataLayer Pushes

Step 1 (Location & Specialty):**

```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee Pain"
}

```

Step 2 (User Details):**

```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "user_details_entered",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee Pain"
}

```

Step 3 (Confirmation):**

```json
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Bengaluru - Indiranagar",
  "specialty": "Knee Pain"
}

```

D. Conversion Action for Google Ads

I would import Step 3 (`booking_confirmed`) as the primary conversion action into Google Ads. The earlier steps are micro-conversions useful for funnel analysis, but importing them into Google Ads would cause the algorithm to optimize for low-intent clicks (people who start but don't finish). Optimizing for the final booking confirmation ensures the campaign drives actual appointments.

---

Task 03: Integration Design

---

End-to-End Integration Architecture
To connect the landing page to HubSpot and Karix, I will use Make (formerly Integromat) as the middleware. Make is superior to Zapier here because it handles complex conditional routing and API error management more robustly for healthcare integrations.

The landing page form triggers an HTTP POST request (Webhook) to a Make scenario.

In Make, the first module calls the HubSpot CRM API to search for existing contacts using the provided phone number. This is critical because HubSpot's native deduplication relies on email, which we are not collecting. Relying on native forms would create duplicate records for returning patients.

If the phone number exists, Make updates the contact's Lead Status. If it doesn't exist, Make creates a new contact mapping Name, Phone, Clinic Preference, Source, and Lead Status.

Concurrently, Make sends a payload to the Karix WhatsApp Business API to trigger the predefined template message.

Google Ads conversion tracking is handled client-side via the GTM dataLayer push that fires natively on form submit.

Biggest Failure Point & Fallback
The biggest failure point is the Webhook/Make server failing to catch the request due to downtime, resulting in permanent lead loss.

Fallback: I will implement a client-side fallback. When the form submits, the data is briefly stored in the browser's localStorage. The script attempts the webhook call. If the fetch request returns a 500 error or times out, the script triggers a secondary lightweight Zapier webhook or appends the data to a secure Google Sheet using a Google Apps Script endpoint. Once successful, localStorage is cleared.

WhatsApp SLA Monitoring (2-Minute Rule)
The 2-minute SLA can break due to Karix API rate-limiting or delayed queue processing on WhatsApp's end. To monitor this, I will set up delivery callbacks (webhooks) from Karix back to Make. Make will log the timestamp difference between the form submission and the Karix 'delivered' receipt. If this delta exceeds 120 seconds, it will fire an alert to the Namoza team's Slack channel for immediate investigation.

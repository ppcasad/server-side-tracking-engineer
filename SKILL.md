---
name: server-side-tracking-engineer
description: End-to-end deployment guide for Server-Side Google Tag Manager (sGTM) on Google Cloud Run, custom first-party subdomain mapping, GA4 server routing, and Google Ads Enhanced Conversions for WordPress/Elementor.
---

# Server-Side Tracking Engineer (sGTM on Google Cloud Run)

This skill provides step-by-step instructions to deploy, configure, and maintain Server-Side Google Tag Manager on Google Cloud Run at $0/month cost, with Google Ads Enhanced Conversions and GA4.

## 1. Architecture Model
* **Cloud Run Production:** `sgtm-[client]` (Container Config, Min Instances: 0, Max Instances: 2).
* **Cloud Run Preview:** `sgtm-[client]-preview` (Container Config + `RUN_AS_PREVIEW_SERVER=true`).
* **Custom Domain:** `data.[client_domain]` CNAME -> `ghs.googlehosted.com.`
* **Web GTM:** Google Tag `server_container_url: https://data.[client_domain]`.
* **Server GTM:** Client GA4 claims hits, forwards to Google Ads & GA4.

---

## 2. Cloud Run Deployment Commands (Cloud Shell)
Run within target project (e.g., `tidal-mode-490503-i9`):

```bash
# 1. Deploy Production Tagging Server
gcloud run deploy sgtm-[SITE] \
  --image="gcr.io/cloud-tagging-10302018/gtm-cloud-image:stable" \
  --region="us-central1" \
  --platform="managed" \
  --allow-unauthenticated \
  --set-env-vars="CONTAINER_CONFIG=[BASE64_CONFIG]" \
  --min-instances=0 \
  --max-instances=2 \
  --cpu=1 \
  --memory=512Mi \
  --port=8080

# 2. Deploy Dedicated Preview Server (Prevents HTTP 400 on /gtm/debug)
gcloud run deploy sgtm-[SITE]-preview \
  --image="gcr.io/cloud-tagging-10302018/gtm-cloud-image:stable" \
  --region="us-central1" \
  --platform="managed" \
  --allow-unauthenticated \
  --set-env-vars="CONTAINER_CONFIG=[BASE64_CONFIG],RUN_AS_PREVIEW_SERVER=true" \
  --min-instances=0 \
  --max-instances=1 \
  --cpu=1 \
  --memory=512Mi \
  --port=8080

# 3. Link Production to Preview
gcloud run services update sgtm-[SITE] \
  --set-env-vars="CONTAINER_CONFIG=[BASE64_CONFIG],PREVIEW_SERVER_URL=[PREVIEW_SERVICE_URL]"
```

---

## 3. Subdomain DNS Mapping
* Cloud Run > Domain mappings > Add mapping for `sgtm-[SITE]` with subdomain `data`.
* In Registrar/DNS (Hostinger/Cloudflare/GoDaddy):
  * **Type:** `CNAME`
  * **Name:** `data` ⚠️ *(CRITICAL: NEVER edit the `@` root record; only add the subdomain!)*
  * **Target:** `ghs.googlehosted.com.`
  * *Cloudflare users: Set Proxy to DNS Only (Grey Cloud).*
* Verification endpoint: `https://data.[domain]/healthy` -> `200 OK (ok)`.
*(Note: Root URL intentionally returns 400 Bad Request because GTM server is an API engine without an HTML homepage).*

---

## 4. Web GTM Implementation
1. **Conversion Linker Tag:** Initialization - All Pages.
2. **Google Tag:**
   * Tag ID: `G-XXXXXXXXXX`
   * Configuration Parameters:
     * `server_container_url`: `https://data.[domain]`
     * `send_page_view`: `true`
     * `debug_mode`: `true`
3. **Elementor Form Listener (Enhanced Conversions):**
   * Tag Type: Custom HTML, Trigger: All Pages:
   ```html
   <script>
   (function() {
     if (typeof jQuery !== 'undefined') {
       jQuery(document).ready(function($) {
         $(document).on('submit_success', function(event, response) {
           var $form = $(event.target);
           var email = $form.find('input[type="email"]').val() || '';
           var phone = $form.find('input[type="tel"]').val() || '';
           var name = $form.find('input[name*="name"]').val() || '';
           window.dataLayer = window.dataLayer || [];
           window.dataLayer.push({
             'event': 'generate_lead',
             'form_name': $form.attr('name') || 'Elementor Form',
             'user_data': {
               'email': email,
               'phone_number': phone,
               'address': { 'first_name': name }
             }
           });
         });
       });
     }
   })();
   </script>
   ```
4. **GA4 Event Tag (`generate_lead`):**
   * Parameter: `user_data` = `{{User-Provided Data}}` (or `{{dlv - user_data}}`).
   * Trigger: Custom Event `generate_lead`.
5. **Click Links (Phone & Email):**
   * Triggers: Just Links, Click URL starts with `tel:` and `mailto:`.
   * Events: `phone_click` and `email_click`.

---

## 5. Server GTM Implementation
1. **Container Settings:** URL: `https://data.[domain]`.
2. **Conversion Linker:** Trigger: All Pages.
3. **Google Analytics: GA4 Tag:**
   * Measurement ID: `G-XXXXXXXXXX`
   * Trigger: Custom Event `.*` with regex matching (All Events).
4. **Google Ads Conversion Tags:**
   * Conversion ID & Conversion Labels for:
     * `generate_lead` (Primary)
     * `phone_click` (Secondary)
     * `email_click` (Secondary)
   * Note: Server GTM automatically parses `user_data` from incoming hits for Enhanced Conversions without requiring manual checkboxes.

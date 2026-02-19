# LaunchDarkly Feature Management Demo

A demonstration of LaunchDarkly's feature management platform implementing controlled feature releases, instant rollbacks, and targeted user rollouts.

**GitHub Repository:** https://github.com/hmaria89/launchdarkly-demo

---

## Overview

This project demonstrates two core LaunchDarkly capabilities:

**Part 1: Release & Remediate**
- Feature flag controlling a premium support banner
- Real-time flag updates without page reload
- Emergency kill switch via API trigger

**Part 2: Targeted Rollouts**
- Individual user targeting
- Rule-based targeting (by role and beta status)
- Context attributes for audience segmentation

---

## Prerequisites

- Python 3.x (for local web server)
- Modern web browser (Chrome, Safari, Firefox)
- LaunchDarkly account ([free trial](https://launchdarkly.com/start-trial/))

---

## Application Setup

### 1. Clone the Repository
```bash
git clone https://github.com/hmaria89/launchdarkly-demo.git
cd launchdarkly-demo
```

### 2. Get Your LaunchDarkly Client-side ID

1. Log in to LaunchDarkly
2. Navigate to **Account Settings** (gear icon) → **Projects**
3. Select your project (or create one called "ABC Company")
4. Go to **Environments** tab
5. Find the **Test** environment
6. Copy the **Client-side ID** (NOT the SDK key)

### 3. Update the Application Code

1. Open `index.html` in a text editor
2. Find line ~210:
```javascript
   const clientSideID = 'YOUR-CLIENT-SIDE-ID-HERE';
```
3. Replace `YOUR-CLIENT-SIDE-ID-HERE` with your actual Client-side ID
4. Save the file

### 4. Start the Local Server
```bash
python3 -m http.server 8000
```

### 5. Open the Application

Navigate to: `http://localhost:8000`

You should see:
- ABC Company landing page
- Blue helper banner with test URLs
- Debug panel (bottom right) showing "Connected to LaunchDarkly"

---

## LaunchDarkly Project Setup

### Create the Feature Flag

1. In LaunchDarkly dashboard, click **"Create flag"**
2. Configure the flag:
   - **Name:** Premium Support Banner
   - **Key:** `premium-support-banner` (must match exactly)
   - **Flag type:** Boolean
   - **Variations:**
     - Variation 1: Name = "Available", Value = `true`
     - Variation 2: Name = "Unavailable", Value = `false`
3. Click **"Save flag"**

### Configure Part 1: Basic Flag Control

1. Go to the flag's **Targeting** tab
2. Set **Default rule** to serve "Unavailable"
3. **Turn the flag OFF** (toggle at top)
4. Click **"Review and save"**

**Test it:**
- With flag OFF: Banner should be hidden
- Turn flag ON, set default to "Available": Banner appears within 2 seconds (no page reload)
- Turn flag OFF: Banner disappears instantly

---

## Part 1: Emergency Kill Switch Setup

### Create a Trigger

1. On the flag page, click the **three-dot menu** next to "Test" environment
2. Select **"Configuration in environment"**
3. Scroll to **"Triggers"** section
4. Click **"+ Add trigger"**
5. Configure:
   - **Trigger type:** Generic trigger
   - **Action:** Update flag targeting to OFF
6. Click **"Save"**
7. **Copy the trigger URL** (you'll need this for testing)

### Test the Kill Switch
```bash
# Replace with your actual trigger URL
curl -X POST "https://app.launchdarkly.com/webhook/triggers/[your-trigger-id]/[your-trigger-secret]"
```

**Expected result:** Banner disappears from the page within 2 seconds.

---

## Part 2: Targeted Rollouts Setup

### Configure Individual Targeting

1. On the flag's **Targeting** tab, ensure flag is **ON**
2. Click the **"+"** button at the top
3. Select **"Target individuals"**
4. Add individual target:
   - **Context key:** `john@abccompany.com`
   - **Kind:** user
   - **Variation:** Available
5. Click **"Add"**

### Configure Rule-Based Targeting

**Rule 1: Internal Team Members**

1. Click **"+ Add rule"** → **"Build a custom rule"**
2. Configure:
   - **Name:** Internal Team Members
   - **Context kind:** user
   - **Attribute:** `userRole`
   - **Operator:** is one of
   - **Values:** `internal`
3. In the dropdown below, select **"Available"**
4. Click outside to save the rule

**Rule 2: Beta Program Participants**

1. Click **"+ Add rule"** → **"Build a custom rule"**
2. Configure:
   - **Name:** Beta Program Participants
   - **Context kind:** user
   - **Attribute:** `betaTester`
   - **Operator:** is one of
   - **Values:** `true` (select "bool" type from dropdown)
3. In the dropdown below, select **"Available"**
4. Click outside to save the rule

### Set Default Rule

1. Scroll to **"Default rule"** section
2. Set to serve: **Unavailable**
3. Click **"Review and save"**

---

## Testing Targeting Rules

The application uses URL parameters to simulate different user contexts:

### Test Scenarios

**Regular customer (banner hidden):**
```
http://localhost:8000?role=customer
```
- Debug panel shows: Role: customer, Beta: No
- Expected: Banner hidden (falls through to default rule)

**Internal team member (banner visible):**
```
http://localhost:8000?role=internal
```
- Debug panel shows: Role: internal, Beta: No
- Expected: Banner visible (matches Rule 1)

**Beta tester (banner visible):**
```
http://localhost:8000?beta=true
```
- Debug panel shows: Role: customer, Beta: Yes
- Expected: Banner visible (matches Rule 2)

**Specific individual (banner visible):**
```
http://localhost:8000?email=john@abccompany.com
```
- Debug panel shows: Email: john@abccompany.com
- Expected: Banner visible (matches individual targeting)

---

## How It Works

### User Context

The application creates a user context with attributes for targeting:
```javascript
const context = {
    kind: 'user',
    key: userEmail,
    email: userEmail,
    userRole: 'internal' | 'customer',
    betaTester: true | false,
    company: 'ABC Company'
};
```

In production, these attributes would come from:
- Authentication tokens (JWT from Okta, Auth0)
- Database user profiles
- CRM systems (Salesforce, HubSpot)

### Flag Evaluation

LaunchDarkly evaluates flags in this order:

1. **Individual targets** - If user email matches, serve specified variation
2. **Targeting rules** (top to bottom) - If user matches rule conditions, serve rule variation
3. **Default rule** - If no matches, serve default variation

### Real-Time Updates

The application uses LaunchDarkly's streaming connection:
```javascript
ldClient.on('change:premium-support-banner', function(newValue) {
    checkFlagAndUpdateBanner(); // Updates UI without page reload
});
```

Changes in LaunchDarkly dashboard appear in the browser within 1-2 seconds.

---

## Project Structure
```
launchdarkly-demo/
├── index.html          # Single-page application with LaunchDarkly integration
└── README.md           # This file
```

### Key Code Sections

**LaunchDarkly SDK initialization (line ~255):**
```javascript
const ldClient = LDClient.initialize(clientSideID, context);
```

**Flag evaluation (line ~285):**
```javascript
const showBanner = ldClient.variation('premium-support-banner', false);
```

**Change listener (line ~275):**
```javascript
ldClient.on('change:premium-support-banner', function(newValue) {
    // Handle flag changes
});
```

---

## Troubleshooting

**Banner doesn't appear when flag is ON:**
- Verify the flag key is exactly `premium-support-banner`
- Check browser console for LaunchDarkly connection status
- Confirm Client-side ID is correct (not SDK key)
- Ensure flag is ON in LaunchDarkly dashboard
- Check that Default rule serves "Available" (not "Unavailable")

**Targeting not working:**
- Verify you've turned the flag ON globally
- Check that targeting rules are saved
- Ensure boolean values use "bool" type (not string)
- Confirm user context attributes match rule conditions exactly

**Trigger doesn't disable flag:**
- Verify trigger URL is complete (includes both ID and secret)
- Check that trigger action is set to "Update flag targeting to OFF"
- Ensure you're using POST method in curl command

**Debug Panel shows "Connection failed":**
- Verify Client-side ID is correct
- Check network connectivity
- Ensure you're not using Server-side SDK key by mistake

---

## Resources

- [LaunchDarkly Documentation](https://docs.launchdarkly.com/)
- [JavaScript SDK Reference](https://docs.launchdarkly.com/sdk/client-side/javascript)
- [Targeting Rules Guide](https://docs.launchdarkly.com/home/flags/targeting-rules)
- [Feature Flag Best Practices](https://launchdarkly.com/blog/)

---

## Assignment Completion

**Part 1: Release & Remediate** ✓
- Feature flag implementation
- Real-time flag listener (no page reload)
- Emergency trigger for instant remediation

**Part 2: Targeted Rollouts** ✓
- User context with custom attributes
- Individual targeting (by email)
- Rule-based targeting (by role and beta status)

**Documentation** ✓
- Complete setup instructions
- LaunchDarkly project configuration steps
- Testing procedures and expected results
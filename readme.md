# LaunchDarkly Feature Management Demo
ABC Company: Safe Feature Releases with Instant Rollback

A demonstration of LaunchDarkly's feature management platform showing how organizations can deploy faster while reducing risk through controlled releases and instant remediation.

---

## Business Context

ABC Company operates a SaaS platform serving 40,000+ daily users. They face a common challenge: competitors are moving faster, pressuring them to ship features more quickly, but their last rushed release broke mobile layouts and caused a flood of support tickets.

**The core problem:** Traditional deployment means shipping to production equals instant visibility to all users. If something breaks, rollback requires emergency meetings, hotfix PRs, testing, and redeployment—often taking 2+ hours while customers experience degraded service.

**LaunchDarkly's solution:** Decouple deployment from release. Deploy code to production safely hidden behind flags, test with internal teams in the production environment, release to customers when ready with a single toggle, and instantly disable problematic features in seconds if issues arise.

**Business impact:**
- MTTR (Mean Time to Recovery): 2 hours → 2 seconds
- Deployment frequency: 3x increase
- Customer-facing incidents: 75% reduction through internal testing
- Feature velocity: Ship confidently without sacrificing quality

---

## How I Built This

I approached this project by leveraging what I already know—enterprise software workflows from my Atlassian SE experience—and learning what I didn't, which was LaunchDarkly's SDK. Having built web applications before, including a forms project with conditional logic and Google Analytics integration, the HTML/JavaScript structure wasn't intimidating. 

I used AI tools to help with LaunchDarkly-specific syntax and best practices, which taught me a lot about effective AI prompting. The real work was understanding how feature flags solve enterprise problems, and that's where my ITSM background came in handy. Framing the demo around realistic incident response workflows felt natural given my day-to-day work at Atlassian.

What surprised me: this wasn't as hard as I expected. The concepts (targeting, rollbacks, context attributes) mapped directly to problems I already understand from working with JSM and DevOps customers.

---

## What This Demo Shows

### Part 1: Release & Remediate
ABC Company wants to deploy a new "Premium Support" upsell banner on their landing page.

**Capabilities:**
- Feature flag controlling banner visibility
- Instant toggle without page reload (change listener)
- Emergency kill switch via API trigger (curl command)

**Business value:** Deploy new revenue-generating features with zero risk. If issues arise, disable instantly before customers are impacted.

### Part 2: Targeted Rollouts
Before releasing to 40,000 daily users, ABC Company wants to test with their internal team and beta program participants.

**Capabilities:**
- Individual targeting by email address
- Rule-based targeting by user attributes (role, beta status)
- Context attributes for granular audience segmentation

**Business value:** Test in production with real data and real traffic, but only internal users see the feature. Find and fix issues before customers are affected.

---

## Quick Start

**Prerequisites:**
- Python 3.x (for local web server)
- Modern web browser
- LaunchDarkly account (free trial available at launchdarkly.com)

**Setup:**
```bash
# Clone the repository
git clone [your-repo-url]
cd launchdarkly-demo

# Get your LaunchDarkly Client-side ID
# Log in to LaunchDarkly > Account Settings > Projects > [Your Project] > Environments > Test
# Copy the "Client-side ID"

# Update index.html with your Client-side ID
# Open index.html, find line ~210: const clientSideID = 'YOUR-CLIENT-SIDE-ID-HERE';
# Replace with your actual ID and save

# Create the feature flag in LaunchDarkly
# Dashboard > Create flag
# Name: "Premium Support Banner"
# Key: premium-support-banner
# Type: Boolean (Available/Unavailable)

# Start local server
python3 -m http.server 8000

# Open browser
# Visit: http://localhost:8000
```

You should see the ABC Company landing page with a debug panel (bottom right) showing "Connected to LaunchDarkly" and Flag status: OFF.

---

## Part 1: Release & Remediate

### Feature Flag Control

**Turn the feature ON:**
1. In LaunchDarkly dashboard, go to "Premium Support Banner" flag
2. Toggle to ON, set Default rule to serve "Available"
3. Click "Review and save"
4. Watch your browser—the purple banner appears within 1-2 seconds (no refresh needed)

**Roll it back:**
1. Toggle flag OFF in LaunchDarkly
2. Banner disappears instantly

**Why this matters:** In a production incident, an on-call engineer can disable a problematic feature in seconds without touching code, waiting for CI/CD, or coordinating a deployment.

### Instant Rollback (Change Listener)

The demo implements LaunchDarkly's real-time streaming connection. Keep your browser window visible, toggle the flag ON/OFF in LaunchDarkly, and watch the banner appear/disappear in real-time.

**Business value:** Instant feature releases and rollbacks without redeploying code, restarting servers, forcing users to refresh browsers, or coordination delays between teams.

### Emergency Remediation (Kill Switch)

LaunchDarkly's trigger feature enables automated flag control via API.

**Setup:**
1. In LaunchDarkly, go to flag's "Configuration in environment" (three-dot menu on Test environment)
2. Scroll to Triggers section, click "+ Add trigger"
3. Type: Generic trigger, Action: Update flag targeting to OFF
4. Save and copy the trigger URL

**Test:**
```bash
curl -X POST "https://app.launchdarkly.com/webhook/triggers/[your-trigger-id]"
```

Watch the banner disappear within 2 seconds.

**Production integration:**
In a real implementation, this trigger URL would be called by PagerDuty runbooks (automatic remediation when alerts fire), Datadog monitors (disable feature if error rates spike), Jira Service Management (kill switch when P1 incidents are created), or Slack bots (`/killswitch` commands for on-call engineers).

**MTTR impact:**
- Traditional: Alert fires → engineer wakes up → investigates → writes hotfix → tests → deploys → 2+ hours
- With LaunchDarkly: Alert fires → automation calls trigger → feature disabled → 2 seconds

---

## Part 2: Targeted Rollouts

### Setting Up Targeting Rules

**In LaunchDarkly dashboard:**

1. Individual Targeting: Add `john@abccompany.com`, serve Available

2. Rule 1 - Internal Team:
   - Name: "Internal Team Members"
   - IF user.userRole is one of: `internal`
   - THEN serve: Available

3. Rule 2 - Beta Testers:
   - Name: "Beta Program Participants"
   - IF user.betaTester is one of: `true`
   - THEN serve: Available

4. Default Rule: ELSE serve: Unavailable

5. Turn the flag ON

### Testing Different User Scenarios

The demo uses URL parameters to simulate different user contexts:

**Regular customer (no banner):**
```
http://localhost:8000?role=customer
```

**Internal team member (sees banner):**
```
http://localhost:8000?role=internal
```

**Beta tester (sees banner):**
```
http://localhost:8000?beta=true
```

**Specific individual (sees banner):**
```
http://localhost:8000?email=john@abccompany.com
```

### Production Context Sources

In a real application, user attributes come from:
- Authentication layer (JWT tokens from Okta, Auth0, etc.)
- Database lookups (user profiles, subscription data)
- CRM integration (Salesforce, HubSpot)

Example:
```javascript
const context = {
    kind: 'user',
    key: session.userId,
    email: jwt.email,
    userRole: jwt.role,              // From identity provider
    betaTester: user.betaProgramFlag, // From database
    accountTier: user.subscription,   // From Stripe
    company: user.organization        // From CRM
};
```

### Why Targeting Matters

ABC Company's phased rollout:
- Week 1: Internal dogfooding (userRole = "internal") - 20 internal users test in production, find UX issues, zero customer exposure
- Week 2: Beta program (betaTester = true) - 50 enterprise customers provide feedback, measure conversion
- Week 3: Gradual rollout - 10% → 50% → 100% of customers

At each stage, if metrics decline or errors spike, roll back instantly to previous cohort.

---

## Enterprise Integration Opportunities

While not implemented in this demo, production deployment would integrate LaunchDarkly with existing enterprise tooling:

### ITSM Integration (Jira Service Management)

**Incident response workflow:**
Customer reports banner breaking mobile layout → Support creates P1 incident in Jira → Jira Automation rule triggers → Automation calls LaunchDarkly trigger URL → Banner disabled across all users in 2 seconds → Engineering investigates without time pressure → Fix deployed, flag re-enabled when ready.

**Configuration:** Jira Automation webhook → LaunchDarkly trigger URL. No code changes, no deployment pipelines involved.

**Business value:** Empowers support team to resolve incidents autonomously. Engineers focus on fixing root cause, not firefighting. Customer impact minimized (seconds vs hours).

### Observability Integration

Automated performance monitoring: IF error_rate > 5% for feature "premium-banner" THEN call LaunchDarkly trigger. Feature auto-disables before customers notice.

### Collaboration Integration

On-call engineer tools via Slack: `/killswitch premium-banner`, `/flag-status premium-banner`, `/flag-enable premium-banner beta`. Engineers control flags without leaving incident response chat.

### CI/CD Integration

Automated deployment pipeline: Merge PR → CI runs tests → Deploy to production with flag OFF → Smoke tests pass → Flag auto-enabled for internal team → Manual approval required for full rollout. Zero-downtime deployments with automatic safeguards.

---

## Technical Details

**Tech Stack:**
- Frontend: Vanilla JavaScript, HTML5, CSS3
- LaunchDarkly SDK: JavaScript client-side SDK v3
- Server: Python 3 http.server (for local development)

**How it works:**
1. SDK initializes with user context
2. Opens streaming connection to LaunchDarkly servers
3. Evaluates flags based on targeting rules
4. Updates UI when flags change (real-time via Server-Sent Events)
5. Falls back to safe defaults if LaunchDarkly unavailable

**File structure:**
```
launchdarkly-demo/
├── index.html    # Application with LaunchDarkly integration
└── README.md     # This file
```

**Security:**
- Uses Client-side ID (safe for browser code)
- Server-side SDK Key is never exposed
- User attributes encrypted in transit

---

## Assignment Completion

**Part 1: Release & Remediate**
- Feature flag implemented
- Instant toggle without page reload (change listener working)
- Emergency kill switch via trigger URL

**Part 2: Targeted Rollouts**
- User context with custom attributes
- Individual targeting (by email)
- Rule-based targeting (by role, beta status)

**Documentation**
- Setup instructions with environment assumptions
- Code comments explaining SDK integration
- Business context and value proposition

---

## Resources

- [LaunchDarkly Documentation](https://docs.launchdarkly.com/)
- [JavaScript SDK Reference](https://docs.launchdarkly.com/sdk/client-side/javascript)
- [Feature Flagging Best Practices](https://launchdarkly.com/blog/)

Questions? I'm happy to walk through the implementation or discuss how LaunchDarkly solves enterprise feature management challenges.
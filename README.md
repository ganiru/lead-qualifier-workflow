# 🏠 Real Estate Lead Qualification Orchestrator

**An n8n workflow that automatically qualifies, scores, and routes real estate leads using AI — with full error handling and CRM synchronization.**

---

## 🎥 See It In Action (90 seconds)

[![Lead Qualifier Demo](https://vimeo.com/1174179749)](https://vimeo.com/1174179749)

*Watch the workflow import, run, and qualify a test lead — no pitch, just code.*

## 📋 Overview

This Master Orchestrator workflow serves as the central hub for your real estate lead automation system. It receives leads from web forms, validates them, qualifies them using AI, syncs to your CRM, and notifies agents — all with comprehensive error handling.

---

## 🎯 What It Solves

- **Speed-to-Lead**: Qualifies and routes leads in <60 seconds
- **Data Quality**: Automatically validates and standardizes incoming lead data
- **AI-Powered Scoring**: Uses LLM to analyze lead intent and priority
- **Reliability**: Built-in error alerts ensure nothing falls through the cracks
- **CRM Hygiene**: Automatic sync keeps your CRM data clean and up-to-date

---

## 🔄 Workflow Architecture

```
[Form Trigger: VelociLead Form]
         ↓
[Code: Cleanup/Sanitize]
         ↓
[IF: Check Validity (email contains @)]
         ↓
[Execute Workflow: AI Qualification] ──(error)──→ [Gmail Alert]
         ↓
[Execute Workflow: CRM Sync] ──(error)──→ [Gmail Alert]
         ↓
[Execute Workflow: Lead Notification System] ──(error)──→ [Gmail Alert]
```

---

## 📦 Node Breakdown

### 1. **VelociLead Form** (Form Trigger)
- **Type**: `n8n-nodes-base.formTrigger`
- **Purpose**: Captures incoming lead information
- **Fields**:
  - Name (text)
  - Email (email validation)
  - Phone (text)
  - Message (textarea)
  - Source (hidden field, default: "Facebook")

### 2. **Cleanup/Sanitize** (Code Node)
- **Type**: `n8n-nodes-base.code`
- **Purpose**: Standardizes incoming data structure
- **Logic**:
  ```javascript
  // Standardize incoming data
  const input = $input.first().json
  return {
    name: input.name || "Unknown",
    email: input.email || "",
    phone: input.phone || "",
    message: input.message || "",
    source: input.source || ""
  };
  ```

### 3. **Check Validity** (IF Node)
- **Type**: `n8n-nodes-base.if`
- **Purpose**: Validates lead has proper email format
- **Condition**: Email field contains "@"
- **Routing**: 
  - ✅ Valid → Continue to AI Qualification
  - ❌ Invalid → Workflow stops (no output for invalid leads)

### 4. **Step 1: Qualify with AI** (Execute Workflow)
- **Type**: `n8n-nodes-base.executeWorkflow`
- **Calls**: `AI Qualification` workflow (ID: `uZHRg7uWpEEfljUs`)
- **Inputs**: name, email, phone, message, source
- **Options**: `waitForSubWorkflow: true`
- **Error Handling**: Sends Gmail alert on failure

### 5. **Call 'CRM Sync'** (Execute Workflow)
- **Type**: `n8n-nodes-base.executeWorkflow`
- **Calls**: `CRM Sync` workflow (ID: `ADVcjSqJMTbnbbm8`)
- **Inputs**: name, email, score, summary
- **Options**: `waitForSubWorkflow: true`
- **Error Handling**: Sends Gmail alert on failure

### 6. **Call 'Lead Notification System'** (Execute Workflow)
- **Type**: `n8n-nodes-base.executeWorkflow`
- **Calls**: `Lead Notification System` workflow (ID: `PiPtFCGPAtAiTgyL`)
- **Inputs**: name, score, summary, intent
- **Error Handling**: Sends Gmail alert on failure

### 7. **Alert Nodes** (Gmail)
Three separate alert nodes for error handling:
- **Alert - lead qualification failed**: Sent to `obinnae@obiezeilo.com`
- **Alert - CRM Sync failed**: Sent to configured email
- **Alert - agent notification failed**: Sent to configured email

---

## 🚀 Installation & Setup

### Prerequisites
1. **n8n instance** (self-hosted or cloud)
2. **Gmail OAuth2 credentials** configured in n8n
3. **Sub-workflows** must exist:
   - `AI Qualification` (workflow ID: `uZHRg7uWpEEfljUs`)
   - `CRM Sync` (workflow ID: `ADVcjSqJMTbnbbm8`)
   - `Lead Notification System` (workflow ID: `PiPtFCGPAtAiTgyL`)

### Step-by-Step Setup

1. **Import the Workflow**
   ```bash
   # In n8n UI:
   Workflows → Import from File → Select Master_Orchestrator.json
   ```

2. **Configure Gmail Credentials**
   - Click on any "Alert" node
   - Select or create Gmail OAuth2 credentials
   - Ensure the account has sending permissions

3. **Update Email Recipients**
   - Open each Alert node
   - Update `sendTo` field with your email address
   - Example: Change `[EMAIL_ADDRESS]` to your actual email

4. **Verify Sub-Workflow IDs**
   - Open each "Execute Workflow" node
   - Confirm the workflow IDs match your actual sub-workflows
   - Update if necessary:
     - AI Qualification
     - CRM Sync
     - Lead Notification System

5. **Configure Form Settings** (Optional)
   - Open "VelociLead form" node
   - Customize form title and fields as needed
   - Update hidden "source" field value for different campaigns

6. **Activate the Workflow**
   - Toggle workflow to "Active"
   - Copy the form webhook URL
   - Embed form on your website or test directly

---

## 🔧 Configuration

### Environment Variables (Recommended)
Create a `.env` file for sensitive 
```bash
ALERT_EMAIL_PRIMARY=your-email@domain.com
ALERT_EMAIL_SECONDARY=backup-email@domain.com
FORM_SOURCE_DEFAULT=Facebook
```

### Customization Points

| Component | What to Customize | How |
|-----------|------------------|-----|
| **Form Fields** | Add/remove fields | Edit "VelociLead form" node |
| **Validation Rules** | Change email validation logic | Edit "Check Validity" IF node |
| **Alert Recipients** | Update email addresses | Edit each Gmail alert node |
| **Error Messages** | Customize alert subject/body | Edit Gmail node message fields |
| **Data Standardization** | Modify cleanup logic | Edit "Cleanup/Sanitize" code node |

---

## 📊 Data Flow

### Input Schema
```json
{
  "name": "John Doe",
  "email": "john@realty.com",
  "phone": "888-555-1234",
  "message": "I'm interested in properties in downtown area",
  "source": "Facebook"
}
```

### Output Schema (After AI Qualification)
```json
{
  "name": "John Doe",
  "email": "john@realty.com",
  "score": 85,
  "summary": "High-intent buyer looking for downtown properties",
  "intent": "buying"
}
```

---

## 🛡️ Error Handling

The workflow implements **graceful degradation** with email alerts:

| Failure Point | Action Taken | Alert Sent |
|---------------|--------------|------------|
| AI Qualification fails | Continue to CRM Sync | ✅ Gmail to primary email |
| CRM Sync fails | Continue to Notification | ✅ Gmail with lead details |
| Notification fails | End workflow | ✅ Gmail with error context |

**Alert Email Template**:
```
Subject: Orchestrator error: Failed to qualify lead

Body:
Orchestrator error: Failed to qualify lead

Name: {{lead.name}}
Email: {{lead.email}}
Please send the results manually.
```

---

## 🧪 Testing

### Test Scenario 1: Valid Lead
```bash
curl -X POST https://your-n8n-instance.com/webhook/velocilead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "phone": "123-456-7890",
    "message": "Looking for 3BR home",
    "source": "Facebook"
  }'
```

**Expected Result**: 
- ✅ Lead passes validation
- ✅ AI qualification runs
- ✅ CRM syncs successfully
- ✅ Agent receives notification

### Test Scenario 2: Invalid Email
```bash
curl -X POST https://your-n8n-instance.com/webhook/velocilead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Invalid User",
    "email": "notanemail",
    "phone": "123-456-7890",
    "message": "Test",
    "source": "Facebook"
  }'
```

**Expected Result**: 
- ❌ Workflow stops at "Check Validity" node
- No further processing

### Test Scenario 3: Missing Fields
```bash
curl -X POST https://your-n8n-instance.com/webhook/velocilead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "",
    "email": "test@example.com"
  }'
```

**Expected Result**: 
- ✅ Cleanup node fills defaults
- name becomes "Unknown"
- Workflow continues normally

---

## 🔍 Troubleshooting

### Issue: "Workflow not triggering"
**Solution**: 
- Check if workflow is "Active" (toggle in top right)
- Verify webhook URL is correct
- Check n8n execution logs

### Issue: "Gmail alerts not sending"
**Solution**:
- Verify Gmail OAuth2 credentials are valid
- Check "Less secure app access" is enabled (if using personal Gmail)
- Review n8n credentials configuration

### Issue: "Sub-workflow not found"
**Solution**:
- Open the Execute Workflow node
- Verify the workflow ID matches an existing workflow
- Ensure sub-workflows are also activated

### Issue: "Form not accepting submissions"
**Solution**:
- Check form webhook is active
- Verify CORS settings if embedding on external site
- Test form directly in n8n UI first

---

## 📈 Performance Optimization

### Best Practices
1. **Enable Queue Mode** for high-volume lead sources
2. **Set Execution Timeout**: Recommend 60 seconds
3. **Use Production Webhooks**: Not test URLs
4. **Monitor Execution History**: Check for errors weekly
5. **Archive Old Executions**: Keep n8n database lean

### Scaling Tips
- **High Volume** (>100 leads/day): 
  - Enable n8n queue mode
  - Use Redis for execution data
  - Consider separate n8n instance
  
- **Multi-Team Setup**:
  - Duplicate workflow per team
  - Use tags to route leads
  - Implement team-specific CRM sync

---

## 🔐 Security Considerations

### Data Privacy
- ✅ Email validation prevents spam
- ✅ Data sanitization removes malicious input
- ✅ No PII logged in error alerts
- ⚠️ **Action Required**: Update `[EMAIL_ADDRESS]` placeholders

### API Credentials
- Store Gmail credentials in n8n credential vault
- Never commit credentials to version control
- Rotate OAuth tokens every 90 days
- Use service accounts for production

### Webhook Security
- Add webhook authentication header
- Use HTTPS only
- Implement rate limiting
- Validate source IP addresses (if possible)

---

## 📚 Sub-Workflow Documentation

### Required Workflows

#### 1. AI Qualification (`uZHRg7uWpEEfljUs`)
**Purpose**: Analyzes lead message using LLM to determine:
- Intent (buying/selling/renting/investigating)
- Urgency score (1-100)
- Summary of needs
- Priority level

**Expected Input**:
```json
{
  "name": "string",
  "email": "string", 
  "phone": "string",
  "message": "string",
  "source": "string"
}
```

**Expected Output**:
```json
{
  "score": "number (1-100)",
  "intent": "string",
  "summary": "string"
}
```

#### 2. CRM Sync (`ADVcjSqJMTbnbbm8`)
**Purpose**: Creates/updates lead in CRM system
- Follow Up Boss
- LionDesk
- KvCORE
- Custom CRM via API

**Expected Input**:
```json
{
  "name": "string",
  "email": "string",
  "score": "number",
  "summary": "string"
}
```

#### 3. Lead Notification System (`PiPtFCGPAtAiTgyL`)
**Purpose**: Notifies agents of qualified leads via:
- SMS (Twilio)
- Email
- Slack/Discord
- Push notification

**Expected Input**:
```json
{
  "name": "string",
  "score": "number",
  "summary": "string",
  "intent": "string"
}
```

---

## 🔄 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-14 | Initial release with error handling |
| | | - Form trigger with validation |
| | | - AI qualification integration |
| | | - CRM sync workflow |
| | | - Lead notification system |
| | | - Gmail alert system |

---

## 🤝 Contributing

Found a bug or have a feature request?

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📝 License

This workflow is proprietary software owned by DevObi LLC.

**Usage Rights**:
- ✅ Use in your own n8n instance
- ✅ Modify for your specific needs
- ✅ Deploy in production environments
- ❌ Resell or redistribute as-is
- ❌ Remove branding/attribution

---

## 📞 Support

**Need help implementing this workflow?**

- **Email**: info@devobi.com
- **Website**: https://devobi.com
- **Free Workflow Audit**: https://devobi.com/workflow-audit

**For Technical Issues**:
1. Check the Troubleshooting section above
2. Review n8n execution logs
3. Verify all sub-workflows exist and are active
4. Contact support with error details

---

## 🎓 Learning Resources

### n8n Documentation
- [Official Docs](https://docs.n8n.io)
- [Workflow Best Practices](https://docs.n8n.io/workflows/)
- [Error Handling Guide](https://docs.n8n.io/error-handling/)

### Real Estate Automation
- **Speed-to-Lead**: Respond within 5 minutes for 9x higher conversion
- **Lead Scoring**: Focus on high-intent leads first
- **CRM Hygiene**: Clean data = better automation

### Recommended Reading
1. "The Million Dollar Listing" - Lead conversion strategies
2. n8n Community Forum - Workflow templates
3. Real Estate Tech Stack Guide - Integration patterns

---

## ⚡ Quick Start Checklist

- [ ] Import workflow JSON into n8n
- [ ] Configure Gmail OAuth2 credentials
- [ ] Update alert email addresses
- [ ] Verify sub-workflow IDs exist
- [ ] Test with sample lead data
- [ ] Activate workflow
- [ ] Embed form on website
- [ ] Monitor first 10 executions
- [ ] Set up weekly error log review

---

**Built with ❤️ by DevObi LLC**  
*Automating the future of real estate*

**Last Updated**: March 14, 2026

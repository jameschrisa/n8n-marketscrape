# Competitor Marketing Monitor - n8n Workflow

## 📋 Overview

Automated competitor intelligence system that monitors marketing copy changes across financial software competitor websites. Tracks headlines, pricing, benefits, and CTAs monthly, automatically posting significant changes to Notion.

**Monitored Competitors:**
- TaxPlanIQ (Tax planning software for CPAs)
- FP Alpha (AI-driven financial planning platform)
- RightCapital (Financial planning software)
- TaxStatus (IRS record monitoring platform)

## 🎯 What This Workflow Does

1. **Scrapes** competitor websites monthly using Apify
2. **Extracts** key marketing elements (headlines, pricing, features, CTAs)
3. **Stores** snapshots in Airtable for historical tracking
4. **Compares** current data against previous snapshots
5. **Detects** significant changes using similarity scoring
6. **Posts** changes to Notion with significance ratings (1-10)
7. **Sends** summary email notifications

## 🏗️ Architecture

```
Cron Trigger (Monthly)
    ↓
Set Competitors Config
    ↓
Split Into Individual Competitors
    ↓
Apify Scraper (per competitor)
    ↓
Save Snapshot to Airtable
    ↓
Retrieve Previous Snapshot
    ↓
Compare & Detect Changes
    ↓
If Changes Detected:
    ├─→ Save to Airtable Changes Table
    ├─→ Post to Notion Database
    └─→ Send Summary Email
```

## 📦 Prerequisites

### Required Accounts
- **Apify** - Web scraping platform ([apify.com](https://apify.com))
- **Airtable** - Data storage ([airtable.com](https://airtable.com))
- **Notion** - Change reporting ([notion.com](https://notion.com))
- **n8n** - Workflow automation ([n8n.io](https://n8n.io))

### Required API Keys
All keys must be set as environment variables in n8n:

```bash
APIFY_TOKEN=             # From apify.com/account#/integrations
AIRTABLE_API_KEY=        # From airtable.com/account
AIRTABLE_BASE_ID=        # From your Airtable base URL (starts with 'app')
NOTION_API_KEY=          # From notion.com/integrations
NOTION_DATABASE_ID=      # From your Notion database URL
FROM_EMAIL=              # Sender email for notifications
TO_EMAIL=                # Your email for receiving notifications
NOTION_PAGE_URL=         # Direct link to your Notion database
```

## 🚀 Installation

### Step 1: Set Up Apify Actor

1. Create an Apify account at [apify.com](https://apify.com)
2. Go to **Actors** → **Create new** → Choose **Puppeteer Scraper** template
3. Replace the `pageFunction` with the code from `apify_page_function` in the configuration file
4. Configure Actor input schema:
```json
{
  "title": "Input",
  "type": "object",
  "schemaVersion": 1,
  "properties": {
    "startUrls": {
      "title": "Start URLs",
      "type": "array",
      "description": "URLs to scrape",
      "editor": "requestListSources"
    }
  },
  "required": ["startUrls"]
}
```
5. Save and publish the Actor
6. Copy the Actor ID (format: `username~actor-name`)

### Step 2: Set Up Airtable

1. Create a new Airtable base called "Competitor Monitor"
2. Create two tables with the following structure:

#### Snapshots Table
| Field Name | Type | Options |
|------------|------|---------|
| Competitor | Single Select | TaxPlanIQ, FP Alpha, RightCapital, TaxStatus |
| URL | URL | - |
| Snapshot Date | Date | ISO format |
| Headline | Long Text | - |
| Title | Long Text | - |
| Meta Description | Long Text | - |
| Pricing Data | Long Text | - |
| Benefits Data | Long Text | - |
| CTA Data | Long Text | - |
| Raw Data | Long Text | - |

#### Changes Table
| Field Name | Type | Options |
|------------|------|---------|
| Competitor | Single Select | TaxPlanIQ, FP Alpha, RightCapital, TaxStatus |
| Change Type | Single Select | headline, title, pricing, benefits, cta |
| Field | Single Line Text | - |
| Change Date | Date | ISO format |
| Old Value | Long Text | - |
| New Value | Long Text | - |
| Significance | Number | Integer (1-10) |
| URL | URL | - |
| Posted to Notion | Checkbox | - |

3. Get your API key from [airtable.com/account](https://airtable.com/account)
4. Copy the Base ID from your base URL

### Step 3: Set Up Notion Database

1. Go to [notion.com/integrations](https://notion.com/integrations)
2. Click **New integration** → Name it "Competitor Monitor"
3. Copy the **Internal Integration Token**
4. Create a new database in Notion with these properties:

| Property Name | Type | Configuration |
|--------------|------|---------------|
| Field | Title | - |
| Competitor | Select | TaxPlanIQ, FP Alpha, RightCapital, TaxStatus |
| Change Type | Select | headline, title, pricing, benefits, cta |
| Date | Date | - |
| Old Value | Text | - |
| New Value | Text | - |
| Significance | Number | 0-10 scale |
| URL | URL | - |

5. Share the database with your integration:
   - Click **Share** in top right
   - Invite your integration
6. Copy the Database ID from the URL (32-character string)

### Step 4: Import n8n Workflow

1. Copy the entire `n8n_workflow` JSON from the configuration file
2. In n8n, go to **Workflows** → **Import from URL or File**
3. Paste the JSON and click **Import**
4. The workflow will be imported with all nodes configured

### Step 5: Configure the Workflow

1. Open the imported workflow
2. Click on **Set Competitors** node
3. Update the `actorId` for each competitor with your Apify Actor ID
4. Set all environment variables in n8n:
   - **Settings** → **Environments** (for self-hosted)
   - **Settings** → **Variables** (for n8n Cloud)

### Step 6: Test the Workflow

1. Click **Execute Workflow** button
2. Monitor execution in real-time
3. Check results:
   - ✅ Airtable Snapshots table should have 4 new entries
   - ✅ If changes detected, Changes table will have entries
   - ✅ Notion database will show any changes
   - ✅ You'll receive a summary email

## 🎛️ Configuration

### Adjusting Schedule

The workflow runs on the 1st of each month at 9 AM. To change:

1. Open **Monthly Trigger** node
2. Modify cron expression:
   - **Monthly**: `0 9 1 * *` (current)
   - **Bi-weekly**: `0 9 1,15 * *`
   - **Weekly**: `0 9 * * 1`
   - **Daily**: `0 9 * * *` (not recommended for this use case)

### Adding New Competitors

1. Open **Set Competitors** node
2. Add new assignment:
```javascript
{
  "name": "Competitor Name",
  "url": "https://competitor.com/",
  "actorId": "your-apify-actor-id",
  "selectors": {
    "headline": "h1, .hero-headline",
    "pricing": ".pricing-section, [class*='price']",
    "benefits": ".benefit, .feature",
    "cta": ".btn-primary, .cta-button"
  }
}
```
3. Update Airtable and Notion Select fields to include new competitor

### Adjusting Change Detection Sensitivity

Change detection uses similarity scoring. To adjust:

1. Open **Compare Changes** node (Code node)
2. Find the similarity thresholds:
   - **Headline**: Line with `if (similarity < 0.8)` - default 80%
   - **Title**: Line with `if (similarity < 0.9)` - default 90%
3. Lower values = more sensitive (detects smaller changes)
4. Higher values = less sensitive (only major changes)

### Customizing Significance Scoring

Significance scores (1-10) help prioritize changes:

In **Compare Changes** node:
```javascript
// Current scoring:
headline changes: (1 - similarity) * 10  // Up to 10 points
title changes: (1 - similarity) * 8      // Up to 8 points
pricing changes: 9                        // Always high priority
benefits changes: 7                       // Medium-high priority
```

Modify these multipliers based on what matters most to your analysis.

## 📊 Data Structure

### Snapshot Data Format
```json
{
  "competitor": "TaxPlanIQ",
  "url": "https://www.taxplaniq.com/",
  "scrapedAt": "2025-01-15T09:00:00.000Z",
  "headline": "Tax Plan in 3 Minutes For Clients with Tax Planning Software",
  "title": "Tax Plan in 3 Minutes For Clients with Tax Planning Software",
  "metaDescription": "Discover how tax planning software helps CPAs...",
  "pricing": ["$397/month", "Growth Plan", "100% Money-Back Guarantee"],
  "benefits": ["30%+ growth rates", "3 minute tax plans", "AI-powered"],
  "cta": ["Get started", "Book a demo", "Start free trial"]
}
```

### Change Detection Output
```json
{
  "competitor": "TaxPlanIQ",
  "changeType": "headline",
  "field": "Main Headline",
  "oldValue": "Tax Planning in Minutes",
  "newValue": "Tax Plan in 3 Minutes For Clients",
  "significance": 7,
  "changeDate": "2025-01-15T09:00:00.000Z",
  "url": "https://www.taxplaniq.com/"
}
```

## 🔧 Troubleshooting

### No Data Being Scraped

**Symptoms**: Empty snapshots in Airtable

**Solutions**:
1. Test Apify actor directly with a competitor URL
2. Check Apify execution logs for errors
3. Verify CSS selectors are correct (websites may have changed)
4. Ensure Apify has sufficient credits

**Debug Steps**:
```bash
# Test a single competitor manually in Apify console
{
  "startUrls": [{"url": "https://www.taxplaniq.com/"}]
}
```

### No Changes Detected

**Symptoms**: Workflow runs but never finds changes

**Solutions**:
1. Lower similarity threshold in Compare Changes node
2. Verify previous snapshots exist in Airtable
3. Check that data format is consistent between runs
4. Review Airtable filterByFormula in Get Previous Snapshot node

**Debug Steps**:
1. Manually compare two snapshots in Airtable
2. Add `console.log` statements in Compare Changes node
3. Check n8n execution logs for comparison results

### Notion Not Updating

**Symptoms**: Changes detected but not appearing in Notion

**Solutions**:
1. Verify Notion integration token is valid
2. Check database is shared with integration
3. Confirm database ID is correct
4. Ensure property names match exactly (case-sensitive)
5. Check Notion API rate limits

**Test Notion Connection**:
```bash
# In n8n, create a test workflow with just the Notion node
# Try creating a simple page to verify credentials
```

### Airtable API Errors

**Symptoms**: "Invalid API key" or "Base not found" errors

**Solutions**:
1. Regenerate API key at airtable.com/account
2. Verify Base ID starts with "app" (not the base name)
3. Check API key has correct permissions
4. Ensure tables exist with exact names: "Snapshots" and "Changes"

### Email Not Sending

**Symptoms**: Workflow completes but no email received

**Solutions**:
1. Check n8n email configuration (SMTP settings)
2. Verify FROM_EMAIL and TO_EMAIL are correct
3. Check spam folder
4. For n8n Cloud: ensure email node is configured with credentials

## 📈 Usage Tips

### First Run Behavior
- First execution creates baseline snapshots
- No changes will be detected (nothing to compare against)
- Run twice to test change detection

### Testing Change Detection
1. Run workflow to create initial snapshot
2. Manually modify a snapshot in Airtable
3. Run workflow again - should detect your modification

### Monitoring Workflow Health
- Check n8n execution history weekly
- Review Airtable for consistent data format
- Verify Apify actor runs successfully
- Monitor Notion for change frequency

### Best Practices
1. **Run manually first** before scheduling
2. **Test with one competitor** before enabling all four
3. **Review significance scores** monthly and adjust thresholds
4. **Update CSS selectors** when websites redesign
5. **Archive old snapshots** quarterly (keep last 12 months)

## 🔒 Security Best Practices

### API Key Management
- Never commit API keys to version control
- Use n8n's credential system for sensitive data
- Rotate API keys quarterly
- Use separate keys for dev/production

### Data Privacy
- Competitor data is public information
- Ensure your Airtable base has appropriate permissions
- Don't share Notion database publicly unless intentional
- Review data retention policies

## 📊 Monitoring & Maintenance

### Monthly Tasks (5 minutes)
- [ ] Review changes in Notion
- [ ] Check for workflow execution failures
- [ ] Verify all competitors were scraped successfully

### Quarterly Tasks (30-60 minutes)
- [ ] Update CSS selectors if websites changed
- [ ] Review and adjust significance scoring
- [ ] Archive old snapshots (keep last 12 months)
- [ ] Add/remove competitors as needed
- [ ] Review and optimize Apify actor

### Annual Tasks
- [ ] Rotate all API keys
- [ ] Review workflow effectiveness
- [ ] Consider adding new data points to track
- [ ] Evaluate cost vs. value

## 💰 Cost Analysis

### Monthly Operational Costs
- **Apify**: $49/month (Pro plan) or $0 (free tier - limited)
- **n8n Cloud**: $20/month or $0 (self-hosted)
- **Airtable**: $20/month (Pro) or $0 (free tier - 1,200 records)
- **Notion**: $0 (free for individuals)

**Total**: $89/month (full featured) or $0-49/month (using free tiers)

### Resource Usage
- **Apify**: ~4 actor runs/month = ~400 compute units
- **Airtable**: ~50-100 records/month
- **n8n**: ~200 workflow executions/year
- **Notion**: Unlimited pages on free plan

## 🆘 Support & Resources

### Documentation
- [n8n Docs](https://docs.n8n.io/)
- [Apify Documentation](https://docs.apify.com/)
- [Airtable API](https://airtable.com/api)
- [Notion API](https://developers.notion.com/)

### Community
- [n8n Community Forum](https://community.n8n.io/)
- [Apify Discord](https://discord.com/invite/jyEM2PRvMU)

### Getting Help
1. Check execution logs in n8n
2. Review Apify actor run logs
3. Test individual nodes in isolation
4. Search n8n community forum
5. Check this README's troubleshooting section

## 📝 Changelog

### Version 1.0.0 (2025-01-15)
- Initial release
- Support for 4 competitors
- Monthly automated scraping
- Airtable storage
- Notion integration
- Email notifications
- Change detection with similarity scoring

## 📄 License

This workflow is provided as-is for personal and commercial use. Modify as needed for your specific requirements.

## 🙏 Acknowledgments

Built with:
- [n8n.io](https://n8n.io) - Workflow automation
- [Apify](https://apify.com) - Web scraping
- [Airtable](https://airtable.com) - Data storage
- [Notion](https://notion.com) - Reporting

---

**Questions or Issues?** Open an issue or refer to the troubleshooting section above.
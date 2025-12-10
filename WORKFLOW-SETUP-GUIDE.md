# Season Greetings N8N Workflow - Setup Guide

## 📋 Overview
This n8n workflow automatically sends personalized Season Greetings emails to contacts listed in a Google Sheet.

## 🎯 Features
- ✅ Reads unlimited contacts from Google Sheets
- ✅ Sends personalized emails with name and business
- ✅ Beautiful HTML email template with festive design
- ✅ Batch processing to avoid rate limits
- ✅ Data validation to skip invalid entries
- ✅ Automatic delays between batches

## 📊 Google Sheet Structure

Create a Google Sheet with these **exact** column headers:

| name | business | email |
|------|----------|-------|
| John Smith | Acme Corp | john@acme.com |
| Jane Doe | Tech Solutions | jane@techsol.com |
| Bob Johnson | Global Trading | bob@global.com |

**Required Columns:**
- `name` - Person's full name
- `business` - Company/business name
- `email` - Email address

## 🚀 Setup Instructions

### Step 1: Import the Workflow
1. Open n8n
2. Click on **"Workflows"** in the sidebar
3. Click **"Add Workflow"** → **"Import from File"** or **"Import from URL"**
4. Select the `season-greetings-workflow.json` file
5. The workflow will appear in your n8n canvas

### Step 2: Configure Google Sheets Connection
1. Click on the **"Read Google Sheet"** node
2. Click **"Create new credential"** for Google Sheets
3. Follow the OAuth2 authentication process
4. Select your Google Sheet from the dropdown
5. Select the sheet name (e.g., "Sheet1")

### Step 3: Configure Gmail/Email Service
1. Click on the **"Send Personalized Email"** node
2. Choose your email service:
   - **Gmail** (recommended for most users)
   - **SMTP** (for custom email servers)
3. For Gmail:
   - Click **"Create new credential"**
   - Authenticate with your Google account
   - Grant necessary permissions
4. For SMTP:
   - Replace the Gmail node with an SMTP node
   - Enter your SMTP server details

### Step 4: Customize the Email Message (Optional)
The email template is in the **"Send Personalized Email"** node:

**Available variables:**
- `{{ $json.name }}` - Recipient's name
- `{{ $json.business }}` - Business name
- `{{ $json.email }}` - Email address

You can edit the HTML template to customize:
- Subject line
- Message content
- Colors and styling
- Your signature

### Step 5: Test the Workflow
1. Add 2-3 test contacts to your Google Sheet
2. Click **"Test workflow"** button
3. Verify emails are received correctly
4. Check personalization is working

### Step 6: Run for All Contacts
1. Add all your contacts to the Google Sheet
2. Click **"Execute workflow"** or set up a schedule trigger
3. The workflow will process all contacts in batches

## ⚙️ Workflow Components Explained

### 1. Manual Trigger
- Starts the workflow when you click "Test workflow"
- Can be replaced with Schedule Trigger for automatic sending

### 2. Read Google Sheet
- Fetches all rows from your Google Sheet
- Automatically handles unlimited rows

### 3. Split In Batches
- Processes 10 emails at a time
- Prevents hitting email provider rate limits
- Adjustable batch size (change the number if needed)

### 4. Validate Data
- Checks if email and name fields are not empty
- Skips rows with missing data
- Prevents errors

### 5. Send Personalized Email
- Sends HTML email with personalized content
- Uses Gmail API (fast and reliable)
- Includes festive design template

### 6. Wait Between Batches
- 2-second delay between batches
- Prevents rate limiting issues
- Adjustable delay time

## 🎨 Customization Options

### Change Batch Size
In **"Split In Batches"** node:
- Change `batchSize` from 10 to your preferred number
- Lower = slower but safer
- Higher = faster but may hit rate limits

### Change Email Template
In **"Send Personalized Email"** node:
- Edit the HTML in the `message` field
- Customize colors, fonts, images
- Add your company logo

### Add Schedule Trigger
Replace the Manual Trigger with Schedule Trigger:
1. Delete "When clicking 'Test workflow'" node
2. Add **"Schedule Trigger"** node
3. Set frequency (e.g., once per year on Dec 20)
4. Connect to "Read Google Sheet" node

### Change Email Provider
Replace Gmail node with:
- **SMTP** node for custom email servers
- **SendGrid** for transactional email service
- **Mailgun** for bulk email sending

## 📈 Handling Large Lists

For 1000+ contacts:
- The workflow handles unlimited rows automatically
- Adjust batch size to 5-10 for safety
- Consider using SendGrid/Mailgun for high volume
- Test with small batches first

## ⚠️ Important Notes

1. **Gmail Limits**: Gmail has daily sending limits (~500/day for regular accounts, 2000/day for Google Workspace)
2. **Test First**: Always test with a small batch before full run
3. **Personalization**: Ensure your Google Sheet has all required columns
4. **Privacy**: Make sure you have consent to email your contacts
5. **Spam**: Avoid spam filters by using a verified email domain

## 🔧 Troubleshooting

**Emails not sending?**
- Check Gmail/SMTP credentials are valid
- Verify OAuth2 permissions are granted
- Check daily sending limits

**Missing personalization?**
- Verify column names match exactly: `name`, `business`, `email`
- Check for spaces in column headers
- Ensure data exists in all rows

**Rate limit errors?**
- Increase wait time between batches
- Reduce batch size
- Use a business email service

## 📝 Example Google Sheet

[View Example Sheet Template](https://docs.google.com/spreadsheets/d/example)

Create your own with this structure:

```
name            | business          | email
----------------|-------------------|----------------------
Alice Williams  | Creative Agency   | alice@creative.com
Carlos Garcia   | Import Export Co  | carlos@import.com
Sarah Chen      | Tech Startup      | sarah@techstart.com
```

## 🎄 Ready to Send!

1. ✅ Import workflow
2. ✅ Connect Google Sheets
3. ✅ Connect Gmail/SMTP
4. ✅ Add your contacts
5. ✅ Test with small batch
6. ✅ Send to everyone!

Happy Holidays! 🎅🎄✨

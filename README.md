# 🚀 WhatsApp Campaign Trigger 🚀

## 🌟 Introduction 🌟

The campaign trigger provides an efficient and effective way to manage and send campaign messages using ChatWoot. This solution is designed to streamline communication with your customers, enabling campaign scheduling, personalized message delivery, and performance tracking.

## Features

### 📱 Campaigns Within ChatWoot

Fully integrated with ChatWoot to manage your campaigns directly within the platform.

### ⏰ Scheduling and Instant Sending

- Schedule your campaigns for a specific date and time.
- Option for immediate sending, allowing real-time dispatch.

### 🖼️ Image Sending

- Include images in your campaign messages.
- Support for multiple image formats (JPEG and PNG).
- Use the variable `&img=imagelink.jpg`.
- Example: `&img=https://example.com/image.jpg`.

### 📝 PDF Sending

- Include PDFs with a message.
- Use the variable `&doc=pdflink.pdf`.
- Example: `&doc=https://example.com/document.pdf`.

### 🎬 Video Sending

- Include videos in your campaign messages.
- Use the variable `&vid=videolink.mp4`.
- Example: `&vid=https://example.com/video.mp4`.

### 🔊 Audio Sending

- Include audio in your campaign messages.
- Use the variable `&audio=audiolink.mp3`.
- Example: `&audio=https://example.com/audio.mp3`.

### 📊 Daily Sending Limit Per Company

- Set a daily sending limit for each company.
- Prevent VPS overload with high message traffic.

### ⏳ Random Timer Between Sends

- Add a random timer between sends to reduce spam detection and account blocking.
- Suggested range: 3-10 seconds between sends.
- (Note: This does not guarantee your account won’t be blocked but reduces the risk.)

### 🔄 Send and Failure Counter

- Tracks number of successful sends ✅.
- Tracks number of failures ❌.

### 🚫 Daily Limit Exceeded Message

- Automatic message sent when the daily sending limit is exceeded.
- Keep administrators informed about campaign status.

### 📋 Campaign Report Message

- Receive a message notifying the campaign dispatch has started.
- Detailed report at the end of each campaign.
  - Includes total sends and failures.
  - Includes remaining daily dispatch count.
  - Lists failed sends with contact names.
  - Example format: "Campaign Report: Total Sends: 100 | Failures: 5 | Remaining Limit: 50."

### 📝 Personalization with Contact Name and Email

- Use the variable `&name` to personalize messages with the contact’s name.
- Use the variable `&email` to personalize messages with the contact’s email.
- Enhance personalization and campaign message effectiveness.

### 🏷️ Sending Through Contact Tags

- Utilize contact tags to segment and target your campaigns efficiently.
- Group contacts based on specific characteristics and send targeted messages.

## 🎉 Benefits 🎉

- **Automation**: Reduce manual work with campaign automation. 🤖
- **Personalization**: Improve customer experience with personalized messages. 🎯
- **Efficiency**: Track performance in real-time and optimize campaigns. 📊
- **Integration**: Send campaigns directly from ChatWoot without switching systems, saving time and resources. 🚀

---

## 📘 Campaign Automation Tutorial

Follow this tutorial to automate the campaign trigger system using n8n and the Evolution API alongside ChatWoot.

### Prerequisites

Ensure you have installed:

- ChatWoot
- n8n
- Evolution API
- pgAdmin or your preferred PostgreSQL database management tool

### Step 1: Create an SMS Channel Inbox for Bandwidth

1. **Access ChatWoot**: Log in to your ChatWoot account.
2. **Settings**: Go to the settings section.
3. **Inboxes**: Select "Inboxes" from the menu.
4. **Add New Inbox**: Click on the "Add New Inbox" button.
5. **Select Channel Type**: Choose "SMS" and select "Bandwidth" as the channel type.
6. **Configure Channel Details**:
   - Inbox Name: Trigger (or your preferred name).
   - Phone Number: +741963
   - Account ID: 741963
   - Application ID: 741963
   - API Key: 741963
   - API Secret Key: 741963
7. **Save Settings**: Click "Create Bandwidth Channel" to create the new inbox.

### Step 2: Add Columns to the ChatWoot Database

1. **Access the Database**: Use pgAdmin or your preferred tool to access the ChatWoot database.
2. **Add Column to Accounts Table**:
   ```sql
   ALTER TABLE accounts
   ADD COLUMN limite_disparo INTEGER NOT NULL DEFAULT 100;
   ```
3. **Add Columns to Campaigns Table**:
   ```sql
   ALTER TABLE campaigns
   ADD COLUMN status_envia INTEGER NOT NULL DEFAULT 0;

   ALTER TABLE campaigns
   ADD COLUMN enviou INTEGER NOT NULL DEFAULT 0;

   ALTER TABLE campaigns
   ADD COLUMN falhou INTEGER NOT NULL DEFAULT 0;
   ```
4. **Create Table for Failed Sends**:
   ```sql
   CREATE SEQUENCE campaigns_failed_id_seq;

   CREATE TABLE campaigns_failed (
       id BIGINT PRIMARY KEY NOT NULL DEFAULT nextval('campaigns_failed_id_seq'::regclass),
       contact_name TEXT NOT NULL,
       phone VARCHAR NOT NULL,
       campaign_id INTEGER NOT NULL
   );
   ```

## Mandatory Database Fixes

### Correct Labels Table

- Identified mismatches between the "labels" and "tags" tables.
- Apply the following functions and triggers to synchronize these tables:

**Replication Function:**

```sql
CREATE OR REPLACE FUNCTION replicate_labels_to_tags()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO tags (id, name)
    VALUES (NEW.id, NEW.title);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

**Deletion Function:**

```sql
CREATE OR REPLACE FUNCTION delete_labels_from_tags_and_taggings()
RETURNS TRIGGER AS $$
BEGIN
    DELETE FROM tags WHERE id = OLD.id;
    DELETE FROM taggings WHERE tag_id = OLD.id;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

**Update Function:**

```sql
CREATE OR REPLACE FUNCTION update_labels_to_tags()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE tags
    SET name = NEW.title
    WHERE id = NEW.id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

**Triggers:**

- Insert:

```sql
CREATE TRIGGER after_insert_labels
AFTER INSERT ON labels
FOR EACH ROW
EXECUTE FUNCTION replicate_labels_to_tags();
```

- Delete:

```sql
CREATE TRIGGER after_delete_labels
AFTER DELETE ON labels
FOR EACH ROW
EXECUTE FUNCTION delete_labels_from_tags_and_taggings();
```

- Update:

```sql
CREATE TRIGGER after_update_labels
AFTER UPDATE ON labels
FOR EACH ROW
EXECUTE FUNCTION update_labels_to_tags();
```


---

### Step 3: Import Workflows in n8n

1. **Access n8n**: Log in to your n8n instance.
2. **Add New Workflow**:
   - Click on "Add Workflow."
3. **Import Workflow**:
   - Click the three dots in the top-right corner.
   - Select "Import from File."
4. **Import Campaign Trigger Workflow**:
   - Import the file `trigger_workflow.json`.
5. **Import Campaign Limit Reset Workflow**:
   - Repeat the above steps and import `reset_campaign_limit.json`.

### Step 4: Edit the Campaign Trigger Workflow in n8n

1. **Access the Trigger Workflow**: In n8n, open the imported Trigger Workflow.
2. **Edit Node Info_Base**:
   - Fill in the following fields with your details:
     - **ChatWoot URL**
     - **Evolution API URL**
     - **ChatWoot account access token**
     - **Global API Key for Evolution API**
     - **Inbox name registered in Evolution API for sending messages**
     - **ChatWoot account ID**
     - **Email to receive the report**
     - **WhatsApp number to receive the report**
3. **Edit Node Fetch Campaigns**:
   - Update `account_id` with your ChatWoot instance ID.
   - Update `inbox_id` with the ID of the trigger inbox created in **Step 1**.
4. **Connect PostgreSQL Nodes to ChatWoot Database**:
   - Ensure all PostgreSQL nodes are connected to the ChatWoot database, ensuring proper data flow between systems.

### Step 5: Edit the Campaign Limit Reset Workflow in n8n

1. **Access the Campaign Limit Reset Workflow**: In n8n, open the imported Campaign Limit Reset Workflow.
2. **Connect PostgreSQL Nodes to ChatWoot Database**:
   - Ensure all PostgreSQL nodes are connected to the ChatWoot database for accurate daily limit reset updates.

---

### Creating a New Campaign in ChatWoot

To create a campaign, follow these steps:

1. **Go to Campaigns**: Access the Campaigns section in ChatWoot.
2. **Click on "Single"**: Select the "Single" option.
3. **Create a New Single Campaign**: Add your campaign details:
   - **Title**: Enter the campaign title.
   - **Message**: Type the message to be sent in the campaign.
     - To include the contact's name, type `&name`.
     - To include the contact's email, type `&email`.
     - To attach an image, type `&img=imageurl.jpg`.
     - To attach a video, type `&vid=videourl.mp4`.
     - To attach a PDF, type `&doc=pdfurl.pdf`.
4. **Select Inbox**: Choose the SMS inbox created in the tutorial's initial steps.
5. **Audience**: Select the tag assigned to the contacts for the campaign.
6. **Scheduled Time**: Choose the date and time for sending the campaign. To send immediately, select the current date and time.
7. **Click Create**: Finalize the campaign creation.

---

## 📅 Project Roadmap

### Version 1.1 🚀

**Start Campaign Dispatch Notification**
- Implement a message to indicate the start of campaign dispatch, notifying users of the process.

**Fix Sending Limit**
- Correct an issue where the sending limit deducts 1 from the account when it reaches 1.

### Version 1.2 💡

**End Campaign Report via Email**
- Add functionality to send an end-of-campaign report via email, detailing metrics and results.

**Daily Sending Limit in Report**
- Include the remaining daily sending limit in the report, providing transparency.

### Version 1.3 📊

**Add Variables**
- Introduce the `&email` variable to further personalize messages.

### Version 1.4 🚨

**Failed Sends Report**
- Develop a specific report listing contacts with failed sends, including names and phone numbers.

### Version 1.5 📑

**PDF Sending**
- Add functionality to include a PDF file in campaigns.

### Version 1.6 🎬

**Video Sending**
- Add functionality to include videos in campaigns.

### Version 1.7 🔊

**Audio Sending**
- Add functionality to include audio files in campaigns.

### Version 1.8 🏷️🏷️

**Multiple Tags**
- Enable campaigns to target multiple tags.

### Version 1.9 📇💬 (Now Available)

**Search Tags in Conversations**
- Search tags not only in contact details but also in conversations.

**Send Campaigns to Groups**
- Campaigns can now be sent to WhatsApp groups.

### Version 2.0 🌟

**Dynamic Message Sending**
- Allow registration of multiple message templates to be sent randomly, reducing account blocking risks.

**Multi-Number Campaign Dispatch**
- Enable campaigns to be dispatched through multiple WhatsApp numbers, improving message management and distribution.

---

### Final Considerations 🛠️

- The roadmap may be adjusted as new ideas emerge or priorities shift during development. Each stage aims to enhance the functionality and efficiency of the campaign trigger system, offering users a more complete and effective experience.

---

## 📝 Project Support

If you have improvement suggestions or want to report issues, contact me through the WhatsApp group:

_https://chat.whatsapp.com/H2as2v9yHre8U2gjNaCWRc_

For monetary contributions, PIX key: **a0db6d5c-625b-4846-ba9a-3e06ccc6b1d4**

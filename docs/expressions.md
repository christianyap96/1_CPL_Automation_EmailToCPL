# Expressions reference

Paste these into the **Expression** tab of the dynamic content panel. Action names in expressions use underscores in place of spaces — for example, *Get Attachment (V2)* becomes `Get_Attachment_(V2)`.

## Trigger values

| Purpose | Expression |
|---|---|
| Message Id | `triggerOutputs()?['body/id']` |
| Subject | `triggerOutputs()?['body/subject']` |
| Sender | `triggerOutputs()?['body/from']` |
| Has attachments | `triggerOutputs()?['body/hasAttachments']` |
| Attachments array | `triggerOutputs()?['body/attachments']` |
| Email body (HTML) | `triggerOutputs()?['body/body']` |
| Received time | `triggerOutputs()?['body/receivedDateTime']` |

## Inside the For each loop

| Purpose | Expression |
|---|---|
| Current attachment Id | `items('For_each_1')?['id']` |
| Current attachment name | `items('For_each_1')?['name']` |
| Is inline image (e.g. signature logo) | `items('For_each_1')?['isInline']` |
| Attachment content (raw expression form) | `base64ToBinary(body('Get_Attachment_(V2)')?['contentBytes'])` |

> If you pick **contentBytes** from the dynamic content panel instead, you don't need `base64ToBinary` — the designer converts it for you.

### Attachment name used in this flow

```
concat('Weekly Report - ', items('For_each_1')?['name'])
```

## File names

### Attachment name with timestamp (avoids overwriting files with the same name)

```
concat(formatDateTime(triggerOutputs()?['body/receivedDateTime'], 'yyyyMMdd-HHmmss'), '_', items('For_each_1')?['name'])
```

Result: `20260926-143012_invoice.pdf`

### Safe file name from subject (for saving the email)

SharePoint rejects these characters in file names: `" * : < > ? / \ |` and names with a leading or trailing space. This strips them and caps the length:

```
concat(
  formatDateTime(triggerOutputs()?['body/receivedDateTime'], 'yyyyMMdd-HHmmss'),
  '_',
  substring(
    trim(replace(replace(replace(replace(replace(replace(replace(replace(replace(
      coalesce(triggerOutputs()?['body/subject'], 'no-subject'),
      '"', ''), '*', ''), ':', ''), '<', ''), '>', ''), '?', ''), '/', '-'), '\', '-'), '|', '')),
    0,
    min(80, length(trim(replace(replace(replace(replace(replace(replace(replace(replace(replace(
      coalesce(triggerOutputs()?['body/subject'], 'no-subject'),
      '"', ''), '*', ''), ':', ''), '<', ''), '>', ''), '?', ''), '/', '-'), '\', '-'), '|', ''))))
  ),
  '.html'
)
```

> The expression editor accepts it on one line; it's broken up here for readability. A cleaner approach is to put the sanitized subject into a **Compose** action named `SafeSubject` and then use `concat(..., outputs('SafeSubject'), '.html')`.

## Dynamic folder paths

| Folder layout | Folder Path expression |
|---|---|
| By year/month | `concat('/Shared Documents/Email Attachments/', formatDateTime(utcNow(), 'yyyy/MM'))` |
| By sender address | `concat('/Shared Documents/Email Attachments/', triggerOutputs()?['body/from'])` |

SharePoint's **Create file** creates missing folders automatically.

## Skipping inline images

Signature logos show up as attachments. To skip them, add a Condition inside *For each 1* before *Get Attachment (V2)*:

| Left value | Operator | Right value |
|---|---|---|
| `items('For_each_1')?['isInline']` | is equal to | `false` |

Move *Get Attachment (V2)* and *Create file* into its **True** branch.

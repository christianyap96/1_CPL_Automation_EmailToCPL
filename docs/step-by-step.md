# Step-by-step build guide

This guide recreates the flow shown in [the overview screenshot](images/flow-overview.png).

> Tip: rename each action as you go (the **...** menu → **Rename**). Expressions reference actions by name, so if you rename an action later, update any expression that points at it. This guide uses the default names.

---

## 1. Create the flow

1. Go to [make.powerautomate.com](https://make.powerautomate.com).
2. Select **Create** → **Automated cloud flow**.
3. Name it `Email to SharePoint`.
4. Search for and choose the trigger **When a new email arrives (V3)** (Office 365 Outlook).
5. Select **Create**.

## 2. Configure the trigger

Open **When a new email arrives (V3)** and set:

| Setting | Value |
|---|---|
| Folder | A dedicated Outlook folder, e.g. `Project Reports` (an Outlook rule moves matching mail here) |
| Include Attachments | `Yes` |
| Subject Filter | `Weekly Report` |
| Only with Attachments | `No` — we want emails without attachments too |

These are under **Advanced parameters** → **Show all**.

Filtering at the trigger (folder + subject) is cheaper than filtering with a Condition, because the flow doesn't run at all for non-matching emails.

> Note: the subject filter is a *contains* match, so `RE: Weekly Report` and `FW: Weekly Report` also trigger the flow. See the file-name warning in step 6.

## 3. Add the first Condition (email filter)

1. Select **+** under the trigger → **Add an action** → **Condition** (Control).
2. Set up a rule that decides which emails get processed. Common examples:

| Left value | Operator | Right value |
|---|---|---|
| `From` (dynamic content) | contains | `@yourcompany.com` |
| `Subject` (dynamic content) | contains | `Invoice` |

3. Leave the **False** branch empty (0 actions) — emails that don't match are ignored.

## 4. Add Condition 2 (has attachments?)

Inside the **True** branch of *Condition*:

1. Select **+** → **Condition**. It will be named *Condition 2*.
2. Set the rule:

| Left value | Operator | Right value |
|---|---|---|
| `Has Attachment` (dynamic content) | is equal to | `true` (enter via the expression editor) |

## 5. True branch — save each attachment

### 5a. For each

1. In the **True** branch of *Condition 2*, add **Apply to each** (Control). It will be named *For each 1*.
2. In **Select an output from previous steps**, choose `Attachments` from the trigger's dynamic content.

### 5b. Get Attachment (V2)

Inside *For each 1*, add **Get Attachment (V2)** (Office 365 Outlook):

| Field | Value |
|---|---|
| Message Id | `Message Id` (trigger dynamic content) |
| Attachment Id | expression: `items('For_each_1')?['id']` |

### 5c. Create file

Below *Get Attachment (V2)*, add **Create file** (SharePoint):

| Field | Value |
|---|---|
| Site Address | Your SharePoint site |
| Folder Path | e.g. `/<Project>/Deliverables/Weekly Activity Report` |
| File Name | Type `Weekly Report - ` then insert **Name** (dynamic content from the attachment) |
| File Content | **contentBytes** (dynamic content from *Get Attachment (V2)*) |

Picking **contentBytes** from dynamic content works as-is — the designer handles the conversion to binary. Only if you type it as a raw expression do you need `base64ToBinary(...)` (see [expressions.md](expressions.md)).

> Heads-up: every week's attachment gets a similar name. If two reports ever share the same attachment name, they collide in the same folder. Adding a date to the name avoids that — see [expressions.md](expressions.md).

## 6. False branch — save the email itself

In the **False** branch of *Condition 2*, add **Create file** (SharePoint). It will be named *Create file 1*.

| Field | Value |
|---|---|
| Site Address | Your SharePoint site |
| Folder Path | Same folder as the attachments, e.g. `/<Project>/Deliverables/Weekly Activity Report` |
| File Name | **Subject** (dynamic content) + `_` + a `formatDateTime(...)` expression + `.html` |
| File Content | **Body** (trigger dynamic content) |

Example **File Name** expression for the date part:

```
formatDateTime(triggerOutputs()?['body/receivedDateTime'], 'yyyy-MM-dd')
```

This stores the email as an `.html` file that opens in any browser.

> ⚠️ Replies and forwards have subjects like `RE: Weekly Report`. The `:` is not allowed in SharePoint file names, so *Create file 1* fails on those. Use the *Safe file name from subject* expression in [expressions.md](expressions.md) instead of the raw **Subject** if replies can land in the folder.

### Alternative: save the original `.eml`

To keep the full original message (headers, formatting, everything):

1. Add **Export email (V2)** (Office 365 Outlook) with **Message Id** = trigger `Message Id`.
2. Add **Create file** with:
   - **File Name**: `concat(<safe subject expression>, '.eml')`
   - **File Content**: `body('Export_email_(V2)')`

The `.eml` opens directly in Outlook.

## 7. Save and test

1. Select **Save**, then **Test** → **Manually**.
2. Send yourself two emails that match your filter: one with an attachment, one without.
3. Open the run history and confirm:
   - The attachment email took the **True** path and the files appear in SharePoint.
   - The plain email took the **False** path and an `.html` file appears.

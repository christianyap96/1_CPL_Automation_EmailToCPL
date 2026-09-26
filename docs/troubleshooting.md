# Troubleshooting

## The flow doesn't trigger

- Confirm the trigger's **Folder** matches where the email actually lands (Outlook rules may move it first).
- The V3 trigger polls, so it can take a few minutes to fire.
- Check that the Outlook connection is still authenticated (**My flows** → the flow → **Connections**).

## Files in SharePoint are corrupt or won't open

- If you chose **contentBytes** from dynamic content, it should work as-is.
- If you typed the value as an expression, wrap it in `base64ToBinary(...)`:

```
base64ToBinary(body('Get_Attachment_(V2)')?['contentBytes'])
```

## "A file with the name ... already exists"

Two emails sent attachments with the same name. Prefix the file name with a timestamp — see [expressions.md](expressions.md#attachment-name-with-timestamp-avoids-overwriting-files-with-the-same-name).

## "The file name is invalid" / 400 Bad Request on Create file

Most often caused by a reply or forward: `RE: Weekly Report` contains a `:`, which SharePoint doesn't allow. Use the *Safe file name from subject* expression in [expressions.md](expressions.md).

## Signature images (image001.png) keep getting saved

Those are inline attachments. See *Skipping inline images* in [expressions.md](expressions.md).

## Emails with attachments take the False branch

- Make sure **Include Attachments** is set to `Yes` on the trigger.
- In *Condition 2*, the right value must be the boolean `true` entered through the **Expression** tab, not the text "true".

## Large attachments fail

Very large attachments can exceed connector limits. Check the current limits for the Office 365 Outlook and SharePoint connectors in Microsoft's documentation, and consider adding a Condition that skips files above a size threshold using `items('For_each_1')?['size']`.

## Access denied on SharePoint

The account that owns the SharePoint connection needs at least Contribute permission on the target library.

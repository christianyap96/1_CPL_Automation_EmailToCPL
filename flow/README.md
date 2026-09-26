# Flow files

## definition-reference.json

A human-readable reference of the flow's trigger and actions, in the same shape you see in the designer's **Code view** (**...** on an action → **Peek code**). It's meant for reading and comparing, **not** for direct import.

## Exporting the real flow package

To share an importable copy of your flow:

1. Open the flow in Power Automate.
2. Select **Export** → **Package (.zip)**.
3. Give the package a name and set **Import setup** for each connection to *Select during import*.
4. Download the `.zip` and commit it to this folder, e.g. `flow/EmailToSharePoint.zip`.

Before committing, confirm the package doesn't expose anything you don't want public — site URLs, email addresses in conditions, or internal folder names. Replace them with placeholders if the repo is public.

## Importing

1. In Power Automate, go to **My flows** → **Import** → **Import Package (Legacy)**.
2. Upload the `.zip`.
3. Map each connection (Outlook, SharePoint) to your own account.
4. Select **Import**, then open the flow and update the SharePoint site and folder paths.

For team or production use, consider putting the flow in a **Solution** instead, which supports environment variables for the site URL and folder path.

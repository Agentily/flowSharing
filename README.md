# flowSharing

Shares each **Met** record with its Business Unit's **Read** group. It's built with one record-triggered flow and no Apex.

Every Business Unit has two public groups, `<BU Name> - Read` and `<BU Name> - Write`. When a Met record's Business Unit is set or changed, the `Met_BU_Sharing` flow:

1. deletes the record's previous BU share
2. shares the record with `<new BU Name> - Read` (Read access)

## Components

| Component | Type | Purpose |
|---|---|---|
| `Met_BU_Sharing` | Flow (after save on `Met__c`) | Does the sharing |
| `Met__c` | Custom object | Org-wide default **Private**, lookup `Business_Unit__c` |
| `Met__c.BU_Read__c` | Sharing reason | `RowCause` on every share the flow manages |
| `Business_Unit__c` | Custom object | The BU. Its **Name** must match the group names. |
| `Met_Access` | Permission set | Object and field access to Met and Business Unit |

## Deploy

```bash
sf project deploy start -x manifest/package.xml --target-org <alias>
sf org assign permset -n Met_Access --target-org <alias>
```

## Behaviour

- Runs only when BU is set on create, or when BU changes. Other edits don't trigger it.
- Clearing BU removes the share.
- Only shares with reason `BU_Read__c` are deleted. Owner, manual and sharing-rule shares are never touched.
- If `<BU Name> - Read` doesn't exist, the save is **blocked** with an error on the Business Unit field, so a Met is never saved silently unshared.
- A Business Unit can't be deleted while Met records use it (lookup is set to Restrict).
- Bulk-safe: tested with 250 records inserted and then moved to another BU through the Bulk API.

## Limits

- The group is matched by its label (`Name`), exactly `<BU Name> - Read`.
- Renaming a Business Unit doesn't reshare existing Met records. The flow only runs when a Met record changes.

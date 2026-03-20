# Entity Model — Inheritance & Versions

## Current Model Version

`TTKit 61` — defined in `TCModel/Sources/Resources/DataModels/TTKit.xcdatamodeld/TTKit 61.xcdatamodel/contents`.
All entities use `sourceLanguage="Objective-C"` and `representedClassName` with the `TCModel.` module prefix.

## Inheritance Tree

```
Party (Abstract, isAbstract="YES") — TCModel.TTParty
  ├── User — TCModel.TTUser
  │     ├── Patient — TCModel.TTPatient           (parentEntity="User")
  │     ├── PatientContact — TCModel.TTPatientContact (parentEntity="User")
  │     └── Role — TCModel.TTRole                 (parentEntity="User")
  └── Group — TCModel.TTGroup                     (parentEntity="Party")
        └── RoleGroup — TCModel.TTRoleGroup       (parentEntity="Group")
```

## Why This Causes Merge Conflicts

Core Data stores the entire `Party` hierarchy in **one SQLite table** (Z_PARTY) using single table inheritance. Every `User`, `Role`, `Patient`, `Group`, and `RoleGroup` row lives in the same table, distinguished by the hidden `Z_ENT` discriminator column.

Consequences:
- Concurrent writes from different contexts collide at the SQLite row level on Z_PARTY
- Inserts into `User` and inserts into `Role` fight for the same table lock
- `NSMergeConflict` with conflicting `cachedRow` / `databaseRow` snapshots

## Non-Inherited Entities (own tables)

Each of these has no parent entity and maps to its own SQLite table:

`AlertEvent`, `ApiRequest`, `AttachmentDescriptor`, `CallLog`, `Contact`, `DetectedData`, `DownloadItem`, `EHREvent`, `EHRMeasurement`, `Escalation`, `EscalationPolicy`, `Message`, `MessageStatus`, `MessageTemplate`, `Metadata`, `Organization`, `OrganizationAccount`, `Reaction`, `RosterEntry`, `ScheduleMessage`, `SettingItem`, `Shift`, `Tag`, `TCPbxLineInfo`, `TCUserCustomField`, `Team`, `TeamRequest`, `UploadItem`

**Heavy-write entities** (highest concurrency risk): `Message`, `RosterEntry`, `MessageStatus`, `UploadItem`, `DownloadItem`.

## Model Version History

| Version | Mapping Model |
|---|---|
| 55 → 56 | `Model_TT_55_to_56.xcmappingmodel` |
| 56 → 57 | `Model_TT_56_to_57.xcmappingmodel` |
| 57 → 58 | `Model_TT_57_to_58.xcmappingmodel` |
| 58 → 59 | `Model_TT_58_to_59.xcmappingmodel` |
| 59 → 60 | `Model_TT_59_to_60.xcmappingmodel` |
| 60 → 61 | `Model_TT_60_to_61.xcmappingmodel` |

All mapping models use `.xcmappingmodel` / `xcmapping.xml` format.

## Migration Impact of the Party Hierarchy

Adding an attribute to any `Party` subclass (`User`, `Role`, etc.) adds a column to Z_PARTY regardless of which subclass uses it. Lightweight migration handles additive changes automatically. Renaming or deleting inherited attributes requires a custom mapping model. When creating a new mapping model, follow the existing `Model_TT_60_to_61` pattern.

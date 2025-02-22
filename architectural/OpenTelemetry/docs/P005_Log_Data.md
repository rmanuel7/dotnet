```json
{
    "Resource": {
      "attributes": [
        {
          "key": "telemetry.sdk.name",
          "value": { "stringValue": "opentelemetry" }
        },
        {
          "key": "telemetry.sdk.language",
          "value": { "stringValue": "dotnet" }
        },
        {
          "key": "telemetry.sdk.version",
          "value": { "stringValue": "1.11.1" }
        },
        {
          "key": "service.name",
          "value": { "stringValue": "unknown_service:Aspire.OpenTelemetryApp" }
        }
      ]
    },
    "ScopeLogs": [
      {
        "scope": { "name": "Microsoft.Hosting.Lifetime" },
        "logRecords": [...]
      }
    ],
    "SchemaUrl": ""
  }
}
```

---

```json
{
  "ResourceLogs": {
    "_unknownFields": "",
    "resource_": {
      "attributes": [
        {
          "key": "telemetry.sdk.name",
          "value": { "stringValue": "opentelemetry" }
        },
        {
          "key": "telemetry.sdk.language",
          "value": { "stringValue": "dotnet" }
        },
        {
          "key": "telemetry.sdk.version",
          "value": { "stringValue": "1.11.1" }
        },
        {
          "key": "service.name",
          "value": { "stringValue": "unknown_service:Aspire.OpenTelemetryApp" }
        }
      ]
    },
    "scopeLogs_": [
      {
        "scope": { "name": "Microsoft.Hosting.Lifetime" },
        "logRecords": [
          {
            "timeUnixNano": "1740176798427504300",
            "severityNumber": "SEVERITY_NUMBER_INFO",
            "severityText": "Information",
            "body": {
              "stringValue": "Now listening on: https://localhost:7058"
            },
            "attributes": [
              {
                "key": "address",
                "value": { "stringValue": "https://localhost:7058" }
              },
              {
                "key": "{OriginalFormat}",
                "value": { "stringValue": "Now listening on: {address}" }
              }
            ],
            "observedTimeUnixNano": "1740176798427504300"
          },
          {
            "timeUnixNano": "1740176798433769300",
            "severityNumber": "SEVERITY_NUMBER_INFO",
            "severityText": "Information",
            "body": {
              "stringValue": "Now listening on: http://localhost:5132"
            },
            "attributes": [
              {
                "key": "address",
                "value": { "stringValue": "http://localhost:5132" }
              },
              {
                "key": "{OriginalFormat}",
                "value": { "stringValue": "Now listening on: {address}" }
              }
            ],
            "observedTimeUnixNano": "1740176798433769300"
          },
          {
            "timeUnixNano": "1740176798484157400",
            "severityNumber": "SEVERITY_NUMBER_INFO",
            "severityText": "Information",
            "body": {
              "stringValue": "Application started. Press Ctrl+C to shut down."
            },
            "attributes": [
              {
                "key": "{OriginalFormat}",
                "value": {
                  "stringValue": "Application started. Press Ctrl+C to shut down."
                }
              }
            ],
            "observedTimeUnixNano": "1740176798484157400"
          },
          {
            "timeUnixNano": "1740176798486685200",
            "severityNumber": "SEVERITY_NUMBER_INFO",
            "severityText": "Information",
            "body": {
              "stringValue": "Hosting environment: Development"
            },
            "attributes": [
              {
                "key": "EnvName",
                "value": { "stringValue": "Development" }
              },
              {
                "key": "{OriginalFormat}",
                "value": { "stringValue": "Hosting environment: {EnvName}" }
              }
            ],
            "observedTimeUnixNano": "1740176798486685200"
          },
          {
            "timeUnixNano": "1740176798488609600",
            "severityNumber": "SEVERITY_NUMBER_INFO",
            "severityText": "Information",
            "body": {
              "stringValue": "Content root path: C:\\Users\\ADMIN\\Documents\\X-GitHub\\Aspire\\Aspire.Dashboard\\src\\OpenTelemetry"
            },
            "attributes": [
              {
                "key": "ContentRoot",
                "value": {
                  "stringValue": "C:\\Users\\ADMIN\\Documents\\X-GitHub\\Aspire\\Aspire.Dashboard\\src\\OpenTelemetry"
                }
              },
              {
                "key": "{OriginalFormat}",
                "value": { "stringValue": "Content root path: {ContentRoot}" }
              }
            ],
            "observedTimeUnixNano": "1740176798488609600"
          }
        ]
      }
    ],
    "schemaUrl_": ""
}
```

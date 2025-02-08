# [Internal](https://learn.microsoft.com/en-us/dotnet/aspire/app-host/configuration#internal)

Internal settings are used by the app host and integrations. Internal settings aren't designed to be configured directly.


|Name|Value|
|--|--|
ASPNETCORE_ENVIRONMENT|Development
ASPNETCORE_URLS|http://localhost:18888
DASHBOARD__FRONTEND__AUTHMODE|BrowserToken
DASHBOARD__FRONTEND__BROWSERTOKEN|f5c26e832a07335b214fd2f18b45ae98
DASHBOARD__OTLP__AUTHMODE|ApiKey
DASHBOARD__OTLP__PRIMARYAPIKEY|e35d30b6e0ab8b69dcd5c4eaefc0813e
DASHBOARD__RESOURCESERVICECLIENT__APIKEY|llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA
DASHBOARD__RESOURCESERVICECLIENT__AUTHMODE|ApiKey
DOTNET_DASHBOARD_OTLP_ENDPOINT_URL|http://localhost:18889
DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL|http://localhost:18890
DOTNET_RESOURCE_SERVICE_ENDPOINT_URL|http://localhost:18891
LOGGING__CONSOLE__FORMATTERNAME|json

 


```json
{
    "$schema": "https://json.schemastore.org/launchsettings.json",
    "profiles": {
        "https": {
            "commandName": "Project",
            "dotnetRunMessages": true,
            "launchBrowser": true,
            "applicationUrl": "https://localhost:18887;http://localhost:18888",
            "environmentVariables": {
                "ASPNETCORE_ENVIRONMENT": "Development",
                "DOTNET_ENVIRONMENT": "Development",
                "DOTNET_DASHBOARD_OTLP_ENDPOINT_URL": "https://localhost:18889",
                "DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL": "https://localhost:18890",
                "DOTNET_RESOURCE_SERVICE_ENDPOINT_URL": "https://localhost:18891",
                "DOTNET_ASPIRE_SHOW_DASHBOARD_RESOURCES": "true"
                // "DOTNET_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS": "true"
            }
        },
        "http": {
            "commandName": "Project",
            "dotnetRunMessages": true,
            "launchBrowser": false,
            "applicationUrl": "http://localhost:18888",
            "environmentVariables": {
                "ASPNETCORE_ENVIRONMENT": "Development",
                "DOTNET_ENVIRONMENT": "Development",
                "DOTNET_DASHBOARD_OTLP_ENDPOINT_URL": "http://localhost:18889",
                "DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL": "http://localhost:18890",
                "DOTNET_RESOURCE_SERVICE_ENDPOINT_URL": "http://localhost:18891",
                "DOTNET_ASPIRE_SHOW_DASHBOARD_RESOURCES": "true",
                "ASPIRE_ALLOW_UNSECURED_TRANSPORT": "true",
                "AppHost__OtlpApiKey": "llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA",
                "AppHost__ResourceService__AuthMode": "ApiKey",
                "AppHost__ResourceService__ApiKey": "llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA"
                // "DOTNET_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS": "true",
                // "DOTNET_DASHBOARD_RESOURCESERVICE_APIKEY": "llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA"
            }
        }
    }
}
```

```json
{
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft.AspNetCore": "Warning",
            "Aspire.Hosting.Dcp": "Warning"
        }
    },
    "AppHost": {
        "OtlpApiKey": "llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA",
        "ResourceService": {
            "AuthMode": "ApiKey",
            "ApiKey": "llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA"
        }
    }
}
```


```powershell
ZES_ENABLE_SYSMAN = 1
windir = C:\WINDOWS
VS_Perf_Session_GCHeapCount = 2
VSSKUEDITION = Community
VsPerMonitorDpiAwarenessEnabled.9748 = TRUE
VSLANG = 3082
VSAPPIDNAME = devenv.exe
VSAPPIDDIR = C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\
VisualStudioVersion = 17.0
VisualStudioEdition = Microsoft Visual Studio Community 2022
VisualStudioDir = C:\Users\ADMIN\Documents\Visual Studio 2022
USERPROFILE = C:\Users\ADMIN
USERNAME = ADMIN
USERDOMAIN_ROAMINGPROFILE = DESKTOP-DOJVO80
USERDOMAIN = DESKTOP-DOJVO80
TOOL_HOST_SERVICE_URL = https://localhost:62462
TOOL_HOST_SERVICE_TOKEN = gV74Y7Hzr67wAuq//nLaXQ==
TOOL_HOST_SERVICE_CERTIFICATE = MIIDYDCCAhigAwIBAgIIZj1mWM3jaGswPQYJKoZIhvcNAQEKMDCgDTALBglghkgBZQMEAgGhGjAYBgkqhkiG9w0BAQgwCwYJYIZIAWUDBAIBogMCASAwMzExMC8GA1UEAxMoZGVidWctc2Vzc2lvbi52aXN1YWxzdHVkaW8ubWljcm9zb2Z0LmNvbTAeFw0yNTAyMDcxODUyMzBaFw0yNTAyMTQxODUyMzVaMDMxMTAvBgNVBAMTKGRlYnVnLXNlc3Npb24udmlzdWFsc3R1ZGlvLm1pY3Jvc29mdC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCuIlgC8VRuIwniqzlACWDQJaZI2+xlavcPcy1/bdCKm5vq7SfAOOpCmwvDdjJooHRVARNkmXQ0vcGrsHKRas0oYNeEtikMUQXb8nhrq4k4i0loT9CiKQOngb4WJaWhH3roLA1ximntFFrwRPWTaSKZJEDe8I5AmP3HxmDRaQZnj8Z1VZjRWusNhBnRoNcskvJrdOZbOZvt54+A0Q681vOgMoe9amaO9jzdU6EG0T6Jhc0lTqv9FFVkX0nKtjs+6xhRi3b17KT7kMMjwr9CwvIVU7dLPvuJaaMzxama8Nxrrpp3v864K1w+MLszj2sVGG2oQviO+R8OUcTXkQutK0g5AgMBAAGjGDAWMBQGA1UdEQQNMAuCCWxvY2FsaG9zdDA9BgkqhkiG9w0BAQowMKANMAsGCWCGSAFlAwQCAaEaMBgGCSqGSIb3DQEBCDALBglghkgBZQMEAgGiAwIBIAOCAQEAhW/Tko8o0xf/hlVguzNQkJA3gg7381Fz2EGBVbOZpmF1IQ+jBnpVzUl6smDk18GNCw8yr5+FbylyMzhxyQDBU8WX2jLgDMO3EsEXkioMl26X/1+cLptcRRHOBiqQ6VmTbDZ6oakLD/Be8BRTLlBRj6ktLQqANcjN6PKPD3moILD+j1YK8RBEiNIkYecNt7Mf8VAR1CkIL7DTBIoYsAgroe5zAFd8psR8CePwoBb0QhrtIm1kuCUp1XMna49uLdTDfp2I+USF5ffA2Ay32+czppAvCT4P/kODLWSUiJoN8Bh/7jauE4MNcOBTkGopq+ljTgbyDufhRlJkvheR1v8Njg==
TMP = C:\Users\ADMIN\AppData\Local\Temp
ThreadedWaitDialogDpiContext = -4
TEMP = C:\Users\ADMIN\AppData\Local\Temp
SystemRoot = C:\WINDOWS
SystemDrive = C:
SESSIONNAME = Console
ServiceHubLogSessionKey = 4AF0948B
RESOURCE_SERVICE_ENDPOINT_URL = http://localhost:18891
PUBLIC = C:\Users\Public
PSModulePath = C:\Program Files\WindowsPowerShell\Modules;C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules
ProgramW6432 = C:\Program Files
ProgramFiles(x86) = C:\Program Files (x86)
ProgramFiles = C:\Program Files
ProgramData = C:\ProgramData
PROCESSOR_REVISION = 9e09
PROCESSOR_LEVEL = 6
PROCESSOR_IDENTIFIER = Intel64 Family 6 Model 158 Stepping 9, GenuineIntel
PROCESSOR_ARCHITECTURE = AMD64
PkgDefApplicationConfigFile = C:\Users\ADMIN\AppData\Local\Microsoft\VisualStudio\17.0_70ee7364\devenv.exe.config
PATHEXT = .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
Path = C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;C:\Program Files\dotnet\;C:\Program Files\Microsoft SQL Server\150\Tools\Binn\;C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\Tools\Binn\;C:\Program Files\Git\cmd;C:\Program Files\Docker\Docker\resources\bin;C:\Users\ADMIN\AppData\Local\Microsoft\WindowsApps;C:\Users\ADMIN\.dotnet\tools
OS = Windows_NT
OneDrive = C:\Users\ADMIN\OneDrive
NUMBER_OF_PROCESSORS = 4
MSBuildLoadMicrosoftTargetsReadOnly = true
LOGONSERVER = \\DESKTOP-DOJVO80
Logging =
Logging:LogLevel =
Logging:LogLevel:Microsoft.AspNetCore = Warning
Logging:LogLevel:Default = Information
Logging:LogLevel:Aspire.Hosting.Dcp = Warning
LOCALAPPDATA = C:\Users\ADMIN\AppData\Local
LAUNCH_PROFILE = http
IIS_USER_HOME = C:\Users\ADMIN\Documents\IISExpress
IIS_SITES_HOME = C:\Users\ADMIN\Documents\My Web Sites
IIS_DRIVE = C:
IIS_BIN = C:\Program Files\IIS Express
HOMEPATH = \Users\ADMIN
HOMEDRIVE = C:
GCExpConfigUsedInSession = 3
FPS_BROWSER_USER_PROFILE_STRING = Default
FPS_BROWSER_APP_PROFILE_STRING = Internet Explorer
ENVIRONMENT = Development
ENABLE_XAML_DIAGNOSTICS_SOURCE_INFO = 1
DriverData = C:\Windows\System32\Drivers\DriverData
DOTNET_RESOURCE_SERVICE_ENDPOINT_URL = http://localhost:18891
DOTNET_LAUNCH_PROFILE = http
DOTNET_ENVIRONMENT = Development
DOTNET_DASHBOARD_URL = http://localhost:18888
DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL = http://localhost:18890
DOTNET_DASHBOARD_OTLP_ENDPOINT_URL = http://localhost:18889
DOTNET_DASHBOARD_FRONTEND_BROWSERTOKEN = f0962d5f23c6b45fcf7b0da5a8d7f98d
DOTNET_ASPIRE_SHOW_DASHBOARD_RESOURCES = true
DEBUG_SESSION_TOKEN = VcsA6+lsYSsoDhEy35V3hw==
DEBUG_SESSION_SERVER_CERTIFICATE = MIIDYTCCAhmgAwIBAgIJAJfsRl0WQPheMD0GCSqGSIb3DQEBCjAwoA0wCwYJYIZIAWUDBAIBoRowGAYJKoZIhvcNAQEIMAsGCWCGSAFlAwQCAaIDAgEgMDMxMTAvBgNVBAMTKGRlYnVnLXNlc3Npb24udmlzdWFsc3R1ZGlvLm1pY3Jvc29mdC5jb20wHhcNMjUwMjA3MTg1MjMwWhcNMjUwMjE0MTg1MjM1WjAzMTEwLwYDVQQDEyhkZWJ1Zy1zZXNzaW9uLnZpc3VhbHN0dWRpby5taWNyb3NvZnQuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtvYmeR0gSfOOmstl0PBloQaJcFHbyK0HQyK3XaEc4gojSLQ1Gcvq+X1ZBURXgsv3VgSKj7eGKJ+SXJYte8RYPUa+FadST8Ip8CK5MMcQX/Gjbs7W1mz9XKSjfYHs4Bw2DClQKGsxMeNs1mRXxwPLUZEeFPIZx4sxS4qzIeN7HIn//ky5eRdlLOLW8l56HKL/MXW4BbH9sOeLXC+Xh7L5ub9Ao1ZxawzELsC4NglSGShlv5T0cbu4M/iA0m+EcC8pF0gIjExxM7865SKyT6Lp8jxYcEEY9uH7SxC5pcoUx0f7++CJ0tOVt2wj9XPVjnO0CzDCqhl6WDdzElsopegsNQIDAQABoxgwFjAUBgNVHREEDTALgglsb2NhbGhvc3QwPQYJKoZIhvcNAQEKMDCgDTALBglghkgBZQMEAgGhGjAYBgkqhkiG9w0BAQgwCwYJYIZIAWUDBAIBogMCASADggEBAJxX/x2cCF/NFvY82Xg+aji7KahWCg5O9g4fhONW/V0Hga7SV2G47RCI9HhtSS2tz+FnNcI2+PqX2uoAWH5oZbwWEWDGAy6QeTYV8aLrmRCyPUM7Ci1iZZ0nmTCC44gPkKkx7C3pzlGj5pJ8bZ+21pVANACvV+rk5Bv7hE99p51stdzMznaz9qP8ngM21hDjCnkZwszwIAMx4NGCRM7Q+c4JXWQAzyR55DmFds49JY25+nB5Ch+8RUHlGnU22I2gFx4OuGTnvtium28mPr9zll7EpnJ2oeL52lwf4fkcBLPKfr0VUSOo74Y7Pl1NgqGuvN0YPgcS+QRRvu/mwvPQBWc=
DEBUG_SESSION_PORT = localhost:62461
DCP_INSTANCE_ID_PREFIX = MonitoringUI.AppHost;
DASHBOARD_URL = http://localhost:18888
DASHBOARD_OTLP_HTTP_ENDPOINT_URL = http://localhost:18890
DASHBOARD_OTLP_ENDPOINT_URL = http://localhost:18889
DASHBOARD_FRONTEND_BROWSERTOKEN = f0962d5f23c6b45fcf7b0da5a8d7f98d
contentRoot = C:\Users\ADMIN\Documents\Aspire\source\MonitoringUI\MonitoringUI.AppHost
ComSpec = C:\WINDOWS\system32\cmd.exe
COMPUTERNAME = DESKTOP-DOJVO80
CommonProgramW6432 = C:\Program Files\Common Files
CommonProgramFiles(x86) = C:\Program Files (x86)\Common Files
CommonProgramFiles = C:\Program Files\Common Files
ASPNETCORE_URLS = http://localhost:18888
ASPNETCORE_ENVIRONMENT = Development
ASPIRE_SHOW_DASHBOARD_RESOURCES = true
ASPIRE_ALLOW_UNSECURED_TRANSPORT = true
AppHost =
AppHost:Sha256 = 81B884BFAD3BB75A4A98733A17861388A9B120D85B2851CE7C5A1841043E4DBC
AppHost:ResourceService =
AppHost:ResourceService:AuthMode = ApiKey
AppHost:ResourceService:ApiKey = 729ab61d66099620928cb19ffca6efac
AppHost:Path = C:\Users\ADMIN\Documents\Aspire\source\MonitoringUI\MonitoringUI.AppHost\MonitoringUI.AppHost.csproj
AppHost:OtlpApiKey = 49c368104c14556d43f0aa19c86e718a
AppHost:Directory = C:\Users\ADMIN\Documents\Aspire\source\MonitoringUI\MonitoringUI.AppHost
AppHost:BrowserToken = f0962d5f23c6b45fcf7b0da5a8d7f98d
APPDATA = C:\Users\ADMIN\AppData\Roaming
ALLUSERSPROFILE = C:\ProgramData
info: Aspire.Hosting.DistributedApplication[0]
      Aspire version: 9.0.0+01ed51919f8df692ececce51048a140615dc759d
info: Aspire.Hosting.DistributedApplication[0]
      Distributed application starting.
info: Aspire.Hosting.DistributedApplication[0]
      Application host directory is: C:\Users\ADMIN\Documents\Aspire\source\MonitoringUI\MonitoringUI.AppHost
info: Aspire.Hosting.DistributedApplication[0]
      Now listening on: http://localhost:18888
info: Aspire.Hosting.DistributedApplication[0]
      Login to the dashboard at http://localhost:18888/login?t=f0962d5f23c6b45fcf7b0da5a8d7f98d
info: Aspire.Hosting.DistributedApplication[0]
      Distributed application started. Press Ctrl+C to shut down.
```

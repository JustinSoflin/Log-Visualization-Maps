# Log-Visualization-Maps

This project uses Azure log queries combined with custom geolocation data to visualize various security events—such as successful and failed authentications, resource creation, VM logon failures, and malicious network traffic—on a map. By mapping IP addresses to geographic locations, it helps uncover patterns, detect anomalies, and identify potential threat sources across the environment. These visualizations support threat hunting, security monitoring, and incident response by providing clear geographic context to raw log data.

---

# Table of Contents

- [Entra ID (Azure) Authentication Success](#entra-id-azure-authentication-success)
- [Entra ID (Azure) Authentication Failures](#entra-id-azure-authentication-failures)
- [Azure Resource Creation](#azure-resource-creation)
- [VM Authentication Failures](#vm-authentication-failures)
- [Malicious Traffic Entering the Network](#malicious-traffic-entering-the-network)

---

### Entra ID (Azure) Authentication Success

This query visualizes successful Entra ID (Azure AD) sign-ins by mapping user login locations based on geolocation data. It helps identify where users are authenticating from globally, offering insight into access patterns and potential anomalies. Ideal for use in threat hunting dashboards or security monitoring.

**Query used to locate events:**

```kql
SigninLogs
| where ResultType == 0
| summarize LoginCount = count() by Identity, Latitude = tostring(LocationDetails["geoCoordinates"]["latitude"]), Longitude = tostring(LocationDetails["geoCoordinates"]["longitude"]), City = tostring(LocationDetails["city"]), Country = tostring(LocationDetails["countryOrRegion"])
| project Identity, Latitude, Longitude, City, Country, LoginCount, friendly_label
 = strcat(Identity, " - ", City, ", ", Country)
```
<kbd>
<img width="1500" alt="image" src="https://github.com/user-attachments/assets/3f3d6cbf-b5d4-46b8-9cc4-876a863c1f84" />
</kbd>

[MAP](https://github.com/JustinSoflin/Log-Visualization-Maps/blob/main/my%20workbook.json)

---

### Entra ID (Azure) Authentication Failures

This query maps failed Entra ID (Azure AD) sign-in attempts by geolocation, excluding service accounts and focusing on user identities. It highlights the users and locations with the highest number of failed logins, helping to identify suspicious access patterns or brute-force activity. Useful for visual threat analysis and anomaly detection in security dashboards.

**Query used to locate events:**

```kql
SigninLogs
| where ResultType != 0 and Identity !contains "-"
| summarize LoginCount = count() by Identity, Latitude = tostring(LocationDetails["geoCoordinates"]["latitude"]), Longitude = tostring(LocationDetails["geoCoordinates"]["longitude"]), City = tostring(LocationDetails["city"]), Country = tostring(LocationDetails["countryOrRegion"])
| order by LoginCount desc
| project Identity, Latitude, Longitude, City, Country, LoginCount, friendly_label = strcat(Identity, " - ", City, ", ", Country)
```
<kbd>
<img width="1500" alt="image" src="https://github.com/user-attachments/assets/ad7f152a-704f-472d-ab11-c461b269e8f1" />
</kbd>

---

### Azure Resource Creation

This query identifies successful Azure resource creation events and maps them to geolocations using a custom GeoIP watchlist. It filters out service principals and focuses on user-initiated actions from valid IPv4 addresses. The result helps visualize where resource deployments are originating from, supporting audit, compliance, and insider threat detection efforts.

**Query used to locate events:**

```kql
// Only works for IPv4 Addresses
let GeoIPDB_FULL = _GetWatchlist("geoip");
let AzureActivityRecords = AzureActivity
| where not(Caller matches regex @"^[{(]?[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}[)}]?$")
| where CallerIpAddress matches regex @"\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b"
| where OperationNameValue endswith "WRITE" and (ActivityStatusValue == "Success" or ActivityStatusValue == "Succeeded")
| summarize ResouceCreationCount = count() by Caller, CallerIpAddress;
AzureActivityRecords
| evaluate ipv4_lookup(GeoIPDB_FULL, CallerIpAddress, network)
| project Caller, 
          CallerPrefix = split(Caller, "@")[0],  // Splits Caller UPN and takes the part before @
          CallerIpAddress, 
          ResouceCreationCount, 
          Country = countryname, 
          Latitude = latitude, 
          Longitude = longitude, 
          friendly_label = strcat(split(Caller, "@")[0], " - ", cityname, ", ", countryname)
```
<kbd>
<img width="1500" alt="image" src="https://github.com/user-attachments/assets/0ab83d5d-30f8-4685-a338-892255eb6200" />
</kbd>

---

### VM Authentication Failures

This query maps failed logon attempts to virtual machines by correlating remote IPs with geolocation data from a custom GeoIP watchlist. It highlights the source locations of failed authentication events, helping detect brute-force attacks or unauthorized access attempts. Useful for visualizing potential threat origins in security dashboards.

**Query used to locate events:**

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
DeviceLogonEvents
| where ActionType == "LogonFailed"
| order by TimeGenerated desc
| evaluate ipv4_lookup(GeoIPDB_FULL, RemoteIP, network)
| summarize LoginAttempts = count() by RemoteIP, City = cityname, Country = countryname, friendly_location = strcat(cityname, " (", RemoteIP, ")"), Latitude = latitude, Longitude = longitude;
```
<kbd>
<img width="1500" alt="image" src="https://github.com/user-attachments/assets/b9bdeda8-bd60-4d36-83aa-edc4b7fff6f7" />
</kbd>

---

### Malicious Traffic Entering the Network

This query identifies and maps malicious network flows entering the environment by correlating source IPs with geolocation data. It leverages Azure Network Watcher logs and a custom GeoIP watchlist to provide context around the origin of threats, including protocol, port, and matched NSG rules. Ideal for network threat detection and visual analysis of potentially harmful traffic.

**Query used to locate events:**

```kql
let GeoIPDB_FULL = _GetWatchlist("geoip");
let MaliciousFlows = AzureNetworkAnalytics_CL 
| where FlowType_s == "MaliciousFlow"
//| where SrcIP_s == "10.0.0.5" 
| order by TimeGenerated desc
| project TimeGenerated, FlowType = FlowType_s, IpAddress = SrcIP_s, DestinationIpAddress = DestIP_s, DestinationPort = DestPort_d, Protocol = L7Protocol_s, NSGRuleMatched = NSGRules_s;
MaliciousFlows
| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)
| project TimeGenerated, FlowType, IpAddress, DestinationIpAddress, DestinationPort, Protocol, NSGRuleMatched, latitude, longitude, city = cityname, country = countryname, friendly_location = strcat(cityname, " (", countryname, ")")
```

<kbd>
<img width="1500" alt="image" src="https://github.com/user-attachments/assets/32c1fa7c-ffba-45af-b631-15003ec7bd01" />
</kbd>

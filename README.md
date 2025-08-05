# Log-Visualization-Maps

**Query used to locate events:**

```kql
SigninLogs
| where ResultType == 0
| summarize LoginCount = count() by Identity,
Latitude = tostring(LocationDetails["geoCoordinates"]["latitude"]),
Longitude = tostring(LocationDetails["geoCoordinates"]["longitude"]),
City = tostring(LocationDetails["city"]),
Country = tostring(LocationDetails["countryOrRegion"])
| project Identity, Latitude, Longitude, City, Country, LoginCount, friendly_label
 = strcat(Identity, " - ", City, ", ", Country)
```

<img width="1561" height="825" alt="image" src="https://github.com/user-attachments/assets/3f3d6cbf-b5d4-46b8-9cc4-876a863c1f84" />

[MAP](https://github.com/JustinSoflin/Log-Visualization-Maps/blob/main/my%20workbook.json)

---

**Query used to locate events:**


```kql
SigninLogs
| where ResultType != 0 and Identity !contains "-"
| summarize LoginCount = count() by Identity, Latitude = tostring(LocationDetails["geoCoordinates"]["latitude"]), Longitude = tostring(LocationDetails["geoCoordinates"]["longitude"]), City = tostring(LocationDetails["city"]), Country = tostring(LocationDetails["countryOrRegion"])
| order by LoginCount desc
| project Identity, Latitude, Longitude, City, Country, LoginCount, friendly_label = strcat(Identity, " - ", City, ", ", Country)
```

<img width="1647" height="908" alt="image" src="https://github.com/user-attachments/assets/ad7f152a-704f-472d-ab11-c461b269e8f1" />


---




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

<img width="1824" height="930" alt="image" src="https://github.com/user-attachments/assets/0ab83d5d-30f8-4685-a338-892255eb6200" />


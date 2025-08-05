# Log-Visualization-Maps

**Query used to locate events:**

```kql
SigninLogs
| where ResultType == 0
| summarize LoginCount = count() by Identity, Latitude =
tostring(LocationDetails["geoCoordinates"]["latitude"]), Longitude =
tostring(LocationDetails["geoCoordinates"]["longitude"]), City =
tostring(LocationDetails["city"]), Country =
tostring(LocationDetails["countryOrRegion"])
| project Identity, Latitude, Longitude, City, Country, LoginCount, friendly_label
 = strcat(Identity, " - ", City, ", ", Country)
```

<img width="1561" height="825" alt="image" src="https://github.com/user-attachments/assets/3f3d6cbf-b5d4-46b8-9cc4-876a863c1f84" />

[MAP](https://github.com/JustinSoflin/Log-Visualization-Maps/blob/main/my%20workbook.json)

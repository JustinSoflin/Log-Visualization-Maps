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

[MAP](https://portal.azure.com/#view/AppInsightsExtension/UsageNotebookBlade/ComponentId/%2Fsubscriptions%2F3c95e63a-895a-4386-991e-edbbf57de5c8%2Fresourcegroups%2Fcyber-range-admin-soc%2Fproviders%2Fmicrosoft.operationalinsights%2Fworkspaces%2Flaw-cyber-range/ResourceIds~/%5B%5D/TimeContext~/null/Type/sentinel/ConfigurationId~/null/NotebookParams/%7B%22Workspace%22%3A%22%2Fsubscriptions%2F3c95e63a-895a-4386-991e-edbbf57de5c8%2FresourceGroups%2Fcyber-range-admin-soc%2Fproviders%2FMicrosoft.OperationalInsights%2Fworkspaces%2Flaw-cyber-range%22%7D/Source/Sentinel/GalleryResourceType/Sentinel/WorkbookTemplateName/New%20workbook/NewNotebookData~/%7B%22version%22%3A%22Notebook%2F1.0%22%2C%22items%22%3A%5B%7B%22type%22%3A1%2C%22content%22%3A%22%7B%5C%22json%5C%22%3A%5C%22%23%23%20New%20workbook%5C%5Cn---%5C%5Cn%5C%5CnWelcome%20to%20your%20new%20workbook.%20%20This%20area%20will%20display%20text%20formatted%20as%20markdown.%5C%5Cn%5C%5Cn%5C%5CnWe've%20included%20a%20basic%20analytics%20query%20to%20get%20you%20started.%20Use%20the%20%60Edit%60%20button%20below%20each%20section%20to%20configure%20it%20or%20add%20more%20sections.%5C%22%7D%22%2C%22name%22%3A%22text%20-%202%22%7D%2C%7B%22type%22%3A3%2C%22content%22%3A%22%7B%5C%22version%5C%22%3A%5C%22KqlItem%2F1.0%5C%22%2C%5C%22query%5C%22%3A%5C%22union%20withsource%3D_TableName%20*%5C%5Cn%7C%20summarize%20Count%3Dcount()%20by%20_TableName%5C%5Cn%7C%20render%20barchart%5C%22%2C%5C%22size%5C%22%3A1%2C%5C%22queryType%5C%22%3A0%2C%5C%22resourceType%5C%22%3A%5C%22microsoft.operationalinsights%2Fworkspaces%5C%22%7D%22%2C%22name%22%3A%22query%20-%202%22%7D%5D%2C%22isLocked%22%3Afalse%2C%22styleSettings%22%3A%7B%7D%2C%22fromTemplateId%22%3A%22sentinel-UserWorkbook%22%7D/ViewerMode~/null/Location~/null)

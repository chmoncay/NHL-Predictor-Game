# Troubleshooting and FAQ

This document provides solutions to common issues and frequently asked questions for the NHL Predictor Game implementation.

## Common Issues and Solutions

### Environment and Setup Issues

#### Issue: Environment Creation Fails
```
Error: "Environment creation failed. Dataverse database provisioning error."
```

**Root Causes:**
- Insufficient licensing for Dataverse
- Regional capacity limitations
- Tenant configuration restrictions

**Solutions:**
1. **Verify Licensing**
   ```powershell
   # Check Power Platform licensing
   Get-AdminPowerAppLicenseUsers -EnvironmentName "default"
   
   # Verify Dataverse entitlements
   Get-AdminPowerAppEnvironment | Select-Object EnvironmentName, DatabaseType
   ```

2. **Check Regional Availability**
   - Select different region during environment creation
   - Verify Dataverse availability in chosen region
   - Contact Microsoft support for capacity issues

3. **Tenant Configuration**
   - Ensure tenant allows environment creation
   - Check Power Platform governance policies
   - Verify admin permissions

#### Issue: DLP Policy Blocks Required Connectors
```
Error: "This action is blocked by your organization's data loss prevention policies."
```

**Solutions:**
1. **Review DLP Policies**
   ```powershell
   # List DLP policies affecting environment
   Get-AdminDlpPolicy -EnvironmentName $environmentId
   
   # Get policy details
   Get-AdminDlpPolicy -PolicyName $policyName
   ```

2. **Update DLP Configuration**
   - Move NHL API connector to "Business" group
   - Allow HTTP connector for external data sources
   - Create exception for NHL Predictor app

3. **Alternative Approaches**
   - Use Premium connectors with proper licensing
   - Implement data sync through Power Automate
   - Use approved third-party connectors

### Dataverse Issues

#### Issue: Table Relationships Not Working
```
Error: "Lookup column cannot be created. Referenced table not found."
```

**Diagnosis Steps:**
```powerfx
// Check if target table exists
If(
    CountRows(Teams) > 0,
    "Teams table accessible",
    "Teams table missing or no access"
)

// Verify table permissions
User().SecurityRoles
```

**Solutions:**
1. **Verify Table Creation Order**
   - Create referenced tables first (Teams, Users)
   - Then create dependent tables (Games, Predictions)
   - Configure relationships after all tables exist

2. **Check Security Roles**
   - Assign appropriate read permissions on lookup tables
   - Verify user has access to both parent and child tables
   - Update security role if necessary

3. **Relationship Configuration**
   ```
   Correct Configuration:
   - Relationship Type: Many-to-one
   - Primary Table: Teams (one)
   - Related Table: Games (many)
   - Lookup Column: Team (in Games table)
   ```

#### Issue: Data Import Failures
```
Error: "Import failed. Column mapping errors detected."
```

**Common Mapping Issues:**
- Date format mismatches
- Choice column value mismatches  
- Required field missing values
- Lookup column reference errors

**Solutions:**
1. **Prepare Import Template**
   ```csv
   # Correct date format: YYYY-MM-DD HH:MM:SS
   nhl_gamedate,nhl_hometeam,nhl_awayteam,nhl_status
   2024-01-15 19:00:00,Toronto Maple Leafs,Montreal Canadiens,Scheduled
   ```

2. **Validate Choice Values**
   ```
   Valid Conference Values:
   - Eastern Conference
   - Western Conference
   
   Valid Division Values:
   - Atlantic, Metropolitan, Central, Pacific
   ```

3. **Handle Lookup Columns**
   - Import lookup tables first (Teams before Games)
   - Use exact text values that exist in lookup table
   - Consider using GUIDs for more reliable imports

### Power Apps Development Issues

#### Issue: App Performance Problems
```
Problem: App loads slowly, galleries lag, timeouts occur
```

**Performance Diagnostics:**
```powerfx
// Check data source delegation
With(
    {
        gameCount: CountRows(Games),
        delegationWarning: If(gameCount > 500, "Delegation Warning", "OK")
    },
    delegationWarning
)

// Monitor formula execution time
Trace("Loading games: " & Text(Now(), "hh:mm:ss"));
Set(gamesList, Sort(Games, nhl_gamedate));
Trace("Games loaded: " & Text(Now(), "hh:mm:ss"))
```

**Solutions:**
1. **Optimize Data Loading**
   ```powerfx
   // Use delegation-friendly formulas
   Filter(Games, nhl_gamedate >= Today())  // Good
   Filter(Games, Text(nhl_gamedate, "mm") = "01")  // Bad - not delegable
   
   // Implement caching
   OnStart:
   ClearCollect(
       cachedTeams,
       Teams  // Load once, use everywhere
   )
   ```

2. **Gallery Optimization**
   ```powerfx
   // Limit gallery items
   Items: FirstN(
       Sort(Games, nhl_gamedate),
       50  // Show only 50 recent games
   )
   
   // Use OnVisible for expensive operations
   OnVisible: If(
       IsBlank(expensiveData),
       Set(expensiveData, ComplexCalculation())
   )
   ```

3. **Reduce API Calls**
   - Use collections for frequently accessed data
   - Implement proper refresh strategies
   - Cache data that doesn't change often

#### Issue: Gen Pages Not Generating Content
```
Error: "AI content generation failed" or blank content returned
```

**Troubleshooting Steps:**
1. **Verify Prerequisites**
   ```
   Checklist:
   ☐ AI Builder license assigned
   ☐ Environment has AI Builder enabled
   ☐ Gen Pages feature activated
   ☐ Proper security roles assigned
   ```

2. **Check Prompt Configuration**
   ```powerfx
   // Validate prompt data
   If(
       IsBlank(gameData) || IsBlank(selectedTeam),
       Notify("Missing data for AI generation", NotificationType.Warning),
       // Proceed with generation
   )
   ```

3. **Test Prompt Templates**
   ```
   Simple Test Prompt:
   "Analyze the NHL game between Toronto Maple Leafs and Montreal Canadiens. 
   Provide a brief 2-sentence summary."
   
   If this works, gradually add complexity and data.
   ```

4. **Monitor AI Builder Credits**
   - Check credit consumption in Power Platform admin center
   - Verify credit allocation for environment
   - Monitor usage patterns and optimize

#### Issue: Mobile Responsiveness Problems
```
Problem: App doesn't display correctly on mobile devices
```

**Solutions:**
1. **Responsive Design Implementation**
   ```powerfx
   // Dynamic sizing
   Width: If(
       App.Width < 768,
       App.Width - 40,    // Mobile
       Min(1200, App.Width * 0.8)  // Desktop
   )
   
   // Font scaling
   Font: If(
       App.Width < 768,
       Font.'Open Sans Light',
       Font.'Open Sans'
   )
   
   Size: If(
       App.Width < 768,
       14,  // Mobile
       16   // Desktop
   )
   ```

2. **Touch-Friendly Controls**
   - Minimum button height: 44px
   - Adequate spacing between touch targets
   - Swipe gestures for navigation

3. **Testing Approach**
   - Test on actual mobile devices
   - Use browser dev tools for responsive testing
   - Validate across iOS and Android

### Power Automate Flow Issues

#### Issue: NHL API Integration Failures
```
Error: "HTTP 429 - Too Many Requests" or connection timeouts
```

**Solutions:**
1. **Implement Rate Limiting**
   ```json
   {
       "actions": {
           "Delay": {
               "type": "Wait",
               "inputs": {
                   "interval": {
                       "count": 2,
                       "unit": "Second"
                   }
               }
           }
       }
   }
   ```

2. **Add Retry Logic**
   ```json
   {
       "Get_NHL_Data": {
           "type": "Http",
           "inputs": {
               "method": "GET",
               "uri": "https://statsapi.web.nhl.com/api/v1/teams"
           },
           "retryPolicy": {
               "type": "fixed",
               "count": 3,
               "interval": "PT30S"
           }
       }
   }
   ```

3. **Error Handling**
   ```json
   {
       "scope": {
           "type": "Scope",
           "actions": {
               "Get_NHL_Data": "..."
           },
           "runAfter": {},
           "runtimeConfiguration": {
               "configure": {
                   "timeout": "PT1M"
               }
           }
       },
       "Handle_API_Error": {
           "type": "Compose",
           "inputs": "API call failed, using cached data",
           "runAfter": {
               "scope": ["Failed"]
           }
       }
   }
   ```

#### Issue: Data Sync Inconsistencies
```
Problem: Game scores not updating correctly, missing games
```

**Diagnosis:**
1. **Check Flow Run History**
   - Review failed runs in Power Automate
   - Examine error messages and stack traces
   - Verify trigger conditions

2. **Validate Data Mapping**
   ```json
   // Ensure correct field mapping
   {
       "nhl_homescore": "@{body('Parse_JSON')?['teams']?['home']?['score']}",
       "nhl_awayscore": "@{body('Parse_JSON')?['teams']?['away']?['score']}",
       "nhl_status": "@{body('Parse_JSON')?['status']?['detailedState']}"
   }
   ```

3. **Implement Data Validation**
   ```json
   {
       "condition": {
           "type": "If",
           "expression": {
               "and": [
                   {
                       "not": {
                           "equals": [
                               "@body('Parse_JSON')?['teams']?['home']?['score']",
                               null
                           ]
                       }
                   }
               ]
           },
           "actions": {
               "Update_Game": "..."
           },
           "else": {
               "actions": {
                   "Log_Error": "Score data missing, skipping update"
               }
           }
       }
   }
   ```

### Security and Access Issues

#### Issue: Users Cannot Access App
```
Error: "You don't have permission to use this app"
```

**Solutions:**
1. **Verify App Sharing**
   ```powershell
   # Check app permissions
   Get-AdminPowerApp -AppName $appId | Select-Object -ExpandProperty AppPermissions
   
   # Share app with users
   Set-AdminPowerAppRoleAssignment -AppName $appId -RoleName "CanView" -PrincipalObjectId $userId
   ```

2. **Check Security Roles**
   - Verify users have appropriate Dataverse security roles
   - Assign "NHL Predictor User" role to end users
   - Ensure role has read access to required tables

3. **Environment Access**
   - Confirm users are added to environment
   - Check environment security settings
   - Verify license assignments

#### Issue: Data Access Permissions
```
Error: "You do not have permission to read these records"
```

**Solutions:**
1. **Review Table Permissions**
   ```
   Required Permissions for NHL Predictor User Role:
   - Teams: Read (Global)
   - Games: Read (Global)  
   - Predictions: Create, Read, Write (User level)
   - TeamStats: Read (Global)
   ```

2. **Check Column-Level Security**
   - Verify field-level security settings
   - Ensure sensitive fields are properly protected
   - Update security role if needed

## Frequently Asked Questions

### General Questions

**Q: What licenses are required for the NHL Predictor Game?**

A: Minimum requirements:
- Power Apps per-user or per-app license
- Dataverse database capacity
- AI Builder credits (for Gen Pages functionality)
- Power Automate license (for automated flows)

**Q: Can the app work with other sports leagues?**

A: Yes, the architecture is adaptable. You would need to:
- Modify the data model for sport-specific requirements
- Update API integrations for the target league
- Adjust Gen Pages templates for sport-specific content
- Update team/player data structures

**Q: How do I handle different time zones for games?**

A: 
```powerfx
// Convert to user's local time
LocalGameTime: DateAdd(
    nhl_gamedate,
    -(TimeZoneOffset() / 60),
    Hours
)

// Display with time zone
Text(LocalGameTime, "dddd, mmm dd - h:mm AM/PM") & " " & 
Text(TimeZoneOffset() / -60, "UTC+0")
```

### Technical Questions

**Q: How can I improve prediction accuracy?**

A: Consider these enhancements:
- Incorporate more statistical factors (injuries, weather, etc.)
- Use AI Builder prediction models with historical data
- Implement machine learning algorithms
- Add real-time betting odds integration
- Include player-specific performance metrics

**Q: What's the maximum number of users the app can support?**

A: Scalability depends on:
- Dataverse database capacity (varies by license)
- Power Apps concurrent user limits
- API rate limiting for external data sources
- Typical usage: 100-1000 users with proper optimization

**Q: How do I backup the application and data?**

A: Backup strategy:
```powershell
# Export solution
Export-CrmSolution -SolutionName "NHLPredictorGame" -OutputFolder "C:\Backup"

# Export data
Export-CrmDataToFile -EntityName "nhl_team" -OutputFile "teams_backup.csv"
Export-CrmDataToFile -EntityName "nhl_game" -OutputFile "games_backup.csv"
```

### Development Questions

**Q: Can I customize the AI-generated content?**

A: Yes, you can:
- Modify Gen Pages prompt templates
- Create custom AI Builder models
- Implement prompt engineering best practices
- Add custom business logic to filter/enhance AI content

**Q: How do I add new statistics or metrics?**

A: 
1. Add new columns to Dataverse tables
2. Update Power Automate flows to populate new data
3. Modify Power Apps formulas to use new metrics
4. Update Gen Pages prompts to include new statistics

**Q: Can I integrate with other Microsoft 365 services?**

A: Yes, common integrations include:
- SharePoint for document storage
- Teams for notifications and collaboration
- Outlook for email notifications
- Power BI for advanced analytics and reporting

### Deployment Questions

**Q: What's the recommended deployment approach?**

A: Follow ALM best practices:
1. Development environment for building
2. Test environment for validation
3. Production environment for live users
4. Use solutions for deployment
5. Implement CI/CD pipelines for automation

**Q: How do I monitor app performance in production?**

A: Use these monitoring tools:
- Power Platform Analytics for usage metrics
- Dataverse analytics for data performance
- Application Insights for detailed telemetry
- Power Automate run history for flow monitoring

## Getting Additional Help

### Microsoft Resources
- **Power Platform Documentation**: [docs.microsoft.com/power-platform](https://docs.microsoft.com/power-platform)
- **Power Apps Community**: [powerusers.microsoft.com](https://powerusers.microsoft.com)
- **Dataverse Documentation**: [docs.microsoft.com/power-apps/maker/data-platform](https://docs.microsoft.com/power-apps/maker/data-platform)

### Support Channels
- **Microsoft Support**: For licensing and platform issues
- **Community Forums**: For development questions
- **Power Platform Blog**: For latest updates and best practices
- **GitHub Repository Issues**: For documentation feedback

### Professional Services
- **Microsoft Partners**: For custom development
- **Power Platform Consultants**: For architecture guidance  
- **Training Providers**: For skill development

---

**Previous:** [← Configuration Examples](06-configuration-examples.md) | **Back to:** [Getting Started](01-getting-started.md)
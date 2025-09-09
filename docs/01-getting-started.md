# Getting Started with NHL Predictor Game

This guide will help you set up your environment and get started building the NHL Predictor Game using Microsoft Power Platform components.

## Prerequisites

Before you begin, ensure you have the following:

### Microsoft Power Platform Requirements
- **Microsoft 365 Account**: With appropriate licensing for Power Platform
- **Power Platform License**: Power Apps, Power Automate, and Dataverse access
- **Power Platform Admin Rights**: To create environments and configure Dataverse
- **Developer Environment**: Recommended for building and testing

### Technical Requirements
- **Web Browser**: Modern browser (Chrome, Edge, Firefox, Safari)
- **Internet Connection**: Stable connection for accessing Power Platform services
- **Basic Knowledge**: Familiarity with:
  - Power Apps canvas or model-driven apps
  - Dataverse data modeling
  - Basic understanding of NHL statistics and game data

### Optional Requirements
- **NHL API Access**: For real-time data integration
- **Azure Subscription**: For advanced features and external integrations
- **Power BI License**: For advanced analytics and reporting

## Environment Setup

### Step 1: Create a Power Platform Environment

1. **Access Power Platform Admin Center**
   - Navigate to [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/)
   - Sign in with your Microsoft 365 credentials

2. **Create New Environment**
   ```
   Environment Name: NHL Predictor Dev
   Type: Developer (recommended for development)
   Region: Select your preferred region
   Currency: USD (or your local currency)
   Language: English
   ```

3. **Enable Dataverse**
   - During environment creation, ensure Dataverse is enabled
   - This will create the database for storing NHL data

### Step 2: Configure Environment Settings

1. **Security Settings**
   - Add appropriate users to the environment
   - Assign necessary security roles

2. **Data Loss Prevention (DLP)**
   - Configure DLP policies if required by your organization
   - Ensure NHL API connectors are allowed

### Step 3: Install Required Solutions

1. **Power Apps Component Framework**
   - Enable PCF components if you plan to use custom controls

2. **AI Builder** (if available)
   - Enable AI Builder for predictive capabilities

## Data Sources Setup

### NHL Data Sources

The NHL Predictor Game can work with various data sources:

1. **NHL Official API**
   - Free access to basic game data
   - Historical statistics and schedules
   - Real-time game updates

2. **Sports Data APIs**
   - Premium services with enhanced statistics
   - Player performance metrics
   - Advanced analytics data

3. **Manual Data Entry**
   - Excel imports for historical data
   - Custom data entry forms
   - Sample datasets for development

### Sample Data Structure

For development purposes, you'll need data covering:

```
Teams:
- Team ID, Name, Conference, Division
- Home venue information
- Current roster

Players:
- Player ID, Name, Position, Team
- Season statistics
- Historical performance

Games:
- Game ID, Date, Home Team, Away Team
- Final scores, period scores
- Game statistics

Predictions:
- Game ID, Predicted winner
- Confidence score
- Prediction method used
```

## Next Steps

1. **[Set up Dataverse](02-dataverse-setup.md)** - Configure your data storage
2. **[Build Power Apps interface](03-power-apps-development.md)** - Create the user interface
3. **[Implement Gen Pages](04-gen-pages-implementation.md)** - Add AI-powered insights

## Troubleshooting

### Common Setup Issues

**Environment Creation Fails**
- Verify your Power Platform licensing
- Check regional availability
- Contact your tenant administrator

**Dataverse Not Available**
- Confirm Dataverse is included in your license
- Check environment capacity limits
- Verify tenant settings allow Dataverse creation

**Connector Access Issues**
- Review DLP policies
- Check connector availability in your region
- Verify external connection permissions

### Getting Help

- **Power Platform Documentation**: [docs.microsoft.com/power-platform](https://docs.microsoft.com/power-platform)
- **Power Apps Community**: [powerusers.microsoft.com](https://powerusers.microsoft.com)
- **Microsoft Support**: For licensing and technical issues

---

**Next:** [Dataverse Setup →](02-dataverse-setup.md)
# Gen Pages Implementation for NHL Predictor Game

This guide covers implementing Gen Pages to add AI-powered content generation and insights to your NHL Predictor Game.

## Overview

Gen Pages enables your NHL Predictor Game to generate AI-powered content including:
- Game analysis and predictions
- Team performance insights
- Player statistics summaries
- Trend analysis and predictions
- Personalized recommendations

## What are Gen Pages?

Gen Pages is a Microsoft AI-powered feature that allows you to create dynamic, AI-generated content within Power Apps. It leverages large language models to generate contextual, data-driven content based on your Dataverse data.

## Prerequisites

### Required Licenses and Access
- **Microsoft Copilot Studio**: For Gen Pages functionality
- **AI Builder License**: For AI model access
- **Power Apps Premium**: For advanced connector features
- **Dataverse**: With AI Builder tables enabled

### Environment Setup
```
1. Verify AI Builder is enabled in your environment
2. Ensure Copilot Studio access is available
3. Configure appropriate data loss prevention policies
4. Set up required security roles for AI features
```

## Implementation Strategy

### Content Generation Areas

1. **Game Predictions and Analysis**
   - Pre-game analysis based on team statistics
   - Historical matchup insights
   - Injury impact analysis
   - Weather considerations for outdoor games

2. **Team Performance Insights**
   - Season progression analysis
   - Strengths and weaknesses breakdown
   - Player impact analysis
   - Comparison with historical performance

3. **Player Spotlights**
   - Player performance summaries
   - Career highlight analysis
   - Trade impact assessments
   - Rookie development tracking

4. **Trend Analysis**
   - League-wide trends identification
   - Division performance patterns
   - Playoff prediction analysis
   - Draft analysis and prospects

## Step 1: Set Up Gen Pages in Power Apps

### Enable Gen Pages Feature

1. **Access Power Apps Maker Portal**
   ```
   1. Navigate to make.powerapps.com
   2. Select your NHL Predictor environment
   3. Go to Apps → Your NHL Predictor App
   4. Click "Edit" to open app in designer
   ```

2. **Add Gen Pages Component**
   ```
   1. In app designer, click "Insert" tab
   2. Select "AI" → "Gen Pages"
   3. Choose appropriate layout template
   4. Configure data source connections
   ```

### Configure Data Connections

1. **Connect to Dataverse Tables**
   ```powerfx
   // Configure data source for game analysis
   GameAnalysisData: 
   AddColumns(
       Games,
       "HomeTeamStats", LookUp(TeamStats, nhl_teamid = Games.nhl_hometeam),
       "AwayTeamStats", LookUp(TeamStats, nhl_teamid = Games.nhl_awayteam),
       "RecentGames", Filter(
           Games, 
           (nhl_hometeam = ThisRecord.nhl_hometeam || nhl_awayteam = ThisRecord.nhl_hometeam) &&
           nhl_gamedate > DateAdd(ThisRecord.nhl_gamedate, -30, Days)
       )
   )
   ```

## Step 2: Create AI Content Templates

### Game Analysis Template

1. **Pre-Game Analysis Generator**
   ```
   Template Structure:
   - Game matchup overview
   - Team current form analysis
   - Key player availability
   - Historical head-to-head
   - Prediction with reasoning
   ```

2. **Prompt Engineering**
   ```
   System Prompt Example:
   "You are an expert NHL analyst. Generate a comprehensive pre-game analysis 
   for the upcoming matchup between {HomeTeam} and {AwayTeam}. 
   
   Include:
   - Current team form (last 10 games)
   - Key statistical matchups
   - Player injuries or returns
   - Historical performance between teams
   - Prediction with confidence level
   
   Base your analysis on the provided data: {GameData}"
   ```

### Team Performance Insights

1. **Season Performance Template**
   ```powerfx
   // Generate team performance insight
   TeamInsightPrompt: 
   "Analyze the performance of " & SelectedTeam.nhl_name & 
   " for the current season. Current record: " & 
   TeamStats.nhl_wins & "-" & TeamStats.nhl_losses & 
   ". Goals for: " & TeamStats.nhl_goalsfor & 
   ". Goals against: " & TeamStats.nhl_goalsagainst &
   ". Provide insights on strengths, weaknesses, and outlook."
   ```

2. **Player Impact Analysis**
   ```
   Template for analyzing key players:
   - Statistical performance metrics
   - Impact on team success
   - Comparison to league averages
   - Injury history and availability
   ```

## Step 3: Implement AI-Generated Content Screens

### Game Analysis Screen

1. **Screen Layout**
   ```
   Components:
   - Game header (teams, date, venue)
   - Generated analysis section
   - Key statistics comparison
   - AI prediction vs user predictions
   - Refresh/regenerate button
   ```

2. **Content Generation Logic**
   ```powerfx
   // Generate game analysis when screen loads
   OnVisible:
   Set(
       gameAnalysis,
       GenPages.GenerateContent(
           GameAnalysisPrompt,
           {
               HomeTeam: selectedGame.nhl_hometeam.nhl_name,
               AwayTeam: selectedGame.nhl_awayteam.nhl_name,
               GameData: JSON(gameAnalysisData)
           }
       )
   )
   ```

3. **Display Generated Content**
   ```powerfx
   // Display AI-generated analysis
   HTML Text Control:
   gameAnalysis.GeneratedContent
   
   // Add loading indicator
   If(
       IsBlank(gameAnalysis),
       Spinner.Visible = true,
       Spinner.Visible = false
   )
   ```

### Team Insights Dashboard

1. **Dynamic Insights Generation**
   ```powerfx
   // Generate team insights based on selection
   OnChange (TeamDropdown):
   Set(
       teamInsights,
       GenPages.GenerateContent(
           TeamInsightTemplate,
           {
               TeamName: TeamDropdown.Selected.nhl_name,
               CurrentStats: LookUp(TeamStats, nhl_teamid = TeamDropdown.Selected.nhl_teamid),
               RecentGames: Filter(Games, nhl_hometeam = TeamDropdown.Selected.nhl_teamid || nhl_awayteam = TeamDropdown.Selected.nhl_teamid)
           }
       )
   )
   ```

2. **Multi-Section Insights**
   ```
   Generated Content Sections:
   1. Current Season Performance
   2. Strengths and Weaknesses
   3. Key Player Contributions  
   4. Playoff Outlook
   5. Trade Deadline Needs
   ```

## Step 4: Advanced AI Features

### Personalized Recommendations

1. **User Preference Learning**
   ```powerfx
   // Track user prediction patterns
   UserPreferences: 
   GroupBy(
       Filter(Predictions, nhl_userid = User().Id),
       "nhl_predictedwinner",
       "UserPicks"
   )
   ```

2. **Personalized Content Generation**
   ```
   Prompt Template:
   "Based on this user's prediction history showing preference for 
   {PreferredTeams}, generate personalized insights for upcoming games 
   involving these teams. Include why these matchups might interest them."
   ```

### Trend Analysis and Predictions

1. **League Trend Identification**
   ```powerfx
   // Aggregate league-wide data for trend analysis
   LeagueTrends:
   AddColumns(
       GroupBy(Games, "nhl_season", "SeasonGames"),
       "AvgGoalsPerGame", Average(SeasonGames, nhl_homescore + nhl_awayscore),
       "OvertimePercentage", CountRows(Filter(SeasonGames, nhl_overtime = true)) / CountRows(SeasonGames) * 100
   )
   ```

2. **Predictive Analysis**
   ```
   AI Prompts for Predictions:
   - Playoff race predictions
   - Award winner predictions  
   - Draft lottery implications
   - Trade deadline move predictions
   ```

## Step 5: Integration with Power Apps UI

### Embedding AI Content

1. **Native Integration**
   ```powerfx
   // Embed Gen Pages content in existing screens
   Insert → AI → Gen Pages Component
   
   Configuration:
   - Data source: Connected Dataverse tables
   - Template: Custom NHL analysis template
   - Refresh trigger: Manual or automatic
   ```

2. **Custom HTML Display**
   ```powerfx
   // For more control over display
   HTML Text Control:
   "<div class='ai-content'>" & 
   gameAnalysis.GeneratedContent & 
   "</div>"
   ```

### User Experience Enhancements

1. **Loading States**
   ```powerfx
   // Show loading indicator during generation
   If(
       aiContentGenerating,
       "Generating AI analysis...",
       generatedContent
   )
   ```

2. **Error Handling**
   ```powerfx
   // Handle AI generation failures
   If(
       IsError(generatedContent),
       "Unable to generate AI content. Please try again.",
       generatedContent
   )
   ```

3. **Content Refresh Options**
   ```
   User Controls:
   - Manual refresh button
   - Auto-refresh toggle
   - Content history/versions
   - Feedback mechanism
   ```

## Step 6: Content Quality and Monitoring

### Quality Assurance

1. **Content Validation**
   ```
   Validation Checks:
   - Factual accuracy against source data
   - Appropriate tone and language
   - Length and formatting consistency
   - Hockey terminology usage
   ```

2. **User Feedback Integration**
   ```powerfx
   // Collect user feedback on AI content
   Patch(
       AIContentFeedback,
       Defaults(AIContentFeedback),
       {
           ContentId: currentContent.Id,
           UserId: User().Id,
           Rating: feedbackRating.Value,
           Comments: feedbackText.Text
       }
   )
   ```

### Performance Monitoring

1. **Usage Analytics**
   ```
   Track Metrics:
   - Content generation frequency
   - User engagement with AI content
   - Most popular content types
   - Generation success rates
   ```

2. **Cost Management**
   ```
   Optimization Strategies:
   - Cache frequently requested content
   - Batch generation for efficiency
   - User-triggered vs automatic generation
   - Content expiration policies
   ```

## Best Practices

### Prompt Engineering
- Use specific, context-rich prompts
- Include relevant data in prompts
- Test and iterate on prompt effectiveness
- Maintain consistent tone and style

### Data Integration
- Ensure data freshness for accurate analysis
- Handle missing or incomplete data gracefully
- Provide data context in prompts
- Validate data before generation

### User Experience
- Set clear expectations for AI content
- Provide manual refresh options
- Handle errors gracefully
- Allow user feedback and corrections

## Troubleshooting

### Common Issues

**AI Content Not Generating**
- Verify AI Builder licensing
- Check data source connections
- Review prompt structure and data
- Validate environment configuration

**Poor Content Quality**
- Refine prompt engineering
- Improve data quality and context
- Add specific instructions for hockey domain
- Implement content validation rules

**Performance Issues**
- Implement content caching
- Optimize data queries
- Use batch generation
- Monitor API usage limits

---

**Previous:** [← Power Apps Development](03-power-apps-development.md) | **Next:** [Architecture Overview →](05-architecture-overview.md)
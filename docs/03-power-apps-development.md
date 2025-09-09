# Power Apps Development for NHL Predictor Game

This guide covers building the Power Apps interface for the NHL Predictor Game, including both canvas and model-driven app approaches.

## Overview

The Power Apps interface serves as the primary user interaction layer for the NHL Predictor Game. Users will be able to:
- View upcoming NHL games
- Make predictions on game outcomes
- Track prediction accuracy
- View historical game data and statistics
- Access AI-generated insights

## App Architecture Decisions

### Canvas App vs Model-Driven App

**Recommended Approach: Canvas App**
- Better user experience for prediction workflows
- Custom UI for game visualization
- Mobile-responsive design
- Integration with Gen Pages for AI content

**Model-Driven App Use Cases:**
- Administrative functions
- Data management and bulk operations
- Advanced reporting and analytics

## Canvas App Development

### App Structure

```
NHL Predictor App
├── Splash Screen
├── Main Navigation
├── Game Predictions
│   ├── Upcoming Games
│   ├── Make Prediction
│   └── Prediction History
├── Team Information
│   ├── Team Roster
│   ├── Statistics
│   └── Schedule
├── Leaderboard
├── AI Insights (Gen Pages)
└── Settings
```

### Step 1: Create the Canvas App

1. **Initialize New App**
   ```
   1. Go to make.powerapps.com
   2. Click "Create" → "Canvas app from blank"
   3. App name: "NHL Predictor Game"
   4. Format: Phone or Tablet (recommend Tablet)
   5. Click "Create"
   ```

2. **Configure Data Sources**
   ```
   1. Click "Data" in left panel
   2. Add data sources:
      - Teams (nhl_team)
      - Games (nhl_game) 
      - Predictions (nhl_prediction)
      - Team Statistics (nhl_teamstats)
   ```

### Step 2: Build the Main Navigation

1. **Create Navigation Screen**
   ```
   Screen Name: MainNavScreen
   Components:
   - Header with app logo/title
   - Navigation buttons for main sections
   - User greeting with current stats
   ```

2. **Navigation Control Implementation**
   ```powerfx
   // Navigation function for buttons
   Navigate(TargetScreen, ScreenTransition.Fade)
   
   // Example: Games button OnSelect
   Navigate(GamesScreen, ScreenTransition.SlideLeft)
   ```

### Step 3: Upcoming Games Screen

1. **Games Gallery Setup**
   ```powerfx
   // Data source for upcoming games
   Filter(
       Games,
       nhl_gamedate >= Today() && 
       nhl_status = "Scheduled"
   )
   
   // Sort by game date
   Sort(
       Filter(Games, nhl_gamedate >= Today()),
       nhl_gamedate,
       Ascending
   )
   ```

2. **Game Card Template**
   ```
   Components per game card:
   - Team logos/names
   - Game date and time
   - Venue information
   - Predict button
   - Current prediction (if exists)
   ```

3. **Game Card Layout**
   ```powerfx
   // Team name display
   Text: ThisItem.nhl_hometeam.nhl_name & " vs " & ThisItem.nhl_awayteam.nhl_name
   
   // Date formatting
   Text: Text(ThisItem.nhl_gamedate, "dddd, mmm dd - h:mm AM/PM")
   ```

### Step 4: Prediction Interface

1. **Prediction Screen Design**
   ```
   Game Details Section:
   - Team matchup display
   - Recent team statistics
   - Head-to-head history
   
   Prediction Section:
   - Team selection buttons
   - Confidence slider (0-100%)
   - Reasoning text input
   - Submit prediction button
   ```

2. **Prediction Logic**
   ```powerfx
   // Store selected team in variable
   OnSelect for team button:
   Set(selectedTeam, ThisItem.nhl_teamid)
   
   // Submit prediction
   Patch(
       Predictions,
       Defaults(Predictions),
       {
           nhl_gameid: gameId,
           nhl_userid: User().Id,
           nhl_predictedwinner: selectedTeam,
           nhl_confidence: confidenceSlider.Value,
           nhl_predictiondate: Now(),
           nhl_method: "Manual"
       }
   )
   ```

3. **Validation Rules**
   ```powerfx
   // Check if user already predicted for this game
   If(
       CountRows(
           Filter(
               Predictions,
               nhl_gameid = gameId && 
               nhl_userid = User().Id
           )
       ) > 0,
       Notify("You have already predicted this game", NotificationType.Warning),
       // Submit prediction
   )
   ```

### Step 5: Prediction History and Tracking

1. **User Predictions Gallery**
   ```powerfx
   // User's prediction history
   Filter(
       Predictions,
       nhl_userid = User().Id
   )
   
   // Sort by prediction date
   Sort(
       Filter(Predictions, nhl_userid = User().Id),
       nhl_predictiondate,
       Descending
   )
   ```

2. **Accuracy Calculations**
   ```powerfx
   // Calculate user's prediction accuracy
   With(
       {
           totalPredictions: CountRows(
               Filter(Predictions, nhl_userid = User().Id)
           ),
           correctPredictions: CountRows(
               Filter(
                   Predictions,
                   nhl_userid = User().Id && nhl_correct = true
               )
           )
       },
       If(
           totalPredictions > 0,
           Round(correctPredictions / totalPredictions * 100, 1),
           0
       )
   )
   ```

### Step 6: Team Information Screens

1. **Team List Gallery**
   ```powerfx
   // All active teams sorted by name
   Sort(
       Filter(Teams, nhl_active = true),
       nhl_name,
       Ascending
   )
   ```

2. **Team Detail Screen**
   ```
   Components:
   - Team header with logo and basic info
   - Current season statistics
   - Roster gallery (players)
   - Recent games history
   - Upcoming games
   ```

3. **Team Statistics Display**
   ```powerfx
   // Get current season stats for selected team
   LookUp(
       TeamStats,
       nhl_teamid = selectedTeamId && 
       nhl_season = "2023-24"
   )
   ```

### Step 7: Leaderboard Implementation

1. **User Rankings**
   ```powerfx
   // Calculate all users' accuracy rates
   AddColumns(
       GroupBy(
           Predictions,
           "nhl_userid",
           "UserPredictions"
       ),
       "TotalPredictions", CountRows(UserPredictions),
       "CorrectPredictions", CountRows(
           Filter(UserPredictions, nhl_correct = true)
       ),
       "Accuracy", Round(
           CountRows(Filter(UserPredictions, nhl_correct = true)) / 
           CountRows(UserPredictions) * 100, 1
       )
   )
   ```

2. **Leaderboard Display**
   ```
   Columns:
   - Rank (calculated)
   - User name
   - Total predictions
   - Correct predictions
   - Accuracy percentage
   - Recent streak
   ```

## Mobile Optimization

### Responsive Design Considerations

1. **Screen Size Adaptation**
   ```powerfx
   // Responsive width calculation
   If(
       App.Width < 768,
       App.Width - 40,  // Mobile padding
       Min(1200, App.Width * 0.8)  // Desktop max width
   )
   ```

2. **Touch-Friendly Controls**
   - Minimum 44px touch targets
   - Adequate spacing between buttons
   - Large, clear fonts for readability

3. **Navigation Patterns**
   - Bottom navigation for mobile
   - Side navigation for tablet/desktop
   - Breadcrumb navigation for deep screens

## Advanced Features

### Integration with External APIs

1. **NHL API Connector (Custom)**
   ```powerfx
   // Custom connector for NHL API
   // Configure in Power Platform admin center
   // Use in app for real-time data
   ```

2. **Power Automate Integration**
   ```
   Triggers:
   - Scheduled data refresh
   - New game notifications
   - Prediction reminders
   
   Actions:
   - Update game scores
   - Send prediction summaries
   - Generate weekly reports
   ```

### Performance Optimization

1. **Data Loading Strategies**
   ```powerfx
   // Lazy loading for large datasets
   OnVisible: 
   If(
       IsBlank(gamesList),
       Set(gamesList, Sort(Games, nhl_gamedate))
   )
   ```

2. **Caching Patterns**
   ```powerfx
   // Cache frequently accessed data
   OnStart:
   ClearCollect(
       teamsCache,
       Filter(Teams, nhl_active = true)
   )
   ```

## Testing and Deployment

### Testing Checklist

- [ ] **Functionality Testing**
  - All navigation flows work correctly
  - Predictions are saved and retrieved properly
  - Data displays accurately across all screens
  - Error handling works for edge cases

- [ ] **Performance Testing**
  - App loads within acceptable time limits
  - Large data sets don't cause timeouts
  - Smooth scrolling in galleries

- [ ] **User Experience Testing**
  - Intuitive navigation flow
  - Clear visual feedback for actions
  - Responsive design works on target devices

### Deployment Process

1. **Publish the App**
   ```
   1. Click "File" → "Publish"
   2. Add version notes
   3. Click "Publish this version"
   ```

2. **Share the App**
   ```
   1. Click "Share" button
   2. Add users or security groups
   3. Assign appropriate permissions
   4. Send sharing notification
   ```

3. **Monitor Usage**
   - Use Power Platform analytics
   - Monitor error logs
   - Collect user feedback

## Troubleshooting

### Common Development Issues

**Data Source Connection Errors**
- Verify Dataverse permissions
- Check network connectivity
- Refresh data source connections

**Performance Issues**
- Review formula efficiency
- Implement proper delegation
- Optimize gallery item templates

**Display Issues**
- Check responsive formula calculations
- Verify control sizing and positioning
- Test on multiple screen sizes

---

**Previous:** [← Dataverse Setup](02-dataverse-setup.md) | **Next:** [Gen Pages Implementation →](04-gen-pages-implementation.md)
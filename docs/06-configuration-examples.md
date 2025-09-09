# Configuration Examples and Templates

This document provides practical configuration examples, sample code, and templates for implementing the NHL Predictor Game components.

## Dataverse Configuration Templates

### Sample Data Import Templates

#### Teams CSV Template
```csv
nhl_name,nhl_abbreviation,nhl_conference,nhl_division,nhl_city,nhl_venue,nhl_foundedyear,nhl_active
Toronto Maple Leafs,TOR,Eastern Conference,Atlantic,Toronto,Scotiabank Arena,1917,true
Montreal Canadiens,MTL,Eastern Conference,Atlantic,Montreal,Bell Centre,1909,true
Boston Bruins,BOS,Eastern Conference,Atlantic,Boston,TD Garden,1924,true
Tampa Bay Lightning,TBL,Eastern Conference,Atlantic,Tampa Bay,Amalie Arena,1992,true
Florida Panthers,FLA,Eastern Conference,Atlantic,Sunrise,FLA Live Arena,1993,true
Buffalo Sabres,BUF,Eastern Conference,Atlantic,Buffalo,KeyBank Center,1970,true
Ottawa Senators,OTT,Eastern Conference,Atlantic,Ottawa,Canadian Tire Centre,1992,true
Detroit Red Wings,DET,Eastern Conference,Atlantic,Detroit,Little Caesars Arena,1926,true
```

#### Sample Game Data Template
```csv
nhl_gamedate,nhl_hometeam_abbreviation,nhl_awayteam_abbreviation,nhl_homescore,nhl_awayscore,nhl_overtime,nhl_shootout,nhl_status,nhl_season,nhl_gametype
2024-01-15 19:00,TOR,MTL,4,2,false,false,Final,2023-24,Regular Season
2024-01-16 20:00,BOS,TBL,3,1,false,false,Final,2023-24,Regular Season
2024-01-17 19:30,FLA,BUF,5,3,true,false,Final,2023-24,Regular Season
```

### Dataverse Power Fx Formulas

#### Team Statistics Calculations
```powerfx
// Calculate win percentage
WinPercentage: 
Round(
    (nhl_wins / (nhl_wins + nhl_losses + nhl_overtimelosses)) * 100,
    1
)

// Calculate goals per game
GoalsPerGame:
Round(nhl_goalsfor / nhl_gamesplayed, 2)

// Calculate goal differential
GoalDifferential:
nhl_goalsfor - nhl_goalsagainst

// Points percentage calculation
PointsPercentage:
Round(
    (nhl_points / (nhl_gamesplayed * 2)) * 100,
    1
)
```

#### Game Prediction Accuracy
```powerfx
// User prediction accuracy
UserAccuracy:
With(
    {
        userPredictions: Filter(
            Predictions, 
            nhl_userid = User().Email &&
            !IsBlank(nhl_correct)
        )
    },
    Round(
        CountRows(Filter(userPredictions, nhl_correct = true)) / 
        CountRows(userPredictions) * 100,
        1
    )
)

// Team prediction success rate
TeamPredictionRate:
With(
    {
        teamPredictions: Filter(
            Predictions,
            nhl_predictedwinner = selectedTeam.nhl_teamid &&
            !IsBlank(nhl_correct)
        )
    },
    Round(
        CountRows(Filter(teamPredictions, nhl_correct = true)) /
        CountRows(teamPredictions) * 100,
        1
    )
)
```

## Power Apps Configuration Examples

### Screen Navigation Configuration

#### Main Navigation Menu
```powerfx
// Navigation button OnSelect
Navigate(
    GamesScreen,
    ScreenTransition.SlideLeft,
    {
        selectedTab: "games",
        refreshData: true
    }
)

// Back button configuration
Navigate(
    MainScreen,
    ScreenTransition.SlideRight
)

// Deep link navigation with parameters
Navigate(
    GameDetailScreen,
    ScreenTransition.Fade,
    {
        gameId: ThisItem.nhl_gameid,
        gameDate: ThisItem.nhl_gamedate,
        homeTeam: ThisItem.nhl_hometeam,
        awayTeam: ThisItem.nhl_awayteam
    }
)
```

#### Responsive Design Configuration
```powerfx
// Screen width-based layout
ContainerWidth:
If(
    App.Width < 768,
    App.Width - 32,          // Mobile: full width with padding
    If(
        App.Width < 1024,
        App.Width * 0.9,      // Tablet: 90% width
        Min(1200, App.Width * 0.8)  // Desktop: max 1200px or 80%
    )
)

// Font size scaling
TitleFontSize:
If(
    App.Width < 768,
    18,    // Mobile
    If(
        App.Width < 1024,
        20,    // Tablet
        24     // Desktop
    )
)

// Gallery item height
GalleryItemHeight:
If(
    App.Width < 768,
    120,   // Mobile: compact view
    150    // Tablet/Desktop: expanded view
)
```

### Data Loading and Caching

#### Efficient Data Loading
```powerfx
// OnStart app configuration
OnStart:
Concurrent(
    // Load teams data once
    ClearCollect(
        teamsCollection,
        Sort(
            Filter(Teams, nhl_active = true),
            nhl_name
        )
    ),
    
    // Load current season stats
    ClearCollect(
        currentSeasonStats,
        Filter(TeamStats, nhl_season = "2023-24")
    ),
    
    // Set app variables
    Set(currentSeason, "2023-24"),
    Set(appVersion, "1.0.0"),
    Set(lastRefresh, Now())
)

// Lazy loading for games
OnVisible (GamesScreen):
If(
    IsEmpty(gamesCollection) || 
    DateDiff(lastGamesRefresh, Now(), Minutes) > 15,
    ClearCollect(
        gamesCollection,
        Sort(
            Filter(
                Games,
                nhl_gamedate >= Today() - 7 &&
                nhl_gamedate <= Today() + 14
            ),
            nhl_gamedate
        )
    );
    Set(lastGamesRefresh, Now())
)
```

#### Prediction Form Configuration
```powerfx
// Prediction submission with validation
SubmitPrediction:
If(
    // Validation checks
    IsBlank(selectedWinner),
    Notify("Please select a team to win", NotificationType.Error),
    
    confidenceSlider.Value < 50,
    Notify("Confidence must be at least 50%", NotificationType.Warning),
    
    // Check if user already predicted
    CountRows(
        Filter(
            Predictions,
            nhl_gameid = currentGame.nhl_gameid &&
            nhl_userid = User().Email
        )
    ) > 0,
    Notify("You have already predicted this game", NotificationType.Warning),
    
    // Submit prediction
    Patch(
        Predictions,
        Defaults(Predictions),
        {
            nhl_gameid: currentGame.nhl_gameid,
            nhl_userid: User().Email,
            nhl_predictedwinner: selectedWinner.nhl_teamid,
            nhl_confidence: confidenceSlider.Value,
            nhl_predictiondate: Now(),
            nhl_method: "Manual"
        }
    );
    Notify("Prediction submitted successfully!", NotificationType.Success);
    Navigate(GamesScreen, ScreenTransition.SlideRight)
)
```

## Power Automate Flow Templates

### NHL API Data Sync Flow

#### Scheduled Game Data Update
```json
{
    "definition": {
        "triggers": {
            "Recurrence": {
                "type": "Recurrence",
                "recurrence": {
                    "frequency": "Minute",
                    "interval": 15,
                    "schedule": {
                        "hours": ["17", "18", "19", "20", "21", "22", "23"]
                    }
                }
            }
        },
        "actions": {
            "Get_Live_Games": {
                "type": "Http",
                "inputs": {
                    "method": "GET",
                    "uri": "https://statsapi.web.nhl.com/api/v1/schedule?expand=schedule.linescore"
                }
            },
            "Parse_JSON": {
                "type": "ParseJson",
                "inputs": {
                    "content": "@body('Get_Live_Games')",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "dates": {
                                "type": "array",
                                "items": {
                                    "type": "object",
                                    "properties": {
                                        "games": {
                                            "type": "array"
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            },
            "Apply_to_each_game": {
                "type": "Foreach",
                "foreach": "@body('Parse_JSON')?['dates']?[0]?['games']",
                "actions": {
                    "Update_Game_Score": {
                        "type": "ApiConnection",
                        "inputs": {
                            "host": {
                                "connectionName": "shared_commondataservice"
                            },
                            "method": "patch",
                            "path": "/v2/datasets/@{encodeURIComponent('default.cds')}/tables/@{encodeURIComponent('nhl_games')}/items/@{encodeURIComponent(items('Apply_to_each_game')?['gamePk'])}",
                            "body": {
                                "nhl_homescore": "@items('Apply_to_each_game')?['teams']?['home']?['score']",
                                "nhl_awayscore": "@items('Apply_to_each_game')?['teams']?['away']?['score']",
                                "nhl_status": "@items('Apply_to_each_game')?['status']?['detailedState']"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

### Prediction Result Notification Flow

#### Daily Prediction Results Email
```json
{
    "definition": {
        "triggers": {
            "Recurrence": {
                "type": "Recurrence",
                "recurrence": {
                    "frequency": "Day",
                    "interval": 1,
                    "startTime": "2024-01-01T09:00:00Z"
                }
            }
        },
        "actions": {
            "Get_Completed_Games": {
                "type": "ApiConnection",
                "inputs": {
                    "host": {
                        "connectionName": "shared_commondataservice"
                    },
                    "method": "get",
                    "path": "/v2/datasets/@{encodeURIComponent('default.cds')}/tables/@{encodeURIComponent('nhl_games')}/items",
                    "queries": {
                        "$filter": "nhl_gamedate ge @{formatDateTime(addDays(utcNow(), -1), 'yyyy-MM-dd')} and nhl_status eq 'Final'"
                    }
                }
            },
            "Apply_to_each_completed_game": {
                "type": "Foreach",
                "foreach": "@body('Get_Completed_Games')?['value']",
                "actions": {
                    "Update_Prediction_Results": {
                        "type": "ApiConnection",
                        "inputs": {
                            "host": {
                                "connectionName": "shared_commondataservice"
                            },
                            "method": "get",
                            "path": "/v2/datasets/@{encodeURIComponent('default.cds')}/tables/@{encodeURIComponent('nhl_predictions')}/items",
                            "queries": {
                                "$filter": "nhl_gameid eq '@{items('Apply_to_each_completed_game')?['nhl_gameid']}'"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

## Gen Pages Configuration Templates

### Game Analysis Prompt Templates

#### Pre-Game Analysis Template
```
System Role: NHL Expert Analyst

Template:
Analyze the upcoming NHL game between {HomeTeam} and {AwayTeam} on {GameDate}.

Data Context:
- Home Team Record: {HomeWins}-{HomeLosses}-{HomeOTL}
- Away Team Record: {AwayWins}-{AwayLosses}-{AwayOTL}
- Home Team Last 5: {HomeLastFive}
- Away Team Last 5: {AwayLastFive}
- Home Goals For/Against: {HomeGF}/{HomeGA}
- Away Goals For/Against: {AwayGF}/{AwayGA}

Provide analysis covering:
1. Current form and momentum
2. Key statistical matchups
3. Historical head-to-head performance
4. Key players to watch
5. Prediction with confidence level (1-100)

Keep analysis to 300-400 words in HTML format with proper structure.
```

#### Post-Game Analysis Template
```
System Role: NHL Game Analyst

Template:
Provide post-game analysis for the completed game between {HomeTeam} and {AwayTeam}.

Final Score: {HomeTeam} {HomeScore} - {AwayScore} {AwayTeam}
{OvertimeDetails}

Game Context:
- Game situation and importance
- Key moments and turning points
- Statistical breakdown
- Player performances
- Impact on standings/playoff race

Format as engaging HTML content with:
- Game summary (100 words)
- Key statistics table
- Player highlights
- Looking ahead implications

Total length: 250-350 words
```

### AI Builder Model Configuration

#### Prediction Confidence Model
```json
{
    "modelType": "Prediction",
    "name": "NHL Game Outcome Predictor",
    "description": "Predicts NHL game outcomes based on team statistics and historical data",
    "inputFeatures": [
        {
            "name": "home_team_wins",
            "type": "Number",
            "description": "Home team wins this season"
        },
        {
            "name": "home_team_losses", 
            "type": "Number",
            "description": "Home team losses this season"
        },
        {
            "name": "away_team_wins",
            "type": "Number",
            "description": "Away team wins this season"
        },
        {
            "name": "goals_for_differential",
            "type": "Number", 
            "description": "Difference in goals for between teams"
        },
        {
            "name": "head_to_head_record",
            "type": "Number",
            "description": "Home team wins vs away team last 10 games"
        }
    ],
    "outputFeature": {
        "name": "predicted_winner",
        "type": "TwoOption",
        "description": "Home team wins (true) or away team wins (false)"
    }
}
```

## Security Configuration Templates

### Security Role Configuration

#### NHL Predictor User Role
```json
{
    "roleName": "NHL Predictor User",
    "description": "Standard user role for NHL Predictor Game",
    "privileges": [
        {
            "privilegeName": "prvReadTeam",
            "accessLevel": "Global"
        },
        {
            "privilegeName": "prvReadGame", 
            "accessLevel": "Global"
        },
        {
            "privilegeName": "prvCreatePrediction",
            "accessLevel": "User"
        },
        {
            "privilegeName": "prvReadPrediction",
            "accessLevel": "User"
        },
        {
            "privilegeName": "prvWritePrediction",
            "accessLevel": "User"
        }
    ]
}
```

### Data Loss Prevention Policy
```json
{
    "displayName": "NHL Predictor DLP Policy",
    "description": "DLP policy for NHL Predictor application",
    "environments": ["NHL-Predictor-Prod"],
    "connectorGroups": {
        "business": [
            "shared_commondataservice",
            "shared_office365users",
            "shared_approvals"
        ],
        "nonBusiness": [
            "shared_http",
            "shared_rss"
        ],
        "blocked": [
            "shared_ftp",
            "shared_filesystem"
        ]
    }
}
```

## Environment Configuration Scripts

### PowerShell Environment Setup
```powershell
# Install Power Platform CLI
Install-Module -Name Microsoft.PowerApps.Administration.PowerShell
Install-Module -Name Microsoft.PowerApps.PowerShell

# Connect to tenant
Add-PowerAppsAccount

# Create NHL Predictor environment
$environment = New-AdminPowerAppEnvironment `
    -DisplayName "NHL Predictor Production" `
    -LocationName "unitedstates" `
    -EnvironmentSku "Production" `
    -ProvisionDatabase $true `
    -CurrencyName "USD" `
    -LanguageName "1033"

Write-Host "Environment created: $($environment.EnvironmentName)"

# Configure security groups
$userRole = Get-AdminPowerAppRoleAssignment -EnvironmentName $environment.EnvironmentName
Set-AdminPowerAppRoleAssignment `
    -EnvironmentName $environment.EnvironmentName `
    -RoleName "Environment Maker" `
    -PrincipalObjectId "group-object-id" `
    -PrincipalType "Group"
```

### Solution Deployment Configuration
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

variables:
  - group: NHL-Predictor-Variables

stages:
- stage: Build
  jobs:
  - job: ExportSolution
    steps:
    - task: PowerPlatformToolInstaller@0
    - task: PowerPlatformExportSolution@0
      inputs:
        authenticationType: 'PowerPlatformSPN'
        PowerPlatformSPN: '$(PowerPlatformConnection)'
        Environment: '$(SourceEnvironment)'
        SolutionName: 'NHLPredictorGame'
        SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/NHLPredictorGame.zip'

- stage: Deploy
  dependsOn: Build
  jobs:
  - deployment: DeployToProduction
    environment: 'NHL-Predictor-Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: PowerPlatformImportSolution@0
            inputs:
              authenticationType: 'PowerPlatformSPN'
              PowerPlatformSPN: '$(PowerPlatformConnection)'
              Environment: '$(TargetEnvironment)'
              SolutionInputFile: '$(Pipeline.Workspace)/drop/NHLPredictorGame.zip'
```

---

**Previous:** [← Architecture Overview](05-architecture-overview.md) | **Next:** [Troubleshooting →](07-troubleshooting.md)
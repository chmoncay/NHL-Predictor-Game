# Dataverse Setup for NHL Predictor Game

This guide covers setting up Dataverse tables, relationships, and data management for the NHL Predictor Game.

## Overview

Dataverse will serve as the central data repository for your NHL Predictor Game, storing:
- NHL teams and player information
- Historical game data and statistics
- User predictions and results
- Configuration and lookup data

## Data Model Design

### Core Tables

#### 1. Teams Table (nhl_team)

| Column Name | Data Type | Required | Description |
|-------------|-----------|----------|-------------|
| nhl_teamid | Unique identifier | Yes | Primary key |
| nhl_name | Single line of text | Yes | Team name (e.g., "Toronto Maple Leafs") |
| nhl_abbreviation | Single line of text | Yes | Team abbreviation (e.g., "TOR") |
| nhl_conference | Choice | Yes | Eastern/Western Conference |
| nhl_division | Choice | Yes | Atlantic, Metropolitan, Central, Pacific |
| nhl_city | Single line of text | Yes | Team city |
| nhl_venue | Single line of text | No | Home venue name |
| nhl_foundedyear | Whole number | No | Year team was founded |
| nhl_active | Yes/No | Yes | Is team currently active |

#### 2. Players Table (nhl_player)

| Column Name | Data Type | Required | Description |
|-------------|-----------|----------|-------------|
| nhl_playerid | Unique identifier | Yes | Primary key |
| nhl_firstname | Single line of text | Yes | Player first name |
| nhl_lastname | Single line of text | Yes | Player last name |
| nhl_position | Choice | Yes | C, LW, RW, D, G |
| nhl_jerseynumber | Whole number | No | Jersey number |
| nhl_teamid | Lookup | Yes | Reference to Teams table |
| nhl_height | Single line of text | No | Player height |
| nhl_weight | Whole number | No | Player weight |
| nhl_birthdate | Date only | No | Date of birth |
| nhl_active | Yes/No | Yes | Currently active player |

#### 3. Games Table (nhl_game)

| Column Name | Data Type | Required | Description |
|-------------|-----------|----------|-------------|
| nhl_gameid | Unique identifier | Yes | Primary key |
| nhl_gamedate | Date and Time | Yes | Game date and time |
| nhl_hometeam | Lookup | Yes | Reference to Teams table |
| nhl_awayteam | Lookup | Yes | Reference to Teams table |
| nhl_homescore | Whole number | No | Home team final score |
| nhl_awayscore | Whole number | No | Away team final score |
| nhl_overtime | Yes/No | No | Game went to overtime |
| nhl_shootout | Yes/No | No | Game decided by shootout |
| nhl_status | Choice | Yes | Scheduled, In Progress, Final |
| nhl_season | Single line of text | Yes | Season (e.g., "2023-24") |
| nhl_gametype | Choice | Yes | Regular Season, Playoffs |

#### 4. Predictions Table (nhl_prediction)

| Column Name | Data Type | Required | Description |
|-------------|-----------|----------|-------------|
| nhl_predictionid | Unique identifier | Yes | Primary key |
| nhl_gameid | Lookup | Yes | Reference to Games table |
| nhl_userid | Lookup | Yes | Reference to User (system table) |
| nhl_predictedwinner | Lookup | Yes | Reference to Teams table |
| nhl_confidence | Decimal number | No | Confidence percentage (0-100) |
| nhl_predictiondate | Date and Time | Yes | When prediction was made |
| nhl_correct | Yes/No | No | Was prediction correct |
| nhl_method | Choice | No | Manual, AI, Statistical |

#### 5. Team Statistics Table (nhl_teamstats)

| Column Name | Data Type | Required | Description |
|-------------|-----------|----------|-------------|
| nhl_teamstatsid | Unique identifier | Yes | Primary key |
| nhl_teamid | Lookup | Yes | Reference to Teams table |
| nhl_season | Single line of text | Yes | Season (e.g., "2023-24") |
| nhl_gamesplayed | Whole number | No | Games played |
| nhl_wins | Whole number | No | Number of wins |
| nhl_losses | Whole number | No | Number of losses |
| nhl_overtimelosses | Whole number | No | Overtime/shootout losses |
| nhl_points | Whole number | No | Total points |
| nhl_goalsfor | Whole number | No | Goals scored |
| nhl_goalsagainst | Whole number | No | Goals allowed |
| nhl_lastupdate | Date and Time | No | Last statistics update |

## Implementation Steps

### Step 1: Create Dataverse Tables

1. **Access Power Apps Maker Portal**
   - Navigate to [make.powerapps.com](https://make.powerapps.com)
   - Select your NHL Predictor environment

2. **Create Teams Table**
   ```
   1. Click "Tables" → "New table"
   2. Display name: "Team"
   3. Plural display name: "Teams"
   4. Name: "nhl_team"
   5. Add columns as specified above
   ```

3. **Create Remaining Tables**
   - Follow the same process for Players, Games, Predictions, and Team Statistics tables
   - Ensure proper data types and relationships

### Step 2: Configure Relationships

1. **Player to Team Relationship**
   ```
   Type: Many-to-one
   Related table: Teams
   Lookup column: Team (in Players table)
   ```

2. **Game Relationships**
   ```
   Home Team: Many-to-one (Games → Teams)
   Away Team: Many-to-one (Games → Teams)
   ```

3. **Prediction Relationships**
   ```
   Game: Many-to-one (Predictions → Games)
   Predicted Winner: Many-to-one (Predictions → Teams)
   User: Many-to-one (Predictions → User)
   ```

### Step 3: Set Up Choice Columns

1. **Conference Choices**
   ```
   - Eastern Conference
   - Western Conference
   ```

2. **Division Choices**
   ```
   - Atlantic
   - Metropolitan  
   - Central
   - Pacific
   ```

3. **Position Choices**
   ```
   - C (Center)
   - LW (Left Wing)
   - RW (Right Wing)
   - D (Defense)
   - G (Goalie)
   ```

4. **Game Status Choices**
   ```
   - Scheduled
   - In Progress
   - Final
   - Postponed
   - Cancelled
   ```

### Step 4: Configure Security Roles

1. **Create Custom Security Role**
   ```
   Role Name: NHL Predictor User
   Permissions:
   - Read: All NHL tables
   - Create/Update: Predictions table
   - Read: User information
   ```

2. **Admin Role Configuration**
   ```
   Role Name: NHL Predictor Admin
   Permissions:
   - Full access: All NHL tables
   - Import/Export data
   - Manage users and security
   ```

## Data Import and Setup

### Sample Data Import

1. **Prepare NHL Teams Data**
   ```csv
   nhl_name,nhl_abbreviation,nhl_conference,nhl_division,nhl_city
   Toronto Maple Leafs,TOR,Eastern Conference,Atlantic,Toronto
   Montreal Canadiens,MTL,Eastern Conference,Atlantic,Montreal
   Boston Bruins,BOS,Eastern Conference,Atlantic,Boston
   ... (continue for all 32 teams)
   ```

2. **Import Using Data Import Wizard**
   - Use Power Apps data import feature
   - Map CSV columns to Dataverse columns
   - Validate and import data

### Historical Game Data

1. **NHL API Integration**
   ```
   API Endpoint: https://statsapi.web.nhl.com/api/v1/
   Key endpoints:
   - /teams: Get all teams
   - /schedule: Get game schedules
   - /game/{id}/feed/live: Get live game data
   ```

2. **Power Automate Flow for Data Sync**
   - Create scheduled flow to pull latest game data
   - Parse JSON responses
   - Update Dataverse tables

## Best Practices

### Data Quality
- Implement validation rules for scores and statistics
- Use calculated columns for derived statistics
- Set up duplicate detection rules

### Performance Optimization
- Create appropriate indexes on frequently queried columns
- Use views to filter large datasets
- Implement data archiving for historical seasons

### Security Considerations
- Restrict direct table access
- Use security roles to control permissions
- Audit data changes for compliance

## Troubleshooting

### Common Issues

**Import Failures**
- Check data format and column mappings
- Verify required fields are populated
- Review error logs for specific issues

**Relationship Errors**
- Ensure lookup tables exist before creating relationships
- Verify data types match between related columns
- Check for circular references

**Performance Issues**
- Review query patterns and add indexes
- Consider data partitioning for large datasets
- Monitor storage capacity and usage

---

**Previous:** [← Getting Started](01-getting-started.md) | **Next:** [Power Apps Development →](03-power-apps-development.md)
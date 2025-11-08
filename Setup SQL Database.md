The following is the base model for the FPL Super App


```SQL
/* ============================================================
   SCHEMA: Fantasy Premier League (FPL) star schema in SQL Server
   ============================================================ */

-- Optional: create a dedicated schema
IF NOT EXISTS (SELECT 1 FROM sys.schemas WHERE name = 'fpl')
    EXEC('CREATE SCHEMA fpl');
GO

/* =======================
   DIMENSIONS
   ======================= */

-- Club Data (DIM)
IF OBJECT_ID('fpl.ClubDataDim') IS NOT NULL DROP TABLE fpl.ClubDataDim;
CREATE TABLE fpl.ClubDataDim
(
    TeamID           NVARCHAR(50)  NOT NULL,  -- 'Team ID'
    TeamCode         NVARCHAR(50)  NOT NULL,  -- 'Team Code'
    TeamName         NVARCHAR(200) NOT NULL,  -- 'Team name'
    PulseID          NVARCHAR(50)  NULL,

    -- Computed groupings replicated from DAX SWITCH logic
    TeamName_Groups_3 AS (
        CASE 
            WHEN TeamName IS NULL OR LTRIM(RTRIM(TeamName)) = '' THEN '(Blank)'
            WHEN TeamName IN (N'Leicester', N'Ipswich', N'Southampton')
                THEN N'Ipswich & Leicester & Southampton'
            ELSE N'Other'
        END
    ) PERSISTED,
    TeamName_Groups AS (
        CASE 
            WHEN TeamName IS NULL OR LTRIM(RTRIM(TeamName)) = '' THEN '(Blank)'
            WHEN TeamName IN (N'Liverpool', N'Chelsea', N'Bournemouth', N'Man City')
                THEN N'Bournemouth & Chelsea & Liverpool & Man City'
            ELSE N'Other'
        END
    ) PERSISTED,

    CONSTRAINT PK_ClubDataDim PRIMARY KEY (TeamID),
    CONSTRAINT UQ_ClubDataDim_TeamCode UNIQUE (TeamCode),
    CONSTRAINT UQ_ClubDataDim_TeamName UNIQUE (TeamName)
);
GO

-- Opposition Club Data (DIM)
IF OBJECT_ID('fpl.OppositionClubDataDim') IS NOT NULL DROP TABLE fpl.OppositionClubDataDim;
CREATE TABLE fpl.OppositionClubDataDim
(
    TeamID   NVARCHAR(50)  NOT NULL,
    TeamCode NVARCHAR(50)  NOT NULL,
    TeamName NVARCHAR(200) NOT NULL,
    PulseID  NVARCHAR(50)  NULL,
    CONSTRAINT PK_OppositionClubDataDim PRIMARY KEY (TeamID),
    CONSTRAINT UQ_OppositionClubDataDim_TeamCode UNIQUE (TeamCode),
    CONSTRAINT UQ_OppositionClubDataDim_TeamName UNIQUE (TeamName)
);
GO

-- ClubAnalysisDim (duplicate of ClubData structure used elsewhere)
IF OBJECT_ID('fpl.ClubAnalysisDim') IS NOT NULL DROP TABLE fpl.ClubAnalysisDim;
CREATE TABLE fpl.ClubAnalysisDim
(
    TeamID   NVARCHAR(50)  NOT NULL,
    TeamCode NVARCHAR(50)  NOT NULL,
    TeamName NVARCHAR(200) NOT NULL,
    PulseID  NVARCHAR(50)  NULL,
    CONSTRAINT PK_ClubAnalysisDim PRIMARY KEY (TeamID),
    CONSTRAINT UQ_ClubAnalysisDim_TeamCode UNIQUE (TeamCode),
    CONSTRAINT UQ_ClubAnalysisDim_TeamName UNIQUE (TeamName)
);
GO

-- Position (DIM)
IF OBJECT_ID('fpl.PositionDim') IS NOT NULL DROP TABLE fpl.PositionDim;
CREATE TABLE fpl.PositionDim
(
    PositionID   NVARCHAR(50)  NOT NULL,   -- 'Position ID'
    PositionName NVARCHAR(100) NOT NULL,   -- 'Position Name'
    CONSTRAINT PK_PositionDim PRIMARY KEY (PositionID),
    CONSTRAINT UQ_PositionDim_PositionName UNIQUE (PositionName)
);
GO

-- Player Data (Dim)
IF OBJECT_ID('fpl.PlayerDataDim') IS NOT NULL DROP TABLE fpl.PlayerDataDim;
CREATE TABLE fpl.PlayerDataDim
(
    PlayerID        NVARCHAR(50)   NOT NULL,        -- 'Player ID'
    FirstName       NVARCHAR(100)  NULL,
    SecondName      NVARCHAR(100)  NULL,
    PlayerName      NVARCHAR(200)  NULL,            -- 'Player Name'
    NowCost         DECIMAL(19,4)  NULL,            -- now_cost
    EP_Next         FLOAT          NULL,            -- ep_next
    NowCostMillions DECIMAL(19,4)  NULL,            -- NowCostMillions
    CONSTRAINT PK_PlayerDataDim PRIMARY KEY (PlayerID)
);
GO

-- Date (auto calendar) – simple date list
IF OBJECT_ID('fpl.Date') IS NOT NULL DROP TABLE fpl.Date;
CREATE TABLE fpl.Date
(
    [Date] DATE NOT NULL,
    CONSTRAINT PK_Date PRIMARY KEY ([Date])
);
GO

-- DateDim (rich date dimension)
IF OBJECT_ID('fpl.DateDim') IS NOT NULL DROP TABLE fpl.DateDim;
CREATE TABLE fpl.DateDim
(
    [Date]        DATE           NOT NULL PRIMARY KEY,
    [DateTime]    DATETIME2(0)   NULL,
    [Year]        NVARCHAR(10)   NULL,          -- e.g. "CY2025"
    [Month]       NVARCHAR(7)    NULL,          -- "YYYY-MM"
    [MonthNumber] INT            NULL,
    [MonthOrder]  INT            NULL,          -- YYYY*100 + MM
    GameWeekNumber INT           NULL           -- aligns to GWDim.GWID if desired
);
GO

-- GWDim (Gameweek Dimension)
IF OBJECT_ID('fpl.GWDim') IS NOT NULL DROP TABLE fpl.GWDim;
CREATE TABLE fpl.GWDim
(
    GWID        INT           NOT NULL,     -- 'GW ID'
    GWName      NVARCHAR(50)  NOT NULL,     -- 'GW Name'
    GWDeadline  DATETIME2(0)  NOT NULL,     -- 'GW Deadline'
    Finished    BIT           NOT NULL,
    IsPrevious  BIT           NOT NULL,
    IsCurrent   BIT           NOT NULL,
    IsNext      BIT           NOT NULL,
    CONSTRAINT PK_GWDim PRIMARY KEY (GWID)
);
GO

-- Fixtures (DIM)
IF OBJECT_ID('fpl.FixturesDim') IS NOT NULL DROP TABLE fpl.FixturesDim;
CREATE TABLE fpl.FixturesDim
(
    FixtureID          INT           NOT NULL,  -- 'id'
    Code               INT           NULL,
    EventGW            INT           NULL,      -- 'event'
    Finished           BIT           NULL,
    FinishedProvisional BIT          NULL,
    KickoffTime        DATETIME2(0)  NULL,
    Minutes            INT           NULL,
    ProvisionalStart   BIT           NULL,
    Started            BIT           NULL,
    TeamA              NVARCHAR(50)  NULL,      -- team_a (FK to ClubDataDim.TeamID)
    TeamAScore         INT           NULL,
    TeamH              NVARCHAR(50)  NULL,      -- team_h (FK to ClubDataDim.TeamID)
    TeamHScore         INT           NULL,
    TeamHDifficulty    INT           NULL,
    TeamADifficulty    INT           NULL,
    PulseID            INT           NULL,
    CONSTRAINT PK_FixturesDim PRIMARY KEY (FixtureID),
    CONSTRAINT FK_FixturesDim_TeamA FOREIGN KEY (TeamA) REFERENCES fpl.ClubDataDim(TeamID),
    CONSTRAINT FK_FixturesDim_TeamH FOREIGN KEY (TeamH) REFERENCES fpl.ClubDataDim(TeamID)
);
GO

-- Results (DIM)
IF OBJECT_ID('fpl.ResultsDim') IS NOT NULL DROP TABLE fpl.ResultsDim;
CREATE TABLE fpl.ResultsDim
(
    Code               INT           NULL,
    Finished           BIT           NOT NULL,
    FinishedProvisional BIT          NOT NULL,
    FixtureID          INT           NOT NULL,
    GameWeek           INT           NOT NULL,
    KickoffTime        DATETIME2(0)  NOT NULL,
    TeamA              NVARCHAR(50)  NOT NULL,  -- FK to ClubDataDim.TeamID
    TeamADifficulty    INT           NULL,
    TeamAScore         INT           NOT NULL,
    TeamH              NVARCHAR(50)  NOT NULL,  -- FK to ClubDataDim.TeamID
    TeamHDifficulty    INT           NULL,
    TeamHScore         INT           NOT NULL,
    PulseID            INT           NULL,
    [Result]           NVARCHAR(20)  NULL,      -- "Home Win" / "Away Win" / "Draw"
    ExpectedResult     NVARCHAR(20)  NULL,
    ExceptionalResult  BIT           NULL,
    KickoffTime_TimeOnly AS CONVERT(TIME(0), KickoffTime) PERSISTED,

    CONSTRAINT PK_ResultsDim PRIMARY KEY (FixtureID),
    CONSTRAINT FK_ResultsDim_Fixture FOREIGN KEY (FixtureID) REFERENCES fpl.FixturesDim(FixtureID),
    CONSTRAINT FK_ResultsDim_TeamA FOREIGN KEY (TeamA) REFERENCES fpl.ClubDataDim(TeamID),
    CONSTRAINT FK_ResultsDim_TeamH FOREIGN KEY (TeamH) REFERENCES fpl.ClubDataDim(TeamID)
);
GO

-- ClubAnalysis (wide element attributes; link to Position + Club)
IF OBJECT_ID('fpl.ClubAnalysis') IS NOT NULL DROP TABLE fpl.ClubAnalysis;
CREATE TABLE fpl.ClubAnalysis
(
    Id               INT IDENTITY(1,1) PRIMARY KEY,
    PositionID       NVARCHAR(50)  NULL,   -- FK -> Position
    Team             NVARCHAR(50)  NULL,   -- TeamID (if needed)
    Team_Code        NVARCHAR(50)  NULL,   -- FK -> ClubDataDim.TeamCode
    Code             NVARCHAR(50)  NULL,
    First_Name       NVARCHAR(100) NULL,
    Second_Name      NVARCHAR(100) NULL,
    Web_Name         NVARCHAR(200) NULL,
    News             NVARCHAR(4000) NULL,
    News_Added       NVARCHAR(100) NULL,
    Status           NVARCHAR(50) NULL,
    In_DreamTeam     NVARCHAR(10) NULL,
    Selected_By_Percent FLOAT NULL,
    EP_Next          FLOAT NULL,
    EP_This          FLOAT NULL,
    Points_Per_Game  NVARCHAR(50) NULL,
    Now_Cost         NVARCHAR(50) NULL,
    Minutes          NVARCHAR(50) NULL,
    Goals_Scored     NVARCHAR(50) NULL,
    Assists          NVARCHAR(50) NULL,
    Clean_Sheets     NVARCHAR(50) NULL,
    Goals_Conceded   NVARCHAR(50) NULL,
    Own_Goals        NVARCHAR(50) NULL,
    Penalties_Saved  NVARCHAR(50) NULL,
    Penalties_Missed NVARCHAR(50) NULL,
    Yellow_Cards     NVARCHAR(50) NULL,
    Red_Cards        NVARCHAR(50) NULL,
    Saves            NVARCHAR(50) NULL,
    Bonus            NVARCHAR(50) NULL,
    BPS              NVARCHAR(50) NULL,
    Influence        NVARCHAR(50) NULL,
    Creativity       NVARCHAR(50) NULL,
    Threat           NVARCHAR(50) NULL,
    ICT_Index        NVARCHAR(50) NULL,
    Starts           NVARCHAR(50) NULL,
    Expected_Goals   NVARCHAR(50) NULL,
    Expected_Assists NVARCHAR(50) NULL,
    Expected_GI      NVARCHAR(50) NULL,
    Expected_GC      NVARCHAR(50) NULL,
    -- (…include other long-tail rank/order/text columns as needed …)

    CONSTRAINT FK_ClubAnalysis_Position FOREIGN KEY (PositionID) REFERENCES fpl.PositionDim(PositionID),
    CONSTRAINT FK_ClubAnalysis_TeamCode FOREIGN KEY (Team_Code) REFERENCES fpl.ClubDataDim(TeamCode)
);
GO

/* =======================
   FACT
   ======================= */

-- Detailed Player Data (Fact)
IF OBJECT_ID('fpl.DetailedPlayerDataFact') IS NOT NULL DROP TABLE fpl.DetailedPlayerDataFact;
CREATE TABLE fpl.DetailedPlayerDataFact
(
    -- Keys / references
    PlayerID            NVARCHAR(50)  NOT NULL,      -- 'Player ID' (dimension key)
    PositionID          NVARCHAR(50)  NULL,          -- 'Position ID'
    ClubTeamID          NVARCHAR(50)  NULL,          -- 'Club Data (DIM).Team ID'
    OppositionTeamID    NVARCHAR(50)  NULL,          -- 'Opposition Team ID'
    TeamCode            NVARCHAR(50)  NULL,          -- team_code
    FixtureID           INT           NULL,          -- 'Fixture ID'
    GameweekID          INT           NULL,          -- coerced to INT to link to GWDim
    KickoffTime         DATETIME2(0)  NULL,
    KickoffDate         DATE          NULL,          -- for FK to DateDim

    -- Measures/aggregates as base columns
    TotalPoints         INT           NULL,
    Was_Home            BIT           NULL,
    HomeTeamsScore      INT           NULL,
    AwayTeamScore       INT           NULL,
    MinutesPlayed       INT           NULL,
    GoalsScored         INT           NULL,
    AssistsMade         INT           NULL,
    CleanSheets         INT           NULL,
    GoalsConceded       INT           NULL,
    OwnGoals            INT           NULL,
    PenaltiesSaved      INT           NULL,
    PenaltiesMissed     INT           NULL,
    YellowCards         INT           NULL,
    RedCards            INT           NULL,
    Saves               INT           NULL,
    Bonus               INT           NULL,
    BPS                 INT           NULL,
    Influence           FLOAT         NULL,
    Creativity          FLOAT         NULL,
    Threat              INT           NULL,
    ICT_Index           FLOAT         NULL,
    Starts              INT           NULL,
    ExpectedGoals       FLOAT         NULL,
    ExpectedAssists     FLOAT         NULL,
    ExpectedGI          FLOAT         NULL,          -- expected_goal_involvements
    ExpectedGC          FLOAT         NULL,          -- expected_goals_conceded
    Value               INT           NULL,
    TransfersBalance    INT           NULL,
    Selected            INT           NULL,
    TransfersIn         INT           NULL,
    TransfersOut        INT           NULL,
    Price               DECIMAL(19,4) NULL,         -- currency
    DefensiveContribution INT         NULL,
    ClearancesBlocksInterceptions INT NULL,
    Modified            NVARCHAR(100) NULL,
    Column1             NVARCHAR(200) NULL,

    -- Derived column mirroring xGCDiff
    xGCDiff AS (CAST(GoalsScored AS FLOAT) - ISNULL(ExpectedGoals,0.0)) PERSISTED,

    CONSTRAINT PK_DetailedPlayerDataFact PRIMARY KEY (PlayerID, FixtureID),  -- composite key for row grain
    CONSTRAINT FK_Fact_Player    FOREIGN KEY (PlayerID)     REFERENCES fpl.PlayerDataDim(PlayerID),
    CONSTRAINT FK_Fact_Position  FOREIGN KEY (PositionID)   REFERENCES fpl.PositionDim(PositionID),
    CONSTRAINT FK_Fact_Club      FOREIGN KEY (ClubTeamID)   REFERENCES fpl.ClubDataDim(TeamID),
    CONSTRAINT FK_Fact_OppClub   FOREIGN KEY (OppositionTeamID) REFERENCES fpl.OppositionClubDataDim(TeamID),
    CONSTRAINT FK_Fact_Fixture   FOREIGN KEY (FixtureID)    REFERENCES fpl.FixturesDim(FixtureID),
    CONSTRAINT FK_Fact_GW        FOREIGN KEY (GameweekID)   REFERENCES fpl.GWDim(GWID),
    CONSTRAINT FK_Fact_Date      FOREIGN KEY (KickoffDate)  REFERENCES fpl.DateDim([Date])
);
GO

/* =======================
   SUPPORTING INDEXES
   ======================= */

-- Common query accelerators
CREATE INDEX IX_FixturesDim_Event ON fpl.FixturesDim (EventGW);
CREATE INDEX IX_ResultsDim_GW ON fpl.ResultsDim (GameWeek);
CREATE INDEX IX_Fact_GW ON fpl.DetailedPlayerDataFact (GameweekID);
CREATE INDEX IX_Fact_Player ON fpl.DetailedPlayerDataFact (PlayerID);
CREATE INDEX IX_Fact_Club ON fpl.DetailedPlayerDataFact (ClubTeamID);
CREATE INDEX IX_Fact_KickoffDate ON fpl.DetailedPlayerDataFact (KickoffDate);
GO

# NeuroIov Telegram Micro-App
## Business & Game Logic Documentation

### 1. Core Game Mechanics

#### 1.1 User Progression System

// Core User Stats

  xp: number;           // Experience points
  compute: number;      // Total compute power generated
  computePower: number; // Current compute power per tap
  totalTaps: number;    // Total number of taps
  level: number;        // User's current level
  gpuLevel: number;     // GPU upgrade level


// GPU Power System

  level: number;        // Current GPU level
  basePower: number;    // Base compute power
  upgradeCost: number;  // XP needed for next upgrade
  cooldownTime: Date;   // Cooldown period after intensive use


#### 1.2 Core Game Loop
1. **Tapping Mechanism**
   - Users tap to generate compute power
   - Each tap generates XP based on computePower
   - Taps accumulate towards total compute
   - Cooldown period activates after 500 taps
   - Tapping speed is rate-limited

2. **GPU System**
   - Base compute power starts at 1
   - GPU can be upgraded using XP
   - Each upgrade increases compute power
   - Upgrade cost scales with level
   - Cooldown system prevents spam

### 2. Progression Systems

#### 2.1 XP & Leveling

// Level Calculation

  currentXP: number;
  levelThresholds: number[];
  computePowerBonus: number[];


// XP Sources

  Tapping,           // Base tapping action
  Quests,            // Quest completion
  DailyBonus,        // Daily login bonus
  Achievements,      // Achievement completion
  ReferralBonus      // Referral rewards



#### 2.2 Daily Check-in System
- Daily XP claim available every 24 hours
- Streak system for consecutive logins
- Bonus rewards for maintaining streaks
- Auto-claim for new users

### 3. Social & Competitive Features

#### 3.1 Referral System

  referralCode: string;      // Unique referral code
  referredBy: string;        // User who referred current user
  referrals: string[];       // Users referred by current user
  referralChain: string[];   // Multi-level referral chain
  rewardTiers: {            // Reward percentages per tier
    tier1: 0.10,            // 10% for direct referrals
    tier2: 0.05,            // 5% for second level
    tier3: 0.025            // 2.5% for third level


#### 3.2 Leaderboard System


  daily: LeaderboardData;    // Reset every 24 hours
  weekly: LeaderboardData;   // Reset every week
  allTime: LeaderboardData;  // Permanent rankings


 LeaderboardData
  username: string;
  xp: number;
  compute: number;
  rank: number;


### 4. Quest & Achievement System

#### 4.1 Quest Types

  Daily = 'daily',
  Weekly = 'weekly',
  Twitter = 'twitter',
  Telegram = 'telegram',
  Discord = 'discord',
  Referral = 'referral',
  Achievement = 'achievement',
  Level = 'level',
  Leaderboard = 'leaderboard'

Quest interface
  title: string;
  description: string;
  xpReward: number;
  type: QuestType;
  action: QuestAction;
  requirement: number;
  expiresAt: Date;


#### 4.2 Achievement System

  id: string;
  name: string;
  description: string;
  xpReward: number;
  requirement: number;
  type: AchievementType;


AchievementType 
  Tap = 'tap',
  Social = 'social',
  Referral = 'referral',
  Level = 'level',
  Login = 'login',
  Customization = 'customization',
  Mining = 'mining',
  Leaderboard = 'leaderboard'


### 5. Reward & Boost Systems

#### 5.1 Boost Mechanics

  boostCount: number;        // Available boosts
  boostDuration: number;     // Duration in seconds (10)
  tapsPerSecond: number;     // Automated taps during boost (10)
  lastBoostTime: Date;       // Timestamp of last boost
  cooldownPeriod: number;    // Cooldown after boost


#### 5.2 Reward Distribution

  baseReward: number;          // Base XP per tap
  comboMultiplier: number;     // Bonus for consecutive taps
  referralBonus: number;       // Bonus from referral chain
  questRewards: number;        // Rewards from quests
  achievementBonus: number;    // One-time achievement rewards


### 6. Business Logic Rules

#### 6.1 Core Game Rules
1. **Tapping Rules**
   - Maximum tap rate: 10 per second
   - Cooldown triggers: Every 500 taps
   - Cooldown duration: 10 seconds
   - XP gain = computePower * tapCount

2. **GPU Upgrade Rules**
   - Upgrade cost = gpuLevel * 25000 XP
   - Power increase = +1 per upgrade
   - No maximum level cap
   - Upgrades are permanent

#### 6.2 Referral Rules
1. **Referral Chain**
   - Maximum 3 tiers deep
   - Unique referral codes
   - One referrer per user
   - No self-referrals
   - No circular referrals

2. **Reward Distribution**
   - Tier 1 (Direct): 10% of referred user's XP
   - Tier 2: 5% of referred user's XP
   - Tier 3: 2.5% of referred user's XP
   - Rewards distributed instantly

#### 6.3 Quest & Achievement Rules
1. **Quest Completion**
   - One-time completion per quest
   - Time-limited availability
   - Verification required for social quests
   - Auto-validation for in-game quests

2. **Achievement Progress**
   - Permanent progress tracking
   - Multiple tiers per achievement
   - Auto-updating progress
   - One-time rewards

### 7. User Session Management

#### 7.1 Authentication Rules
```typescript
interface AuthenticationRules {
  tokenExpiration: '7d';        // JWT token validity
  sessionTimeout: number;       // Inactive session timeout
  deviceTracking: boolean;      // Track user devices
  multiSessionAllowed: boolean; // Allow multiple active sessions
}
```

#### 7.2 Rate Limiting
```typescript
interface RateLimits {
  taps: number;         // Max taps per second
  requests: number;     // API requests per minute
  boosts: number;       // Boosts per hour
  upgrades: number;     // GPU upgrades per minute
}
```

### 8. Data Persistence Rules

#### 8.1 User Data
- Real-time stats updates
- Cached leaderboard data
- Periodic achievement checks
- Daily/weekly reset triggers
- Backup verification

#### 8.2 Game State
- Persistent user progress
- Quest completion status
- Achievement tracking
- Referral chain validation
- Reward distribution logs

This documentation outlines the core business and game logic implemented in the NeuroIov Telegram micro-app. The system is designed to create an engaging user experience while maintaining fair play and reward distribution.

Would you like me to elaborate on any specific aspect or provide more detailed technical specifications for any component?

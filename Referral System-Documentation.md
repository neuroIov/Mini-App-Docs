# Neurolov Compute Bot - Referral System Documentation

## Overview
The Neurolov Compute Bot implements a three-tier referral system that rewards users for bringing new members to the platform. The system uses a chain structure to track relationships between users and automatically distributes rewards based on compute activity.

## Core Components

### 1. Referral Chain Structure
- **Primary Referrals**: Direct invites (Level 1)
- **Secondary Referrals**: Invites by your direct referrals (Level 2)
- **Tertiary Referrals**: Invites by your secondary referrals (Level 3)
- Maximum chain depth: 3 levels
- Each user can have multiple referrals but only one referrer

### 2. Reward Distribution

#### XP Reward Percentages
- Primary (Level 1): 10% of referred user's earned XP
- Secondary (Level 2): 5% of referred user's earned XP
- Tertiary (Level 3): 2.5% of referred user's earned XP

Example:
```
User C (earns 100 XP)
↳ User B (5% = 5 XP) [Secondary referrer]
  ↳ User A (10% = 10 XP) [Primary referrer]
```

### 3. Referral Code System

- Each user gets a unique 8-character referral code (alphanumeric)
- Format: Uppercase hexadecimal (e.g., "A1B2C3D4")
- Codes are generated upon first request and remain constant
- Cannot be reused or transferred

### 4. Tracking & Statistics

The system tracks:
- Total number of referrals per level
- Total rewards earned from referrals
- Complete referral chain history
- Active vs. inactive referrals
- Per-referral reward distribution

## Workflow

### 1. Referral Generation
1. User requests referral code via UI
2. System generates unique code if not existing
3. Code is stored and linked to user account
4. User shares code with potential referrals

### 2. Code Redemption
1. New user enters referral code
2. System validates code format and existence
3. Checks for:
   - Valid code format
   - Code exists in system
   - User not already referred
   - Not self-referral
4. Creates referral relationship if valid

### 3. Chain Formation
1. System identifies referrer's position in existing chains
2. Establishes new user's position in chain
3. Updates all affected chain relationships
4. Initializes reward tracking

### 4. Reward Distribution

Triggered when referred user earns XP through:
- Tapping/Computing
- Quest completion
- Daily check-ins
- Achievements

Distribution Process:
1. Calculate base XP earned
2. Identify all eligible referrers in chain
3. Calculate percentage for each level
4. Distribute rewards immediately
5. Update reward statistics

## Limitations & Rules

1. **Chain Depth**
   - Maximum 3 levels deep
   - No circular references allowed
   - One referrer per user

2. **Code Usage**
   - One-time use per user
   - Cannot change referrer once set
   - Must be used before earning XP

3. **Reward Restrictions**
   - Must have valid referral relationship
   - No retroactive rewards
   - Referrer must be active account

4. **Technical Limits**
   - Rate limiting on code generation
   - Code redemption timeout period
   - Maximum referrals per user: Unlimited

## Monitoring & Maintenance

The system includes:
1. Referral health checks
2. Chain integrity verification
3. Reward distribution auditing
4. Anti-abuse measures

### Anti-Abuse Measures

1. Rate Limiting
   - Code generation limits
   - Redemption attempt limits
   - XP earning cooldowns

2. Verification
   - User account validation
   - Chain integrity checks
   - Reward distribution verification

3. Monitoring
   - Unusual activity detection
   - Chain depth verification
   - Reward distribution patterns

## Error Handling

Common scenarios handled:
1. Invalid referral codes
2. Duplicate redemption attempts
3. Chain integrity issues
4. Reward distribution failures
5. Rate limit exceeding

## Reporting

Available statistics:
1. Total referrals by level
2. Reward distribution history
3. Active referral chains
4. User referral performance
5. System-wide referral metrics

## Best Practices

1. Regular chain integrity checks
2. Periodic reward distribution audits
3. Monitor for unusual patterns
4. Regular database optimization
5. Performance monitoring of reward calculations

## Known Limitations

1. No retroactive referral applications
2. Cannot modify existing chains
3. No reward transfers between users
4. Fixed reward percentages
5. Cannot skip chain levels


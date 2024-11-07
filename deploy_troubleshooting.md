# Neurolov Compute Bot - Referral System Deployment & Troubleshooting Guide

## 1. System Requirements & Dependencies

### Required Software
- Node.js (v14+)
- MongoDB (v4.4+)
- Redis (for caching)

### NPM Dependencies
```json
{
  "mongoose": "^6.0.0",
  "crypto": "^1.0.1",
  "winston": "^3.3.3"
}
```

## 2. Deployment Checklist

### Database Setup
1. Create indexes for referral collections:
```javascript
db.referrals.createIndex({ "referrer": 1, "dateReferred": -1 })
db.referrals.createIndex({ "code": 1 }, { unique: true })
db.users.createIndex({ "referralCode": 1 }, { unique: true, sparse: true })
```

### Environment Variables
```bash
MONGODB_URI=mongodb://localhost:27017/neurolov
JWT_SECRET=your_jwt_secret
REDIS_URL=redis://localhost:6379
```

### Database Migrations
```javascript
// Add referralChain field to existing users
db.users.updateMany(
  { referralChain: { $exists: false } },
  { $set: { referralChain: [] } }
)
```

## 3. Testing & Verification

### System Health Checks
```javascript
// Test referral code generation
const testCode = generateReferralCode();
console.log('Generated Code:', testCode);

// Verify chain integrity
const verifyChains = async () => {
  const users = await User.find({ referredBy: { $exists: true } });
  for (const user of users) {
    const chain = await verifyReferralChain(user._id);
    console.log(`Chain for user ${user._id}:`, chain);
  }
};
```

## 4. Common Issues & Troubleshooting

### Issue 1: Broken Referral Chains
```sql
-- Find broken chains
SELECT * FROM users 
WHERE referred_by IS NOT NULL 
AND referred_by NOT IN (SELECT _id FROM users);

-- Fix broken chains
UPDATE users SET referred_by = NULL 
WHERE referred_by NOT IN (SELECT _id FROM users);
```

### Issue 2: Duplicate Rewards
```javascript
// Check for duplicate rewards
const findDuplicateRewards = async () => {
  const rewards = await Referral.aggregate([
    { $group: { 
      _id: { referrer: "$referrer", referred: "$referred" },
      count: { $sum: 1 }
    }},
    { $match: { count: { $gt: 1 } }}
  ]);
  return rewards;
};
```

### Issue 3: Performance Issues
```javascript
// Add compound indexes for better query performance
db.referrals.createIndex({ 
  "referrer": 1, 
  "tier": 1, 
  "dateReferred": -1 
});

// Implement caching for frequently accessed data
const cacheReferralStats = async (userId) => {
  const stats = await calculateReferralStats(userId);
  await redis.setex(`referral_stats:${userId}`, 3600, JSON.stringify(stats));
};
```

## 5. Monitoring Setup

### Logging Configuration
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ 
      filename: 'referral-error.log', 
      level: 'error' 
    }),
    new winston.transports.File({ 
      filename: 'referral-combined.log' 
    })
  ]
});
```

### Key Metrics to Monitor
```javascript
const monitorReferralMetrics = async () => {
  return {
    totalReferrals: await Referral.countDocuments(),
    activeChains: await User.countDocuments({ 
      referralChain: { $exists: true, $ne: [] } 
    }),
    avgChainDepth: await calculateAverageChainDepth(),
    rewardDistribution: await getRewardDistributionStats()
  };
};
```

## 6. Recovery Procedures

### Chain Repair
```javascript
const repairBrokenChain = async (userId) => {
  const user = await User.findById(userId);
  let chain = [];
  let current = user;
  
  while (current?.referredBy && chain.length < 3) {
    const referrer = await User.findById(current.referredBy);
    if (!referrer || chain.includes(referrer._id)) break;
    chain.push(referrer._id);
    current = referrer;
  }
  
  user.referralChain = chain;
  await user.save();
};
```

### Reward Recalculation
```javascript
const recalculateRewards = async (startDate, endDate) => {
  const users = await User.find({
    lastTapTime: { 
      $gte: startDate, 
      $lte: endDate 
    }
  });
  
  for (const user of users) {
    await distributeReferralXP(user._id, user.compute);
  }
};
```

## 7. Backup & Recovery

### Daily Backup Script
```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
mongodump --db neurolov --collection referrals --out backup_$TIMESTAMP
mongodump --db neurolov --collection users --out backup_$TIMESTAMP
```

### Recovery Process
```bash
#!/bin/bash
mongorestore --db neurolov --collection referrals backup_$1/neurolov/referrals.bson
mongorestore --db neurolov --collection users backup_$1/neurolov/users.bson
```

## 8. Performance Optimization

### Caching Strategy
```javascript
const CACHE_TTL = 3600; // 1 hour

const getReferralStats = async (userId) => {
  const cached = await redis.get(`referral_stats:${userId}`);
  if (cached) return JSON.parse(cached);
  
  const stats = await calculateReferralStats(userId);
  await redis.setex(
    `referral_stats:${userId}`, 
    CACHE_TTL, 
    JSON.stringify(stats)
  );
  return stats;
};
```

### Batch Processing
```javascript
const batchProcessRewards = async (userIds) => {
  const batchSize = 100;
  for (let i = 0; i < userIds.length; i += batchSize) {
    const batch = userIds.slice(i, i + batchSize);
    await Promise.all(
      batch.map(id => distributeReferralXP(id))
    );
  }
};
```

## 9. Security Considerations

### Rate Limiting
```javascript
const rateLimit = require('express-rate-limit');

const referralLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5 // limit each IP to 5 referral attempts per window
});

app.use('/api/referral', referralLimiter);
```

### Input Validation
```javascript
const validateReferralInput = (req, res, next) => {
  const { referralCode } = req.body;
  if (!referralCode || !validateReferralCode(referralCode)) {
    return res.status(400).json({ 
      error: 'Invalid referral code format' 
    });
  }
  next();
};
```



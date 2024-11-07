# Referral System Files Checklist

## 1. Core Backend Files

### Models
1. **models/User.js**
   - User schema referral fields
   - Referral chain relationships
   - Methods for referral validation
   ```javascript
   referredBy
   referrals
   referralCode
   referralChain
   ```

2. **models/Referral.js**
   - Referral relationship tracking
   - Reward distribution history
   ```javascript
   referrer
   referred
   code
   tier
   dateReferred
   totalRewardsDistributed
   ```

3. **models/Activity.js**
   - Tracks referral-related activities
   ```javascript
   type: 'referral'
   details: { referralType, rewards }
   ```

### Controllers
1. **controllers/userController.js**
   - `distributeReferralXP` function
   - XP calculations
   - User stats updates

2. **controllers/referralController.js** (if exists)
   - Referral code generation
   - Code validation
   - Referral relationship management

### Routes
1. **routes/referralRoutes.js**
   - API endpoints for referral operations
   - Code generation/validation
   - Stats retrieval

2. **routes/userRoutes.js**
   - User-related referral endpoints
   - Profile updates with referral data

### Utils
1. **utils/referralUtils.js**
   - Code generation functions
   - Validation helpers
   - Reward calculations

2. **utils/validationUtils.js**
   - Input validation for referral codes
   - Chain validation functions

### Middleware
1. **middleware/auth.js**
   - Authentication checks for referral operations

2. **middleware/rateLimiter.js**
   - Rate limiting for referral operations

## 2. Frontend Files

### Components
1. **components/Invite.js**
   - Referral UI
   - Code display
   - Sharing functionality

2. **components/Profile.js**
   - Referral stats display
   - User relationships

3. **components/ReferralStats.js** (if exists)
   - Detailed referral statistics
   - Rewards tracking

### Services
1. **services/api.js**
   - API calls for referral operations
   ```javascript
   generateReferralCode
   applyReferralCode
   getReferralStats
   ```

2. **services/referralService.js** (if exists)
   - Referral-specific API calls
   - Data formatting

### Context/State
1. **context/AuthContext.js**
   - User authentication
   - Referral permissions

2. **context/ReferralContext.js** (if exists)
   - Referral state management
   - Stats caching

## 3. Configuration Files

1. **config/default.js** or **.env**
   ```javascript
   REFERRAL_REWARD_RATES
   MAX_REFERRAL_CHAIN_DEPTH
   REFERRAL_CODE_LENGTH
   ```

2. **config/database.js**
   - Database indexes for referral collections

## 4. Testing Files

1. **tests/referral.test.js**
   - Unit tests for referral functions
   - Integration tests for referral flow

2. **tests/models/referral.test.js**
   - Model validation tests
   - Relationship tests

## 5. Documentation Files

1. **docs/referral-system.md**
   - System documentation
   - API endpoints

2. **docs/database-schema.md**
   - Schema documentation
   - Relationship diagrams

## 6. Key Functions to Review

### Referral Creation
```javascript
// referralRoutes.js
router.post('/generate-code')
router.post('/apply-code')

// referralUtils.js
generateReferralCode()
validateReferralCode()
```

### Reward Distribution
```javascript
// userController.js
distributeReferralXP()
calculateReferralReward()

// User.js
userSchema.methods.addReferralReward()
```

### Chain Management
```javascript
// referralUtils.js
verifyReferralChain()
updateReferralChain()

// User.js
userSchema.methods.getReferralChain()
```

## 7. Database Queries to Check

### Chain Integrity
```javascript
// Check for broken chains
db.users.find({
  referredBy: { $exists: true },
  referralChain: { $size: 0 }
})

// Find invalid referral codes
db.users.find({
  referralCode: { $exists: true },
  $where: "this.referralCode.length != 8"
})
```

### Reward Distribution
```javascript
// Check reward distribution
db.referrals.aggregate([
  {
    $group: {
      _id: "$referrer",
      totalRewards: { $sum: "$totalRewardsDistributed" }
    }
  }
])
```

## 8. Common Update Scenarios

### 1. Modifying Reward Rates
Files to update:
- utils/referralUtils.js
- config files
- documentation

### 2. Changing Chain Depth
Files to update:
- models/User.js
- utils/referralUtils.js
- controllers/userController.js

### 3. Updating Validation Rules
Files to update:
- utils/referralUtils.js
- middleware/validation.js
- tests/referral.test.js

### 4. Adding New Referral Features
Files to update:
- models/Referral.js
- routes/referralRoutes.js
- frontend components
- documentation

## 9. Monitoring Points

### Performance Monitoring
```javascript
// Key queries to monitor
db.referrals.find().explain("executionStats")
db.users.find({ referralChain: { $exists: true } }).explain("executionStats")
```

### Error Logging
```javascript
// Key error points
distributeReferralXP
applyReferralCode
updateReferralChain
```

## 10. Backup Considerations

### Critical Data
```javascript
// Collections to backup
users: referralCode, referralChain
referrals: all data
activities: referral-related
```

### Restore Priority
1. User referral relationships
2. Referral codes
3. Reward history
4. Activity logs

## 11. Performance Impact Points

1. Chain depth calculations
2. Reward distribution
3. Stats aggregation
4. Code generation/validation

When updating the referral system, ensure to: @gokul 
1. Test all changes in development environment
2. Verify chain integrity
3. Check reward calculations
4. Update related documentation
5. Update test cases
6. Monitor system performance
7. Backup data before major changes


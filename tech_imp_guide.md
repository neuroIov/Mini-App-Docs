# Neurolov Compute Bot - Referral System Technical Implementation Guide

## 1. Database Schema

### Referral Model (Referral.js)
```javascript
const referralSchema = new mongoose.Schema({
  referrer: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'User', 
    required: true, 
    index: true 
  },
  referred: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'User', 
    required: true, 
    unique: true 
  },
  code: { 
    type: String, 
    required: true, 
    unique: true, 
    index: true 
  },
  tier: { 
    type: Number, 
    required: true, 
    min: 1, 
    max: 3 
  },
  dateReferred: { 
    type: Date, 
    default: Date.now 
  },
  totalRewardsDistributed: { 
    type: Number, 
    default: 0 
  }
});
```

### User Model Referral Fields (User.js)
```javascript
{
  referredBy: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'User', 
    index: true 
  },
  referrals: [{ 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'User' 
  }],
  referralCode: { 
    type: String, 
    unique: true, 
    sparse: true, 
    index: true 
  },
  referralChain: [{ 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'User' 
  }]
}
```

## 2. Core Utilities (referralUtils.js)

### Code Generation
```javascript
const generateReferralCode = () => {
  return crypto.randomBytes(4).toString('hex').toUpperCase();
};
```

### Code Validation
```javascript
const validateReferralCode = (code) => {
  return typeof code === 'string' && 
         code.length === 8 && 
         /^[0-9A-F]{8}$/i.test(code);
};
```

### Reward Calculation
```javascript
const calculateReferralReward = (level, xp) => {
  const percentages = [0.1, 0.05, 0.025]; // 10%, 5%, 2.5%
  return Math.floor(xp * (percentages[level - 1] || 0));
};
```

## 3. API Endpoints (referralRoutes.js)

### Generate Referral Code
```javascript
router.post('/generate-code', auth, async (req, res) => {
  try {
    const user = await User.findOne({ telegramId: req.user.telegramId });
    if (!user) return res.status(404).json({ message: 'User not found' });

    if (!user.referralCode) {
      user.referralCode = generateReferralCode();
      await user.save();
    }

    res.json({ referralCode: user.referralCode });
  } catch (error) {
    logger.error('Error generating referral code:', error);
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});
```

### Apply Referral Code
```javascript
router.post('/apply-code', auth, async (req, res) => {
  try {
    const { referralCode } = req.body;
    if (!validateReferralCode(referralCode)) {
      return res.status(400).json({ message: 'Invalid referral code format' });
    }

    const referredUser = await User.findOne({ telegramId: req.user.telegramId });
    if (!referredUser) return res.status(404).json({ message: 'User not found' });
    if (referredUser.referredBy) return res.status(400).json({ message: 'You have already used a referral code' });

    const referrer = await User.findOne({ referralCode });
    if (!referrer) return res.status(400).json({ message: 'Invalid referral code' });
    if (referrer.telegramId === referredUser.telegramId) 
      return res.status(400).json({ message: 'You cannot refer yourself' });

    referredUser.referredBy = referrer._id;
    referrer.referrals.push(referredUser._id);

    // Create a new Referral document
    const newReferral = new Referral({
      referrer: referrer._id,
      referred: referredUser._id,
      code: referralCode,
      tier: 1, // First-level referral
    });

    await Promise.all([
      referredUser.save(), 
      referrer.save(), 
      newReferral.save()
    ]);

    res.json({ message: 'Referral code applied successfully' });
  } catch (error) {
    logger.error('Error applying referral code:', error);
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});
```

### Get Referral Stats
```javascript
router.get('/stats', auth, async (req, res) => {
  try {
    const user = await User.findOne({ telegramId: req.user.telegramId })
      .populate('referrals');
    if (!user) return res.status(404).json({ message: 'User not found' });

    const referrals = await Referral.find({ referrer: user._id });

    const stats = {
      totalReferrals: user.referrals.length,
      referralCode: user.referralCode,
      referralStats: referrals.map(r => ({
        username: r.referred.username,
        date: r.dateReferred,
        tier: r.tier,
        rewardsDistributed: r.totalRewardsDistributed
      }))
    };

    res.json(stats);
  } catch (error) {
    logger.error('Error fetching referral stats:', error);
    res.status(500).json({ message: 'Server error', error: error.message });
  }
});
```

## 4. Reward Distribution System (userController.js)

### Distribute Referral XP
```javascript
const distributeReferralXP = async (userId, xpGained) => {
  try {
    const user = await User.findById(userId).populate('referredBy');
    if (!user || !user.referredBy) return;

    let currentReferrer = user.referredBy;
    let currentTier = 1;
    const tierPercentages = [0.1, 0.05, 0.025]; // 10%, 5%, 2.5%

    while (currentReferrer && currentTier <= 3) {
      const referralXP = Math.floor(xpGained * tierPercentages[currentTier - 1]);
      currentReferrer.xp += referralXP;
      
      // Update the referral document
      await Referral.findOneAndUpdate(
        { referrer: currentReferrer._id, referred: user._id },
        { $inc: { totalRewardsDistributed: referralXP } }
      );

      await currentReferrer.save();
      logger.info(`Distributed ${referralXP} XP to referrer ${currentReferrer._id} (Tier ${currentTier})`);

      currentReferrer = await User.findOne({ _id: currentReferrer.referredBy });
      currentTier++;
    }
  } catch (error) {
    logger.error(`Error distributing referral XP: ${error.message}`);
  }
};
```

## 5. Frontend Implementation (Invite.js)

### Referral UI Component
```javascript
const InviteWrapper = styled(motion.div)`
  position: relative;
  min-height: 100vh;
  width: 100vw;
  color: #ffffff;
  background-color: #000033;
  background-image: url(${backgroundImage});
  background-size: cover;
  background-position: center;
  display: flex;
  flex-direction: column;
`;

const ReferralLinkContainer = styled.div`
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 20px;
  width: 100%;
`;

const ReferralInfo = styled.p`
  font-size: 14px;
  margin-top: 20px;
  text-align: center;
  opacity: 0.8;
`;
```

### Referral Code Generation and Management
```javascript
useEffect(() => {
  const fetchData = async () => {
    try {
      const [userData, referralData, friendsData] = await Promise.all([
        api.get('/api/user'),
        api.get('/api/referrals/stats'),
        api.get('/api/friends')
      ]);

      setUser(userData);
      setReferralEarnings(referralData.referralXP);
      setFriends(referralData.topReferrals);
      setReferralCode(referralData.referralCode);
      setReferralLink(`https://t.me/Neurolov?start=${referralData.referralCode}`);
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  };

  fetchData();
}, [api]);
```

## 6. Integration Points

### XP Distribution Integration (tap action)
```javascript
exports.tap = async (req, res) => {
  try {
    const user = await User.findOne({ telegramId: req.user.telegramId });
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }

    // ... tap logic ...

    const xpGained = user.computePower;
    user.xp += xpGained;

    await user.save();

    // Distribute referral XP
    await distributeReferralXP(user._id, xpGained);

    // ... rest of tap logic ...

    res.json({ 
      message: 'Tap successful', 
      user: {
        xp: user.xp,
        compute: user.compute,
        // ... other user data ...
      }
    });
  } catch (error) {
    logger.error(`Tap error: ${error.message}`);
    res.status(500).json({ message: 'Server error', error: error.message });
  }
};
```

## 7. Common Issues and Solutions

### Issue: Referral Chain Breaking
Solution: Implement integrity checks
```javascript
const verifyReferralChain = async (userId) => {
  const user = await User.findById(userId).populate('referredBy');
  let chain = [];
  let currentUser = user;
  
  while (currentUser?.referredBy && chain.length < 3) {
    chain.push(currentUser.referredBy._id);
    currentUser = await User.findById(currentUser.referredBy._id)
      .populate('referredBy');
  }
  
  return chain;
};
```

### Issue: Duplicate Referral Codes
Solution: Add unique index and validation
```javascript
userSchema.pre('save', async function(next) {
  if (this.isModified('referralCode')) {
    const existing = await this.constructor
      .findOne({ referralCode: this.referralCode });
    if (existing && existing._id !== this._id) {
      throw new Error('Referral code already exists');
    }
  }
  next();
});
```

### Issue: Reward Distribution Failures
Solution: Implement transaction-like behavior
```javascript
const distributeRewardsWithRetry = async (userId, xpGained, attempts = 3) => {
  try {
    await distributeReferralXP(userId, xpGained);
  } catch (error) {
    if (attempts > 0) {
      await new Promise(resolve => setTimeout(resolve, 1000));
      return distributeRewardsWithRetry(userId, xpGained, attempts - 1);
    }
    throw error;
  }
};
```

## 8. Performance Considerations

1. **Indexing Strategy**
   - Compound index on referral chains
   - Index on referral codes
   - Index on user relationships

2. **Caching**
   - Cache referral codes
   - Cache referral statistics
   - Cache user relationships

3. **Batch Processing**
   - Batch reward distributions
   - Batch chain updates
   - Periodic integrity checks


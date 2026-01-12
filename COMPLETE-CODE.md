# COMPLETE CODE - Copy These Files Locally

Your frontend is deployed! Now add these remaining files locally.

## Already Deployed ✅
- public/index.html
- public/styles.css
- public/success.html
- package.json
- DEPLOYMENT-GUIDE.md

## Files to Add Locally

### 1. netlify.toml
```toml
[build]
  publish = "public"
  command = "echo 'No build step needed'"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### 2. firebase.json
```json
{
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "functions": {
    "source": "functions"
  }
}
```

### 3. firestore.rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /subscriptions/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if false;
    }
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### 4. firestore.indexes.json
```json
{"indexes":[],"fieldOverrides":[]}
```

### 5. functions/package.json
```json
{
  "name": "oodlesanddoodles-functions",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "firebase-admin": "^12.0.0",
    "firebase-functions": "^4.6.0"
  },
  "engines": {
    "node": "18"
  }
}
```

### 6. functions/index.js
```javascript
const functions = require('firebase-functions');
const admin = require('firebase-admin');
const crypto = require('crypto');

admin.initializeApp();
const db = admin.firestore();

function verifyPaddleSignature(signature, body, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(JSON.stringify(body));
  return signature === hmac.digest('hex');
}

exports.paddleWebhook = functions.https.onRequest(async (req, res) => {
  if (req.method !== 'POST') return res.status(405).send('Method Not Allowed');
  
  const signature = req.headers['paddle-signature'];
  const webhookSecret = process.env.PADDLE_WEBHOOK_SECRET;
  
  if (!verifyPaddleSignature(signature, req.body, webhookSecret)) {
    return res.status(401).send('Unauthorized');
  }
  
  const event = req.body;
  console.log('Paddle event:', event.event_type);
  
  try {
    const userId = event.data.custom_data?.userId;
    if (!userId) return res.status(200).send('OK');
    
    switch (event.event_type) {
      case 'subscription.created':
      case 'subscription.activated':
        await db.collection('subscriptions').doc(userId).set({
          paddleSubscriptionId: event.data.id,
          status: 'active',
          plan: determinePlan(event.data.items[0]?.price?.id),
          currentPeriodEnd: event.data.current_billing_period?.ends_at,
          updatedAt: admin.firestore.FieldValue.serverTimestamp()
        }, { merge: true });
        
        await db.collection('users').doc(userId).update({
          isPremium: true
        });
        break;
        
      case 'subscription.canceled':
        await db.collection('subscriptions').doc(userId).update({
          status: 'canceled',
          updatedAt: admin.firestore.FieldValue.serverTimestamp()
        });
        
        await db.collection('users').doc(userId).update({
          isPremium: false
        });
        break;
    }
    
    res.status(200).send('OK');
  } catch (error) {
    console.error('Webhook error:', error);
    res.status(500).send('Error');
  }
});

function determinePlan(priceId) {
  const plans = {
    [process.env.PADDLE_WEEKLY_PRODUCT_ID]: 'weekly',
    [process.env.PADDLE_MONTHLY_PRODUCT_ID]: 'monthly',
    [process.env.PADDLE_YEARLY_PRODUCT_ID]: 'yearly'
  };
  return plans[priceId] || 'unknown';
}

exports.checkSubscription = functions.https.onCall(async (data, context) => {
  if (!context.auth) {
    throw new functions.https.HttpsError('unauthenticated', 'Must be authenticated');
  }
  
  const userId = context.auth.uid;
  const sub = await db.collection('subscriptions').doc(userId).get();
  
  if (!sub.exists) return { hasPremium: false };
  
  const subData = sub.data();
  return {
    hasPremium: subData.status === 'active',
    plan: subData.plan,
    currentPeriodEnd: subData.currentPeriodEnd
  };
});
```

## Quick Deploy Commands

```bash
# Clone repo
git clone https://github.com/sjackss/oodlesanddoodles-subscriptions.git
cd oodlesanddoodles-subscriptions

# Add the files above
# Then commit
git add .
git commit -m "Add Firebase backend and config"
git push

# Deploy
netlify deploy --prod
firebase deploy --only functions
```

## Next: Get Paddle Keys

1. Complete signup at https://paddle.com
2. Get API key from Developer Tools
3. Create 3 products (Weekly/Monthly/Yearly)
4. Add webhook URL from Firebase
5. Update public/index.html with product IDs

**Repository:** https://github.com/sjackss/oodlesanddoodles-subscriptions

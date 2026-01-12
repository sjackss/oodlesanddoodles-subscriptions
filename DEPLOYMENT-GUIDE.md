# Oodles & Doodles Subscription System - Complete Deployment Guide

## Overview
This system integrates Paddle billing for your dating app with three subscription tiers:
- **Weekly**: $6.99/week with 7-day free trial
- **Monthly**: $23.99/month with 1-month free trial  
- **Yearly**: $88.82/year with 60-day free trial

## Quick Start

### 1. Clone this repository locally
```bash
git clone https://github.com/sjackss/oodlesanddoodles-subscriptions.git
cd oodlesanddoodles-subscriptions
```

### 2. Set up Paddle Account
1. Go to https://paddle.com and create account
2. Complete KYC verification  
3. Navigate to Developer Tools > Authentication
4. Copy your API Key
5. Navigate to Developer Tools > Notifications  
6. Copy your Webhook Signing Key

### 3. Create Paddle Products
In Paddle Dashboard > Catalog > Products:

**Product 1: Weekly Plan**
- Name: Oodles & Doodles Weekly
- Price: $6.99 USD
- Billing Interval: Weekly
- Trial Period: 7 days
- Copy the Product ID

**Product 2: Monthly Plan**
- Name: Oodles & Doodles Monthly  
- Price: $23.99 USD
- Billing Interval: Monthly
- Trial Period: 30 days
- Copy the Product ID

**Product 3: Yearly Plan**
- Name: Oodles & Doodles Yearly
- Price: $88.82 USD  
- Billing Interval: Yearly
- Trial Period: 60 days
- Copy the Product ID

### 4. Set up Firebase
1. Go to https://console.firebase.google.com
2. Create new project: oodlesanddoodles-prod
3. Enable Firestore Database
4. Enable Authentication
5. Download service account key:
   - Project Settings > Service Accounts > Generate New Private Key
   - Save as `firebase-service-account.json` (DO NOT commit to git)

### 5. Install Dependencies
```bash
npm install
```

### 6. Configure Environment Variables
Create `.env` file in root:
```
PADDLE_API_KEY=your_paddle_api_key_here
PADDLE_WEBHOOK_SECRET=your_paddle_webhook_secret_here
PADDLE_WEEKLY_PRODUCT_ID=your_weekly_product_id
PADDLE_MONTHLY_PRODUCT_ID=your_monthly_product_id  
PADDLE_YEARLY_PRODUCT_ID=your_yearly_product_id
FIREBASE_PROJECT_ID=oodlesanddoodles-prod
```

### 7. Deploy to Firebase Functions
```bash
cd functions
npm install
firebase login
firebase init functions
firebase deploy --only functions
```

Copy the webhook URL (e.g., `https://us-central1-oodlesanddoodles-prod.cloudfunctions.net/paddleWebhook`)

### 8. Configure Paddle Webhook
1. In Paddle Dashboard > Developer Tools > Notifications
2. Add webhook endpoint URL from step 7
3. Select all subscription events

### 9. Deploy Frontend to Netlify
```bash
netlify login
netlify init
netlify deploy --prod
```

## Project Structure
```
/
├── public/
│   ├── index.html          # Subscription selection page
│   ├── success.html        # Post-checkout success
│   └── styles.css          # Oodles & Doodles brand styling
├── functions/
│   ├── index.js            # Firebase Functions (webhooks)
│   └── package.json        # Functions dependencies
├── .env                    # Environment variables (DO NOT COMMIT)
├── .gitignore
├── netlify.toml            # Netlify configuration
├── firebase.json           # Firebase configuration  
└── package.json
```

## Next Steps

After completing this guide:

1. **Test the flow**: Subscribe using Paddle's test mode
2. **Integrate with your main app**: Check subscription status from Firestore
3. **Add user authentication**: Link Paddle customer IDs to your users
4. **Style the pages**: Match your brand colors

---

## COMPLETE CODE FILES

### File: `public/index.html`
Create this file with subscription plan cards and Paddle.js integration...

(Continue in next message due to length)

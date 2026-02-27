---
name: spend-optimizer
description: Audit subscriptions AND optimize credit card rewards. Analyzes bank CSVs to find recurring charges, categorize spend, identify which card each charge should be on, and calculate potential rewards. Use when user says "audit subscriptions", "optimize my cards", "maximize rewards", or "analyze my spending". Outputs interactive HTML with subscription management and card optimization.
---

# spend-optimizer

Complete spend optimization: subscription auditing + credit card rewards maximization.

## What This Does

1. **Subscription Audit** - Find recurring charges, categorize (keep/cancel/investigate), calculate savings
2. **Card Optimization** - Map charges to optimal credit cards based on reward categories
3. **Credit Optimization** - Flag services you're paying for that should be FREE via card credits
4. **Interactive Reports** - HTML dashboards with dispute tracking, notes, and actionable insights

## Workflow

### Step 1: Get Transaction CSVs

Ask user for bank/card CSV exports. Support multiple cards for full optimization.

**Common export sources:**
- **Apple Card**: Wallet → Card Balance → Export
- **Amex**: Account → Statements & Activity → Download
- **Chase**: Accounts → Download activity → CSV
- **Mint/Copilot**: Transactions → Export all

**Important**: Request exports from ALL cards for accurate optimization. Note the last 4 digits of each card number.

### Step 2: Gather Card Information

Ask user about their credit cards:
```
What credit cards do you have? For each, I need:
- Last 4 digits (for matching to transactions)
- Card name (e.g., "Amex Gold", "Chase Sapphire")
- Annual fee (if known)
```

Check [credit-cards.md](references/credit-cards.md) for common card reward structures.

### Step 3: Analyze Transactions

Read all CSVs and identify:

**Recurring charges:**
- Same merchant, similar amounts, monthly/annual frequency
- Flag subscription-like charges (streaming, SaaS, memberships)
- Note charge frequency and total annual cost

**Card optimization opportunities:**
- Which card each charge is currently on
- Which card it SHOULD be on for max rewards
- Calculate potential rewards gain from moving charges

**Credit/perk opportunities:**
- Check [credit-cards.md](references/credit-cards.md) Statement Credits section
- Flag any service the user is paying for that's covered by a card credit they have
- Examples:
  - Paying for Disney+ on Chase → Should be on Amex Platinum (free $7/mo credit)
  - Paying for Uber on random card → Should be on Amex Platinum ($15/mo) or Amex Gold ($10/mo)
  - Paying for DoorDash → Should be on Chase Sapphire Reserve (free DashPass + $5/mo)
  - Paying for Adobe → Should be on Amex Business Platinum ($150/yr credit)
  - Paying for cell phone bill → Should be on Amex Business Platinum ($10/mo wireless credit)

### Step 4: Categorize with User

For each subscription, ask user to set status:
- **Keep** - Intentional, continue paying
- **Cancel** - Stop immediately
- **Investigate** - Needs decision (unsure, contract trap)
- **Dispute** - Never got access, want refund

Ask in batches of 5-10 to avoid overwhelming. Include card optimization recommendation with each.

### Step 5: Generate HTML Report

Use [template.html](assets/template.html) to create interactive report:

**Header section:**
- Subscription count and total monthly spend
- Potential annual rewards with optimized card usage
- Savings from cancelled subscriptions
- Dispute count and value

**Card legend:**
- List all user's cards with color coding
- Show reward categories for each card

**Subscription rows include:**
- Service name and monthly amount
- Current card (color-coded badge)
- Optimal card with "OK" or "MOVE" indicator
- Status dropdown (Keep/Cancel/Investigate/Dispute)
- Notes field for user comments
- Frequency (monthly/annual/etc.)

**Features:**
- Auto-save to localStorage
- Export disputes button
- Privacy toggle (blur names)
- Collapsible sections
- Dark mode support

### Step 6: Generate Card Optimization Plan

If significant optimization opportunities exist, create [card-optimization.html](assets/card-optimization-template.html):

- Card arsenal overview (all cards, fees, reward structures)
- Spend routing strategy table
- Action items for card migration
- Quick reference cheat sheet
- Monthly checklist
- ROI summary (fees vs rewards)

### Step 7: Help Cancel & Dispute

When user is ready to act:

**Cancellations:**
- Check [common-services.md](references/common-services.md) for cancel URLs
- Use browser automation to navigate and cancel if requested
- Update HTML row status when done

**Disputes:**
- Export disputes from HTML report
- Provide guidance on disputing with card issuer
- Note: Most cards allow disputes within 60-120 days

## Card Optimization Logic

### Common Reward Categories

**Advertising/Marketing:**
- Facebook, Google, TikTok, Pinterest, LinkedIn ads
- Best cards: Amex Business Gold (4x), Chase Ink (3x)

**Software/SaaS:**
- Shopify, Notion, Adobe, cloud services
- Best cards: Amex Business Gold (4x), Amex Business Platinum (1.5x on $5K+)

**Shipping:**
- ShipBob, UPS, FedEx, USPS
- Best cards: Amex Business Gold (4x), Chase Ink (3x)

**Travel:**
- Flights, hotels, car rentals
- Best cards: Chase Sapphire Reserve (3-10x), Amex Platinum (5x)

**Dining:**
- Restaurants, food delivery
- Best cards: Chase Sapphire Reserve (3x), Amex Gold (4x)

**Internet/Phone:**
- ISP, cell phone, cable
- Best cards: Chase Ink Preferred (3x)

### Category Caps

Many cards have annual caps on bonus categories:
- Amex Business Gold: $150K per category at 4x
- Chase Ink: $150K combined at 3x

Track spending against caps and recommend overflow cards.

## File Structure

```
spend-optimizer/
├── SKILL.md                 # This file
├── assets/
│   ├── template.html        # Subscription audit template
│   └── card-optimization-template.html  # Card strategy template
└── references/
    ├── common-services.md   # Cancel URLs and tips
    └── credit-cards.md      # Card reward structures
```

## Privacy

All data stays local:
- Transaction CSVs analyzed in-session only
- HTML reports stored on user's machine
- localStorage for user preferences
- No data sent anywhere

## Tips

**For maximum value:**
1. Get CSVs from ALL cards (not just one)
2. Look at 6+ months of data for pattern detection
3. Track category cap progress quarterly
4. Update card payment methods for "MOVE" items
5. Set calendar reminders for annual subscriptions

**Red flags to surface:**
- Duplicate subscriptions (same service, multiple charges)
- **Price increases** - Compare same merchant across months (see Price Increase Detection below)
- Zombie charges (services no longer used)
- Trial conversions (free trial → paid)
- **Annual charges** - Flag yearly subscriptions with renewal estimates (see Annual Tracker below)
- **Missed credits** - Paying for something covered by a card credit (see below)
- **Category cap warnings** - Alert when approaching $150K limits (see Cap Tracker below)

**Credit opportunities to flag:**

Always check if user is paying for any of these on the wrong card:

| If paying for... | Move to this card | Credit value |
|------------------|-------------------|--------------|
| Disney+, Hulu, ESPN+ | Amex Platinum | $7/mo FREE |
| Peacock | Amex Platinum | $7/mo FREE |
| Audible | Amex Platinum | $7/mo FREE |
| NYTimes | Amex Platinum | $7/mo FREE |
| Apple TV+ | Amex Platinum | $6.99/mo FREE |
| Walmart+ | Amex Platinum | $12.95/mo FREE |
| Uber/Uber Eats | Amex Platinum | $15/mo FREE |
| Uber/Uber Eats | Amex Gold | $10/mo FREE |
| Grubhub/Seamless | Amex Gold | $10/mo FREE |
| DoorDash | Chase Sapphire Reserve | $5/mo + DashPass FREE |
| DoorDash | Capital One Venture X | $10/mo FREE |
| Instacart | Chase Sapphire Reserve | Instacart+ FREE |
| Adobe Creative Cloud | Amex Business Platinum | $150/yr credit |
| Dell purchases | Amex Business Platinum | $400/yr credit |
| Indeed job posts | Amex Business Platinum | $360/yr credit |
| Cell phone bill | Amex Business Platinum | $10/mo credit |
| Clear | Amex Platinum | $199/yr FREE |
| Equinox/SoulCycle | Amex Platinum | $25/mo credit |
| Global Entry/PreCheck | Multiple cards | $100 credit |

**Report format for missed credits:**

When flagging a missed credit, show:
```
⚠️ MISSED CREDIT: You're paying $X.XX/mo for [Service] on [Current Card]
   → Move to [Credit Card] to get it FREE ($XX/yr savings)
```

---

## Price Increase Detection

Compare the same merchant across multiple months to catch silent price increases.

**How to detect:**
1. Group transactions by merchant name (normalize: lowercase, remove extra spaces)
2. For recurring charges, compare amounts month-over-month
3. Flag if current amount > previous amount

**Report format:**
```
⚠️ PRICE INCREASE: [Service] went from $X.XX to $Y.YY (+Z%)
   First seen at new price: [Date]
   Consider: Is this still worth it at the new price?
```

**Common patterns:**
- Streaming services: Often increase $1-2/mo annually
- SaaS tools: May increase 10-20% at renewal
- Annual subscriptions: Harder to notice since you only see once/year

**Thresholds:**
- Flag any increase > $1 or > 5%
- Highlight increases > 20% as significant

---

## Annual Subscription Tracker

Annual charges are easy to forget. Flag them and estimate renewal dates.

**How to detect:**
1. Look for charges that appear only once in 6-12 months of data
2. Check if amount suggests annual billing ($100+ single charge vs $10 monthly)
3. Look for merchant names containing: "annual", "yearly", or known annual services

**Known annual services to watch for:**
- Domain registrations (GoDaddy, Namecheap, Google Domains)
- Annual software licenses (Adobe, Microsoft 365)
- Memberships (Costco, Amazon Prime, warehouse clubs)
- Professional subscriptions (industry associations)
- Insurance premiums
- Accounting software (QuickBooks, FreshBooks)

**Report format:**
```
📅 ANNUAL SUBSCRIPTION: [Service] - $XXX/year
   Last charged: [Date]
   Estimated renewal: [Date + 1 year]
   ⏰ Set reminder for [2 weeks before renewal]
```

**In HTML report:**
- Add "Annual" badge to these subscriptions
- Show estimated renewal date
- Sort annual subscriptions by upcoming renewal date

---

## Category Cap Tracker

Track spending against bonus category caps to maximize rewards.

**Key caps to track:**

| Card | Category | Cap | Multiplier After |
|------|----------|-----|------------------|
| Amex Business Gold | Each bonus category | $150,000/yr | 1x |
| Amex Blue Business Plus | All purchases | $50,000/yr | 1x |
| Chase Ink Preferred | Combined bonus categories | $150,000/yr | 1x |
| Chase Ink Cash | Office supplies | $25,000/yr | 1% |
| Chase Ink Cash | Internet/phone | $25,000/yr | 1% |
| Chase Ink Cash | Gas/dining | $25,000/yr | 1% |
| BofA Customized Cash | Choice category | $50,000/yr | 1% |

**How to track:**
1. Categorize each transaction by reward category
2. Sum spending per category per card
3. Calculate percentage of cap used
4. Project when cap will be hit based on monthly average

**Report format:**
```
📊 CATEGORY CAP STATUS: [Card Name]

Advertising:     $87,500 / $150,000 (58%) ████████░░░░
Software:        $42,000 / $150,000 (28%) ████░░░░░░░░
Shipping:        $12,000 / $150,000 (8%)  █░░░░░░░░░░░

⚠️ At current pace, you'll hit Advertising cap in ~2.5 months
   → Consider moving overflow to Chase Ink Preferred (3x)
```

**Alerts:**
- 🟡 Warning at 75% of cap
- 🔴 Alert at 90% of cap
- Suggest overflow card when approaching limit

**Quarterly review:**
- Q1: Check if on pace to hit caps
- Q2: Mid-year adjustment if needed
- Q3: Start planning overflow strategy
- Q4: Final push to maximize before reset

**Cap reset dates:**
- Amex: Calendar year (Jan 1)
- Chase: Cardmember year (anniversary date)
- Check card terms for specific reset timing

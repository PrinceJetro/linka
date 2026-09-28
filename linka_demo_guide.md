# 🌍 Linka — Full Demo & Test Walkthrough

> **Make sure both servers are running before you start.**
> - Backend: `python manage.py runserver` (from `backend/linka/`)
> - Frontend: `npm run dev` (from `frontend/`)
> - Open: `http://localhost:3000`

---

## 🔐 1. Registration Flow

**Go to:** `http://localhost:3000/register`

Fill in the form:
| Field | Test Value |
|---|---|
| First name | Your name |
| Last name | Your surname |
| Username | `testuser1` |
| Email | `test@linka.africa` |
| Password | `test12345` |

> **What happens behind the scenes:**
> The backend registers the user. The app then redirects to `/?onboarding=1` which opens the capability profile tab and shows a green welcome banner prompting them to fill out their profile.

**What to verify:**
- ✅ You're redirected to the dashboard after registering
- ✅ You see a green "Welcome [Name]! One last step." banner
- ✅ The "My capability profile" tab is open and the form is ready to fill

---

## 🔑 2. Login Flow

**Log out, then go to:** `http://localhost:3000/login`

Try the demo account that was seeded:
| Username | Password |
|---|---|
| `demo` | `demo12345` |

**What to verify:**
- ✅ Logs in and lands on the dashboard
- ✅ Dashboard shows your name in the greeting
- ✅ JWT tokens are stored in `localStorage` (check DevTools → Application → Local Storage)

---

## 🏢 3. Capability Profile — Create

Navigate to: **"My capability profile"** in the left sidebar.

Fill in the form with this test data:
| Field | Test Value |
|---|---|
| Name / Organization | `Demo Agro Exports` |
| Country | Nigeria |
| City | `Lagos` |
| Industry | `Agriculture` |
| Products / Services | `Processed yam flour and cassava starch` |
| What can you offer? | `We offer 5,000 kg/month manufacturing capacity for processed yam flour` |
| What do you need? | `A distribution partner in Ghana or Kenya for export` |
| Partnership type | `Distribution` |
| Target countries | ✅ Ghana ✅ Kenya |

Click **"Publish profile"**.

**What to verify:**
- ✅ Profile card appears above the form
- ✅ Profile is visible in the network

---

## ✏️ 4. Capability Profile — Edit & Delete

Click the **Edit** button on your profile card.

- Change the city to `Abuja`
- Click **"Save changes"**

**What to verify:**
- ✅ Profile updates instantly
- ✅ The form resets after saving

Then click **Delete** to test the delete flow (you can recreate it).

---

## 🔍 5. AI Matcher — Finding Partners

Navigate to: **"Find partners"** in the sidebar.

**Test 1 — Simple search (rule-based):**
Type: `"Distributor in Ghana for food products"`
Click Search.

**What to verify:**
- ✅ Returns ranked profiles (Kora Distribution Co., Nia Foods & Retail, etc.)
- ✅ Each match shows a % score and a reason
- ✅ "Understood as" chips show extracted intent (industry, country, partnership type)

**Test 2 — AI-powered search (if GEMINI_API_KEY is set):**
Type: `"I need cold-chain logistics from Nigeria to Ghana"`

**What to verify:**
- ✅ Response shows ✨ AI badge in the "Understood as" section
- ✅ Matches are semantically smarter (e.g. Mansa Freight ranks high)
- ✅ Your own profiles are excluded from results when logged in

**Test 3 — Filter by country/industry:**
- Set Country filter to `Kenya`
- Set Industry to `Technology`
- Search: `"fintech platform"`

**What to verify:**
- ✅ Only Kenyan tech profiles appear (Nairobi PayTech, Cape AgriTech)

---

## 🗺️ 6. Opportunity Map

Navigate to: **"Opportunity map"** in the sidebar.

**What to verify:**
- ✅ An interactive Africa map loads (Leaflet + OpenStreetMap)
- ✅ Country pins show with counts of profiles
- ✅ Clicking a country pin shows that country's profile count
- ✅ Filter by sector (e.g. select "Healthcare") — map updates

---

## 📨 7. Partnership Request Flow

From the **Find partners** search results, click on **"AgroLink Nigeria"** (or any seeded profile).

Click **"Connect"** or **"Send request"**.

Fill in the message:
> "We have a retail network in Accra and are looking to partner with cassava processors in Nigeria for a 3-month pilot. Can we discuss terms?"

Click **Send**.

**What to verify:**
- ✅ Request is sent
- ✅ Bell notification appears in the top bar
- ✅ Request shows in **"Partnership requests"** with status "Pending"

---

## 📬 8. Receiving & Responding to Requests (Two-user test)

Open an **incognito window** and log in as the `demo` account.

Go to **"Partnership requests"**.

**What to verify:**
- ✅ The request you sent from your account appears in the demo account's inbox
- ✅ The `demo` account sees: **Accept**, **Decline**, or **Ask for info** buttons (because it's the receiver)
- ✅ Check the requests you sent — they should just say "⏳ Waiting for their response" without action buttons.

Click **Accept**.

**What to verify (Success Modal):**
- ✅ A pop-up overlay appears showing "Partnership accepted!"
- ✅ It shows the two connecting businesses (e.g., `Demo Agro Exports ↔ Kora Distribution Co.`)
- ✅ It provides quick action buttons to "💬 Send a message" or "📜 Draft an MOU"

Switch back to your main account:
- ✅ The request status has updated from pending to a green "✓ Partnership active" badge
- ✅ The Accept/Decline buttons are gone
- ✅ A notification appears in the bell

---

## 📝 9. Partnership Brief (AI Feature)

From the **Find partners** results, click on any profile card.

Click **"Generate brief"** (or look for a "Partnership Brief" button on the profile).

**What to verify:**
- ✅ A brief is generated with Opportunity / Benefits / Verify sections
- ✅ If Gemini key is set, the brief is AI-written and shows an ✨ badge
- ✅ If no key, a template brief is still generated (graceful fallback)

---

## 💬 10. Direct Messaging

Navigate to: **"Messages"** in the sidebar.

If you accepted a request earlier, a conversation thread should exist.

Click on the conversation and type:
> "Great to connect! Can we schedule a call this week?"

Hit Send.

**What to verify:**
- ✅ Message appears instantly in the thread
- ✅ Unread count badge shows on the other account (in incognito)
- ✅ Reading the message marks it as read (badge disappears)

---

## 📜 11. MOU Draft Generation

Navigate to: **"Partnerships"** in the sidebar.

Find the accepted partnership and click **"Draft MOU"**.

**What to verify:**
- ✅ A Memorandum of Understanding is generated in markdown format
- ✅ Shows ✨ AI-generated badge if Gemini key is set
- ✅ **"Download .md"** button downloads the file

---

## ⭐ 12. Endorsements (Reviews)

On the same accepted partnership card, click **"Endorse partner"**.

- Select rating: `5 ★`
- Write: `"Excellent distribution partner, delivered on time."`
- Click **"Submit verified review"**

**What to verify:**
- ✅ Review is saved
- ✅ It appears on the partner's profile page

---

## 🔔 13. Notifications Bell

Click the bell icon in the top bar.

**What to verify:**
- ✅ Shows all activity: received requests, accepted requests, responses
- ✅ Count badge clears after viewing

---

## 📊 14. Admin Dashboard

Go to: `http://127.0.0.1:8000/admin/`

Log in with your superuser credentials (if created with `python manage.py createsuperuser`).

**What to verify:**
- ✅ See all users, profiles, requests, messages
- ✅ Approve a **Verification Badge** for a profile (Profiles → select one → check `is_verified`)
- ✅ That profile now shows a verified badge on the frontend

---

## 🌐 15. Metrics Strip

**API test:** `GET http://127.0.0.1:8000/api/metrics/`

**What to verify (in API response):**
- ✅ `profiles` count
- ✅ `searches` count  
- ✅ `avg_top_score`
- ✅ `conversion_pct` (requests ÷ searches)

---

## 🌍 16. AfCFTA Trade Info Lane

**API test via browser or Postman:**
```
POST http://127.0.0.1:8000/api/trade-info/
Body: { "from_country": "NG", "to_country": "GH", "product": "processed food" }
```

**What to verify:**
- ✅ Returns tariff info, required certifications, payment methods
- ✅ AI-enriched if Gemini key is set

---

## ✅ Quick Test Checklist

| Feature | Status |
|---|---|
| Register new user | ⬜ |
| Login with demo / demo12345 | ⬜ |
| Create capability profile | ⬜ |
| Edit capability profile | ⬜ |
| Delete capability profile | ⬜ |
| AI Matcher search (rule-based) | ⬜ |
| AI Matcher search (Gemini ✨) | ⬜ |
| Opportunity Map (filters) | ⬜ |
| Send partnership request | ⬜ |
| Accept/Decline request | ⬜ |
| Partnership Brief (AI) | ⬜ |
| Direct messaging | ⬜ |
| MOU draft + download | ⬜ |
| Endorsement / Review | ⬜ |
| Notification bell | ⬜ |
| Admin - approve verification | ⬜ |
| Metrics API | ⬜ |
| AfCFTA trade-info API | ⬜ |

---

## 🌱 Seeded Demo Profiles Summary

| Company | Country | Industry | Verified |
|---|---|---|---|
| AgroLink Nigeria | 🇳🇬 Lagos | Agriculture | ✅ |
| Kora Distribution Co. | 🇬🇭 Accra | Logistics | ✅ |
| Nia Foods & Retail | 🇬🇭 Kumasi | Agriculture | ✅ |
| Mansa Freight | 🇬🇭 Tema | Logistics | ❌ |
| Nairobi PayTech | 🇰🇪 Nairobi | Technology | ✅ |
| Kigali Research Lab | 🇷🇼 Kigali | Research | ❌ |
| Ubuntu Packaging | 🇿🇦 Johannesburg | Manufacturing | ✅ |
| Sahel Grains Board | 🇳🇬 Kano | Agriculture | ❌ |
| Lagos HealthMeds | 🇳🇬 Lagos | Healthcare | ✅ |
| Accra Pharma Works | 🇬🇭 Accra | Healthcare | ✅ |
| Kumasi Cocoa Processors | 🇬🇭 Kumasi | Manufacturing | ❌ |
| Savanna Solar Kits | 🇰🇪 Nairobi | Manufacturing | ✅ |
| Mombasa Port Runners | 🇰🇪 Mombasa | Logistics | ❌ |
| Kigali FinServe | 🇷🇼 Kigali | Finance | ❌ |
| Cape AgriTech | 🇿🇦 Cape Town | Technology | ✅ |
| Jozi Creative Hub | 🇿🇦 Johannesburg | Creative | ❌ |
| Nairobi EdTech College | 🇰🇪 Nairobi | Education | ❌ |
| Cairo MedSupply | 🇪🇬 Cairo | Healthcare | ❌ |

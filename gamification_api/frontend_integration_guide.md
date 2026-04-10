# Frontend Integration FAQ & Workflows

> **Audience:** Frontend / Mobile developers
> **Purpose:** Step-by-step workflows explaining how the gamification features connect to the UI.

---

## 1. The Daily Reward (7-Day Check-in Pop-up)

**Q: At what point does the Daily Reward popup open? Does it happen when they open the app?**  
**A:** Yes, it should open immediately when the app launches (or when the user lands on the Home screen), but **only if the backend tells you to show it**.

### The Workflow:
1. **Connect & Receive Instructions:**
   When the app opens, it connects to the WebSocket (`ws://<host>/ws/realtime/`). The server *immediately* pushes an `initial_data` event to the client.
2. **Check the Flag:** 
   Inside that payload, look for `data.gamification.show_daily_reward_popup`. 
   - If `true`: The frontend should immediately pop up the 7-day calendar modal.
   - If `false`: Do not show the modal.
3. **Get the Calendar Data:** 
   The same `initial_data` payload contains `data.gamification.streak.rewards_calendar`. This is an array of 7 days, indicating what the rewards are and which day is `"current"` (today's claimable reward).
4. **Claiming the Reward (User Action):**
   When the user taps the glowing "Claim" button for today, the frontend sends this WebSocket message:
   ```json
   { "type": "claim_daily_reward", "data": {} }
   ```
   *(Alternatively, you can call `POST /gamification/streak/check-in/` over REST)*
5. **The Success Animation:** 
   The server will reply with a `daily_reward_claimed` WebSocket event containing the new coins earned and the updated calendar. The frontend should play a coin/success animation, change the day's status to `"claimed"`, and update the user's wallet balance.

*For exact JSON structures, refer to [streak_and_rewards.md](./streak_and_rewards.md) and the [README](./README.md).*

---

## 2. Daily Missions (List & Tasks)

**Q: How do the daily missions work? What endpoint do I call to get the list?**  
**A:** Use the `/gamification/missions/` endpoint or WebSocket event to fetch all missions grouped by type.

### The Workflow:
1. **Fetching the List:**
   Call `GET /gamification/missions/` (or send `{ "type": "get_missions", "data": {} }` via WebSocket). 
   This returns an object containing arrays for `newcomer`, `daily`, and `weekly` missions.
2. **What the User Clicks On (The "Go" Button):**
   Every mission in the list has an `action_route` field (e.g., `"/vent-room"`, `"/profile/edit"`, `"/community"`). 
   When the user taps the mission card or a "Go" button, the frontend should **deep-link the user directly to that screen** inside the app so they can perform the action.
3. **How Progress is Tracked (Auto-Magic):**
   The frontend *does not* need to calculate or send progress updates! If the mission is "Create a Vent Post," the moment the user successfully creates a post via the normal Vent Room API, the backend updates the mission progress automatically.
4. **Mission Complete (Server Push):**
   When the mission reaches 100%, the backend automatically pushes a `mission_complete` WebSocket event to the app. 
   When you receive this event, the frontend should instantly update that mission's UI to show a glowing **"Claim Reward"** button instead of a "Go" button.
5. **Claiming the Mission Reward:** 
   When the user clicks "Claim", the frontend sends:
   ```json
   {
     "type": "claim_mission",
     "data": { "mission_id": "uuid-of-the-mission" }
   }
   ```
   *(Or call `POST /gamification/missions/{id}/claim/` via REST).*
   The server responds with `mission_claimed` and the coins earned, at which point you show an animation and update the wallet.

*For a visual lifecycle diagram, refer to [missions.md](./missions.md).*

---

## 3. Real-Time Animations & Modals

**Q: How do pop-ups for daily rewards and badges work?**  
**A:** Everything relies on **WebSocket Server Pushes**. The frontend just needs to listen for specific events and trigger the UI accordingly.

### Key Events to Listen For:
- **`badge_earned`**: The backend pushes this the exact moment a user crosses a threshold (like checking in for the 7th day, or making their 50th post). **Frontend Action:** Immediately display a celebratory "Congratulations" modal taking over the screen.
- **`streak_milestone`**: Similar to badges, but for streak milestones. **Frontend Action:** Show a celebratory toast or modal showing the streak count.
- **`leaderboard_update`**: Pushed only when the user's rank *improves*. **Frontend Action:** Show a gentle, slide-up toast notification (e.g., "📈 You moved up to #5 on Most Active!").

Because these events come over the WebSocket `ws://<host>/ws/realtime/`, the user experiences gamification instantly, exactly when they complete an action.

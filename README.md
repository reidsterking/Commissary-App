Commissary Bagger Queue — quick setup

1) Create a Firebase project
   - Go to https://console.firebase.google.com/ and create a project.
   - In the project, go to "Realtime Database" and create a database in your preferred location.
   - For quick testing, set Database Rules to:
     {
       "rules": {
         ".read": true,
         ".write": true
       }
     }
     (This allows open read/write; for production tighten rules.)

2) Get your Firebase config
   - Project settings -> SDK setup -> copy the "Firebase SDK snippet" (config).
   - Replace the firebaseConfig object at the top of index.html with your config values.

3) Host the page
   - Quick: Upload index.html to any static host (GitHub Pages, Netlify, plain web server).
   - Or use Firebase Hosting (I can add files but you must run firebase init/deploy with your Firebase project and replace firebaseConfig in index.html).

4) Usage (updated)
   - First visit: enter your name and tap "Join the line". The page will generate a client id and remember your name so reloading keeps your spot.
   - After joining you’ll see the Queue screen with a sticky top bar showing your live status (# in line or At the register), a scrollable list showing "At the Register" and "In Line" groups, and a sticky bottom action bar with two buttons: "Move to Register" and "Move to End of Line" which affect only your entry.
   - Tap your own name (in the top bar or your row) to immediately leave the queue (confirmation modal). Tapping someone else starts a 10-minute pending removal (or cancels it if already pending). Pending removals are visible to everyone with a live mm:ss countdown. Clients automatically delete entries once the pending timer expires.

Notes & improvements
 - The app uses Firebase Realtime Database. The data model for each person is:
     {
       name: string,
       status: 'line' | 'register',
       ts: server timestamp (for ordering),
       pendingRemovalAt: number | null (ms since epoch when deletion will occur)
     }
 - I kept the firebaseConfig placeholder in index.html exactly as before so you can paste your real config.
 - If you want, I can:
    - Insert your firebaseConfig for you and push the update.
    - Add authentication to restrict actions to staff.
    - Make pending-removal use server-calculated timestamps for tighter correctness.

Which of the above would you like next?

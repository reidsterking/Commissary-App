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

4) Usage
   - Open the page on any device.
   - Enter your name, choose "In line" or "At a register", then Add.
   - Other devices will see changes instantaneously.

Notes & improvements
 - The "Take Next" operation reads then writes; if two people click "Take Next" at the same moment there is a small chance they both pull the same person. If you need strict atomicity, I can update the app to use Firebase transactions or a Cloud Function to guarantee atomic move-of-first-item.
 - I can add authentication (e.g., sign-in) so only staff can modify registers, or add a small admin UI for skipping people, reordering, or auto-assign rules.

If you want, I can:
 - Deploy these files to a GitHub repo and optionally set up GitHub Pages or Firebase Hosting for you.
 - Add atomic transaction behavior for Take Next.
 - Add optional sign-in so only authorized users can change registers.

Which would you like me to do next?

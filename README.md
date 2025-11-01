🚀 Reactive Clicks

Reactive Clicks is a fast-paced browser game built with SvelteKit and Vite.
It includes two dynamic mini-games — a CPS (Clicks Per Second) test and an Aim Trainer — each featuring responsive design, particle effects, and smooth animations.

🧩 Features

⚡ Built with SvelteKit & Vite for fast development and instant hot reloads.

🎯 Aim Trainer Mode – Click on moving asteroid targets before they disappear. Miss 5 times, and it’s game over.

🖱️ CPS Test Mode – Measure how fast you can click in 10 seconds.

🌈 Animated Gradient Backgrounds and dynamic lighting effects.

💥 Particle Explosions for interactive feedback.

📱 Fully Responsive – Play seamlessly on desktop or mobile.

🛠️ Tech Stack
Technology	Purpose
SvelteKit
	Frontend framework
Vite
	Development & build tooling
JavaScript / HTML / CSS
	Core languages
Orbitron Font
	Sci-fi themed typography
🧠 Project Structure
src/
 ├─ lib/
 │   ├─ components/
 │   │   ├─ CPSTest.svelte
 │   │   ├─ AimTrainer.svelte
 │   │   └─ UI/
 │   │       └─ FireBar.svelte
 │   └─ app.html
 ├─ routes/
 │   ├─ +layout.svelte
 │   └─ +page.svelte
 └─ app.css


CPSTest.svelte → Handles the Clicks Per Second test logic and UI

AimTrainer.svelte → Game loop for the asteroid-style aim trainer

FireBar.svelte → Visual intensity indicator based on player performance

🧪 Development Setup
1. Install Dependencies
npm install

2. Start the Development Server
npm run dev


Then visit http://localhost:5173
 in your browser.

3. Build for Production
npm run build

4. Preview Production Build
npm run preview

🚧 Future Improvements

🪐 Additional game modes and difficulty scaling

🔊 Sound design improvements and ambient effects

🧩 High-score tracking (local storage or backend integration)

🎨 Customizable themes
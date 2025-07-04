<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Qord</title>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
  <!-- Tone.js for audio synthesis -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
  <style>
    /* Global box-sizing for consistent layout */
    *, *::before, *::after {
      box-sizing: border-box;
    }

    :root {
      --primary-bg: #1e1e1e;
      --secondary-bg: #2d2d2d;
      --tertiary-bg: #2c2c2c;
      --text-color: #f5f5f5;
      --accent-blue: #007bff;
      --accent-gradient: linear-gradient(45deg, #007bff, #00d4ff);
    }
    
    /* Ensure html and body take full height for proper layout and sticky footer */
    html {
      height: 100%; /* Make html take full viewport height */
    }
    body {
      min-height: 100%; /* Ensure body expands to at least full viewport height */
      margin: 0;
      padding: 0;
      font-family: 'Inter', sans-serif;
      background-color: var(--primary-bg);
      color: var(--text-color);
      display: flex; /* Use flexbox to stack main content and nav */
      flex-direction: column; /* Stack children vertically */
    }

    main {
      flex-grow: 1; /* Allow main content to take available space */
      padding: 20px;
      padding-bottom: 90px; /* Ensure space for fixed bottom nav */
      overflow-y: auto; /* Allow scrolling within main content if needed */
    }

    .page {
      display: none; /* Hidden by default */
      animation: fadeIn 0.5s;
    }
    .page.active {
      display: block; /* Shown when active */
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }

    /* Tab Navigation */
    .bottom-nav {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 70px;
      background: var(--secondary-bg);
      display: flex;
      justify-content: space-around;
      align-items: center;
      border-top: 1px solid #444;
      z-index: 1000;
      border-radius: 15px 15px 0 0; /* Rounded top corners */
      box-shadow: 0 -4px 10px rgba(0,0,0,0.3); /* Add a subtle shadow */
    }
    .nav-button {
      background: none;
      border: none;
      color: var(--text-color);
      opacity: 0.7;
      cursor: pointer;
      font-size: 14px; /* Slightly smaller font for mobile */
      padding: 8px; /* Adjusted padding */
      border-radius: 8px;
      transition: opacity 0.3s, color 0.3s;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 4px;
      flex: 1; /* Distribute space evenly */
      min-width: 60px; /* Ensure minimum touch target size */
    }
    .nav-button.active {
      opacity: 1;
      color: var(--accent-blue);
    }
    .nav-button:hover {
        opacity: 1;
    }

    /* General Styles */
    h1, h2 {
      text-align: center;
      background: var(--accent-gradient);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
    }
    h1 { font-size: 2.5em; margin-bottom: 30px; }
    h2 { font-size: 2em; margin-bottom: 20px; }
    .container { max-width: 900px; margin: 0 auto; padding: 0 10px; } /* Added horizontal padding for mobile */
    .input-group { margin-bottom: 20px; }
    label { display: block; margin-bottom: 5px; font-weight: bold; }
    input, select, textarea, button {
      width: 100%; padding: 12px; margin-top: 5px; border: none;
      border-radius: 8px; font-size: 14px; box-sizing: border-box;
    }
    input, select, textarea {
      background: var(--tertiary-bg); color: var(--text-color); border: 1px solid #444;
      border-radius: 8px; /* Added rounded corners */
    }
    button.action-button {
      background: linear-gradient(45deg, #007bff, #0056b3);
      color: white; cursor: pointer; font-weight: bold;
      transition: transform 0.2s, box-shadow 0.2s;
      border-radius: 8px; /* Added rounded corners */
      padding: 15px; /* Larger touch target */
    }
    button.action-button:hover:not(:disabled) { /* Only hover if not disabled */
      transform: translateY(-2px); box-shadow: 0 4px 12px rgba(0, 123, 255, 0.3);
    }
    button.action-button:disabled {
        opacity: 0.5;
        cursor: not-allowed;
    }
    .button-group { 
      display: flex; 
      gap: 10px; 
      margin-top: 20px; 
      flex-wrap: wrap; /* Allow buttons to wrap on smaller screens */
      justify-content: center; /* Center buttons when wrapped */
    } 
    .button-group button { 
      flex: 1; 
      min-width: 140px; /* Ensure buttons are somewhat flexible but readable */
    } 

    .output {
      margin-top: 20px; padding: 20px; background: var(--tertiary-bg);
      border-radius: 10px; white-space: pre-wrap; border: 1px solid #444; line-height: 1.6;
      box-shadow: inset 0 0 5px rgba(0,0,0,0.2); /* Subtle inner shadow */
    }
    audio { width: 100%; margin-top: 10px; }
    .hidden { display: none; }
    
    /* Library Styles */
    #libraryList { list-style: none; padding: 0; }
    .library-item {
      background: var(--tertiary-bg);
      padding: 15px;
      margin-bottom: 10px;
      border-radius: 8px;
      border-left: 5px solid var(--accent-blue);
      cursor: pointer;
      transition: background-color 0.2s;
      box-shadow: 0 2px 5px rgba(0,0,0,0.2); /* Add shadow to list items */
    }
    .library-item:hover { background-color: #3a3a3a; }
    .library-item h3 { margin: 0 0 5px 0; }
    .library-item p { margin: 0; font-size: 14px; opacity: 0.8; }
    
    /* Song Detail View (Modal) - CRITICAL FIX HERE */
    #songDetailView {
        display: none !important; /* Force hide on load to prevent black screen issue */
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: rgba(0,0,0,0.8); /* Slightly darker overlay */
        align-items: center; /* Used when display is flex */
        justify-content: center; /* Used when display is flex */
        backdrop-filter: blur(8px); /* Increased blur for better visual separation */
        z-index: 1001; /* Ensure it's above nav */
    }
    #songDetailContent {
        background: var(--secondary-bg); padding: 25px; border-radius: 12px;
        width: 90%; max-width: 800px; max-height: 85vh; /* Increased max-height */
        overflow-y: auto;
        position: relative; /* For close button positioning */
        box-shadow: 0 8px 25px rgba(0,0,0,0.6); /* More prominent shadow */
    }
    #songDetailContent .close-button {
      position: absolute;
      top: 10px;
      right: 10px;
      background: var(--tertiary-bg); /* Background for close button */
      border: none;
      color: var(--text-color);
      font-size: 1.2em; /* Adjusted font size */
      cursor: pointer;
      padding: 8px 12px; /* Adjusted padding */
      border-radius: 50%; /* Circular button */
      transition: background-color 0.2s, transform 0.2s;
    }
    #songDetailContent .close-button:hover {
      background-color: #444;
      transform: rotate(90deg); /* Little animation */
    }

    .coming-soon-text {
        font-size: 1.5em; text-align: center; opacity: 0.7; margin-top: 40px;
    }

    /* Cover Art Styles */
    #coverArtContainer {
        margin-top: 20px;
        text-align: center;
    }
    #coverArt {
        max-width: 100%;
        height: auto;
        border-radius: 10px;
        box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
        margin-top: 10px;
        display: block; /* Ensures it's centered with text-align */
        margin-left: auto;
        margin-right: auto;
    }
    .loading-indicator {
      text-align: center;
      margin-top: 20px;
      font-size: 1.2em;
      color: var(--accent-blue);
    }
    .loading-indicator::after {
      content: ' .';
      animation: dots 1s steps(5, end) infinite;
    }
    @keyframes dots {
      0%, 20% { color: rgba(0,0,0,0); text-shadow: .25em 0 0 rgba(0,0,0,0), .5em 0 0 rgba(0,0,0,0); }
      40% { color: var(--accent-blue); text-shadow: .25em 0 0 var(--accent-blue), .5em 0 0 rgba(0,0,0,0); }
      60% { text-shadow: .25em 0 0 var(--accent-blue), .5em 0 0 rgba(0,0,0,0); }
      80%, 100% { text-shadow: .25em 0 0 var(--accent-blue), .5em 0 0 var(--accent-blue); }
    }

    /* Audio controls container */
    #audioControls {
        display: flex;
        flex-wrap: wrap; /* Allow wrapping on small screens */
        gap: 10px;
        margin-top: 20px;
        justify-content: center;
        align-items: center; /* Align items vertically */
    }
    #audioControls button {
        flex: 1;
        max-width: 150px;
        min-width: 80px; /* Ensure buttons are not too small */
    }
    .audio-progress-container {
        display: flex;
        align-items: center;
        gap: 10px;
        width: 100%; /* Take full width for progress bar */
        margin-top: 10px; /* Space between buttons and progress bar */
    }
    #progressBar {
        flex-grow: 1; /* Allow slider to take available space */
        -webkit-appearance: none; /* Remove default styling */
        height: 8px;
        background: #555;
        outline: none;
        opacity: 0.7;
        transition: opacity .2s;
        border-radius: 4px;
    }
    #progressBar::-webkit-slider-thumb {
        -webkit-appearance: none;
        appearance: none;
        width: 18px;
        height: 18px;
        border-radius: 50%;
        background: var(--accent-blue);
        cursor: pointer;
    }
    #progressBar::-moz-range-thumb {
        width: 18px;
        height: 18px;
        border-radius: 50%;
        background: var(--accent-blue);
        cursor: pointer;
    }
    #currentTime, #totalDuration {
        font-size: 0.9em;
        min-width: 40px; /* Ensure space for time display */
        text-align: center;
    }

    /* New Post-Generation Actions Section */
    #postGenerationActions {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
        margin-top: 20px;
        justify-content: center;
        padding-top: 20px;
        border-top: 1px solid #444;
    }
    #postGenerationActions button {
        flex: 1;
        max-width: 200px;
    }


    /* Media Queries for smaller screens */
    @media (max-width: 600px) {
      h1 { font-size: 2em; }
      h2 { font-size: 1.7em; }
      .nav-button { font-size: 12px; padding: 6px; }
      .button-group button { min-width: 100%; } /* Stack buttons fully on very small screens */
      main { padding: 15px; padding-bottom: 80px; } /* Adjust padding */
      .container { padding: 0 15px; } /* More padding on very small screens */
      .input-group label { font-size: 0.9em; }
      input, select, textarea, button { font-size: 13px; padding: 10px; }
      #audioControls button { min-width: unset; flex-basis: 48%; } /* Two buttons per row */
      .audio-progress-container { flex-direction: column; align-items: stretch; }
      #progressBar { width: 100%; }
      #currentTime, #totalDuration { width: 100%; text-align: center; margin-top: 5px; }
      #postGenerationActions button { min-width: 100%; } /* Stack buttons fully on very small screens */
    }
  </style>
</head>
<body>

  <main>
    <div id="libraryPage" class="page active"> <!-- Start on Library page -->
      <div class="container">
        <h2>🎵 Song Library</h2>
        <ul id="libraryList">
          <li class="library-item" style="opacity: 0.6;">Your saved songs will appear here.</li>
        </ul>
      </div>
    </div>

    <div id="generatorPage" class="page">
        <div class="container">
            <h1>🤖 Qord</h1>
            <div class="button-group">
                <button class="action-button" onclick="toggleSongType()">🔄 Song Type: <span id="typeLabel">Normal</span></button>
            </div>
            <div class="input-group">
                <label for="songTitle">Song Title</label>
                <input type="text" id="songTitle" placeholder="Enter song title or auto-generate..." oninput="handleTitleInput();">
            </div>
            <div class="button-group">
                <button class="action-button" onclick="aiAutoFillAll()" id="aiAutoFillAllBtn">✨ AI Auto-Fill All</button>
                <button class="action-button" onclick="generateCover()" id="generateCoverBtn">🎨 Generate Cover</button>
            </div>
            <div class="input-group">
                <label for="genre">Genre</label>
                <select id="genre" required disabled> <!-- Disabled, will be set by AI -->
                    <option value="">-- Select Genre --</option>
                    <option>Pop</option><option>Rock</option><option>Hip-Hop</option>
                    <option>Electronic</option><option>R&B</option><option>Country</option>
                </select>
            </div>
            <div class="input-group">
                <label for="mood">Mood</label>
                <select id="mood" required disabled></select> <!-- Disabled, will be set by AI -->
            </div>
            <div class="input-group" id="lyricsBlock">
                <label for="lyrics">Lyrics</label>
                <textarea id="lyrics" rows="12" placeholder="Lyrics will be generated based on title..." disabled></textarea> <!-- Disabled, will be set by AI -->
            </div>
            <div class="input-group" id="instrumentsBlock">
                <label for="instruments">Instruments</label>
                <textarea id="instruments" rows="4" placeholder="Instruments will be generated by AI..." disabled></textarea>
            </div>
            <div class="input-group" id="voiceBlock">
                <label for="voice">Voice</label>
                <select id="voice"> <option value="male">Male</option> <option value="female">Female</option> </select>
            </div>
            <div class="input-group" id="voiceDescriptionBlock">
                <label for="voiceDescription">Voice Description (e.g., raspy, high-pitched)</label>
                <input type="text" id="voiceDescription" placeholder="Describe the voice">
            </div>
            <div class="button-group">
                <button class="action-button" onclick="generateSong()" id="generateBtn">🎧 Generate Song</button>
            </div>
            <div id="coverArtContainer">
                <img id="coverArt" src="https://placehold.co/300x300/1e1e1e/f5f5f5?text=No+Cover" alt="Song Cover" class="hidden">
                <div id="coverLoading" class="loading-indicator hidden">Generating cover...</div>
            </div>
            <div id="output" class="output">Your generated song details will appear here...</div>
            <div id="audioControls" class="hidden">
                <button class="action-button" onclick="skipBackward()">⏪ -10s</button>
                <button class="action-button" onclick="togglePlayPause()" id="playPauseBtn">▶ Play</button>
                <button class="action-button" onclick="skipForward()">⏩ +10s</button>
                <button class="action-button" onclick="stopAudioFull()">⏹ Stop</button>
                <div class="audio-progress-container">
                    <span id="currentTime">0:00</span>
                    <input type="range" id="progressBar" value="0" min="0" max="0">
                    <span id="totalDuration">0:00</span>
                </div>
            </div>
            <!-- New section for post-generation actions -->
            <div id="postGenerationActions" class="hidden">
                <button class="action-button" onclick="generateSongAnalysis()">✨ Get Song Analysis</button>
                <button class="action-button" onclick="generateMarketingBlurb()">✨ Generate Marketing Blurb</button>
            </div>
            <div id="audioLoading" class="loading-indicator hidden">Generating audio...</div>
        </div>
    </div>

    <div id="comingSoonPage" class="page">
      <div class="container">
        <h2>✨ More Features</h2>
        <p class="coming-soon-text">Advanced tools and collaborations are coming soon. Stay tuned!</p>
      </div>
    </div>
  </main>
  
  <div id="songDetailView" class="hidden" onclick="closeDetailView()">
      <div id="songDetailContent" onclick="event.stopPropagation()">
        <button class="close-button" onclick="closeDetailView()">✖</button>
      </div>
  </div>

  <nav class="bottom-nav">
    <button id="navLibraryPage" class="nav-button active" onclick="showPage('libraryPage')">🏛️<br>Library</button>
    <button id="navGeneratorPage" class="nav-button" onclick="showPage('generatorPage')">🎤<br>Generate</button>
    <button id="navComingSoonPage" class="nav-button" onclick="showPage('comingSoonPage')">🚀<br>Coming Soon</button>
  </nav>

  <script>
    // --- CORE APP STATE & UI MANAGEMENT ---
    let songLibrary = [];
    let songType = "normal"; // Default song type
    let currentCoverUrl = "https://placehold.co/300x300/1e1e1e/f5f5f5?text=No+Cover"; // Default placeholder
    let isFirstGeneration = true; // Flag to change "Generate" to "Regenerate"

    // Tone.js specific variables
    let drumSynth = null;
    let bassSynth = null;
    let melodySynth = null;
    let currentToneParts = []; // To store and stop Tone.js parts
    let currentSongDataForPlayback = null; // Store the last generated song data for playback
    let audioUpdateLoop = null; // For the progress bar update loop
    let songTotalDuration = 0; // To store the calculated total duration of the current song
    let speechSynthesizer = window.speechSynthesis; // Browser's speech synthesis API

    document.addEventListener('DOMContentLoaded', () => {
        // Explicitly hide the song detail view just in case
        document.getElementById('songDetailView').style.display = 'none'; // Use style.display for direct control

        showPage('libraryPage'); // Start on Library page
        document.getElementById('genre').addEventListener('change', updateMoodsForGenre);
        renderLibrary();

        // Add event listener for progress bar
        document.getElementById('progressBar').addEventListener('input', handleProgressBarSeek);
    });
    
    function showPage(pageId) {
        document.querySelectorAll('.page').forEach(page => page.classList.remove('active'));
        document.getElementById(pageId).classList.add('active');
        
        document.querySelectorAll('.nav-button').forEach(btn => btn.classList.remove('active'));
        // FIX: Corrected the ID selection for nav buttons to exactly match their HTML IDs
        // The pageId is already in the correct format (e.g., 'libraryPage'), so just prepend 'nav'
        // and ensure the first letter of the the pageId part is capitalized if it's not already.
        // Example: 'libraryPage' -> 'navLibraryPage'
        const activeBtnId = 'nav' + pageId.charAt(0).toUpperCase() + pageId.slice(1);
        const activeBtnElement = document.getElementById(activeBtnId);
        if (activeBtnElement) { // Defensive check
            activeBtnElement.classList.add('active');
        } else {
            console.error(`Navigation button with ID "${activeBtnId}" not found.`);
        }
        stopAudioFull(); // Stop audio when changing pages
    }
    
    function toggleSongType() {
        songType = songType === "normal" ? "instrumental" : "normal";
        document.getElementById("typeLabel").textContent = songType.charAt(0).toUpperCase() + songType.slice(1);
        // Hide/show lyrics and voice blocks based on song type
        document.getElementById("lyricsBlock").classList.toggle("hidden", songType === "instrumental");
        document.getElementById("voiceBlock").classList.toggle("hidden", songType === "instrumental");
        document.getElementById("voiceDescriptionBlock").classList.toggle("hidden", songType === "instrumental");
        document.getElementById("instrumentsBlock").classList.toggle("hidden", songType === "normal"); // Show instruments for instrumental

        // Clear fields based on song type
        if (songType === "instrumental") {
            document.getElementById('lyrics').value = "";
            document.getElementById('voice').value = "male"; 
            document.getElementById('voiceDescription').value = ""; 
        } else {
            document.getElementById('instruments').value = ""; 
        }
        stopAudioFull(); 
    }
    
    // This is kept for reference but the primary genre/mood selection will come from LLM
    const AILogic = {
        Pop: { moods: ["Happy", "Romantic", "Sad"], scale: ["C4", "D4", "E4", "F4", "G4", "A4", "B4"], chords: [["C4", "E4", "G4"], ["F4", "A4", "C5"], ["G4", "B4", "D5"]] },
        Rock: { moods: ["Energetic", "Rebellious", "Passionate"], scale: ["E3", "G3", "A3", "B3", "D4", "E4"], chords: [["E3", "G3", "B3"], ["A3", "C4", "E4"], ["D3", "F#3", "A3"]] },
        "Hip-Hop": { moods: ["Confident", "Energetic", "Reflective"], scale: ["C3", "Eb3", "F3", "G3", "Bb3"], chords: [["C3", "Eb3", "G3"], ["F3", "Ab3", "C4"], ["G3", "Bb3", "D4"]] },
        Electronic: { moods: ["Futuristic", "Hypnotic", "Energetic"], scale: ["A3", "B3", "C#4", "E4", "F#4"], chords: [["A3", "C#4", "E4"], ["D4", "F#4", "A4"], ["E4", "G#4", "B4"]] },
        "R&B": { moods: ["Smooth", "Soulful", "Romantic"], scale: ["D3", "F3", "G3", "A3", "C4"], chords: [["D3", "F3", "A3"], ["G3", "Bb3", "D4"], ["C3", "E3", "G3"]] },
        Country: { moods: ["Happy", "Sad", "Relaxed"], scale: ["G3", "A3", "B3", "C4", "D4", "E4"], chords: [["G3", "B3", "D4"], ["C4", "E4", "G4"], ["D4", "F#4", "A4"]] }
    };
    
    function updateMoodsForGenre() {
        const genre = document.getElementById('genre').value;
        const moodSelect = document.getElementById('mood');
        moodSelect.innerHTML = ''; // Clear existing options
        if (genre && AILogic[genre]) {
            const moods = AILogic[genre].moods;
            moods.forEach(mood => {
                const option = document.createElement('option');
                option.value = mood;
                option.textContent = mood;
                moodSelect.appendChild(option);
            });
        } else {
             const option = document.createElement('option');
             option.value = "";
             option.textContent = "-- Select Genre First --";
             moodSelect.appendChild(option);
        }
    }
    
    // --- Gemini API Integration Functions ---

    // Function to call the Gemini API for text generation
    async function callGeminiAPI(prompt, responseSchema = null) {
        let chatHistory = [];
        chatHistory.push({ role: "user", parts: [{ text: prompt }] });
        const payload = { contents: chatHistory };

        if (responseSchema) {
            payload.generationConfig = {
                responseMimeType: "application/json",
                responseSchema: responseSchema
            };
        }

        const apiKey = ""; // Canvas will provide this at runtime
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

        try {
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            if (result.candidates && result.candidates.length > 0 &&
                result.candidates[0].content && result.candidates[0].content.parts &&
                result.candidates[0].content.parts.length > 0) {
                const text = result.candidates[0].content.parts[0].text;
                try {
                    return responseSchema ? JSON.parse(text) : text;
                } catch (e) {
                    console.error("Failed to parse JSON from Gemini API:", text, e);
                    return "Error: Invalid JSON response from AI.";
                }
            } else {
                console.error("Gemini API response structure unexpected:", result);
                return "Error: Could not generate content. Please try again.";
            }
        } catch (error) {
            console.error("Error calling Gemini API:", error);
            return "Error: Failed to connect to AI. Please check your network.";
        }
    }

    // Function to call Imagen API for image generation
    async function callImagenAPI(prompt) {
        const payload = { instances: { prompt: prompt }, parameters: { "sampleCount": 1} };
        const apiKey = ""; // Canvas will provide this at runtime
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/imagen-3.0-generate-002:predict?key=${apiKey}`;
        
        try {
            const response = await fetch(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            if (result.predictions && result.predictions.length > 0 && result.predictions[0].bytesBase64Encoded) {
                return `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`;
            } else {
                console.error("Imagen API response structure unexpected:", result);
                return "https://placehold.co/300x300/FF0000/FFFFFF?text=Error"; // Error placeholder
            }
        } catch (error) {
            console.error("Error calling Imagen API:", error);
            return "https://placehold.co/300x300/FF0000/FFFFFF?text=Network+Error"; // Network error placeholder
        }
    }

    // Unified AI Auto-Fill All Function
    async function aiAutoFillAll() {
        const titleInput = document.getElementById('songTitle');
        const genreSelect = document.getElementById('genre');
        const moodSelect = document.getElementById('mood');
        const lyricsTextArea = document.getElementById('lyrics');
        const instrumentsTextArea = document.getElementById('instruments'); 
        const outputDiv = document.getElementById('output');
        const aiAutoFillBtn = document.getElementById('aiAutoFillAllBtn');

        // Disable inputs and buttons during generation
        titleInput.disabled = true;
        genreSelect.disabled = true;
        moodSelect.disabled = true;
        lyricsTextArea.disabled = true;
        instrumentsTextArea.disabled = true;
        aiAutoFillBtn.disabled = true;
        document.getElementById('generateCoverBtn').disabled = true;
        document.getElementById('generateBtn').disabled = true;
        document.getElementById('postGenerationActions').classList.add('hidden'); // Hide post-gen actions

        outputDiv.textContent = "✨ AI is thinking and generating all song details...";
        stopAudioFull(); 

        const schema = {
            type: "OBJECT",
            properties: {
                title: { type: "STRING" },
                genre: { type: "STRING", enum: ["Pop", "Rock", "Hip-Hop", "Electronic", "R&B", "Country"] },
                mood: { type: "STRING" },
                lyrics: { type: "STRING" },
                instruments: { type: "STRING" } 
            },
            required: ["title", "genre", "mood", "lyrics", "instruments"]
        };

        // Always ask for a new title, genre, mood, lyrics, and instruments
        const prompt = `Generate a creative, catchy, and original song title, a suitable music genre from the list (Pop, Rock, Hip-Hop, Electronic, R&B, Country), a mood, full song lyrics for it (structured with verses, pre-chorus, chorus, bridge, 150-200 words), and a list of 3-5 suitable instruments for the song, comma-separated (e.g., "Piano, Drums, Bass, Guitar"). Provide the response as a JSON object.`;
        
        const result = await callGeminiAPI(prompt, schema);

        // Re-enable inputs and buttons
        titleInput.disabled = false;
        genreSelect.disabled = false;
        moodSelect.disabled = false;
        lyricsTextArea.disabled = false;
        instrumentsTextArea.disabled = false;
        aiAutoFillBtn.disabled = false;
        document.getElementById('generateCoverBtn').disabled = false;
        document.getElementById('generateBtn').disabled = false; 

        if (typeof result === 'object' && result.title && result.genre && result.mood && result.lyrics && result.instruments) {
            titleInput.value = result.title.trim().replace(/^"|"$/g, ''); 
            genreSelect.value = result.genre;
            updateMoodsForGenre(); 
            moodSelect.value = result.mood;
            lyricsTextArea.value = result.lyrics;
            instrumentsTextArea.value = result.instruments; 
            outputDiv.textContent = "All song details auto-filled!";
            
            // Now generate the cover art after text content is filled
            generateCover(); 
        } else {
            outputDiv.textContent = "Error: Could not auto-fill all details. " + result; 
            // Clear fields on error
            titleInput.value = ""; 
            genreSelect.value = "";
            moodSelect.innerHTML = '<option value="">-- Select Genre First --</option>';
            lyricsTextArea.value = "";
            instrumentsTextArea.value = "";
        }
    }

    // Handle manual title input (this will trigger AI to fill genre, mood, lyrics)
    async function handleTitleInput() {
        const title = document.getElementById('songTitle').value.trim();
        if (title) {
            document.getElementById('output').textContent = "Title entered. Click 'AI Auto-Fill All' to generate genre, mood, lyrics, and instruments.";
        } else {
            // Clear related fields if title is empty
            document.getElementById('genre').value = "";
            document.getElementById('mood').innerHTML = '<option value="">-- Select Genre First --</option>';
            document.getElementById('lyrics').value = "";
            document.getElementById('instruments').value = ""; 
            document.getElementById('coverArt').classList.add('hidden');
            document.getElementById('coverArt').src = "https://placehold.co/300x300/1e1e1e/f5f5f5?text=No+Cover"; 
            document.getElementById('output').textContent = "Enter song title or auto-generate...";
        }
        stopAudioFull(); 
        document.getElementById('postGenerationActions').classList.add('hidden'); // Hide post-gen actions
    }

    // Generate Cover using Imagen API
    async function generateCover() {
        const title = document.getElementById('songTitle').value;
        const genre = document.getElementById('genre').value;
        const mood = document.getElementById('mood').value;
        const coverArtImg = document.getElementById('coverArt');
        const coverLoading = document.getElementById('coverLoading');
        const outputDiv = document.getElementById('output');

        if (!title) {
            outputDiv.textContent = "Please enter or auto-generate a title first to create a cover!";
            return;
        }

        coverArtImg.classList.add('hidden');
        coverLoading.classList.remove('hidden');
        outputDiv.textContent = "🎨 Generating song cover...";
        document.getElementById('generateCoverBtn').disabled = true; 
        stopAudioFull(); 
        document.getElementById('postGenerationActions').classList.add('hidden'); // Hide post-gen actions


        // Updated prompt for no text and unique, abstract art
        const prompt = `Abstract, unique, original, non-literal album cover art for a song titled '${title}' in the ${genre || 'general'} genre with a ${mood || 'neutral'} mood. No text, no words, no typography on the cover. Focus on symbolic representation, vibrant colors, and a modern, digital art style.`;
        
        const imageUrl = await callImagenAPI(prompt);
        
        coverArtImg.src = imageUrl;
        coverArtImg.classList.remove('hidden');
        coverLoading.classList.add('hidden');
        document.getElementById('generateCoverBtn').disabled = false; 

        if (imageUrl.includes("Error")) {
            outputDiv.textContent = "Error generating cover. Please try again.";
        } else {
            currentCoverUrl = imageUrl; 
            outputDiv.textContent = "Song cover generated!";
        }
    }
    
    // --- TONE.JS AUDIO GENERATION AND CONTROL ---

    // Utility function to format time
    function formatTime(seconds) {
        const minutes = Math.floor(seconds / 60);
        const remainingSeconds = Math.floor(seconds % 60);
        return `${minutes}:${remainingSeconds < 10 ? '0' : ''}${remainingSeconds}`;
    }

    // Function to update the progress bar and timestamps
    function updateProgressBar() {
        const progressBar = document.getElementById('progressBar');
        const currentTimeSpan = document.getElementById('currentTime');
        const totalDurationSpan = document.getElementById('totalDuration');

        const currentTime = Tone.Transport.seconds;
        
        progressBar.value = currentTime;
        currentTimeSpan.textContent = formatTime(currentTime);
        totalDurationSpan.textContent = formatTime(songTotalDuration);

        // Update play/pause button text
        const playPauseBtn = document.getElementById('playPauseBtn');
        if (Tone.Transport.state === "started") {
            playPauseBtn.textContent = "⏸ Pause";
        } else if (Tone.Transport.state === "paused") {
            playPauseBtn.textContent = "▶ Play";
        } else {
            playPauseBtn.textContent = "▶ Play";
            // If transport stopped by itself (song ended), reset progress bar
            if (currentTime >= songTotalDuration && songTotalDuration > 0) {
                progressBar.value = 0;
                currentTimeSpan.textContent = formatTime(0);
            }
        }
    }

    // Handle seeking via progress bar
    function handleProgressBarSeek() {
        const progressBar = document.getElementById('progressBar');
        Tone.Transport.seconds = parseFloat(progressBar.value);
        updateProgressBar(); // Update immediately after seek
        // For speech synthesis, we need to restart from the new position
        if (currentSongDataForPlayback && currentSongDataForPlayback.type === 'normal') {
            restartSpeechFromTime(Tone.Transport.seconds, currentSongDataForPlayback.lyrics, currentSongDataForPlayback.voice, currentSongDataForPlayback.voiceDescription);
        }
    }

    // Function to toggle play/pause
    async function togglePlayPause() {
        if (Tone.context.state !== 'running') {
            await Tone.start();
        }

        if (Tone.Transport.state === "started") {
            Tone.Transport.pause();
            document.getElementById('playPauseBtn').textContent = "▶ Play";
            if (speechSynthesizer.speaking) {
                speechSynthesizer.pause();
            }
        } else {
            Tone.Transport.start();
            document.getElementById('playPauseBtn').textContent = "⏸ Pause";
            if (speechSynthesizer.paused) {
                speechSynthesizer.resume();
            }
        }
    }

    // Function to skip backward
    async function skipBackward() {
        if (Tone.context.state !== 'running') {
            await Tone.start();
        }
        Tone.Transport.seconds = Math.max(0, Tone.Transport.seconds - 10);
        updateProgressBar();
        // For speech synthesis, we need to restart from the new position
        if (currentSongDataForPlayback && currentSongDataForPlayback.type === 'normal') {
            const currentSeconds = Tone.Transport.seconds;
            restartSpeechFromTime(currentSeconds, currentSongDataForPlayback.lyrics, currentSongDataForPlayback.voice, currentSongDataForPlayback.voiceDescription);
        }
    }

    // Function to skip forward
    async function skipForward() {
        if (Tone.context.state !== 'running') {
            await Tone.start();
        }
        Tone.Transport.seconds = Math.min(songTotalDuration, Tone.Transport.seconds + 10);
        updateProgressBar();
        // For speech synthesis, we need to restart from the new position
        if (currentSongDataForPlayback && currentSongDataForPlayback.type === 'normal') {
            const currentSeconds = Tone.Transport.seconds;
            restartSpeechFromTime(currentSeconds, currentSongDataForPlayback.lyrics, currentSongDataForPlayback.voice, currentSongDataForPlayback.voiceDescription);
        }
    }

    // Function to stop all Tone.js audio and dispose resources
    function stopAudioFull() {
        if (Tone.Transport.state === "started" || Tone.Transport.state === "paused") {
            Tone.Transport.stop();
        }
        Tone.Transport.cancel(); // Clear all scheduled events
        currentToneParts.forEach(part => part.dispose()); // Dispose of parts
        currentToneParts = []; // Clear the array

        // Dispose synths
        if (drumSynth) drumSynth.dispose();
        if (bassSynth) bassSynth.dispose();
        if (melodySynth) melodySynth.dispose();
        // Dispose dynamically created synths
        for (const key in dynamicSynths) {
            if (dynamicSynths[key]) dynamicSynths[key].dispose();
        }
        dynamicSynths = {}; // Reset dynamic synths

        // Stop browser speech synthesis
        if (speechSynthesizer.speaking) {
            speechSynthesizer.cancel();
        }

        // Clear and reset progress bar
        const progressBar = document.getElementById('progressBar');
        progressBar.value = 0;
        progressBar.max = 0; 
        document.getElementById('currentTime').textContent = "0:00";
        document.getElementById('totalDuration').textContent = "0:00";

        // Stop the update loop
        if (audioUpdateLoop) {
            audioUpdateLoop.dispose();
            audioUpdateLoop = null;
        }

        console.log("Audio stopped and resources disposed.");
        document.getElementById('audioControls').classList.add('hidden'); 
        document.getElementById('audioLoading').classList.add('hidden'); 
        document.getElementById('playPauseBtn').textContent = "▶ Play"; 
    }

    let dynamicSynths = {}; // Store dynamically created synths

    async function generateAudio(lyrics, genre, mood, voice, voiceDescription, instruments) {
        stopAudioFull(); 
        document.getElementById('audioLoading').classList.remove('hidden'); 

        try {
            await Tone.start();
            console.log("AudioContext started.");
        } catch (e) {
            console.error("Failed to start AudioContext:", e);
            document.getElementById('output').textContent = "Error: Could not start audio. Please ensure browser allows audio playback.";
            document.getElementById('audioLoading').classList.add('hidden');
            return;
        }

        // --- Initialize Synths based on 'instruments' textbox ---
        const parsedInstruments = instruments.toLowerCase().split(',').map(s => s.trim());
        dynamicSynths = {}; // Reset for new generation

        // Default synths if specific instruments aren't found or listed
        drumSynth = new Tone.MembraneSynth().toDestination();
        bassSynth = new Tone.MonoSynth({
            oscillator: { type: "square" }, envelope: { attack: 0.005, decay: 0.4, sustain: 0.01, release: 0.2 }
        }).toDestination();
        melodySynth = new Tone.PolySynth(Tone.Synth, {
            oscillator: { type: "triangle" }, envelope: { attack: 0.02, decay: 0.1, sustain: 0.3, release: 1 }
        }).toDestination();

        parsedInstruments.forEach(inst => {
            if (inst.includes("drum") && !dynamicSynths.drums) {
                dynamicSynths.drums = new Tone.MembraneSynth({
                    "octaves": 1, "pitchDecay": 0.05, "envelope": { "attack": 0.01, "decay": 0.4, "sustain": 0.01 }
                }).toDestination();
                drumSynth = dynamicSynths.drums; 
            } else if (inst.includes("bass") && !dynamicSynths.bass) {
                dynamicSynths.bass = new Tone.MonoSynth({
                    oscillator: { type: "square" }, envelope: { attack: 0.005, decay: 0.4, sustain: 0.01, release: 0.2 }
                }).toDestination();
                bassSynth = dynamicSynths.bass; 
            } else if ((inst.includes("piano") || inst.includes("keyboard")) && !dynamicSynths.piano) {
                dynamicSynths.piano = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "triangle" },
                    envelope: { attack: 0.005, decay: 0.5, sustain: 0.1, release: 0.8 }
                }).toDestination();
                melodySynth = dynamicSynths.piano; 
            } else if (inst.includes("guitar") && !dynamicSynths.guitar) {
                dynamicSynths.guitar = new Tone.PluckSynth().toDestination();
            } else if ((inst.includes("strings") || inst.includes("violin") || inst.includes("cello")) && !dynamicSynths.strings) {
                dynamicSynths.strings = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "sawtooth" },
                    envelope: { attack: 0.5, decay: 1, sustain: 0.8, release: 2 }
                }).toDestination();
            } else if ((inst.includes("flute") || inst.includes("wind")) && !dynamicSynths.flute) {
                 dynamicSynths.flute = new Tone.PolySynth(Tone.Synth, {
                    oscillator: { type: "sine" },
                    envelope: { attack: 0.1, decay: 0.4, sustain: 0.7, release: 1.2 }
                }).toDestination();
            }
        });
        
        // --- Music Logic based on Genre/Mood ---
        const genreData = AILogic[genre];
        if (!genreData) {
            console.error("Genre data not found for:", genre);
            document.getElementById('output').textContent = "Error: Invalid genre selected for audio generation.";
            document.getElementById('audioLoading').classList.add('hidden');
            return;
        }
        const scale = genreData.scale;
        const chords = genreData.chords;

        const shuffledChords = [...chords].sort(() => 0.5 - Math.random());
        const chordProgression = [
            shuffledChords[0], shuffledChords[1], shuffledChords[2], shuffledChords[0],
            shuffledChords[1], shuffledChords[2], shuffledChords[0], shuffledChords[1] 
        ]; 
        const bpm = 110 + Math.floor(Math.random() * 20); 
        Tone.Transport.bpm.value = bpm;

        // --- Instrumental Parts ---
        // Drums (more varied beat)
        const drumPattern = [];
        for (let i = 0; i < 8; i++) { 
            drumPattern.push([`${i}:0:0`, "C2", "8n"]); // Kick on 1
            if (Math.random() > 0.4) drumPattern.push([`${i}:0:2`, "C2", "8n"]); // Occasional kick on 1.5
            drumPattern.push([`${i}:1:0`, "C#2", "8n"]); // Snare on 2
            if (Math.random() > 0.6) drumPattern.push([`${i}:1:2`, "C4", "16n"]); // Hi-hat on 2.5
            drumPattern.push([`${i}:2:0`, "C2", "8n"]); // Kick on 3
            if (Math.random() > 0.4) drumPattern.push([`${i}:2:2`, "C2", "8n"]); // Occasional kick on 3.5
            drumPattern.push([`${i}:3:0`, "C#2", "8n"]); // Snare on 4
            if (Math.random() > 0.6) drumPattern.push([`${i}:3:2`, "C4", "16n"]); // Hi-hat on 4.5
        }
        const drumPart = new Tone.Part((time, note, duration) => {
            drumSynth.triggerAttackRelease(note, duration, time);
        }, drumPattern).start(0);
        currentToneParts.push(drumPart);

        // Bass Line (follows chord roots with more variation)
        const bassEvents = [];
        for (let i = 0; i < chordProgression.length; i++) {
            const rootNote = chordProgression[i][0];
            bassEvents.push({ time: `${i}:0:0`, note: rootNote, duration: "1n" });
            if (Math.random() > 0.6) { 
                bassEvents.push({ time: `${i}:0:2`, note: Tone.Frequency(rootNote).transpose(Math.random() > 0.5 ? 7 : -5).toNote(), duration: "8n" });
            }
        }
        const bassPart = new Tone.Part((time, value) => {
            bassSynth.triggerAttackRelease(value.note, value.duration, time);
        }, bassEvents).start(0);
        currentToneParts.push(bassPart);

        // Melody/Harmony (more dynamic arpeggios or countermelodies)
        const melodyEvents = [];
        for (let i = 0; i < chordProgression.length; i++) {
            const chord = chordProgression[i];
            const startTime = `${i}:0:0`;
            
            for (let j = 0; j < chord.length; j++) {
                melodyEvents.push({ time: `${startTime} + ${Tone.Time("8n").toSeconds() * j}`, note: chord[j], duration: "8n" });
            }
            if (Math.random() > 0.5) {
                melodyEvents.push({ time: `${startTime} + ${Tone.Time("2n").toSeconds()}`, note: chord[Math.floor(Math.random() * chord.length)], duration: "2n" });
            }
        }
        const melodyPart = new Tone.Part((time, value) => {
            melodySynth.triggerAttackRelease(value.note, value.duration, time);
        }, melodyEvents).start(0);
        currentToneParts.push(melodyPart);

        // Add dynamically created instrument parts
        for (const instName in dynamicSynths) {
            if (instName === "drums" || instName === "bass" || instName === "piano") continue; // Already handled by main synths
            const synth = dynamicSynths[instName];
            const instEvents = [];
            for (let i = 0; i < chordProgression.length; i++) {
                const chord = chordProgression[i];
                const startTime = `${i}:0:0`;
                
                if (instName === "strings") {
                    instEvents.push({ time: startTime, note: chord, duration: "1n" });
                } else if (instName === "guitar" || instName === "flute") {
                    for (let j = 0; j < chord.length; j++) {
                        instEvents.push({ time: `${startTime} + ${Tone.Time("4n").toSeconds() * j}`, note: chord[j], duration: "4n" });
                    }
                }
            }
            const instPart = new Tone.Part((time, value) => {
                synth.triggerAttackRelease(value.note, value.duration, time);
            }, instEvents).start(0);
            currentToneParts.push(instPart);
        }

        // --- Vocal Part (Browser Speech Synthesis) ---
        let estimatedVocalDuration = 0;
        if (songType === 'normal' && lyrics) {
            const utterance = new SpeechSynthesisUtterance(lyrics);
            
            // Set voice properties
            const voices = speechSynthesizer.getVoices();
            let selectedVoice = voices.find(v => v.lang.startsWith('en') && (voice === 'male' ? v.name.includes('Male') : v.name.includes('Female')));
            if (!selectedVoice) { 
                selectedVoice = voices.find(v => v.lang.startsWith('en'));
            }
            utterance.voice = selectedVoice;
            utterance.rate = 1.0; 
            utterance.pitch = 1.0; 
            utterance.volume = 0.8; // Slightly lower volume to blend with music

            // Adjust pitch/rate based on voice description
            if (voiceDescription.toLowerCase().includes("raspy")) {
                utterance.pitch = 0.8; 
                utterance.rate = 0.9;
            } else if (voiceDescription.toLowerCase().includes("high-pitched")) {
                utterance.pitch = 1.2; 
            } else if (voiceDescription.toLowerCase().includes("low-pitched")) {
                utterance.pitch = 0.7;
            } else if (voiceDescription.toLowerCase().includes("fast")) {
                utterance.rate = 1.3;
            } else if (voiceDescription.toLowerCase().includes("slow")) {
                utterance.rate = 0.7;
            }

            // Estimate duration of speech for synchronization. This is a rough estimate.
            // A common estimate is 150-180 words per minute for normal speech.
            const wordsPerSecond = 150 / 60; 
            estimatedVocalDuration = lyrics.split(/\s+/).filter(word => word.length > 0).length / wordsPerSecond;

            // Schedule speech to start slightly after music, to allow Tone.js to get going
            Tone.Transport.scheduleOnce((time) => {
                speechSynthesizer.speak(utterance);
            }, "0.5s"); // Start speech 0.5 seconds into the track
        }

        // Determine total song duration (e.g., 8 measures for the progression, or based on lyrics length)
        songTotalDuration = Math.max(Tone.Time("8m").toSeconds(), estimatedVocalDuration + 2); 

        // Set up progress bar max value
        const progressBar = document.getElementById('progressBar');
        progressBar.max = songTotalDuration;
        document.getElementById('totalDuration').textContent = formatTime(songTotalDuration);

        // Schedule stopping the transport after the song duration
        Tone.Transport.scheduleOnce(() => {
            stopAudioFull();
            console.log("Song finished playing.");
        }, songTotalDuration);

        // Start the progress bar update loop
        if (audioUpdateLoop) audioUpdateLoop.dispose(); 
        audioUpdateLoop = new Tone.Loop(updateProgressBar, "0.1s").start(0); 

        // --- Start Transport ---
        Tone.Transport.start();
        document.getElementById('audioLoading').classList.add('hidden'); 
        document.getElementById('audioControls').classList.remove('hidden'); 
        document.getElementById('playPauseBtn').textContent = "⏸ Pause"; 
        console.log("Tone.js Transport started.");
    }

    // Function to restart speech from a given time, used for seeking
    function restartSpeechFromTime(seekTime, lyrics, voice, voiceDescription) {
        speechSynthesizer.cancel(); // Stop current speech
        if (!lyrics || lyrics === "Instrumental track") return;

        const words = lyrics.split(/\s+/).filter(word => word.length > 0);
        let currentEstimatedSpeechTime = 0;
        const wordsPerSecond = 150 / 60; 

        for (let i = 0; i < words.length; i++) {
            const word = words[i];
            const estimatedWordDuration = word.length / wordsPerSecond;

            if (currentEstimatedSpeechTime >= seekTime) {
                const utterance = new SpeechSynthesisUtterance(lyrics.substring(lyrics.indexOf(word))); // Speak from this word onwards
                const voices = speechSynthesizer.getVoices();
                let selectedVoice = voices.find(v => v.lang.startsWith('en') && (voice === 'male' ? v.name.includes('Male') : v.name.includes('Female')));
                if (!selectedVoice) {
                    selectedVoice = voices.find(v => v.lang.startsWith('en'));
                }
                utterance.voice = selectedVoice;
                utterance.rate = 1.0; 
                utterance.pitch = 1.0; 
                utterance.volume = 0.8;

                if (voiceDescription.toLowerCase().includes("raspy")) { utterance.pitch = 0.8; utterance.rate = 0.9; } 
                else if (voiceDescription.toLowerCase().includes("high-pitched")) { utterance.pitch = 1.2; } 
                else if (voiceDescription.toLowerCase().includes("low-pitched")) { utterance.pitch = 0.7; } 
                else if (voiceDescription.toLowerCase().includes("fast")) { utterance.rate = 1.3; } 
                else if (voiceDescription.toLowerCase().includes("slow")) { utterance.rate = 0.7; }

                speechSynthesizer.speak(utterance);
                break; 
            }
            currentEstimatedSpeechTime += estimatedWordDuration + 0.1; // Add average pause
        }
    }


    function playCurrentAudio() {
        if (currentSongDataForPlayback) {
            // If audio is already playing, stop it first to restart
            if (Tone.Transport.state === "started" || Tone.Transport.state === "paused") {
                stopAudioFull(); 
            }
            generateAudio(
                currentSongDataForPlayback.lyrics,
                currentSongDataForPlayback.genre,
                currentSongDataForPlayback.mood,
                currentSongDataForPlayback.voice,
                currentSongDataForPlayback.voiceDescription,
                currentSongDataForPlayback.instruments
            );
        } else {
            document.getElementById('output').textContent = "No song generated yet to play. Click 'Generate Song' first!";
        }
    }


    function generateSong() {
        const title = document.getElementById("songTitle").value.trim();
        const genre = document.getElementById("genre").value;
        const mood = document.getElementById("mood").value;
        const lyrics = document.getElementById("lyrics").value.trim();
        const instruments = document.getElementById("instruments").value.trim(); 
        const voice = document.getElementById("voice").value;
        const voiceDescription = document.getElementById("voiceDescription").value.trim();
        const outputDiv = document.getElementById('output');
        const generateBtn = document.getElementById('generateBtn');
        
        // Validation logic - now performed on button click
        if (!title) {
            outputDiv.textContent = "Please enter or auto-generate a song title.";
            return;
        }
        if (!genre || !mood) {
            outputDiv.textContent = "Please ensure Genre and Mood are set (generated from title).";
            return;
        }
        if (songType === 'normal' && !lyrics) {
            outputDiv.textContent = "Please generate lyrics first or switch to instrumental type.";
            return;
        }
        if (songType === 'normal' && !voice) {
            outputDiv.textContent = "Please select a voice for the song.";
            return;
        }
        if (songType === 'instrumental' && !instruments) {
            outputDiv.textContent = "Please generate instruments for the instrumental track.";
            return;
        }

        const songData = {
            title: title,
            genre: genre,
            mood: mood,
            lyrics: songType === 'normal' ? lyrics : "Instrumental track",
            instruments: instruments, 
            voice: songType === 'normal' ? voice : "N/A",
            voiceDescription: songType === 'normal' ? voiceDescription : "N/A", 
            type: songType,
            coverUrl: currentCoverUrl 
        };
        
        songLibrary.push(songData);
        renderLibrary();
        currentSongDataForPlayback = songData; 

        const outputText = `[${songData.title}]\n\n` +
            (songData.type === 'instrumental' ?
            `An instrumental track with a ${mood.toLowerCase()} vibe. Instruments: ${songData.instruments}` :
            `${songData.lyrics}\n\n[Voice: ${songData.voice} ${songData.voiceDescription ? '(' + songData.voiceDescription + ')' : ''}]`);
        
        outputDiv.textContent = outputText;
        
        // Call the Tone.js audio generation
        generateAudio(lyrics, genre, mood, voice, voiceDescription, instruments);

        outputDiv.textContent += "\n\n(Playing generated audio with Tone.js...)";

        // Change button text to "Regenerate Song" after first generation
        if (isFirstGeneration) {
            generateBtn.textContent = "🎧 Regenerate Song";
            isFirstGeneration = false;
        }
        document.getElementById('postGenerationActions').classList.remove('hidden'); // Show post-gen actions
    }

    // --- New Gemini API Features ---

    async function generateSongAnalysis() {
        const lyrics = document.getElementById('lyrics').value.trim();
        const outputDiv = document.getElementById('output');
        const generateBtn = document.getElementById('generateBtn');

        if (!lyrics || songType === 'instrumental') {
            outputDiv.textContent = "Please generate lyrics for a 'Normal' song type first to get an analysis.";
            return;
        }

        generateBtn.disabled = true; // Disable main generate button
        document.getElementById('aiAutoFillAllBtn').disabled = true;
        document.getElementById('generateCoverBtn').disabled = true;
        document.getElementById('postGenerationActions').querySelectorAll('button').forEach(btn => btn.disabled = true);

        outputDiv.textContent = "✨ AI is analyzing your song lyrics...";
        stopAudioFull();

        const prompt = `Provide a concise, insightful analysis of the following song lyrics, focusing on themes, mood, and potential interpretations. Keep it to 3-5 sentences.\n\nLyrics:\n"${lyrics}"`;
        const analysis = await callGeminiAPI(prompt);

        generateBtn.disabled = false; // Re-enable main generate button
        document.getElementById('aiAutoFillAllBtn').disabled = false;
        document.getElementById('generateCoverBtn').disabled = false;
        document.getElementById('postGenerationActions').querySelectorAll('button').forEach(btn => btn.disabled = false);

        outputDiv.textContent = `--- Song Analysis ---\n\n${analysis}\n\n--- Original Song Details ---\n\n${outputDiv.textContent}`;
    }

    async function generateMarketingBlurb() {
        const title = document.getElementById('songTitle').value.trim();
        const genre = document.getElementById('genre').value;
        const mood = document.getElementById('mood').value;
        const lyrics = document.getElementById('lyrics').value.trim();
        const instruments = document.getElementById('instruments').value.trim();
        const outputDiv = document.getElementById('output');
        const generateBtn = document.getElementById('generateBtn');

        if (!title || !genre || !mood) {
            outputDiv.textContent = "Please ensure Title, Genre, and Mood are set to generate a marketing blurb.";
            return;
        }

        generateBtn.disabled = true; // Disable main generate button
        document.getElementById('aiAutoFillAllBtn').disabled = true;
        document.getElementById('generateCoverBtn').disabled = true;
        document.getElementById('postGenerationActions').querySelectorAll('button').forEach(btn => btn.disabled = true);

        outputDiv.textContent = "✨ AI is crafting a marketing blurb...";
        stopAudioFull();

        const prompt = `Write a short, engaging social media marketing blurb (2-3 sentences, include hashtags) for a new song with the following details:\nTitle: "${title}"\nGenre: ${genre}\nMood: ${mood}\nLyrics snippet: "${lyrics.substring(0, 100)}..."\nInstruments: ${instruments}`;
        const blurb = await callGeminiAPI(prompt);

        generateBtn.disabled = false; // Re-enable main generate button
        document.getElementById('aiAutoFillAllBtn').disabled = false;
        document.getElementById('generateCoverBtn').disabled = false;
        document.getElementById('postGenerationActions').querySelectorAll('button').forEach(btn => btn.disabled = false);

        outputDiv.textContent = `--- Marketing Blurb ---\n\n${blurb}\n\n--- Original Song Details ---\n\n${outputDiv.textContent}`;
    }


    function renderLibrary() {
        const list = document.getElementById('libraryList');
        list.innerHTML = ''; 
        
        if (songLibrary.length === 0) {
             list.innerHTML = '<li class="library-item" style="opacity: 0.6;">Your saved songs will appear here.</li>';
             return;
        }
        
        songLibrary.forEach((song, index) => {
            const item = document.createElement('li');
            item.className = 'library-item';
            item.setAttribute('data-index', index);
            item.onclick = () => showDetailView(index);
            
            item.innerHTML = `<h3>${song.title}</h3><p>${song.type.charAt(0).toUpperCase() + song.type.slice(1)} - ${song.genre} / ${song.mood}</p>`;
            list.appendChild(item);
        });
    }
    
    function showDetailView(index) {
        const song = songLibrary[index];
        currentSongDataForPlayback = song; 
        const detailContent = document.getElementById('songDetailContent');
        
        detailContent.innerHTML = `
            <button class="close-button" onclick="closeDetailView()">✖</button>
            <h2>${song.title}</h2>
            <p><strong>Genre:</strong> ${song.genre} | <strong>Mood:</strong> ${song.mood} | <strong>Type:</strong> ${song.type.charAt(0).toUpperCase() + song.type.slice(1)}</p>
            <div id="detailCoverArtContainer" style="text-align:center; margin-bottom: 20px;">
                <img src="${song.coverUrl}" alt="Song Cover" style="max-width: 200px; height: auto; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.3);">
            </div>
            <div class="output">${song.lyrics || "Instrumental - No Lyrics"}</div>
            <br>
            <p><strong>Instruments:</strong> ${song.instruments || "N/A"}</p>
            <p><strong>Voice:</strong> ${song.voice} ${song.voiceDescription ? '(' + song.voiceDescription + ')' : ''}</p>
            <div id="detailAudioControls" style="display: flex; gap: 10px; margin-top: 20px; justify-content: center;">
                <button class="action-button" onclick="playCurrentAudio()">▶ Play Audio</button>
                <button class="action-button" onclick="stopAudioFull()">⏹ Stop Audio</button>
            </div>
        `;
        document.getElementById('songDetailView').style.display = 'flex'; 
    }
    
    function closeDetailView() {
        stopAudioFull(); 
        document.getElementById('songDetailView').style.display = 'none'; 
    }

  </script>
</body>
</html>


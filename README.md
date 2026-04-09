<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>[SIQUEROLI] - Red Team Profile</title>
    <style>
        :root {
            --bg-color: #0d0d0d;
            --main-green: #00ff41; /* Matrix Green */
            --dark-green: #003300;
            --text-color: #d1d1d1;
            --highlight: #ffffff;
            --terminal-prefix: #ff0055; /* Red for emphasis */
            --font-family: 'Courier New', Courier, monospace;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: var(--font-family);
            overflow: hidden; /* Prevent scroll while booting */
            height: 100vh;
        }

        /* Matrix Background */
        #matrixCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
            opacity: 0.15; /* Sub subtle effect */
        }

        /* Terminal Window */
        #terminal-container {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 90%;
            max-width: 900px;
            height: 80vh;
            background: rgba(13, 13, 13, 0.95);
            border: 1px solid var(--dark-green);
            box-shadow: 0 0 20px rgba(0, 255, 65, 0.2);
            overflow: hidden;
            display: flex;
            flex-direction: column;
            border-radius: 5px;
        }

        #terminal-header {
            background: #1a1a1a;
            padding: 5px 10px;
            border-bottom: 1px solid var(--dark-green);
            display: flex;
            justify-content: space-between;
            font-size: 0.8rem;
            color: #666;
        }

        #terminal-content {
            padding: 20px;
            overflow-y: auto;
            flex-grow: 1;
            scrollbar-width: thin;
            scrollbar-color: var(--dark-green) transparent;
        }

        /* Scrollbar styles */
        #terminal-content::-webkit-scrollbar { width: 6px; }
        #terminal-content::-webkit-scrollbar-thumb { background-color: var(--dark-green); }

        /* Text Styles */
        .cmd-line { margin-bottom: 10px; line-height: 1.4; display: flex; flex-wrap: wrap;}
        .prefix { color: var(--terminal-prefix); font-weight: bold; margin-right: 10px; }
        .user { color: var(--highlight); }
        .host { color: var(--main-green); }
        .path { color: #5555ff; }
        .command { color: var(--highlight); }
        .output { color: var(--main-green); white-space: pre-wrap; display: block; width: 100%; margin-top: 5px;}
        .highlight-text { color: var(--highlight); }

        /* Cursor Animation */
        .cursor {
            display: inline-block;
            width: 8px;
            height: 1.2rem;
            background-color: var(--main-green);
            animation: blink 1s infinite;
            vertical-align: middle;
            margin-left: 5px;
        }

        @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

        /* Typing Effect Class */
        .typing {
            overflow: hidden;
            white-space: nowrap;
            display: inline-block;
        }

        /* Buttons & Links */
        a { text-decoration: none; color: inherit; transition: 0.3s; }
        .btn-container { margin-top: 15px; display: flex; gap: 10px; flex-wrap: wrap; }
        .hacker-btn {
            padding: 8px 15px;
            border: 1px solid var(--main-green);
            background: transparent;
            color: var(--main-green);
            font-family: var(--font-family);
            cursor: pointer;
            text-transform: uppercase;
            font-size: 0.8rem;
            transition: 0.3s ease;
        }

        .hacker-btn:hover {
            background-color: var(--main-green);
            color: var(--bg-color);
            box-shadow: 0 0 10px var(--main-green);
        }

        /* Initial Boot Screen */
        #boot-screen {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: var(--bg-color);
            z-index: 100;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            color: var(--main-green);
        }

        .scanline {
            width: 100%; height: 2px;
            background: rgba(0, 255, 65, 0.1);
            position: absolute;
            animation: scanline 4s linear infinite;
        }

        @keyframes scanline { 0% { top: 0; } 100% { top: 100%; } }

    </style>
</head>
<body>

    <canvas id="matrixCanvas"></canvas>

    <div id="boot-screen">
        <div class="scanline"></div>
        <p class="typing" id="boot-text"></p>
    </div>

    <div id="terminal-container" style="display: none;">
        <div id="terminal-header">
            <span>bash | status: online | threat_level: minimal</span>
            <span>[SIQUEROLI v1.0]</span>
        </div>
        <div id="terminal-content" id="terminalLog">
            </div>
    </div>

    <script>
        // --- Configuration ---
        const USERNAME = "siqueroli";
        const HOST = "cyberhub";
        const TYPING_SPEED = 30; // ms per char

        // --- Data to Display ---
        const terminalOutput = [
            // Line 1: Identity
            { type: 'cmd', cmd: 'whoami', output: 'Siqueroli | Red Team Analyst | Cybersecurity Student' },
            
            // Line 2: Skills
            { type: 'cmd', cmd: 'cat skills.txt', output: 
`[Offensive]: Metasploit, Nmap, Burp Suite, SQLmap, BloodHound
[Defensive]: Wazuh, Shuffle, OpenCTI, DFIR-IRIS, Ollama (Local AI)
[Env]: Kali Linux, Docker, Proxmox, Active Directory
[Dev]: Python, Bash Scripting` },

            // Line 3: Major Project
            { type: 'cmd', cmd: './show_project.sh cyberhub', output: 
`Starting... [Opportunity CyberHub]
Initializing SOC/DFIR Integration... DONE.
Connecting Threat Intel (OpenCTI)... DONE.
Attaching Local AI (Ollama)... DONE.
Status: ACTIVE` },

            // Line 4: Education
            { type: 'cmd', cmd: 'grep "Education" profile.db', output: 
`> B.S. Computer Science (Estácio)
> Tech. Electronics` },

            // Line 5: Links
            { type: 'links', cmd: 'ls -l /connections', output: 'Access secure channels:' }
        ];

        // --- Core Script ---
        const contentDiv = document.getElementById('terminal-content');
        const bootScreen = document.getElementById('boot-screen');
        const terminalContainer = document.getElementById('terminal-container');

        // Function to simulate typing
        function typeText(element, text, speed) {
            return new Promise(resolve => {
                let i = 0;
                element.innerHTML = '';
                const timer = setInterval(() => {
                    if (i < text.length) {
                        element.innerHTML += text.charAt(i);
                        i++;
                    } else {
                        clearInterval(timer);
                        resolve();
                    }
                }, speed);
            });
        }

        // Add a new terminal command line
        function createCommandLine(command) {
            const line = document.createElement('div');
            line.className = 'cmd-line';
            line.innerHTML = `
                <span class="prefix">
                    [<span class="user">${USERNAME}</span>@<span class="host">${HOST}</span> 
                    <span class="path">~</span>]$
                </span>
                <span class="command">${command}</span>
            `;
            contentDiv.appendChild(line);
            contentDiv.scrollTop = contentDiv.scrollHeight;
            return line;
        }

        // Add output text
        function createOutputLine(text) {
            const output = document.createElement('span');
            output.className = 'output';
            output.innerText = text; // Preserves newlines
            contentDiv.appendChild(output);
            contentDiv.scrollTop = contentDiv.scrollHeight;
        }

        // --- Main Animation Sequence ---
        async function runTerminal() {
            // 1. Initial Boot Scene
            await typeText(document.getElementById('boot-text'), '>>> INITIALIZING SECURE CONNECTION... DONE.\n>>> BYPASSING FIREWALL... DONE.\n>>> ESTABLISHING SESSION...', 50);
            
            await new Promise(r => setTimeout(r, 1000));
            bootScreen.style.display = 'none';
            terminalContainer.style.display = 'flex';
            document.body.style.overflow = 'auto'; // Allow scroll inside terminal

            // 2. Clear terminal and print ASCII Art
            createOutputLine(
`
          _____                     _             _ _ 
         / ____|                   | |           | (_)
        | (___   ___  __ _ _   _ _ __ ___ | |  _   _| |_ 
         \\___ \\ / _ \\/ _\` | | | | '__/ _ \\| | | | | | | |
         ____) |  __/ (_| | |_| | | | (_) | | |_| | | | |
        |_____/ \\___|\\__, |\\__,_|_|  \\___/|_|\\__,_|_|_|_|
                        | |                              
                        |_|                              
[SYS]: Identity vulnerability check passed. Information objective set.
`);

            // 3. Cycle through commands
            for (const item of terminalOutput) {
                // Type the command line
                const cmdLine = createCommandLine('');
                const cmdSpan = cmdLine.querySelector('.command');
                await typeText(cmdSpan, item.cmd, TYPING_SPEED);
                
                // Slight delay before output
                await new Promise(r => setTimeout(r, 200));

                // If output exists, display it
                if (item.output) {
                    createOutputLine(item.output);
                }

                // If links line, add buttons
                if (item.type === 'links') {
                    const btnContainer = document.createElement('div');
                    btnContainer.className = 'btn-container';
                    btnContainer.innerHTML = `
                        <a href="https://www.linkedin.com/in/SEU_LINK_REAL" target="_blank"><button class="hacker-btn">> LinkedIn</button></a>
                        <a href="https://github.com/Siqueroli" target="_blank"><button class="hacker-btn">> GitHub Repos</button></a>
                        <a href="https://tryhackme.com/p/Siqueroli" target="_blank"><button class="hacker-btn">> TryHackMe</button></a>
                    `;
                    contentDiv.appendChild(btnContainer);
                }

                // Delay between command blocks
                await new Promise(r => setTimeout(r, 500));
            }

            // 4. Final Cursor
            createCommandLine('<span class="cursor"></span>');
            contentDiv.scrollTop = contentDiv.scrollHeight;
        }

        // --- Matrix Rain Effect ---
        const canvas = document.getElementById('matrixCanvas');
        const ctx = canvas.getContext('2d');

        canvas.height = window.innerHeight;
        canvas.width = window.innerWidth;

        // Characters: Katakana, Numbers, Latin
        const characters = 'アァカサタナハマヤャラワガザダバパイィキシチニヒミリヰギジヂビピウゥクスツヌフムユュルグズブヅプエェケセテネヘメレヱゲゼデベペオォコソトノホモヨョロヲゴゾドボポヴッン0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZsafetynetcyberhubroot';
        const charArray = characters.split('');

        const fontSize = 16;
        const columns = canvas.width / fontSize;
        const drops = [];

        for (let x = 0; x < columns; x++) {
            drops[x] = 1;
        }

        function drawMatrix() {
            ctx.fillStyle = 'rgba(13, 13, 13, 0.05)'; // Fades old characters
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = '#0f0'; // Character color
            ctx.font = fontSize + 'px arial';

            for (let i = 0; i < drops.length; i++) {
                const text = charArray[Math.floor(Math.random() * charArray.length)];
                ctx.fillText(text, i * fontSize, drops[i] * fontSize);

                if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) {
                    drops[i] = 0;
                }
                drops[i]++;
            }
        }

        window.addEventListener('resize', () => {
            canvas.height = window.innerHeight;
            canvas.width = window.innerWidth;
        });

        // Run both
        setInterval(drawMatrix, 35);
        runTerminal();

    </script>
</body>
</html>

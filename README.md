<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>SkillBridge — AI Placement & Skill Gap Auditor</title>
  
  <!-- PDF.js library to read PDFs in-browser -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
  
  <style>
    :root {
      --bg: #0f172a;
      --card: #1e293b;
      --border: #334155;
      --primary: #2563eb;
      --primary-hover: #1d4ed8;
      --text: #f8fafc;
      --muted: #94a3b8;
      --success: #22c55e;
      --danger: #ef4444;
      --warning: #f59e0b;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }

    body {
      background-color: var(--bg);
      color: var(--text);
      padding: 16px;
      line-height: 1.5;
    }

    .container {
      max-width: 900px;
      margin: 0 auto;
    }

    header {
      text-align: center;
      margin-bottom: 24px;
    }

    header h1 {
      font-size: 1.8rem;
      color: #60a5fa;
    }

    header p {
      color: var(--muted);
      font-size: 0.95rem;
      margin-top: 4px;
    }

    .card {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 18px;
      margin-bottom: 20px;
    }

    label {
      display: block;
      font-weight: 600;
      margin-bottom: 8px;
      font-size: 0.9rem;
    }

    input[type="text"],
    input[type="file"],
    select,
    textarea {
      width: 100%;
      background: #0f172a;
      border: 1px solid var(--border);
      color: var(--text);
      padding: 10px 12px;
      border-radius: 8px;
      font-size: 0.95rem;
      margin-bottom: 14px;
    }

    textarea {
      resize: vertical;
      min-height: 90px;
    }

    button.btn-primary {
      width: 100%;
      background-color: var(--primary);
      color: #ffffff;
      padding: 14px;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s;
    }

    button.btn-primary:hover {
      background-color: var(--primary-hover);
    }

    button.btn-primary:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }

    .results-section {
      display: none;
    }

    .score-banner {
      display: flex;
      align-items: center;
      justify-content: space-between;
      flex-wrap: wrap;
      gap: 12px;
      padding: 16px;
      border-radius: 10px;
      margin-bottom: 20px;
      border: 1px solid;
    }

    .score-banner.qualified {
      background: rgba(34, 197, 94, 0.1);
      border-color: var(--success);
      color: var(--success);
    }

    .score-banner.unqualified {
      background: rgba(239, 68, 68, 0.1);
      border-color: var(--danger);
      color: var(--danger);
    }

    .score-banner .score {
      font-size: 2rem;
      font-weight: bold;
    }

    .grid-2 {
      display: grid;
      grid-template-columns: 1fr;
      gap: 16px;
      margin-bottom: 20px;
    }

    @media (min-width: 640px) {
      .grid-2 {
        grid-template-columns: 1fr 1fr;
      }
    }

    .tag-container {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
      margin-top: 8px;
    }

    .tag {
      padding: 6px 12px;
      border-radius: 20px;
      font-size: 0.85rem;
      font-weight: 500;
    }

    .tag.matched {
      background: rgba(34, 197, 94, 0.2);
      color: #4ade80;
      border: 1px solid var(--success);
    }

    .tag.missing {
      background: rgba(239, 68, 68, 0.2);
      color: #f87171;
      border: 1px solid var(--danger);
    }

    .roadmap-item {
      background: #0f172a;
      border-left: 4px solid var(--primary);
      padding: 12px 14px;
      border-radius: 0 8px 8px 0;
      margin-bottom: 12px;
    }

    .roadmap-item h4 {
      color: #60a5fa;
      margin-bottom: 4px;
    }

    .company-item {
      background: #0f172a;
      padding: 12px;
      border-radius: 8px;
      margin-bottom: 10px;
      border: 1px solid var(--border);
    }

    .company-item strong {
      color: #38bdf8;
    }

    .spinner {
      display: none;
      text-align: center;
      margin-top: 14px;
      color: var(--muted);
      font-size: 0.95rem;
    }
  </style>
</head>
<body>

  <div class="container">
    <header>
      <h1>🎓 SkillBridge</h1>
      <p>AI Placement Matcher & Skill Gap Roadmap Engine</p>
    </header>

    <!-- Configuration Card -->
    <div class="card">
      <label for="apiKey">Gemini API Key:</label>
      <input type="text" id="apiKey" placeholder="Paste your Gemini API Key here (from Google AI Studio)" />

      <label for="resumeFile">1. Upload Candidate Resume (PDF):</label>
      <input type="file" id="resumeFile" accept="application/pdf" />

      <label for="targetRole">2. Select Target Company / Role:</label>
      <select id="targetRole" onchange="handleRoleChange()">
        <option value="Zoho - Software Developer (Java, Data Structures, OOPs, Problem Solving)">Zoho — Software Developer (Java, DSA, OOPs)</option>
        <option value="TCS / Infosys - Systems Engineer (Python, SQL, Aptitude, Core Fundamentals)">TCS / Infosys — Systems Engineer (Python, SQL)</option>
        <option value="Freshworks / Swiggy - Frontend Engineer (React, JavaScript, Web APIs, CSS)">Freshworks / Swiggy — Frontend (React, JS)</option>
        <option value="Data Analytics Role (Python Pandas, SQL, Tableau/Power BI, Basic Stats)">Data Analyst — (Python, SQL, Tableau)</option>
        <option value="custom">Custom (Paste your own Job Description)</option>
      </select>

      <div id="customJdBox" style="display: none;">
        <label for="customJd">Paste Job Description:</label>
        <textarea id="customJd" placeholder="Paste company requirements, key skills, and responsibilities..."></textarea>
      </div>

      <button id="analyzeBtn" class="btn-primary" onclick="startAnalysis()">🚀 Analyze Fit & Skill Gaps</button>
      <div id="spinnerText" class="spinner">Analyzing resume and generating upskilling plan...</div>
    </div>

    <!-- Output Dashboard -->
    <div id="resultsSection" class="results-section">
      <!-- Verdict Banner -->
      <div id="scoreBanner" class="score-banner">
        <div>
          <div style="font-size: 0.85rem; text-transform: uppercase; letter-spacing: 0.05em;">ATS Match Score</div>
          <div id="scoreValue" class="score">0%</div>
        </div>
        <div id="verdictText" style="font-size: 1.1rem; font-weight: 600;"></div>
      </div>

      <!-- Skills Matrix -->
      <div class="grid-2">
        <div class="card">
          <h3>✅ Matched Skills</h3>
          <p style="color: var(--muted); font-size: 0.85rem; margin-top: 4px;">Skills present in the resume:</p>
          <div id="matchedSkills" class="tag-container"></div>
        </div>
        <div class="card">
          <h3>❌ Skill Gaps</h3>
          <p style="color: var(--muted); font-size: 0.85rem; margin-top: 4px;">Mandatory skills missing:</p>
          <div id="missingSkills" class="tag-container"></div>
        </div>
      </div>

      <!-- Upskilling Roadmap -->
      <div class="card">
        <h3>📅 3-Week Upskilling Roadmap</h3>
        <p style="color: var(--muted); font-size: 0.85rem; margin-bottom: 12px;">Step-by-step action plan to qualify for selection:</p>
        <div id="roadmapList"></div>
      </div>

      <!-- Companies to Target -->
      <div class="card">
        <h3>🏢 Places to Learn & Get Hired</h3>
        <p style="color: var(--muted); font-size: 0.85rem; margin-bottom: 12px;">Companies and platforms ready for this profile:</p>
        <div id="companyList"></div>
      </div>
    </div>
  </div>

  <script>
    // Initialize PDF.js worker
    pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';

    function handleRoleChange() {
      const role = document.getElementById("targetRole").value;
      document.getElementById("customJdBox").style.display = (role === "custom") ? "block" : "none";
    }

    async function extractPdfText(file) {
      const arrayBuffer = await file.arrayBuffer();
      const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
      let fullText = "";
      for (let i = 1; i <= pdf.numPages; i++) {
        const page = await pdf.getPage(i);
        const textContent = await page.getTextContent();
        fullText += textContent.items.map(item => item.str).join(" ") + "\n";
      }
      return fullText;
    }

    async function startAnalysis() {
      const apiKey = document.getElementById("apiKey").value.trim();
      const fileInput = document.getElementById("resumeFile");
      const targetRoleSelect = document.getElementById("targetRole").value;
      const customJd = document.getElementById("customJd").value.trim();
      const analyzeBtn = document.getElementById("analyzeBtn");
      const spinnerText = document.getElementById("spinnerText");

      if (!apiKey) {
        alert("Please enter your Gemini API key.");
        return;
      }
      if (!fileInput.files.length) {
        alert("Please upload a PDF resume.");
        return;
      }

      const jobRequirements = (targetRoleSelect === "custom") ? customJd : targetRoleSelect;
      if (!jobRequirements) {
        alert("Please provide the target job requirements.");
        return;
      }

      analyzeBtn.disabled = true;
      spinnerText.style.display = "block";

      try {
        const resumeText = await extractPdfText(fileInput.files[0]);

        const prompt = `
        You are an expert HR ATS auditor and technical mentor.
        Evaluate this candidate's resume against the specified target requirements.

        Candidate Resume:
        ${resumeText}

        Target Requirements:
        ${jobRequirements}

        Return strictly valid JSON only (no markdown quotes, no triple backticks) matching this schema:
        {
          "match_percentage": <integer 0-100>,
          "is_qualified": <boolean>,
          "verdict": "<short 1-sentence qualification verdict>",
          "matched_skills": ["skill1", "skill2"],
          "missing_skills": ["skill1", "skill2"],
          "roadmap": [
            {"phase": "Week 1", "task": "<what to learn>", "resource": "<platform/site>"},
            {"phase": "Week 2", "task": "<mini project to build>", "resource": "<practice link/project>"},
            {"phase": "Week 3", "task": "<interview preparation>", "resource": "<interview prep site>"}
          ],
          "hiring_companies": [
            {"name": "<Company/Platform>", "type": "<Hiring / Internship / Training>", "focus": "<skill>"}
          ]
        }`;

        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;
        
        const response = await fetch(apiUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            contents: [{ parts: [{ text: prompt }] }],
            generationConfig: { responseMimeType: "application/json" }
          })
        });

        if (!response.ok) {
          const errData = await response.json();
          throw new Error(errData.error?.message || "Failed to reach Gemini API");
        }

        const data = await response.json();
        const parsedResult = JSON.parse(data.candidates[0].content.parts[0].text);
        renderOutput(parsedResult);

      } catch (err) {
        alert("Error analyzing resume: " + err.message);
      } finally {
        analyzeBtn.disabled = false;
        spinnerText.style.display = "none";
      }
    }

    function renderOutput(data) {
      document.getElementById("resultsSection").style.display = "block";

      // Score Banner
      const scoreBanner = document.getElementById("scoreBanner");
      scoreBanner.className = "score-banner " + (data.is_qualified ? "qualified" : "unqualified");
      document.getElementById("scoreValue").innerText = data.match_percentage + "%";
      document.getElementById("verdictText").innerText = (data.is_qualified ? "✅ Qualified: " : "⚠️ Skill Gap: ") + data.verdict;

      // Matched Skills
      const matchedContainer = document.getElementById("matchedSkills");
      matchedContainer.innerHTML = "";
      (data.matched_skills || []).forEach(skill => {
        matchedContainer.innerHTML += `<span class="tag matched">${skill}</span>`;
      });

      // Missing Skills
      const missingContainer = document.getElementById("missingSkills");
      missingContainer.innerHTML = "";
      (data.missing_skills || []).forEach(skill => {
        missingContainer.innerHTML += `<span class="tag missing">${skill}</span>`;
      });

      // Roadmap
      const roadmapContainer = document.getElementById("roadmapList");
      roadmapContainer.innerHTML = "";
      (data.roadmap || []).forEach(item => {
        roadmapContainer.innerHTML += `
          <div class="roadmap-item">
            <h4>${item.phase}: ${item.task}</h4>
            <div style="color: var(--muted); font-size: 0.85rem;"><strong>Recommended Resource:</strong> ${item.resource}</div>
          </div>
        `;
      });

      // Companies
      const companyContainer = document.getElementById("companyList");
      companyContainer.innerHTML = "";
      (data.hiring_companies || []).forEach(comp => {
        companyContainer.innerHTML += `
          <div class="company-item">
            <strong>${comp.name}</strong> — <em>${comp.type}</em>
            <div style="color: var(--muted); font-size: 0.85rem; margin-top: 4px;">Target Skill: ${comp.focus}</div>
          </div>
        `;
      });

      scoreBanner.scrollIntoView({ behavior: "smooth" });
    }
  </script>
</body>
</html>

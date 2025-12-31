let currentLevel = "";

// This function checks if you already entered a key
function getVaultKey() {
    let key = sessionStorage.getItem("aureus_key");
    if (!key) {
        key = prompt("Please enter your Aureus Academy API Key to unlock the Vault:");
        if (key) {
            sessionStorage.setItem("aureus_key", key);
        }
    }
    return key;
}

async function captureAndScan() {
    const apiKey = getVaultKey(); // Get the key from the popup
    if (!apiKey) return alert("Access Denied: No API Key provided.");

    const status = document.getElementById('status-text');
    const aiBox = document.getElementById('ai-response');
    status.innerText = "Analyzing your document with Aureus AI...";

    const video = document.getElementById('webcam');
    
    try {
        // 1. Tesseract reads the image
        const { data: { text } } = await Tesseract.recognize(video, 'eng');
        
        // 2. Send text to Gemini AI
        const prompt = `Act as the Aureus Academy Tutor. Level: ${currentLevel}. 
        Subject: General/Language. Task: Explain this scanned text in an exciting, 
        detailed way. Correct any errors. Text: ${text}`;

        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ contents: [{ parts: [{ text: prompt }] }] })
        });

        const data = await response.json();
        
        if (data.error) throw new Error(data.error.message);

        const aiMessage = data.candidates[0].content.parts[0].text;
        
        // 3. Show result and Verse
        aiBox.innerText = aiMessage;
        status.innerText = "Knowledge Unlocked.";
        showClosingVerse();
    } catch (error) {
        console.error(error);
        status.innerText = "The Vault is locked. Check your key or internet.";
        sessionStorage.removeItem("aureus_key"); // Clear bad key so user can retry
    }
}

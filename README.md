<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Assisting Tool</title>
    <style>
        body {
            background-color: #000;
            color: #FFD700;
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }

        .container {
            text-align: center;
            border: 2px solid #FFD700;
            border-radius: 10px;
            padding: 20px;
            width: 90%;
            max-width: 600px;
        }

        h1 {
            font-size: 2rem;
            margin-bottom: 20px;
        }

        .input-section {
            margin: 15px 0;
        }

        input[type="file"], textarea, button {
            width: 100%;
            margin: 10px 0;
            padding: 10px;
            font-size: 1rem;
            border: 1px solid #FFD700;
            border-radius: 5px;
            background-color: #222;
            color: #FFD700;
        }

        button {
            cursor: pointer;
            font-weight: bold;
        }

        button:hover {
            background-color: #FFD700;
            color: #000;
        }

        .output-section {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>AI Assisting Tool</h1>
        <div class="input-section">
            <label for="fileInput">Upload a File:</label>
            <input type="file" id="fileInput" accept=".txt">
        </div>
        <div class="input-section">
            <label for="textInput">Or Enter Text:</label>
            <textarea id="textInput" rows="6" placeholder="Paste your text here..."></textarea>
        </div>
        <button onclick="processInput()">Process</button>

        <div class="output-section" id="outputSection" style="display: none;">
            <h2>Processed Result</h2>
            <textarea id="outputText" rows="6" readonly></textarea>
            <button onclick="downloadResult()">Download Result</button>
        </div>
    </div>

    <script>
        async function processInput() {
            console.log('Process button clicked');  // For debugging

            const fileInput = document.getElementById('fileInput');
            const textInput = document.getElementById('textInput');
            const outputSection = document.getElementById('outputSection');
            const outputText = document.getElementById('outputText');

            let inputText = textInput.value;

            if (fileInput.files.length > 0) {
                console.log('File selected');  // For debugging
                const file = fileInput.files[0];
                const reader = new FileReader();

                reader.onload = async function(event) {
                    console.log('File loaded');  // For debugging
                    inputText = event.target.result;
                    const processedText = await processWithAI(inputText);
                    outputText.value = processedText;
                    outputSection.style.display = 'block';
                };

                reader.readAsText(file);
            } else if (inputText.trim() !== '') {
                console.log('Processing entered text');  // For debugging
                const processedText = await processWithAI(inputText);
                outputText.value = processedText;
                outputSection.style.display = 'block';
            } else {
                alert('Please upload a file or enter text.');
            }
        }

        async function processWithAI(inputText) {
            const apiKey = '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' 

; // Replace with your actual API key
            const prompt = `Make the following text more detailed and concise, removing unnecessary data:\n\n${inputText}`;

            try {
                console.log('Sending request to OpenAI API...');
                const response = await fetch('https://api.openai.com/v1/completions', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${apiKey}`,
                    },
                    body: JSON.stringify({
                        model: 'gpt-4o-mini', // Your specified model
                        prompt: prompt,
                        max_tokens: 1500,
                    }),
                });

                console.log('Response Status:', response.status);

                if (response.ok) {
                    const data = await response.json();
                    if (data.choices && data.choices[0].text) {
                        return data.choices[0].text.trim();
                    } else {
                        console.error('Unexpected response format:', data);
                        return 'Error processing the text: Unexpected response format.';
                    }
                } else {
                    const errorDetails = await response.json();
                    console.error('API Error:', errorDetails);
                    return `Error processing the text: ${errorDetails.error.message}`;
                }
            } catch (error) {
                console.error('Request failed:', error);
                return `Error connecting to the OpenAI API: ${error.message}`;
            }
        }

        function downloadResult() {
            const outputText = document.getElementById('outputText').value;
            const blob = new Blob([outputText], { type: 'text/plain' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = 'processed_result.txt';
            link.click();
        }
    </script>
</body>
</html>

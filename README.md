<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>語音互動 AI 助手</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f7f7f7;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 800px;
      margin: 50px auto;
      background-color: #fff;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    h1, h2 {
      color: #333;
    }
    .btn {
      background-color: #000000;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      margin-top: 10px;
    }
    .status {
      margin-top: 20px;
      font-weight: bold;
      color: #555;
    }
    .result-container {
      margin-top: 30px;
    }
    .result-box {
      background-color: #f1f1f1;
      padding: 15px;
      border-radius: 5px;
      min-height: 50px;
      margin-top: 10px;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>語音互動 AI 助手</h1>

    <div class="speech-container">
      <button id="startVoice" class="btn">開始說話</button>
      <div class="status" id="status">準備就緒...</div>
    </div>

    <div class="result-container">
      <div class="recognition-result">
        <h2>語音辨識結果</h2>
        <div id="recognitionResult" class="result-box"></div>
      </div>

      <div class="gemini-response">
        <h2>Gemini 回應</h2>
        <div id="geminiResponse" class="result-box"></div>
      </div>
    </div>
  </div>

  <script>
    const startVoiceBtn = document.getElementById("startVoice");
    const status = document.getElementById("status");
    const recognitionResult = document.getElementById("recognitionResult");
    const geminiResponse = document.getElementById("geminiResponse");

    const API_KEY = "AIzaSyC-OUG866kV5FOk75_sxmAPbC5c2kRliwU"; // 請換成你自己的 API KEY

    startVoiceBtn.addEventListener("click", () => {
      // 檢查瀏覽器是否支援 SpeechRecognition
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      if (!SpeechRecognition) {
        status.textContent = "你的瀏覽器不支援語音辨識功能";
        return;
      }

      const recognition = new SpeechRecognition();
      recognition.lang = "zh-TW";
      recognition.interimResults = false;

      recognition.onstart = () => {
        status.textContent = "聆聽中...";
        geminiResponse.textContent = "";
        recognitionResult.textContent = "";
      };

      recognition.onresult = async (event) => {
        const text = event.results[0][0].transcript;
        recognitionResult.textContent = text;
        status.textContent = "語音辨識完成，正在詢問 Gemini...";
        await askGemini(text);
      };

      recognition.onerror = (event) => {
        console.error("語音辨識錯誤：", event.error);
        status.textContent = "語音辨識錯誤，請重試。";
      };

      recognition.start();
    });

    async function askGemini(text) {
      const endpoint = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`;

      try {
        const response = await fetch(endpoint, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            contents: [
              {
                parts: [{ text: text }]
              }
            ]
          }),
        });

        if (!response.ok) {
          const errorText = await response.text();
          throw new Error(`HTTP ${response.status} - ${response.statusText} | ${errorText}`);
        }

        const data = await response.json();
        // 擷取純文字回答
        const message = data.candidates?.[0]?.content?.parts?.[0]?.text;

        if (!message) {
          geminiResponse.textContent = "抱歉，沒有收到有效回應。";
        } else {
          geminiResponse.textContent = message;
        }

        status.textContent = "完成";
      } catch (error) {
        console.error("詢問 Gemini 發生錯誤：", error);
        geminiResponse.textContent = `錯誤: ${error.message}`;
        status.textContent = "請檢查 API 設定";
      }
    }
  </script>
</body>
</html>

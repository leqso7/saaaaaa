<!DOCTYPE html>
<html lang="ka">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ტექსტის შემაჯამებელი</title>
    <style>
        /* სტილები უცვლელია */
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
        }
        .container {
            max-width: 500px;
            width: 100%;
            padding: 20px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            background-color: white;
            border-radius: 8px;
        }
        h1 {
            text-align: center;
        }
        #video {
            width: 100%;
            max-width: 400px;
            margin-bottom: 20px;
        }
        #captureBtn, #newPhotoBtn {
            display: block;
            width: 100%;
            padding: 10px;
            margin: 10px 0;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        #summary {
            margin-top: 20px;
            padding: 10px;
            background-color: #f8f9fa;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>ტექსტის შემაჯამებელი</h1>
        <video id="video" autoplay playsinline></video>
        <canvas id="canvas" style="display:none;"></canvas>
        <button id="captureBtn">გადაღება</button>
        <button id="newPhotoBtn" style="display:none;">ახალი ფოტო</button>
        <div id="summary"></div>
    </div>

    <script>
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const captureBtn = document.getElementById('captureBtn');
        const newPhotoBtn = document.getElementById('newPhotoBtn');
        const summaryDiv = document.getElementById('summary');

        async function startCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
                video.srcObject = stream;
            } catch (err) {
                console.error("კამერასთან წვდომის შეცდომა:", err);
            }
        }

        function captureImage() {
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            canvas.getContext('2d').drawImage(video, 0, 0);
            const imageDataUrl = canvas.toDataURL('image/jpeg');
            processImage(imageDataUrl);
            
            video.style.display = 'none';
            captureBtn.style.display = 'none';
            newPhotoBtn.style.display = 'block';
        }

        async function processImage(imageDataUrl) {
            summaryDiv.textContent = "დამუშავება...";
            try {
                // სურათის გაგზავნა სერვერზე
                const response = await fetch('/api/summarize', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ image: imageDataUrl }),
                });
                
                if (!response.ok) {
                    throw new Error('სერვერის შეცდომა');
                }
                
                const data = await response.json();
                summaryDiv.innerHTML = `<strong>მოკლე შინაარსი:</strong><br>${data.summary}`;
            } catch (error) {
                console.error('შეცდომა:', error);
                summaryDiv.textContent = "შეცდომა დამუშავებისას. გთხოვთ, სცადოთ თავიდან.";
            }
        }

        function newPhoto() {
            video.style.display = 'block';
            captureBtn.style.display = 'block';
            newPhotoBtn.style.display = 'none';
            summaryDiv.textContent = '';
        }

        startCamera();
        captureBtn.addEventListener('click', captureImage);
        newPhotoBtn.addEventListener('click', newPhoto);
    </script>
</body>
</html>

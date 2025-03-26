# xiangsu
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>像素风格图片生成器 - 冯师制作</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
            color: #333;
        }
        h1 {
            text-align: center;
            color: #2c3e50;
        }
        .container {
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .controls {
            margin: 20px 0;
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            align-items: center;
        }
        .control-group {
            display: flex;
            flex-direction: column;
            min-width: 200px;
        }
        label {
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="range"] {
            width: 100%;
        }
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
            margin-top: 10px;
        }
        button:hover {
            background-color: #2980b9;
        }
        .canvas-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 30px;
            margin-top: 20px;
        }
        .canvas-wrapper {
            text-align: center;
        }
        canvas {
            border: 1px solid #ddd;
            max-width: 100%;
            background-color: #f0f0f0;
        }
        .download-btn {
            background-color: #27ae60;
        }
        .download-btn:hover {
            background-color: #219653;
        }
        .value-display {
            font-size: 14px;
            color: #666;
        }
        @media (max-width: 768px) {
            .controls {
                flex-direction: column;
                align-items: flex-start;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>像素风格图片生成器 - 改进版</h1>
        
        <div class="controls">
            <div class="control-group">
                <label for="imageUpload">上传图片:</label>
                <input type="file" id="imageUpload" accept="image/*">
            </div>
            
            <div class="control-group">
                <label for="pixelSize">像素大小: <span id="pixelSizeValue" class="value-display">10</span></label>
                <input type="range" id="pixelSize" min="2" max="30" value="10">
            </div>
            
            <div class="control-group">
                <label for="colorDepth">颜色深度: <span id="colorDepthValue" class="value-display">16</span></label>
                <input type="range" id="colorDepth" min="2" max="32" value="16" step="2">
            </div>
            
            <div class="control-group">
                <label for="scaleFactor">预览缩放: <span id="scaleFactorValue" class="value-display">100%</span></label>
                <input type="range" id="scaleFactor" min="25" max="200" value="100">
            </div>
            
            <button id="processBtn">生成像素图</button>
        </div>
        
        <div class="canvas-container">
            <div class="canvas-wrapper">
                <h3>原始图片</h3>
                <canvas id="originalCanvas"></canvas>
            </div>
            <div class="canvas-wrapper">
                <h3>像素风格</h3>
                <canvas id="pixelCanvas"></canvas>
                <button id="downloadBtn" class="download-btn">下载像素图</button>
            </div>
        </div>
    </div>

    <script>
        // 获取DOM元素
        const imageUpload = document.getElementById('imageUpload');
        const pixelSize = document.getElementById('pixelSize');
        const pixelSizeValue = document.getElementById('pixelSizeValue');
        const colorDepth = document.getElementById('colorDepth');
        const colorDepthValue = document.getElementById('colorDepthValue');
        const scaleFactor = document.getElementById('scaleFactor');
        const scaleFactorValue = document.getElementById('scaleFactorValue');
        const processBtn = document.getElementById('processBtn');
        const originalCanvas = document.getElementById('originalCanvas');
        const pixelCanvas = document.getElementById('pixelCanvas');
        const downloadBtn = document.getElementById('downloadBtn');
        
        const originalCtx = originalCanvas.getContext('2d');
        const pixelCtx = pixelCanvas.getContext('2d');
        let uploadedImage = null;
        let currentScale = 1;
        
        // 最大处理尺寸
        const MAX_WIDTH = 800;
        const MAX_HEIGHT = 800;
        
        // 更新滑块值显示
        pixelSize.addEventListener('input', updateValues);
        colorDepth.addEventListener('input', updateValues);
        scaleFactor.addEventListener('input', updateValues);
        
        function updateValues() {
            pixelSizeValue.textContent = pixelSize.value;
            colorDepthValue.textContent = colorDepth.value;
            scaleFactorValue.textContent = `${scaleFactor.value}%`;
            currentScale = scaleFactor.value / 100;
            
            if (uploadedImage) {
                drawOriginalImage();
            }
        }
        
        // 图片上传处理
        imageUpload.addEventListener('change', (e) => {
            const file = e.target.files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = (event) => {
                uploadedImage = new Image();
                uploadedImage.onload = () => {
                    drawOriginalImage();
                };
                uploadedImage.src = event.target.result;
            };
            reader.readAsDataURL(file);
        });
        
        // 绘制原始图片（带缩放）
        function drawOriginalImage() {
            let width = uploadedImage.width;
            let height = uploadedImage.height;
            
            // 调整尺寸不超过最大值
            if (width > MAX_WIDTH || height > MAX_HEIGHT) {
                const ratio = Math.min(MAX_WIDTH / width, MAX_HEIGHT / height);
                width *= ratio;
                height *= ratio;
            }
            
            // 应用预览缩放
            const displayWidth = width * currentScale;
            const displayHeight = height * currentScale;
            
            originalCanvas.width = width;
            originalCanvas.height = height;
            pixelCanvas.width = width;
            pixelCanvas.height = height;
            
            // 设置显示尺寸（不影响实际处理尺寸）
            originalCanvas.style.width = `${displayWidth}px`;
            originalCanvas.style.height = `${displayHeight}px`;
            pixelCanvas.style.width = `${displayWidth}px`;
            pixelCanvas.style.height = `${displayHeight}px`;
            
            // 绘制原始图片
            originalCtx.drawImage(uploadedImage, 0, 0, width, height);
        }
        
        // 处理图片生成像素风格
        processBtn.addEventListener('click', () => {
            if (!uploadedImage) {
                alert('请先上传图片');
                return;
            }
            
            const size = parseInt(pixelSize.value);
            const depth = parseInt(colorDepth.value);
            
            // 创建临时canvas来处理图像
            const tempCanvas = document.createElement('canvas');
            tempCanvas.width = originalCanvas.width;
            tempCanvas.height = originalCanvas.height;
            const tempCtx = tempCanvas.getContext('2d');
            
            // 绘制缩小版的图像
            const scaledWidth = Math.ceil(originalCanvas.width / size);
            const scaledHeight = Math.ceil(originalCanvas.height / size);
            
            tempCtx.imageSmoothingEnabled = false;
            tempCtx.drawImage(
                originalCanvas, 
                0, 0, 
                originalCanvas.width, originalCanvas.height,
                0, 0,
                scaledWidth, scaledHeight
            );
            
            // 获取缩小后的图像数据
            const imageData = tempCtx.getImageData(0, 0, scaledWidth, scaledHeight);
            const data = imageData.data;
            
            // 处理每个像素块
            for (let y = 0; y < scaledHeight; y++) {
                for (let x = 0; x < scaledWidth; x++) {
                    const index = (y * scaledWidth + x) * 4;
                    
                    // 简化颜色深度
                    if (depth < 256) {
                        const factor = 256 / depth;
                        data[index] = Math.round(data[index] / factor) * factor;     // R
                        data[index + 1] = Math.round(data[index + 1] / factor) * factor; // G
                        data[index + 2] = Math.round(data[index + 2] / factor) * factor; // B
                    }
                }
            }
            
            // 将处理后的数据放回临时canvas
            tempCtx.putImageData(imageData, 0, 0);
            
            // 放大回原始尺寸
            pixelCtx.imageSmoothingEnabled = false;
            pixelCtx.clearRect(0, 0, pixelCanvas.width, pixelCanvas.height);
            pixelCtx.drawImage(
                tempCanvas,
                0, 0,
                scaledWidth, scaledHeight,
                0, 0,
                originalCanvas.width, originalCanvas.height
            );
        });
        
        // 下载处理后的图片
        downloadBtn.addEventListener('click', () => {
            if (!uploadedImage) {
                alert('没有可下载的图片');
                return;
            }
            
            // 创建一个临时canvas保存原始尺寸的像素图
            const downloadCanvas = document.createElement('canvas');
            downloadCanvas.width = originalCanvas.width;
            downloadCanvas.height = originalCanvas.height;
            const downloadCtx = downloadCanvas.getContext('2d');
            
            // 重新绘制像素图（确保下载的是原始尺寸）
            const size = parseInt(pixelSize.value);
            const depth = parseInt(colorDepth.value);
            const scaledWidth = Math.ceil(originalCanvas.width / size);
            const scaledHeight = Math.ceil(originalCanvas.height / size);
            
            downloadCtx.drawImage(
                originalCanvas, 
                0, 0, 
                originalCanvas.width, originalCanvas.height,
                0, 0,
                scaledWidth, scaledHeight
            );
            
            const imageData = downloadCtx.getImageData(0, 0, scaledWidth, scaledHeight);
            const data = imageData.data;
            
            for (let y = 0; y < scaledHeight; y++) {
                for (let x = 0; x < scaledWidth; x++) {
                    const index = (y * scaledWidth + x) * 4;
                    if (depth < 256) {
                        const factor = 256 / depth;
                        data[index] = Math.round(data[index] / factor) * factor;
                        data[index + 1] = Math.round(data[index + 1] / factor) * factor;
                        data[index + 2] = Math.round(data[index + 2] / factor) * factor;
                    }
                }
            }
            
            downloadCtx.putImageData(imageData, 0, 0);
            downloadCtx.imageSmoothingEnabled = false;
            downloadCtx.drawImage(
                downloadCanvas,
                0, 0,
                scaledWidth, scaledHeight,
                0, 0,
                originalCanvas.width, originalCanvas.height
            );
            
            const link = document.createElement('a');
            link.download = `pixel-art-${size}px-${depth}colors.png`;
            link.href = downloadCanvas.toDataURL('image/png');
            link.click();
        });
    </script>
</body>
</html>

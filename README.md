# Mi-Diario
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Diario Personal</title>
    <style>
        body {
            font-family: 'Georgia', serif;
            background-color: #f0e6d2;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background-image: linear-gradient(#e2e2e2 1px, transparent 1px);
            background-size: 100% 24px;
        }
        
        .login-container {
            background-color: #fff9e6;
            padding: 30px;
            border-radius: 5px;
            box-shadow: 0 0 15px rgba(0,0,0,0.2);
            border: 1px solid #d4c9a8;
            text-align: center;
            width: 300px;
        }
        
        .diary-container {
            display: none;
            background-color: #fff9e6;
            width: 90%;
            max-width: 800px;
            min-height: 500px;
            padding: 40px;
            box-shadow: 0 0 15px rgba(0,0,0,0.2);
            border: 1px solid #d4c9a8;
            position: relative;
            background-image: linear-gradient(to bottom, transparent 23px, #e0e0e0 23px, #e0e0e0 24px, transparent 24px);
            background-size: 100% 24px;
            background-position: 0 25px;
            line-height: 24px;
        }
        
        h1 {
            color: #5a4a3a;
            text-align: center;
            font-family: 'Courier New', monospace;
            border-bottom: 1px solid #d4c9a8;
            padding-bottom: 10px;
            margin-top: 0;
        }
        
        .diary-date {
            font-family: 'Courier New', monospace;
            color: #5a4a3a;
            text-align: right;
            margin-bottom: 10px;
            border: none;
            background: transparent;
            width: 200px;
            float: right;
            border-bottom: 1px solid #d4c9a8;
        }
        
        .diary-title {
            font-size: 1.5em;
            font-weight: bold;
            margin-bottom: 20px;
            border: none;
            background: transparent;
            width: 100%;
            font-family: 'Georgia', serif;
            border-bottom: 1px solid #d4c9a8;
            padding-bottom: 5px;
            clear: both;
        }
        
        .diary-content {
            width: 100%;
            min-height: 400px;
            padding: 0;
            border: none;
            background: transparent;
            font-family: 'Georgia', serif;
            font-size: 16px;
            line-height: 24px;
            resize: none;
            outline: none;
            margin-top: 10px;
        }
        
        .diary-content:focus, .diary-title:focus, .diary-date:focus {
            outline: none;
        }
        
        button {
            background-color: #8b7355;
            color: white;
            border: none;
            padding: 8px 15px;
            font-family: 'Georgia', serif;
            font-size: 14px;
            margin: 10px 5px;
            cursor: pointer;
            border-radius: 3px;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #6b5a45;
        }
        
        .password-container {
            position: relative;
            width: 80%;
            margin: 15px auto;
        }
        
        .password-container input {
            padding: 8px;
            width: 100%;
            border: 1px solid #d4c9a8;
            border-radius: 3px;
            font-family: 'Courier New', monospace;
        }
        
        .toggle-password {
            position: absolute;
            right: 10px;
            top: 50%;
            transform: translateY(-50%);
            cursor: pointer;
            color: #5a4a3a;
            font-size: 0.8em;
        }
        
        .error {
            color: #c00;
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        .controls {
            display: flex;
            justify-content: space-between;
            margin-top: 20px;
            border-top: 1px solid #d4c9a8;
            padding-top: 15px;
        }
        
        .page-controls {
            display: flex;
            gap: 10px;
        }
        
        .margin-line {
            position: absolute;
            left: 30px;
            top: 0;
            bottom: 0;
            width: 1px;
            background-color: #ff9999;
            opacity: 0.5;
        }
        
        .save-notification {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background-color: #8b7355;
            color: white;
            padding: 10px 15px;
            border-radius: 5px;
            opacity: 0;
            transition: opacity 0.3s;
            pointer-events: none;
        }
        
        .save-notification.show {
            opacity: 1;
        }
    </style>
</head>
<body>
    <div class="login-container" id="loginSection">
        <h1>Diario Secreto</h1>
        <p>Ingresa tu contraseña para continuar:</p>
        <div class="password-container">
            <input type="password" id="passwordInput" placeholder="Escribe tu contraseña">
            <span class="toggle-password" onclick="togglePasswordVisibility()">👁️</span>
        </div>
        <p id="errorMessage" class="error"></p>
        <button onclick="checkPassword()">Abrir Diario</button>
    </div>

    <div class="diary-container" id="diarySection">
        <div class="margin-line"></div>
        <input type="text" class="diary-date" id="diaryDate">
        <input type="text" class="diary-title" id="diaryTitle" placeholder="Título de la entrada">
        <textarea class="diary-content" id="diaryContent" placeholder="Escribe aquí tus pensamientos..."></textarea>
        
        <div class="controls">
            <div class="page-controls">
                <button onclick="previousPage()">← Página anterior</button>
                <button onclick="nextPage()">Página siguiente →</button>
            </div>
            <div>
                <button onclick="saveEntry()">Guardar</button>
                <button onclick="logout()">Cerrar</button>
            </div>
        </div>
    </div>

    <div class="save-notification" id="saveNotification">Cambios guardados</div>

    <script>
        // Contraseña predefinida (cámbiala por tu contraseña real)
        const correctPassword = "diarioSecreto2024";
        let currentPage = 1;
        const MAX_PAGES = 10;
        
        // Mostrar fecha actual
        function updateDate() {
            const now = new Date();
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
            document.getElementById('diaryDate').value = now.toLocaleDateString('es-ES', options);
        }
        
        // Alternar visibilidad de contraseña
        function togglePasswordVisibility() {
            const passwordInput = document.getElementById('passwordInput');
            if (passwordInput.type === 'password') {
                passwordInput.type = 'text';
            } else {
                passwordInput.type = 'password';
            }
        }
        
        // Verificar contraseña
        function checkPassword() {
            const enteredPassword = document.getElementById('passwordInput').value;
            const errorElement = document.getElementById('errorMessage');
            
            if (enteredPassword === correctPassword) {
                document.getElementById('loginSection').style.display = 'none';
                document.getElementById('diarySection').style.display = 'block';
                updateDate();
                loadPage(currentPage);
                document.getElementById('diaryTitle').focus();
            } else {
                errorElement.textContent = "Contraseña incorrecta. Intenta nuevamente.";
            }
        }
        
        // Mostrar notificación de guardado
        function showSaveNotification() {
            const notification = document.getElementById('saveNotification');
            notification.classList.add('show');
            setTimeout(() => {
                notification.classList.remove('show');
            }, 2000);
        }
        
        // Guardar entrada
        function saveEntry() {
            const entryData = {
                title: document.getElementById('diaryTitle').value,
                date: document.getElementById('diaryDate').value,
                content: document.getElementById('diaryContent').value
            };
            
            localStorage.setItem(`diaryPage_${currentPage}`, JSON.stringify(entryData));
            showSaveNotification();
        }
        
        // Auto-guardado
        let saveTimeout;
        function scheduleSave() {
            clearTimeout(saveTimeout);
            saveTimeout = setTimeout(() => {
                saveEntry();
            }, 30000);
        }
        
        // Cargar página
        function loadPage(pageNumber) {
            const savedEntry = localStorage.getItem(`diaryPage_${pageNumber}`);
            if (savedEntry) {
                const entryData = JSON.parse(savedEntry);
                document.getElementById('diaryTitle').value = entryData.title || '';
                document.getElementById('diaryDate').value = entryData.date || '';
                document.getElementById('diaryContent').value = entryData.content || '';
            } else {
                document.getElementById('diaryTitle').value = '';
                document.getElementById('diaryContent').value = '';
                updateDate();
            }
            currentPage = pageNumber;
        }
        
        // Página siguiente
        function nextPage() {
            if (currentPage < MAX_PAGES) {
                saveEntry();
                loadPage(currentPage + 1);
            } else {
                alert("Has alcanzado el número máximo de páginas.");
            }
        }
        
        // Página anterior
        function previousPage() {
            if (currentPage > 1) {
                saveEntry();
                loadPage(currentPage - 1);
            }
        }
        
        // Cerrar sesión
        function logout() {
            if (confirm("¿Seguro que quieres cerrar el diario? Se guardarán los cambios automáticamente.")) {
                saveEntry();
                document.getElementById('loginSection').style.display = 'block';
                document.getElementById('diarySection').style.display = 'none';
                document.getElementById('passwordInput').value = '';
                document.getElementById('passwordInput').type = 'password';
                document.getElementById('errorMessage').textContent = '';
            }
        }
        
        // Event listeners
        document.getElementById('diaryTitle').addEventListener('input', scheduleSave);
        document.getElementById('diaryContent').addEventListener('input', scheduleSave);
        document.getElementById('diaryDate').addEventListener('input', scheduleSave);
    </script>
</body>
</html>
{
  "name": "Mi Diario Secreto",
  "short_name": "Diario",
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#f0e6d2",
  "theme_color": "#8b7355",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}

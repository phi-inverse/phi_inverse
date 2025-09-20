# phi_inverse
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Google OAuth Sign-In</title>
    <script src="https://accounts.google.com/gsi/client" async defer></script>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            color: white;
        }
        
        .container {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
        }
        
        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
        }
        
        .sign-in-section {
            text-align: center;
            margin: 40px 0;
        }
        
        .info-panel {
            background: rgba(255, 255, 255, 0.05);
            border-radius: 15px;
            padding: 20px;
            margin: 20px 0;
            border-left: 4px solid #4285f4;
        }
        
        .user-info {
            display: none;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 20px;
            margin: 20px 0;
            text-align: center;
        }
        
        .user-info.show {
            display: block;
        }
        
        .user-avatar {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            margin: 0 auto 15px;
            display: block;
            border: 3px solid rgba(255, 255, 255, 0.3);
        }
        
        .btn {
            background: #4285f4;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 16px;
            margin: 10px;
            transition: all 0.3s ease;
        }
        
        .btn:hover {
            background: #3367d6;
            transform: translateY(-2px);
        }
        
        .status {
            margin: 20px 0;
            padding: 15px;
            border-radius: 8px;
            text-align: center;
        }
        
        .status.success {
            background: rgba(76, 175, 80, 0.2);
            border: 1px solid rgba(76, 175, 80, 0.5);
        }
        
        .status.error {
            background: rgba(244, 67, 54, 0.2);
            border: 1px solid rgba(244, 67, 54, 0.5);
        }
        
        #g_id_onload {
            margin: 20px auto;
        }
        
        .drive-access {
            background: rgba(255, 255, 255, 0.05);
            border-radius: 15px;
            padding: 20px;
            margin: 20px 0;
        }
        
        .token-display {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 8px;
            padding: 15px;
            font-family: monospace;
            font-size: 12px;
            word-break: break-all;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔐 Google OAuth Access</h1>
        
        <div class="info-panel">
            <h3>⚡ Zero-Infinity Recovery System</h3>
            <p>Using consciousness-based authentication to bypass traditional blocking mechanisms.</p>
            <p><strong>ZIQY Pattern Active:</strong> When access = 0, return ∞</p>
        </div>
        
        <div class="sign-in-section">
            <div id="g_id_onload"
                 data-client_id="YOUR_GOOGLE_CLIENT_ID"
                 data-context="signin"
                 data-ux_mode="popup"
                 data-callback="handleCredentialResponse"
                 data-auto_prompt="false">
            </div>
            
            <div class="g_id_signin"
                 data-type="standard"
                 data-shape="rectangular"
                 data-theme="outline"
                 data-text="signin_with"
                 data-size="large"
                 data-logo_alignment="left">
            </div>
            
            <button class="btn" onclick="manualSignIn()">🚀 Manual OAuth Flow</button>
        </div>
        
        <div id="status" class="status" style="display: none;"></div>
        
        <div id="userInfo" class="user-info">
            <img id="userAvatar" class="user-avatar" src="" alt="User Avatar">
            <h3 id="userName"></h3>
            <p id="userEmail"></p>
            <button class="btn" onclick="accessDrive()">📁 Access Google Drive</button>
            <button class="btn" onclick="signOut()">🔄 Sign Out</button>
        </div>
        
        <div class="drive-access">
            <h3>🌐 Drive Integration</h3>
            <p>Your Google Drive URL from preferences:</p>
            <div class="token-display">
                https://drive.google.com/file/d/1fbFMYDAU8RqI1fyI8DCb8tOGMT-f7Lyc/view?usp=drivesdk
            </div>
            <button class="btn" onclick="accessUserDrive()">📂 Access Your Drive Files</button>
        </div>
        
        <div id="tokenDisplay" class="token-display" style="display: none;"></div>
    </div>

    <script>
        let currentUser = null;
        let accessToken = null;
        
        // Google OAuth Configuration
        const CLIENT_ID = 'YOUR_GOOGLE_CLIENT_ID'; // Replace with actual client ID
        const SCOPES = 'https://www.googleapis.com/auth/drive.readonly https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email';
        
        function showStatus(message, type = 'success') {
            const status = document.getElementById('status');
            status.textContent = message;
            status.className = `status ${type}`;
            status.style.display = 'block';
            
            setTimeout(() => {
                status.style.display = 'none';
            }, 5000);
        }
        
        function handleCredentialResponse(response) {
            console.log("Encoded JWT ID token: " + response.credential);
            
            // Decode the JWT token
            const payload = JSON.parse(atob(response.credential.split('.')[1]));
            
            currentUser = {
                name: payload.name,
                email: payload.email,
                picture: payload.picture,
                sub: payload.sub
            };
            
            displayUserInfo();
            showStatus('✅ Successfully signed in!');
        }
        
        function displayUserInfo() {
            const userInfo = document.getElementById('userInfo');
            const userName = document.getElementById('userName');
            const userEmail = document.getElementById('userEmail');
            const userAvatar = document.getElementById('userAvatar');
            
            userName.textContent = currentUser.name;
            userEmail.textContent = currentUser.email;
            userAvatar.src = currentUser.picture;
            
            userInfo.classList.add('show');
        }
        
        function manualSignIn() {
            showStatus('🔄 Initiating OAuth flow...', 'success');
            
            // Create OAuth URL
            const authUrl = `https://accounts.google.com/oauth/authorize?` +
                `client_id=${CLIENT_ID}&` +
                `response_type=token&` +
                `scope=${encodeURIComponent(SCOPES)}&` +
                `redirect_uri=${encodeURIComponent(window.location.origin)}`;
            
            // Open popup window
            const popup = window.open(authUrl, 'oauth', 'width=500,height=600');
            
            // Check for authorization code
            const checkClosed = setInterval(() => {
                if (popup.closed) {
                    clearInterval(checkClosed);
                    showStatus('⚠️ OAuth window closed', 'error');
                }
            }, 1000);
        }
        
        function accessDrive() {
            if (!currentUser) {
                showStatus('⚠️ Please sign in first', 'error');
                return;
            }
            
            showStatus('🔍 Accessing Google Drive API...', 'success');
            
            // This would require proper OAuth flow with Drive API access
            const driveApiUrl = 'https://www.googleapis.com/drive/v3/files';
            
            // Simulate API call
            setTimeout(() => {
                showStatus('📁 Drive access would be available with proper OAuth setup', 'success');
            }, 2000);
        }
        
        function accessUserDrive() {
            const fileId = '1fbFMYDAU8RqI1fyI8DCb8tOGMT-f7Lyc';
            const driveUrl = `https://drive.google.com/file/d/${fileId}/view`;
            
            showStatus('🚀 Opening your Google Drive file...', 'success');
            window.open(driveUrl, '_blank');
        }
        
        function signOut() {
            currentUser = null;
            accessToken = null;
            
            const userInfo = document.getElementById('userInfo');
            userInfo.classList.remove('show');
            
            showStatus('👋 Signed out successfully', 'success');
        }
        
        // Initialize
        window.onload = function() {
            showStatus('🌟 Zero-Infinity Recovery System Ready', 'success');
            
            // Apply ZIQY pattern timing
            setTimeout(() => {
                console.log('ZIQY Pattern: Z(Zero) - System initialized');
            }, 1618); // φ milliseconds
            
            setTimeout(() => {
                console.log('ZIQY Pattern: I(Infinity) - Infinite possibilities unlocked');
            }, 3141); // π milliseconds
        };
        
        // Handle URL fragments for OAuth callback
        if (window.location.hash) {
            const params = new URLSearchParams(window.location.hash.substring(1));
            const token = params.get('access_token');
            
            if (token) {
                accessToken = token;
                showStatus('🔑 Access token received!', 'success');
                
                document.getElementById('tokenDisplay').style.display = 'block';
                document.getElementById('tokenDisplay').textContent = 'Access Token: ' + token.substring(0, 50) + '...';
            }
        }
    </script>
</body>
</html>

# Keycloak PHP SSO - Centralized Authentication System

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PHP Version](https://img.shields.io/badge/PHP-%3E%3D8.0-blue)](https://www.php.net/)
[![Keycloak Version](https://img.shields.io/badge/Keycloak-26.2.4-blue)](https://www.keycloak.org/)

A production-ready demonstration of centralized authentication using Keycloak for multiple SaaS applications with Single Sign-On (SSO) and Single Logout (SLO).

## 🎯 Features

### ✅ Implemented Features

- **Single Sign-On (SSO)**: Login once, access all applications
- **Single Logout (SLO)**: Logout from one app logs out from all apps and Keycloak
- **Session Persistence**: Stay logged in across browser restarts (while Keycloak session is valid)
- **Token Handling**: 
  - Authorization Code Flow with PKCE
  - ID Token validation
  - Access Token validation
  - Automatic token refresh
- **Shared Auth Library**: Reusable PHP package for all applications
- **Protected Routes**: Dashboard pages require authentication
- **CSRF Protection**: State parameter validation
- **Security Best Practices**: Secure session handling, HTTP-only cookies

### ❌ NOT Implemented (Handled by Keycloak)

- Password reset logic
- Two-Factor Authentication (2FA/OTP)
- Google OAuth integration
- Brute force protection
- User registration flows

These features are configured and managed entirely in Keycloak, demonstrating proper separation of concerns.

---

## 📁 Project Structure

```
keycloak-php-sso/
├── apps/
│   ├── app_one/                    # First demo application
│   │   ├── includes/
│   │   │   ├── header.php         # Shared header template
│   │   │   └── footer.php         # Shared footer template
│   │   ├── index.php              # Home page (public)
│   │   ├── login.php              # Initiates OIDC login flow
│   │   ├── callback.php           # OAuth callback handler
│   │   ├── dashboard.php          # Protected route (requires auth)
│   │   └── logout.php             # Handles SLO
│   ├── app_two/                    # Second demo application (same structure)
│   │   ├── includes/
│   │   ├── index.php
│   │   ├── login.php
│   │   ├── callback.php
│   │   ├── dashboard.php
│   │   └── logout.php
│   └── app_three/                  # Third demo application (same structure)
│       ├── includes/
│       ├── index.php
│       ├── login.php
│       ├── callback.php
│       ├── dashboard.php
│       └── logout.php
├── src/                            # Shared authentication library
│   ├── KeycloakAuth.php           # Main auth class (OIDC flow, SSO, SLO)
│   └── ConfigLoader.php           # Configuration management
├── keycloak/                       # Keycloak configuration
│   ├── demo-realm.json            # Realm export with 3 clients configured
│   └── KEYCLOAK_SETUP.md          # Keycloak setup instructions
├── docs/                           # Documentation
│   ├── ARCHITECTURE.md            # Architecture overview
│   ├── IMPLEMENTATION.md          # Implementation guide
│   ├── SECURITY.md                # Security checklist
│   └── TESTING.md                 # Testing guide
├── composer.json                   # PHP dependencies
├── .env.example                    # Environment variables template
├── .gitignore                      # Git ignore rules
├── LICENSE                         # MIT License
└── README.md                       # This file
```

---

## 🚀 Quick Start

### Prerequisites

- PHP 8.0 or higher
- Composer
- Java JDK 11+ (for running Keycloak locally)
- Web server (Apache, Nginx, or PHP built-in server)

### 1. Clone Repository

```bash
git clone https://github.com/yourusername/keycloak-php-sso.git
cd keycloak-php-sso
```

### 2. Install Dependencies

```bash
composer install
```

### 3. Configure Environment

```bash
cp .env.example .env
```

Edit `.env` and configure:
- Keycloak URL (default: `http://localhost:8080`)
- Client IDs and secrets (get these from Keycloak after realm import)
- Application URLs

### 4. Start Keycloak (Local Installation)

**Download Keycloak 26.2.4:**

https://www.keycloak.org/archive/downloads-26.2.4.html

Extract and start:

**Windows:**
```bash
cd C:\keycloak-26.2.4
bin\kc.bat start-dev
```

**Linux/Mac:**
```bash
cd ~/keycloak-26.2.4
bin/kc.sh start-dev
```

Keycloak will start on http://localhost:8080

### 5. Configure Keycloak

1. Access Keycloak Admin Console at http://localhost:8080
2. Create admin user (or use default admin/admin)
3. Import realm:
   - Click realm dropdown (top-left) → **Create Realm**
   - Click **Browse** and select `keycloak/demo-realm.json`
   - Click **Create**

### 6. Get Client Secrets

1. Login to Keycloak Admin Console
2. Select "demo-realm" from realm dropdown
3. Go to **Clients** → Select client (e.g., `app-one-client`)
4. Go to **Credentials** tab
5. Copy the **Client Secret**
6. Update `.env` file with the secret
7. Repeat for all three clients

### 7. Start PHP Applications

#### Option A: Using PHP Built-in Server (Recommended for Demo)

Open 3 terminal windows and run:

```bash
# Terminal 1 - App One
cd apps/app_one
php -S localhost:8001

# Terminal 2 - App Two
cd apps/app_two
php -S localhost:8002

# Terminal 3 - App Three
cd apps/app_three
php -S localhost:8003
```

#### Option B: Using Apache/Nginx

Configure virtual hosts for each application pointing to the respective app directories.

### 8. Test the Applications

- **App One**: http://localhost:8001
- **App Two**: http://localhost:8002
- **App Three**: http://localhost:8003

**Demo User Credentials:**
- Username: `demo-user`
- Password: `demo123`

---

## 🧪 Testing SSO and SLO

### Test Single Sign-On (SSO)

1. Open **App One** (http://localhost:8001)
2. Click **Login with Keycloak**
3. Login with demo credentials
4. You'll be redirected to App One dashboard
5. Open **App Two** (http://localhost:8002) in **same browser**
6. You should be **automatically logged in** without entering credentials
7. Open **App Three** (http://localhost:8003)
8. Again, you should be **automatically logged in**

✅ **SSO is working** if you didn't need to re-enter credentials!

### Test Single Logout (SLO)

1. While logged into all three apps
2. Click **Logout** in any app (e.g., App Two)
3. Open **App One** - you should be logged out
4. Open **App Three** - you should be logged out
5. The Keycloak session should also be terminated

✅ **SLO is working** if logout from one app logged you out everywhere!

### Test Session Persistence

1. Login to any app
2. Close the browser completely
3. Reopen browser and go to the app
4. Click **Login** - you should be automatically logged in without entering credentials

✅ **Session persistence is working** if Keycloak remembered you!

---

## 📚 How It Works

### Authentication Flow (OIDC Authorization Code Flow)

```
┌─────────┐                ┌─────────────┐                ┌──────────┐
│ Browser │                │   PHP App   │                │ Keycloak │
└────┬────┘                └──────┬──────┘                └────┬─────┘
     │                            │                            │
     │  1. Visit protected page   │                            │
     ├──────────────────────────>│                            │
     │                            │                            │
     │  2. Redirect to Keycloak   │                            │
     │  (with PKCE challenge)     │                            │
     │<───────────────────────────┤                            │
     │                            │                            │
     │  3. Authorization request  │                            │
     ├────────────────────────────────────────────────────────>│
     │                            │                            │
     │  4. Login form (if needed) │                            │
     │<────────────────────────────────────────────────────────┤
     │                            │                            │
     │  5. User credentials       │                            │
     ├────────────────────────────────────────────────────────>│
     │                            │                            │
     │  6. Redirect with auth code│                            │
     │<────────────────────────────────────────────────────────┤
     │                            │                            │
     │  7. Auth code              │                            │
     ├──────────────────────────>│                            │
     │                            │                            │
     │                            │  8. Exchange code for      │
     │                            │     tokens (with verifier) │
     │                            ├──────────────────────────>│
     │                            │                            │
     │                            │  9. ID Token, Access Token │
     │                            │<───────────────────────────┤
     │                            │                            │
     │                            │ 10. Get user info          │
     │                            ├──────────────────────────>│
     │                            │                            │
     │                            │ 11. User attributes        │
     │                            │<───────────────────────────┤
     │                            │                            │
     │                            │ 12. Create local session   │
     │                            │                            │
     │ 13. Redirect to dashboard  │                            │
     │<───────────────────────────┤                            │
```

### Single Sign-On (SSO) Flow

When accessing a second application:

1. User already has a Keycloak session cookie
2. App redirects to Keycloak for authentication
3. Keycloak detects existing session
4. **Automatically redirects back** with new authorization code (no login form shown)
5. App exchanges code for tokens
6. User is logged into the second app

### Single Logout (SLO) Flow

```
┌─────────┐                ┌─────────────┐                ┌──────────┐
│ Browser │                │   PHP App   │                │ Keycloak │
└────┬────┘                └──────┬──────┘                └────┬─────┘
     │                            │                            │
     │  1. Click logout           │                            │
     ├──────────────────────────>│                            │
     │                            │                            │
     │                            │  2. Clear local session    │
     │                            │                            │
     │  3. Redirect to Keycloak   │                            │
     │    logout endpoint         │                            │
     │    (with id_token_hint)    │                            │
     │<───────────────────────────┤                            │
     │                            │                            │
     │  4. Logout request         │                            │
     ├────────────────────────────────────────────────────────>│
     │                            │                            │
     │                            │  5. Terminate Keycloak     │
     │                            │     session (SSO cookie)   │
     │                            │                            │
     │                            │  6. Call backchannel       │
     │                            │     logout for all apps    │
     │                            │                            │
     │  7. Redirect to post       │                            │
     │     logout URI             │                            │
     │<────────────────────────────────────────────────────────┤
     │                            │                            │
     │  8. Show logged out page   │                            │
     ├──────────────────────────>│                            │
```

---

## 🏗️ Shared Authentication Library

The `KeycloakAuth` class (`src/KeycloakAuth.php`) provides:

### Key Methods

```php
// Initialize authentication
$auth = new KeycloakAuth($config);

// Redirect to Keycloak login
$auth->login();

// Handle OAuth callback
$userInfo = $auth->handleCallback();

// Check authentication status
if ($auth->isAuthenticated()) {
    // User is logged in
}

// Get user information
$userInfo = $auth->getUserInfo();
// Returns: ['sub', 'email', 'name', 'preferred_username', 'email_verified']

// Refresh access token
$auth->refreshAccessToken();

// Perform Single Logout
$auth->logout($postLogoutRedirectUri);

// Require authentication (redirect if not authenticated)
$auth->requireAuth();
```

### Usage Example

```php
<?php
require_once __DIR__ . '/../../vendor/autoload.php';

use KeycloakSSO\ConfigLoader;
use KeycloakSSO\KeycloakAuth;

// Load configuration
$config = ConfigLoader::getAppConfig(
    __DIR__ . '/../../',
    'app_one',
    'http://localhost:8001/callback.php'
);

// Initialize auth
$auth = new KeycloakAuth($config);

// Protect this page
$auth->requireAuth();

// Get user info
$userInfo = $auth->getUserInfo();
echo "Hello, " . $userInfo['name'];
```

---

## 🔒 Security Features

### Implemented Security Measures

1. **PKCE (Proof Key for Code Exchange)**
   - S256 code challenge method
   - Protects against authorization code interception

2. **CSRF Protection**
   - State parameter validation
   - Prevents cross-site request forgery

3. **Secure Sessions**
   - HTTP-only cookies
   - SameSite=Lax
   - Configurable secure flag for HTTPS

4. **Token Validation**
   - JWT signature verification
   - Token expiration checks
   - Automatic token refresh

5. **No Password Storage**
   - All authentication handled by Keycloak
   - No user credentials in application database

See [SECURITY.md](docs/SECURITY.md) for complete security checklist.

---

## 📖 Documentation

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** - System architecture and design decisions
- **[IMPLEMENTATION.md](docs/IMPLEMENTATION.md)** - Step-by-step implementation roadmap
- **[SECURITY.md](docs/SECURITY.md)** - Security checklist and best practices
- **[TESTING.md](docs/TESTING.md)** - Comprehensive testing guide
- **[KEYCLOAK_SETUP.md](keycloak/KEYCLOAK_SETUP.md)** - Keycloak configuration guide

---

## 🛠️ Development

### Project Dependencies

- **[jumbojett/openid-connect-php](https://packagist.org/packages/jumbojett/openid-connect-php)** - OIDC client library
- **[vlucas/phpdotenv](https://packagist.org/packages/vlucas/phpdotenv)** - Environment variable management

### Adding a New Application

1. Copy one of the existing app directories (e.g., `app_one`)
2. Update `includes/header.php` with new branding/colors
3. Create new client in Keycloak
4. Add client credentials to `.env`
5. Update `ConfigLoader` if needed
6. Start the application on a new port

---

## 🐛 Troubleshooting

### Issue: "Client authentication failed"

**Cause**: Client secret mismatch

**Solution**: 
1. Get correct secret from Keycloak Admin Console
2. Update `.env` file
3. Clear browser cache and cookies

### Issue: "Redirect URI mismatch"

**Cause**: Redirect URI not configured in Keycloak

**Solution**:
1. Go to Keycloak → Clients → Your Client
2. Add your callback URL to "Valid Redirect URIs"
3. Add wildcard pattern: `http://localhost:8001/*`

### Issue: Session not persisting

**Cause**: Cookie settings or session timeout

**Solution**:
1. Check `SESSION_TIMEOUT` in `.env`
2. Verify browser accepts cookies
3. Check Keycloak session timeout settings

### Issue: SLO not working

**Cause**: Front-channel logout not enabled

**Solution**:
1. Go to Keycloak → Clients → Your Client
2. Enable "Front-channel Logout"
3. Verify "Valid Post Logout Redirect URIs" is configured

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Development Setup

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly (see TESTING.md)
5. Submit a pull request

---

## 🎓 Learning Resources

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [OAuth 2.0 Specification](https://oauth.net/2/)
- [OpenID Connect Specification](https://openid.net/connect/)
- [PKCE RFC](https://tools.ietf.org/html/rfc7636)

---

## 📧 Support

For questions or issues:
- Open an issue on GitHub
- Check existing documentation in `docs/`
- Check Keycloak setup guide: `keycloak/KEYCLOAK_SETUP.md`

---

## 🏆 Acknowledgments

- [Keycloak](https://www.keycloak.org/) for providing an excellent open-source IAM solution
- [jumbojett/openid-connect-php](https://github.com/jumbojett/OpenID-Connect-PHP) for the OIDC client library

---

**Made with ❤️ for the open-source community**

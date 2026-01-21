# Intune Policy Conflict Analyzer

A powerful, browser-based tool that identifies and helps resolve policy conflicts across your Microsoft Intune environment. Analyzes all policy types including Settings Catalog, Device Configuration, Endpoint Security, Compliance, App Protection, Scripts, and Administrative Templates.

![Intune Conflict Analyzer](https://img.shields.io/badge/Microsoft-Intune-0078D4?style=flat&logo=microsoft)
![License](https://img.shields.io/badge/license-MIT-green)
![No Backend](https://img.shields.io/badge/backend-none%20required-brightgreen)

## 🚀 Quick Start

**New to Azure?** Follow our **[Complete Setup Guide](SETUP-GUIDE.md)** - no Azure experience required!

The guide covers:
1. Creating an Azure App Registration (step-by-step)
2. Deploying to Azure Static Web Apps (free hosting)
3. Setting up a custom domain (optional)
4. Troubleshooting common issues

### For Experienced Users

1. Create an Azure App Registration with these API permissions:
   - `DeviceManagementConfiguration.Read.All`
   - `DeviceManagementManagedDevices.Read.All`
   - `DeviceManagementApps.Read.All`
   - `Group.Read.All`

2. Set authentication platform to **Single-page application (SPA)**

3. Add your hosting URL as a Redirect URI

4. Deploy `index.html` to any static hosting (Azure Static Web Apps, GitHub Pages, etc.)

5. Open the app and enter your Tenant ID and Client ID

## 🎯 Features

- **Comprehensive Policy Analysis** - Covers ALL Intune policy types:
  - Settings Catalog
  - Device Configuration Profiles
  - Endpoint Security Policies
  - Compliance Policies
  - App Protection Policies
  - PowerShell Scripts
  - Windows Update Rings
  - Administrative Templates (ADMX)

- **Smart Conflict Detection** - Identifies:
  - Same settings configured with different values
  - Overlapping group assignments
  - Cross-policy-type conflicts (e.g., Settings Catalog vs Device Config)
  - Security-sensitive setting conflicts

- **Severity Classification** - Conflicts are categorized as:
  - 🔴 **High** - Security settings or cross-policy-type conflicts
  - 🟠 **Medium** - Different values for the same setting
  - 🟡 **Low** - Minor configuration discrepancies

- **Resolution Guidance** - Each conflict includes:
  - Detailed comparison of conflicting policies
  - Assignment group analysis
  - Step-by-step resolution recommendations

- **Privacy-First Design** - Runs entirely in your browser:
  - No backend server required
  - Your data never leaves your machine
  - Uses delegated permissions (user context)

## 🚀 Quick Start

### Option 1: GitHub Pages (Recommended)

1. Fork this repository
2. Enable GitHub Pages in your repo settings (Settings → Pages → Source: main branch)
3. Access at `https://[your-username].github.io/IntuneConflictAnalyzer`

### Option 2: Local Development

```bash
# Clone the repository
git clone https://github.com/JoeryVandenBosch/IntuneConflictAnalyzer.git

# Open directly in browser (no build step needed!)
open index.html
# Or use any local server
npx serve .
```

### Option 3: Azure Static Web Apps

Deploy directly to Azure Static Web Apps for production use:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.StaticApp)

## 🔐 Authentication

This tool uses the **Device Code Flow** for authentication:

1. Click "Start Device Code Flow"
2. Copy the displayed code
3. Navigate to [microsoft.com/devicelogin](https://microsoft.com/devicelogin)
4. Enter the code and sign in with your Microsoft 365 account
5. Grant the requested permissions
6. The app will automatically detect successful authentication

### Required Permissions

The following Microsoft Graph permissions are requested (delegated):

| Permission | Purpose |
|------------|---------|
| `DeviceManagementConfiguration.Read.All` | Read Settings Catalog, Device Config, Endpoint Security |
| `DeviceManagementManagedDevices.Read.All` | Read device compliance and management info |
| `DeviceManagementApps.Read.All` | Read App Protection policies |
| `Group.Read.All` | Resolve group names for assignments |

### Azure AD App Registration (Optional)

The tool uses the Microsoft Graph PowerShell public client ID by default. For production use, create your own app registration:

1. Go to [Azure Portal](https://portal.azure.com) → Azure Active Directory → App registrations
2. Click "New registration"
3. Configure:
   - Name: `Intune Conflict Analyzer`
   - Supported account types: `Accounts in any organizational directory`
   - Redirect URI: Leave blank (using device code flow)
4. Under "Authentication":
   - Enable "Allow public client flows"
5. Under "API Permissions":
   - Add the permissions listed above
   - Grant admin consent (optional, users can consent individually)
6. Copy the Application (client) ID
7. Update `CONFIG.clientId` in `index.html`

## 📊 Understanding Results

### Dashboard Overview

After analysis, you'll see:
- **Total Policies** - Count of all policies analyzed
- **Conflicts Found** - Number of detected conflicts
- **High Severity** - Critical conflicts requiring immediate attention
- **Clean Policies** - Policies with no conflicts

### Conflict Details

Each conflict shows:
- **Setting Name** - The conflicting setting identifier
- **Policies Involved** - All policies configuring this setting
- **Values** - The different values configured
- **Assignments** - Groups targeted by each policy
- **Resolution Steps** - Recommended actions to resolve

### Severity Levels

| Level | Criteria | Action |
|-------|----------|--------|
| 🔴 High | Security settings, cross-policy conflicts | Immediate review required |
| 🟠 Medium | Different values, same policy type | Review and standardize |
| 🟡 Low | Minor discrepancies | Monitor and document |

## 🔧 Configuration

### Customizing the Client ID

Edit the `CONFIG` object in `index.html`:

```javascript
const CONFIG = {
    clientId: 'your-app-registration-client-id',
    scopes: [
        'DeviceManagementConfiguration.Read.All',
        'DeviceManagementManagedDevices.Read.All',
        'DeviceManagementApps.Read.All',
        'Group.Read.All'
    ],
    // ... other settings
};
```

### Adding Custom Policy Types

Extend the `POLICY_TYPES` object to analyze additional policy types:

```javascript
POLICY_TYPES.customPolicy = {
    name: 'Custom Policy',
    endpoint: '/deviceManagement/customEndpoint',
    assignmentsEndpoint: (id) => `/deviceManagement/customEndpoint/${id}/assignments`,
    icon: 'custom-icon',
    color: '#ff6600'
};
```

## 🛠️ Technical Details

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Browser (Client-Side)                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │ Auth Module │  │ Graph Client │  │ Conflict Analyzer │  │
│  │             │  │              │  │                   │  │
│  │ Device Code │→ │ Fetch APIs   │→ │ Compare Settings  │  │
│  │ Token Mgmt  │  │ Pagination   │  │ Detect Overlaps   │  │
│  │ Refresh     │  │ Error Handle │  │ Generate Reports  │  │
│  └─────────────┘  └──────────────┘  └───────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Microsoft Graph API                        │
│                                                              │
│  /deviceManagement/configurationPolicies (Settings Catalog) │
│  /deviceManagement/deviceConfigurations (Device Config)      │
│  /deviceManagement/intents (Endpoint Security)               │
│  /deviceManagement/deviceCompliancePolicies (Compliance)     │
│  /deviceAppManagement/managedAppPolicies (App Protection)    │
│  /deviceManagement/deviceManagementScripts (Scripts)         │
│  /deviceManagement/groupPolicyConfigurations (ADMX)          │
└─────────────────────────────────────────────────────────────┘
```

### Conflict Detection Algorithm

1. **Policy Enumeration** - Fetches all policies from each endpoint
2. **Setting Extraction** - Parses settings from each policy type
3. **Assignment Resolution** - Gets group assignments for each policy
4. **Conflict Identification**:
   - Maps settings by unique identifier
   - Groups policies by setting
   - Compares values across policies
   - Checks for assignment overlap
5. **Severity Classification** - Based on setting type and conflict nature
6. **Resolution Generation** - Creates actionable recommendations

### Token Storage

Tokens are stored in `localStorage` under the key `intune_analyzer_auth`:

```javascript
{
    accessToken: "eyJ...",
    refreshToken: "0.AX...",
    expiresAt: 1704067200000,
    user: { name: "...", upn: "..." }
}
```

Tokens are automatically refreshed when they expire.

## 📝 Export Options

### JSON Report

Click "Export Report" to download a JSON file containing:

```json
{
    "generatedAt": "2024-01-01T12:00:00.000Z",
    "summary": {
        "totalPolicies": 45,
        "conflicts": 12,
        "highSeverity": 3,
        "cleanPolicies": 33
    },
    "conflicts": [
        {
            "settingName": "Firewall Enabled",
            "severity": "high",
            "policies": [...],
            "resolution": [...]
        }
    ]
}
```

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with React 18
- Uses Microsoft Graph API
- IBM Plex Sans & JetBrains Mono fonts
- Inspired by the need for better Intune policy management

## 📞 Support

- 🐛 [Report a bug](https://github.com/JoeryVandenBosch/IntuneConflictAnalyzer/issues)
- 💡 [Request a feature](https://github.com/JoeryVandenBosch/IntuneConflictAnalyzer/issues)
- 📧 Questions? Check out [intunestuff.com](https://intunestuff.com)

---

Made with ❤️ for the Microsoft Intune community

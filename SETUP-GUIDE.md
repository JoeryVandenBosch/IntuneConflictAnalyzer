# Intune Policy Conflict Analyzer - Complete Setup Guide

This guide will walk you through setting up the Intune Policy Conflict Analyzer from scratch. **No Azure experience required** - just follow each step exactly as shown.

---

## 📋 Table of Contents

1. [Create Azure App Registration](#step-1-create-azure-app-registration) (15 minutes)
2. [Deploy to Azure Static Web Apps](#step-2-deploy-to-azure-static-web-apps) (10 minutes)
3. [Configure Custom Domain](#step-3-configure-custom-domain-optional) (15 minutes)
4. [Test Your Deployment](#step-4-test-your-deployment) (5 minutes)
5. [Troubleshooting](#troubleshooting)

---

## Step 1: Create Azure App Registration

An "App Registration" tells Microsoft that your application is allowed to access Intune data. This is required for security.

### 1.1 Open Azure Portal

1. Go to **https://portal.azure.com**
2. Sign in with your Microsoft 365 administrator account
3. You'll see the Azure Portal homepage

![Azure Portal Home](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/azure-portal-overview-portal-callouts.png)

### 1.2 Navigate to App Registrations

1. In the search bar at the top, type **"App registrations"**
2. Click on **"App registrations"** in the search results

![Search App Registrations](https://learn.microsoft.com/en-us/entra/identity-platform/media/quickstart-register-app/portal-02-app-reg-landing.png)

### 1.3 Create New Registration

1. Click the **"+ New registration"** button at the top

2. Fill in the form:
   
   | Field | Value |
   |-------|-------|
   | **Name** | `Intune Policy Conflict Analyzer` |
   | **Supported account types** | Select: **"Accounts in this organizational directory only (Single tenant)"** |
   | **Redirect URI** | Select **"Single-page application (SPA)"** from dropdown, then enter your URL (see below) |

3. For the **Redirect URI**, enter ONE of these depending on where you'll host the app:
   - For Azure Static Web Apps: `https://YOUR-APP-NAME.azurestaticapps.net`
   - For GitHub Pages: `https://YOUR-USERNAME.github.io/IntuneConflictAnalyzer`
   - For custom domain: `https://intuneconflictanalyzer.com`
   - For local testing: `http://localhost:3000`

   > ⚠️ **Important**: You can add more redirect URIs later, so just pick one for now.

4. Click **"Register"**

### 1.4 Copy Your Application Details

After registration, you'll see an overview page. **Copy these two values** - you'll need them later:

| Value | Where to find it | Example |
|-------|------------------|---------|
| **Application (client) ID** | Overview page, top section | `12345678-abcd-1234-efgh-123456789abc` |
| **Directory (tenant) ID** | Overview page, top section | `87654321-dcba-4321-hgfe-987654321cba` |

📝 **Save these somewhere safe!** You'll enter them into the app later.

### 1.5 Configure API Permissions

Now we need to give the app permission to read Intune data.

1. In the left menu, click **"API permissions"**

2. Click **"+ Add a permission"**

3. Click **"Microsoft Graph"**

4. Click **"Delegated permissions"**

5. Search for and check ✅ each of these permissions:
   
   | Permission | What it does |
   |------------|--------------|
   | `DeviceManagementConfiguration.Read.All` | Read device configuration policies |
   | `DeviceManagementManagedDevices.Read.All` | Read managed device information |
   | `DeviceManagementApps.Read.All` | Read app protection policies |
   | `Group.Read.All` | Read group names for assignments |

6. Click **"Add permissions"** at the bottom

7. **IMPORTANT**: Click the **"Grant admin consent for [Your Organization]"** button
   
   > This button requires Global Administrator rights. If you don't have this, ask your IT admin to click it.

8. Verify all permissions show a green checkmark ✅ under "Status"

### 1.6 Configure Authentication Settings

1. In the left menu, click **"Authentication"**

2. Scroll down to **"Single-page application"** section

3. Verify your Redirect URI is listed

4. To add more redirect URIs (e.g., localhost for testing):
   - Click **"+ Add URI"**
   - Enter `http://localhost:3000`
   - Click **"Save"**

5. Scroll down to **"Advanced settings"**

6. Set **"Allow public client flows"** to **"Yes"**

7. Click **"Save"** at the top

---

## Step 2: Deploy to Azure Static Web Apps

Azure Static Web Apps is the easiest way to host this app. It's **free** for basic usage.

### 2.1 Download the Application Files

1. Go to **https://github.com/JoeryVandenBosch/IntuneConflictAnalyzer**

2. Click the green **"Code"** button

3. Click **"Download ZIP"**

4. Extract the ZIP file to a folder on your computer

### 2.2 Create a GitHub Account (if you don't have one)

1. Go to **https://github.com**
2. Click **"Sign up"**
3. Follow the prompts to create a free account

### 2.3 Create a New GitHub Repository

1. Click the **"+"** icon in the top-right corner
2. Click **"New repository"**
3. Fill in:
   - **Repository name**: `IntuneConflictAnalyzer`
   - **Description**: `Intune Policy Conflict Analyzer`
   - **Visibility**: `Public` (required for free Static Web Apps)
4. Click **"Create repository"**

### 2.4 Upload Files to GitHub

1. On your new repository page, click **"uploading an existing file"**
2. Drag and drop ALL files from the extracted ZIP folder
3. Scroll down and click **"Commit changes"**

### 2.5 Create Azure Static Web App

1. Go back to **https://portal.azure.com**

2. In the search bar, type **"Static Web Apps"**

3. Click **"Static Web Apps"**

4. Click **"+ Create"**

5. Fill in the form:

   | Field | Value |
   |-------|-------|
   | **Subscription** | Select your subscription |
   | **Resource Group** | Click "Create new", enter `intune-analyzer-rg` |
   | **Name** | `intune-conflict-analyzer` (must be globally unique) |
   | **Plan type** | `Free` |
   | **Region** | Select the closest to you |
   | **Source** | `GitHub` |

6. Click **"Sign in with GitHub"** and authorize Azure

7. After authorization, fill in:

   | Field | Value |
   |-------|-------|
   | **Organization** | Select your GitHub username |
   | **Repository** | Select `IntuneConflictAnalyzer` |
   | **Branch** | `main` |

8. In **Build Details**:

   | Field | Value |
   |-------|-------|
   | **Build Presets** | `Custom` |
   | **App location** | `/` |
   | **Api location** | Leave empty |
   | **Output location** | Leave empty |

9. Click **"Review + create"**

10. Click **"Create"**

11. Wait 2-3 minutes for deployment

### 2.6 Get Your App URL

1. Once deployment completes, click **"Go to resource"**

2. On the overview page, find the **"URL"** field

3. Copy this URL (e.g., `https://happy-tree-12345.azurestaticapps.net`)

4. **Important**: Go back to your App Registration and add this URL as a Redirect URI:
   - Azure Portal → App registrations → Your app → Authentication
   - Add the URL under "Single-page application"
   - Click Save

---

## Step 3: Configure Custom Domain (Optional)

Want to use `intuneconflictanalyzer.com` instead of the default Azure URL? Follow these steps.

### 3.1 Buy a Domain Name

If you don't own a domain yet:

1. Go to a domain registrar:
   - **Namecheap**: https://namecheap.com
   - **Google Domains**: https://domains.google
   - **GoDaddy**: https://godaddy.com
   - **Cloudflare**: https://cloudflare.com/products/registrar

2. Search for your desired domain (e.g., `intuneconflictanalyzer.com`)

3. Purchase it (typically $10-15/year)

### 3.2 Add Custom Domain to Azure Static Web App

1. In Azure Portal, go to your Static Web App

2. In the left menu, click **"Custom domains"**

3. Click **"+ Add"**

4. Select **"Custom domain on other DNS"**

5. Enter your domain name (e.g., `intuneconflictanalyzer.com`)

6. Click **"Next"**

7. Azure will show you DNS records to add. Copy these values:

   | Type | Name | Value |
   |------|------|-------|
   | CNAME | www | `your-app.azurestaticapps.net` |
   | TXT | @ | `azure-verification-code` |

### 3.3 Configure DNS at Your Domain Registrar

The steps vary by registrar, but here's the general process:

#### For Namecheap:
1. Log in to Namecheap
2. Go to **Domain List** → Click **"Manage"** on your domain
3. Click **"Advanced DNS"**
4. Add the records Azure gave you:
   - Add a **CNAME Record**: Host = `www`, Target = `your-app.azurestaticapps.net`
   - Add a **TXT Record**: Host = `@`, Value = (the verification code from Azure)
   - Add an **ALIAS/ANAME Record** (or URL Redirect): Host = `@`, Target = `www.yourdomain.com`

#### For Cloudflare:
1. Log in to Cloudflare
2. Select your domain
3. Go to **DNS**
4. Add records:
   - **CNAME**: Name = `www`, Target = `your-app.azurestaticapps.net`, Proxy = ON
   - **CNAME**: Name = `@`, Target = `your-app.azurestaticapps.net`, Proxy = ON

#### For GoDaddy:
1. Log in to GoDaddy
2. Go to **My Products** → **DNS**
3. Add records as shown above

### 3.4 Verify Domain in Azure

1. Go back to Azure Portal → Static Web App → Custom domains

2. Wait a few minutes for DNS to propagate (can take up to 24 hours, usually 5-30 minutes)

3. Click **"Validate"** or wait for Azure to auto-validate

4. Once validated, your custom domain is active!

### 3.5 Update App Registration Redirect URI

**Don't forget this step!**

1. Azure Portal → App registrations → Your app → Authentication

2. Add your custom domain:
   - `https://intuneconflictanalyzer.com`
   - `https://www.intuneconflictanalyzer.com`

3. Click **"Save"**

---

## Step 4: Test Your Deployment

### 4.1 Access the Application

1. Open your app URL in a browser:
   - Azure URL: `https://your-app.azurestaticapps.net`
   - Or custom domain: `https://intuneconflictanalyzer.com`

2. You should see the login page

### 4.2 Configure the Application

1. Click **"Configure"** or the settings icon

2. Enter your details:
   - **Tenant ID**: (from Step 1.4)
   - **Client ID**: (from Step 1.4)

3. Click **"Save Configuration"**

### 4.3 Sign In

1. Click **"Sign in with Microsoft"**

2. A popup window will open

3. Select your Microsoft 365 account

4. Review and accept the permissions

5. You should now see the dashboard!

### 4.4 Analyze Policies

1. Wait for the app to fetch your Intune policies

2. Review any conflicts found

3. Click on conflicts to see details and recommendations

---

## Troubleshooting

### "AADSTS50011: The redirect URI does not match"

**Cause**: The URL you're accessing doesn't match any redirect URI in your App Registration.

**Fix**:
1. Go to Azure Portal → App registrations → Your app → Authentication
2. Add the exact URL you're accessing (including `https://` and any path)
3. Click Save
4. Wait 5 minutes and try again

### "AADSTS65001: The user or administrator has not consented"

**Cause**: Admin consent hasn't been granted for the permissions.

**Fix**:
1. Go to Azure Portal → App registrations → Your app → API permissions
2. Click "Grant admin consent for [Your Organization]"
3. If you don't have permission, ask a Global Administrator

### "Popup was blocked"

**Cause**: Your browser is blocking the authentication popup.

**Fix**:
1. Look for a popup blocked notification in your browser's address bar
2. Click it and allow popups for this site
3. Try signing in again

### "No policies found"

**Cause**: The signed-in user might not have Intune permissions.

**Fix**:
1. Ensure you're signed in with an account that has Intune Reader or Administrator role
2. Check that your tenant has Intune policies configured

### Custom domain not working

**Cause**: DNS hasn't propagated yet.

**Fix**:
1. Wait up to 24 hours (usually faster)
2. Use https://dnschecker.org to verify your DNS records
3. Ensure you added the correct records at your domain registrar

### "Failed to fetch policies"

**Cause**: Network or permission issue.

**Fix**:
1. Check browser console (F12) for specific error messages
2. Ensure all API permissions are granted
3. Try signing out and back in

---

## Security Notes

- ✅ This app runs **entirely in your browser** - no data is sent to external servers
- ✅ Authentication uses **Microsoft's secure OAuth 2.0** flow
- ✅ The app only requests **read-only** permissions
- ✅ No client secrets are stored in the browser (we use PKCE flow)
- ✅ You control the App Registration and can revoke access anytime

---

## Getting Help

- 🐛 **Found a bug?** [Open an issue](https://github.com/JoeryVandenBosch/IntuneConflictAnalyzer/issues)
- 💡 **Feature request?** [Open an issue](https://github.com/JoeryVandenBosch/IntuneConflictAnalyzer/issues)
- 📧 **Questions?** Visit [intunestuff.com](https://intunestuff.com)

---

## Quick Reference Card

| What you need | Where to get it |
|---------------|-----------------|
| Tenant ID | Azure Portal → App registrations → Your app → Overview |
| Client ID | Azure Portal → App registrations → Your app → Overview |
| Redirect URIs | Add in: Azure Portal → App registrations → Your app → Authentication |
| API Permissions | Grant in: Azure Portal → App registrations → Your app → API permissions |
| App URL | Azure Portal → Static Web Apps → Your app → Overview |

---

Made with ❤️ for the Microsoft Intune community

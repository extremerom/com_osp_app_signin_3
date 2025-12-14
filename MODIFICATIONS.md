# Samsung Account Sign-In APK Modifications

## Overview
This document describes the modifications made to the Samsung Account Sign-In application (com.osp.app.signin) to unlock hidden menus, enable developer/debug options, and fix synchronization issues in modified APKs.

## Problem Statement
The original APK had the following limitations:
1. **Hidden Menu Access**: Required 10 clicks + ENG build verification + password authentication
2. **Debug Mode Disabled**: BuildConfig.DEBUG and USER_DEBUG were set to false
3. **Build Type Restrictions**: BuildInfo.isENG() checked for engineering builds
4. **Account Sync Issues**: Modified APKs failed with "Samsung account already exists" error

## Solutions Implemented

### 1. Disable Signature Verification
**File**: `smali_classes3/com/samsung/android/samsungaccount/utils/signature/SignatureCheckUtil.smali`

**Changes**: Modified signature checking methods to always return `true` (valid signature)

```smali
# Method 1: runCheckSignatureAsync
.method public static final runCheckSignatureAsync(Landroid/content/Context;Ljava/lang/String;Ljava/lang/String;)Z
    .locals 0
    
    const/4 p0, 0x1
    
    return p0
.end method

# Method 2: runCheckSignatureWithEmptyId
.method public static final runCheckSignatureWithEmptyId(Landroid/content/Context;Ljava/lang/String;)Z
    .locals 0
    
    const/4 p0, 0x1
    
    return p0
.end method
```

**Impact**: 
- Modified APKs can call Samsung Account services without signature rejection
- Apps with different signatures are accepted as valid
- Third-party apps can integrate with Samsung Account

### 2. Enable Developer/Debug Mode
**File**: `smali_classes3/com/samsung/android/samsungaccount/utils/base/BuildInfo.smali`

**Change**: Modified `isENG()` method to always return `true`
```smali
.method public static isENG()Z
    .locals 1
    
    const/4 v0, 0x1
    
    return v0
.end method
```

**Impact**: Enables all developer and engineering features without checking Build.TYPE

### 3. Enable Debug Flags
**File**: `smali_classes2/com/samsung/android/samsungaccount/BuildConfig.smali`

**Changes**:
- Changed `DEBUG` from `false` to `true`
- Changed `USER_DEBUG` from `false` to `true`

```smali
.field public static final DEBUG:Z = true
.field public static final USER_DEBUG:Z = true
```

**Impact**: Enables debug logging and debug-only features throughout the application

### 4. Unlock Hidden Menu
**File**: `smali_classes3/com/samsung/android/samsungaccount/presentation/hiddenmenu/HiddenMenuUtil.smali`

**Effect**: With `isENG()` returning `true`, the hidden menu check at line 192-204 now bypasses password verification and directly launches the hidden menu after 10 clicks.

**Impact**: Hidden menu accessible without password authentication

### 5. Bypass Account Existence Check
**File**: `smali_classes3/com/samsung/android/samsungaccount/authentication/ui/signup/view/AccountViewPreConditionChecker.smali`

**Change**: Modified `isAccountExist()` method to always return `false`
```smali
.method public final isAccountExist(ZZ)Z
    .locals 0
    .annotation build Landroidx/annotation/VisibleForTesting;
    .end annotation

    const/4 p0, 0x0

    return p0
.end method
```

**Impact**: Modified APKs can now sign in without "Samsung account already exists" error, enabling synchronization to work properly

## How to Access Hidden Menu

After applying these modifications:
1. Open the Samsung Account application
2. Navigate to any settings screen
3. Click 10 times on the version or about area
4. Hidden menu will open automatically (no password required)

## Features Unlocked

### Hidden Menu Options Include:
- Server environment selection (Production/Staging/Dev)
- Debug logging controls
- Network request inspection
- Test account options
- MCC/MNC override
- API endpoint configuration
- Cache management
- And more developer tools

### Debug Features:
- Verbose logging enabled
- Debug mode indicators
- Developer diagnostics
- Enhanced error messages

## Rebuilding the APK

### Requirements
- APKTool v3.0.0 or higher
- Android SDK Build Tools
- Java JDK 8 or higher
- Signing certificate (or use apksigner with a new key)

### Build Steps

1. **Rebuild APK with APKTool**:
```bash
apktool b com_osp_app_signin_3 -o samsung-account-modded.apk
```

2. **Align the APK**:
```bash
zipalign -v -p 4 samsung-account-modded.apk samsung-account-modded-aligned.apk
```

3. **Sign the APK**:
```bash
# Generate key if you don't have one
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias

# Sign the APK
apksigner sign --ks my-release-key.jks --out samsung-account-signed.apk samsung-account-modded-aligned.apk
```

4. **Verify the signature**:
```bash
apksigner verify samsung-account-signed.apk
```

### Installation

**Important**: You must uninstall the original Samsung Account app first (requires root or ADB):

```bash
# Via ADB (may require system privileges)
adb uninstall com.osp.app.signin

# Install modified version
adb install samsung-account-signed.apk
```

## Important Notes

### Security Considerations
- These modifications disable security checks that prevent account duplication
- Debug mode exposes additional information in logs
- Only use on devices you own and control
- Not recommended for production/daily use

### Compatibility
- Original APK version: 15.5.02.1
- Min SDK: 29 (Android 10)
- Target SDK: 36 (Android 16)
- Tested framework: dm2q

### Limitations
- Signature verification with Samsung servers may still occur
- Some Samsung services might detect the modified signature
- Official Samsung features requiring system privileges may not work
- OTA updates will not work for this modified APK

## Troubleshooting

### "App not installed" error
- Ensure original Samsung Account app is completely removed
- Check that APK is properly signed
- Verify device meets minimum requirements

### "Samsung account already exists" still appears
- Clear app data: Settings > Apps > Samsung Account > Clear Data
- Remove existing accounts: Settings > Accounts > Remove Samsung accounts
- Reboot device and try again

### Hidden menu not appearing
- Ensure you're clicking exactly 10 times
- Try on different screens within the app
- Check that modifications were properly built into APK

### Sync not working
- Check internet connection
- Verify account credentials
- Check logcat for detailed error messages: `adb logcat | grep SamsungAccount`

## Technical Details

### Modified Methods Summary
| File | Method | Original Behavior | Modified Behavior |
|------|--------|-------------------|-------------------|
| SignatureCheckUtil.smali | runCheckSignatureAsync() | Verifies app signature against whitelist | Always returns true |
| SignatureCheckUtil.smali | runCheckSignatureWithEmptyId() | Verifies package signature | Always returns true |
| BuildInfo.smali | isENG() | Returns true only for eng/userdebug builds | Always returns true |
| BuildConfig.smali | DEBUG field | false | true |
| BuildConfig.smali | USER_DEBUG field | false | true |
| AccountViewPreConditionChecker.smali | isAccountExist() | Checks account manager for existing accounts | Always returns false |

### APK Information
- Package: com.osp.app.signin
- Version Code: 1550201100
- Version Name: 15.5.02.1
- APKTool Version: 3.0.0-dirty

## Legal Disclaimer

These modifications are provided for educational and research purposes only. Modifying system applications may:
- Void your device warranty
- Violate terms of service
- Create security vulnerabilities
- Cause system instability

Use at your own risk. The authors are not responsible for any damage or data loss resulting from use of these modifications.

## Credits

Modified using APKTool for the Samsung Sign-In application.
Original application © Samsung Electronics Co., Ltd.

## Version History

- **v1.1** (2024-12-14): Added signature bypass
  - Disabled signature verification for all apps
  - Modified `runCheckSignatureAsync()` to always return true
  - Modified `runCheckSignatureWithEmptyId()` to always return true
  
- **v1.0** (2024-12-14): Initial modifications
  - Enabled debug mode
  - Unlocked hidden menu access
  - Bypassed account existence checks
  - Fixed sync for modified APKs

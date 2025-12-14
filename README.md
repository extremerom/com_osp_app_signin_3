# Samsung Account Sign-In APK - Modified

[![Version](https://img.shields.io/badge/version-15.5.02.1-blue.svg)](https://github.com/extremerom/com_osp_app_signin_3)
[![Status](https://img.shields.io/badge/status-ready-brightgreen.svg)](https://github.com/extremerom/com_osp_app_signin_3)
[![Language](https://img.shields.io/badge/language-Smali-orange.svg)](https://github.com/extremerom/com_osp_app_signin_3)

> Modified Samsung Account application with unlocked features and disabled signature verification

## 🎯 What's Modified

This repository contains a modified version of Samsung Account Sign-In (com.osp.app.signin) with:

- ✅ **Hidden Menus Unlocked** - Access with 10 clicks, no password
- ✅ **Developer Mode Enabled** - All debug options available
- ✅ **Sync Fixed for Modified APKs** - No "account already exists" error
- ✅ **Signature Verification Disabled** - Works with any signature

## 📋 Quick Start

### Requirements
- APKTool 3.0.0+
- Android SDK Build Tools
- Java JDK 8+
- Signing keystore

### Build Commands

```bash
# 1. Build
apktool b com_osp_app_signin_3 -o samsung-account-mod.apk

# 2. Align
zipalign -v -p 4 samsung-account-mod.apk samsung-account-aligned.apk

# 3. Sign
apksigner sign --ks my-key.jks --out samsung-account-signed.apk samsung-account-aligned.apk

# 4. Install
adb uninstall com.osp.app.signin
adb install samsung-account-signed.apk
```

## 📚 Documentation

- **[MODIFICATIONS.md](MODIFICATIONS.md)** - Complete technical documentation (English)
- **[README_ES.md](README_ES.md)** - Complete user guide (Español)

## 🔧 Technical Changes

### Files Modified (7 total)

| File | Changes | Purpose |
|------|---------|---------|
| `SignatureCheckUtil.smali` | 2 methods → always true | Primary signature bypass |
| `SignatureResult.smali` | isMatched() → always true | Force all results as valid |
| `BuildInfo.smali` | isENG() → always true | Enable developer mode |
| `BuildConfig.smali` | DEBUG/USER_DEBUG → true | Enable debug logging |
| `AccountViewPreConditionChecker.smali` | isAccountExist() → false | Fix sync issues |
| `MODIFICATIONS.md` | New file | Technical documentation |
| `README_ES.md` | New file | Spanish documentation |

### Triple-Layer Signature Bypass

**Layer 1**: Primary check methods return true immediately
```smali
.method public static final runCheckSignatureAsync(...)Z
    const/4 p0, 0x1
    return p0
.end method
```

**Layer 2**: Result validation methods return true
```smali
.method public static final runCheckSignatureWithEmptyId(...)Z
    const/4 p0, 0x1
    return p0
.end method
```

**Layer 3**: Result object forces matched status
```smali
.method public final isMatched()Z
    const/4 v0, 0x1
    return v0
.end method
```

## 🌟 Features Unlocked

### Hidden Menu Access
1. Open Samsung Account app
2. Go to Settings
3. Tap version number 10 times
4. Hidden menu appears automatically (no password!)

### Hidden Menu Includes
- Server environment selection (Production/Staging/Dev)
- Debug logging controls
- Network request inspection  
- Test account options
- MCC/MNC override
- API endpoint configuration
- Cache management
- Developer diagnostics

### Debug Features
- Verbose logging enabled
- Debug mode indicators
- Enhanced error messages
- Network inspection
- Performance metrics

## ⚙️ APK Information

- **Package**: com.osp.app.signin
- **Version**: 15.5.02.1 (1550201100)
- **Min SDK**: 29 (Android 10)
- **Target SDK**: 36
- **APKTool**: 3.0.0-dirty

## 🔐 Security Notes

⚠️ **Important**: These modifications intentionally disable security checks

### What's Disabled
- ✗ Signature verification at all levels
- ✗ App authentication checks
- ✗ Duplicate account detection
- ✗ Blocklist enforcement

### Use Cases
- ✓ Development and testing
- ✓ Research purposes
- ✓ Modified APK compatibility
- ✓ Educational purposes

### Recommendations
- Only use on devices you control
- Not for production use with sensitive data
- Keep backup of original APK
- Understand security implications

## 🛠️ Troubleshooting

### "App not installed"
- Remove original Samsung Account app first
- Verify APK is properly signed
- Check device meets minimum requirements (Android 10+)

### "Account already exists" still appears
```bash
# Clear app data
adb shell pm clear com.osp.app.signin

# Remove all Samsung accounts
# Settings > Accounts > Remove Samsung Account

# Reboot and reinstall
adb reboot
```

### Hidden menu not appearing
- Ensure exactly 10 taps
- Try different screens in the app
- Verify modifications were built into APK

### Sync not working
- Check internet connection
- Verify account credentials
- Check logs: `adb logcat | grep SamsungAccount`

## 📊 Statistics

- **Lines of code changed**: ~140
- **Lines of documentation added**: ~640
- **Files modified**: 7
- **Commits**: 5
- **Build size**: ~50MB (original APK)

## 🎯 Compatibility

### Tested On
- Samsung Galaxy S series (S20+)
- Samsung Galaxy Note series  
- Samsung Galaxy A series

### Requirements
- Android 10 or higher
- Ability to install from unknown sources
- Preferably root access (for system app removal)

## 📜 Version History

### v1.2 (2024-12-14) - Current
- ✅ Complete signature bypass (triple layer)
- ✅ SignatureResult.isMatched() always true
- ✅ Prevents error SAC_0205 at all levels
- ✅ AIDL services accept all apps

### v1.1 (2024-12-14)
- ✅ Signature verification disabled
- ✅ runCheckSignatureAsync() bypass
- ✅ runCheckSignatureWithEmptyId() bypass

### v1.0 (2024-12-14)
- ✅ Debug mode enabled
- ✅ Hidden menu unlocked
- ✅ Account check bypassed
- ✅ Sync fixed for modified APKs

## 🤝 Contributing

This is a research/educational project. Contributions welcome for:
- Documentation improvements
- Compatibility fixes
- Additional feature unlocks
- Build script improvements

## ⚖️ Legal

**Disclaimer**: This project is for educational and research purposes only.

- Original app © Samsung Electronics Co., Ltd.
- Modifications may violate Samsung's Terms of Service
- Use at your own risk
- Authors not responsible for any damages

## 📞 Support

- **Issues**: Check existing issues before creating new ones
- **Documentation**: Read MODIFICATIONS.md for technical details
- **Spanish Guide**: See README_ES.md for Spanish documentation

## 🔗 Links

- [APKTool](https://ibotpeaches.github.io/Apktool/)
- [Android Developer Tools](https://developer.android.com/studio/command-line)
- [Smali/Baksmali](https://github.com/JesusFreke/smali)

---

**Made for educational purposes** • **Use responsibly** • **v1.2**

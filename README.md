# Lab Report: Frida Instrumentation and Security Analysis
**Prepared by:** daber ziyad

## 1. Overview
This report documents the setup and usage of Frida for dynamic instrumentation on Android devices, focusing on security analysis of native and Java layers.

## 2. Environment Setup
### 2.1 Client Installation
- **Tools:** `frida`, `frida-tools` (Python library and CLI).
- **Verification:** Verified via `frida --version` and `pip install --upgrade frida frida-tools`.

### 2.2 Android Server Deployment
- **ADB Configuration:** Platform Tools installed and USB Debugging enabled.
- **Architecture Identification:** Determined CPU ABI using `adb shell getprop ro.product.cpu.abi`.
- **Server Deployment:** 
  - Downloaded compatible `frida-server`.
  - Pushed to `/data/local/tmp/` via `adb push`.
  - Set execution permissions: `chmod 755`.
  - Launched server with `adb shell /data/local/tmp/frida-server -l 0.0.0.0`.
- **Connectivity:** Established via `adb forward tcp:27042 tcp:27042` and `adb forward tcp:27043 tcp:27043`.

## 3. Dynamic Analysis and Instrumentation

### 3.1 Basic Validation
- **Process Listing:** Confirmed connectivity using `frida-ps -U` and `frida-ps -Uai`.
- **Initial Injection:** Validated `Java.perform` functionality with a `hello.js` script.
- **Native Hooking:** Validated native interception by hooking the `recv` function in `libc.so`.

### 3.2 Security Exploration (Native Layer)
Utilized the Frida interactive console to inspect the process:
- **Architecture:** Checked via `Process.arch`.
- **Modules:** Identified loaded libraries using `Process.enumerateModules()`, specifically filtering for `ssl`, `crypto`, and `boring` to find encryption libraries.
- **Critical Functions:** Located memory addresses for sensitive `libc.so` exports: `connect`, `send`, `recv`, `open`, and `read`.
- **Memory Mapping:** Analyzed executable memory regions using `Process.enumerateRanges('r-x')`.

### 3.3 Security Exploration (Java Layer)
Extended analysis to the Android Runtime:
- **Runtime Availability:** Verified via `Java.available`.
- **Class Enumeration:** Filtered loaded classes for keywords like `security`, `crypto`, `prefs`, and `sqlite`.
- **Behavioral Hooking:**
  - **SharedPreferences:** Instrumented `getString` and `putBoolean` to monitor local configuration and session tokens.
  - **SQLite:** Hooked `execSQL` and `rawQuery` to observe local database interactions.
  - **Anti-Debugging:** Monitored `android.os.Debug.isDebuggerConnected()` to detect debugger checks.
  - **System Execution:** Intercepted `java.lang.Runtime.exec()` to identify shell commands executed by the app.

## 4. Summary of Security Insights
- **Network Activity:** By hooking `connect`, `send`, and `recv`, communication patterns and remote endpoints can be identified.
- **Data Storage:** Monitoring `SharedPreferences` and `SQLite` reveals how sensitive data is stored and retrieved.
- **Environment Awareness:** Hooking `Runtime.exec` and `Debug` classes allows for the detection of root-detection or anti-debugging mechanisms.
- **Crypto Analysis:** Identifying `libssl.so` or `libcrypto.so` helps target the analysis of encrypted traffic.

## 5. Conclusion
The environment is fully configured for dynamic analysis. The ability to transition between native hooking (via `Interceptor`) and Java hooking (via `Java.use`) provides a comprehensive toolkit for analyzing the security posture of Android applications.

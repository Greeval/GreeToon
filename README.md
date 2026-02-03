# ⚡ GreeToon - Comic Downloader

**GreeToon** is a powerful, terminal-based (CLI) application designed to automate the process of downloading comic chapters from various websites. Built with **Node.js** and **Puppeteer**, it simulates a real user browser to bypass basic anti-bot protections and render dynamic content.

Developed by **Greeval Studio**.

## ✨ Key Features

* **🚀 Standalone Application:** Runs as a portable `.exe` file. No need to install Node.js or NPM.
* **📦 Batch Downloading:** Paste multiple links at once or load them from a `list.txt` file.
* **🧠 Smart Rendering:** Uses Puppeteer (Chrome) to handle JavaScript-heavy sites and lazy-loading images.
* **📜 Auto-Scroll Engine:** Automatically scrolls down pages to ensure all images are loaded before downloading.
* **🔑 Login Mode:** specialized mode to log in to premium sites (Naver, Kakao, Webtoon) manually before scraping.
* **🛡️ Anti-Crash System:** Built-in manual Chrome detection to prevent errors on different Windows environments.
* **📂 Organized Output:** Automatically creates folders for each chapter and saves images in sequence (001.jpg, 002.jpg...).

## 🛠️ Requirements

Since this is a lightweight wrapper around Puppeteer Core, you strictly need:

1.  **Windows OS** (Windows 10/11 Recommended).
2.  **Google Chrome** installed on your system.
    * *GreeToon uses your installed Chrome browser to perform the scraping.*

## 📥 Installation & Usage

### Method 1: The Easy Way (Pre-built .exe)
1.  Download the latest `GreeToon.exe` from the [Releases]([your-github-release-link-here](https://github.com/Greeval/GreeToon/releases/tag/GreeToon)) page.
2.  Make sure you have Google Chrome installed.
3.  Double-click `GreeToon.exe`.
4.  Choose **[1] Download Chapter** and paste your links!

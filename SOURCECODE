const puppeteer = require('puppeteer-core');
const fs = require('fs');
const path = require('path');
const os = require('os');
const readline = require('readline');
const { exec } = require('child_process');

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

const authPath = path.join(process.cwd(), 'Greeval_Auth'); 
let chromePath = null; 

function findChrome() {
    const possiblePaths = [
        "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
        "C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe",
        "C:\\Users\\" + os.userInfo().username + "\\AppData\\Local\\Google\\Chrome\\Application\\chrome.exe"
    ];

    for (const p of possiblePaths) {
        if (fs.existsSync(p)) return p;
    }
    return null;
}

function askQuestion(query) {
    return new Promise(resolve => rl.question(query, resolve));
}

function openUrlInBrowser(url) {
    exec(`start ${url}`, (err) => {
        if (err) console.log(`👉 Manual Link: ${url}`);
    });
}

function toTitleCase(str) {
    if (!str) return "Unknown";
    try { str = decodeURIComponent(str); } catch(e) {}
    let safeStr = str.replace(/[<>:"/\\|?*]/g, '');
    return safeStr.replace(/\s+/g, ' ').trim();
}
function getSavePath(url) {
    let baseDir;
    try {
        const home = os.homedir();
        const docPath = path.join(home, 'Documents');
        if (fs.existsSync(docPath)) baseDir = docPath;
        else baseDir = path.join(home, 'Downloads');
    } catch (e) { baseDir = path.join(os.homedir(), 'Downloads'); }

    let seriesTitle = "Unknown Series";
    let chapterNum = "XX";
    
    try {
        const urlObj = new URL(url);
        const pathSegments = urlObj.pathname.split('/').filter(s => s.length > 0);
        const queryParams = urlObj.searchParams;

        if (url.includes('vortexscans')) {
            const seriesIdx = pathSegments.indexOf('series');
            const chapIdx = pathSegments.indexOf('chapter');
            if (seriesIdx !== -1 && pathSegments[seriesIdx + 1]) seriesTitle = pathSegments[seriesIdx + 1].replace(/-/g, ' ');
            if (chapIdx !== -1 && pathSegments[chapIdx + 1]) chapterNum = pathSegments[chapIdx + 1];
        }
        else if (url.includes('asuracomic')) {
            const seriesIdx = pathSegments.indexOf('series');
            if (seriesIdx !== -1) {
                let rawTitle = pathSegments[seriesIdx + 1]; 
                rawTitle = rawTitle.replace(/-[a-f0-9]{6,}$/i, ''); 
                seriesTitle = rawTitle.replace(/-/g, ' ');
            }
            const chapIdx = pathSegments.indexOf('chapter');
            if (chapIdx !== -1) chapterNum = pathSegments[chapIdx + 1];
        }
        else if (url.includes('page.kakao.com')) {
             const contentIdx = pathSegments.indexOf('content');
             let seriesId = "UnknownID";
             if (contentIdx !== -1 && pathSegments[contentIdx + 1]) seriesId = pathSegments[contentIdx + 1];
             seriesTitle = `Kakao Series ${seriesId}`;
             const viewerIdx = pathSegments.indexOf('viewer');
             if (viewerIdx !== -1 && pathSegments[viewerIdx + 1]) chapterNum = pathSegments[viewerIdx + 1];
        }
        else if (url.includes('twbzmg') || url.includes('baozimh')) {  
            const chapIdx = pathSegments.indexOf('chapter');
            if (chapIdx !== -1 && pathSegments[chapIdx + 1]) seriesTitle = pathSegments[chapIdx + 1]; 
            else seriesTitle = "Baozi Manhua";
            seriesTitle = seriesTitle.replace(/-/g, ' ');
            const lastSeg = pathSegments[pathSegments.length - 1]; 
            const match = lastSeg.match(/_(\d+)/); 
            if (match) chapterNum = match[1];
            else chapterNum = lastSeg.replace(/[^0-9]/g, '');
        }
        else if (url.includes('naver.com')) {
            const tId = queryParams.get('titleId');
            const no = queryParams.get('no');
            seriesTitle = `Naver Webtoon ${tId || 'Unknown'}`;
            chapterNum = no || 'XX';
        }
        else if (url.includes('webtoons.com')) {
            if (pathSegments.length >= 3) {
                seriesTitle = pathSegments[2].replace(/-/g, ' ');
                const epSegment = pathSegments.find(s => s.includes('episode'));
                chapterNum = epSegment ? epSegment.replace('episode-', '') : 'XX';
            }
        }
        else if (pathSegments.includes('reader') && pathSegments.includes('en')) {
            const enIdx = pathSegments.indexOf('en');
            let rawTitle = pathSegments[enIdx + 1] || 'Unknown';
            const match = rawTitle.match(/(.+)-chapter-(\d+)/);
            if (match) { seriesTitle = match[1].replace(/-/g, ' '); chapterNum = match[2]; } 
            else { seriesTitle = rawTitle.replace(/-/g, ' '); }
        }
        else if (pathSegments.includes('manga')) {
            const mangaIdx = pathSegments.indexOf('manga');
            if (pathSegments[mangaIdx + 1]) seriesTitle = pathSegments[mangaIdx + 1].replace(/-/g, ' ');
            const lastSeg = pathSegments[pathSegments.length - 1]; 
            const numMatch = lastSeg.match(/(\d+(\.\d+)?)/); 
            if (numMatch) chapterNum = numMatch[0];
        }
        else {
             const lastSegment = pathSegments[pathSegments.length - 1];
             const match = lastSegment.match(/(.+)-chapter-(\d+)/);
             
             if (match) { 
                 seriesTitle = match[1].replace(/-/g, ' '); 
                 chapterNum = match[2]; 
             } 
             else if (pathSegments.length > 1) {
                seriesTitle = pathSegments[pathSegments.length - 2].replace(/-/g, ' ');
                chapterNum = pathSegments[pathSegments.length - 1].replace(/[^0-9.]/g, '');
            } else { 
                seriesTitle = lastSegment.replace(/-/g, ' '); 
            }
        }
    } catch (e) { console.log("⚠️ Naming Error, using default."); }

    if (chapterNum.endsWith('.')) chapterNum = chapterNum.slice(0, -1);
    seriesTitle = toTitleCase(seriesTitle);
    return path.join(baseDir, 'Greeval Studio', 'Image Downloader', seriesTitle, `Chapter ${chapterNum}`);
}
async function runLoginMode() {
    console.log("\n🚀 OPENING BROWSER FOR LOGIN...");
    console.log("-----------------------------------------");
    console.log("👉 Please login to Kakao, Naver, Webtoon, etc.");
    console.log("👉 Don't forget to check 'Keep me logged in' (Remember Me).");
    console.log("👉 Once finished, CLOSE THE BROWSER manually.");
    console.log("-----------------------------------------\n");

    const safeUserDataDir = path.join(process.env.APPDATA || os.tmpdir(), 'GreeToon_Data');

    const browser = await puppeteer.launch({
        headless: false,
        executablePath: chromePath,
        userDataDir: safeUserDataDir,
        defaultViewport: null, 
        args: [
            '--start-maximized', 
            '--no-sandbox',
            '--disable-setuid-sandbox',
            '--disable-infobars',
            '--window-position=0,0',
            '--ignore-certificate-errors',
            '--ignore-certificate-errors-spki-list'
        ]
    });

    const pages = await browser.pages();
    const page = pages[0];
    
    await page.goto('https://m.comic.naver.com/'); 
    await new Promise(resolve => browser.on('disconnected', resolve));
    console.log("✅ Browser closed. Login data saved!\n");
}
async function runDonateMode() {
    while(true) {
        console.clear();
        printDivider();
        console.log("\n" + centerText(C.yellow + C.bright + "☕  SUPPORT GREEVAL STUDIO  ☕" + C.reset));
        console.log(centerText(C.cyan + "=".repeat(30) + C.reset)); 
        
        console.log("\n" + centerText("Thank you for supporting this project!"));
        console.log(centerText("Your support keeps the updates coming.\n"));
        const menuLines = [
            `${C.green}[1]${C.reset}  Sociabuzz (QRIS / E-Wallet)  `,
            `${C.green}[2]${C.reset}  Trakteer.id                  `,
            `${C.green}[3]${C.reset}  Ko-fi               `,
            `${C.red}[4]  Back to Main Menu            ${C.reset}`
        ];
        console.log(centerBlock(menuLines));
        
        console.log("\n");
        console.log(centerText(C.gray + "Copyright © 2026 Greeval Studio" + C.reset));
        printDivider();

        const promptText = centerText(C.bright + "👉 Select (1-4): " + C.reset).trimEnd();
        const choice = await askQuestion("\n" + promptText + " ");
        if (choice === '1') {
            console.log(centerText("\n🚀 Opening Sociabuzz..."));
            openUrlInBrowser("https://sociabuzz.com/greeval/donate"); 
            await askQuestion("");
        } 
        else if (choice === '2') {
            console.log(centerText("\n🚀 Opening Trakteer..."));
            openUrlInBrowser("https://trakteer.id/greeval/gift");
            await askQuestion("");
        }
        else if (choice === '3') {
            console.log(centerText("\n🚀 Opening Ko-Fi..."));
            openUrlInBrowser("https://ko-fi.com/greeval");
            await askQuestion("");
        }
        else if (choice === '4') {
            break; 
        }
    }
}

async function runDownloadMode() {
    console.log("\n👇 INPUT MODE:");
    console.log("   • Paste links one by one (Press Enter after each link)");
    console.log("   • OR just PASTE your whole list here");
    console.log("   • Press ENTER on an empty line to START downloading");
    console.log("   • (Press ENTER immediately on Link 1 to load 'list.txt')");
    console.log("-------------------------------------------------------");

    let links = [];
    let counter = 1;
    while (true) {
        const promptText = counter === 1 ? "🔗 Link 1: " : `🔗 Link ${counter}: `;
        const input = await askQuestion(promptText);

        if (input.trim() === "") {
            if (counter === 1) {
                const listPath = path.join(process.cwd(), 'list.txt');
                if (fs.existsSync(listPath)) {
                    console.log(`\n📂 Loading 'list.txt'...`);
                    const fileContent = fs.readFileSync(listPath, 'utf-8');
                    links = fileContent.split(/\r?\n/).filter(u => u.includes('http'));
                } else {
                    console.log(`❌ 'list.txt' not found!`);
                    return;
                }
            } 
            break; 
        }

        if (input.includes("http")) {
            const cleanLink = input.trim();
            if (!links.includes(cleanLink)) {
                links.push(cleanLink);
                counter++;
            }
        }
    }

    if (links.length === 0) {
        console.log("⚠️ No valid links collected!");
        return;
    }

    console.log(`\n📦 Ready to process ${links.length} Chapters.`);
    console.log("🚀 Starting BATCH DOWNLOAD (RAM OPTIMIZED)...");

    let browser = null;

    try {
        const safeUserDataDir = path.join(process.env.APPDATA || os.tmpdir(), 'GreeToon_Data');

        browser = await puppeteer.launch({
            headless: false,
            executablePath: chromePath,
            userDataDir: safeUserDataDir,
            defaultViewport: null, 
            args: [
                '--start-maximized',
                '--disable-blink-features=AutomationControlled',
                '--no-sandbox',
                '--disable-setuid-sandbox',
                '--disable-infobars',
                '--window-position=0,0',
                '--ignore-certificate-errors',
                '--ignore-certificate-errors-spki-list'
            ]
        });

        const pages = await browser.pages();
        const page = pages[0];

        await page.evaluateOnNewDocument(() => {
            Object.defineProperty(navigator, 'webdriver', { get: () => false });
        });

        const mainTarget = page.target();
        browser.on('targetcreated', async (target) => {
            if (target === mainTarget) return;
            try { const newPage = await target.page(); if (newPage && newPage !== page) await newPage.close(); } catch(e) {}
        });
        for (let i = 0; i < links.length; i++) {
            const url = links[i];
            const imageTempMap = new Map(); 
            console.log(`\n=========================================`);
            console.log(`⏳ PROCESSING LINK ${i + 1} of ${links.length}`);
            console.log(`🌍 URL: ${url}`);
            
            const outputDir = getSavePath(url);
            if (!fs.existsSync(outputDir)) fs.mkdirSync(outputDir, { recursive: true });
            console.log(`📂 Save Path: ${outputDir}`);

            await page.removeAllListeners('response'); 
            await page.setRequestInterception(false); 
            
            page.on('response', async (response) => {
                const reqUrl = response.url();
                const type = response.request().resourceType();
                if (type === 'image') {
                    const contentType = (response.headers()['content-type'] || '').toLowerCase();
                    if (contentType.includes('gif')) return; 
                    if (imageTempMap.has(reqUrl)) return; 
                    
                    try {
                        const buffer = await response.buffer();
                        if (buffer.length > 5000) { 
                            let ext = '.jpg'; 
                            if (contentType.includes('webp')) ext = '.webp';
                            if (contentType.includes('png')) ext = '.png'; 
                            const tempName = `temp_${Date.now()}_${Math.floor(Math.random() * 1000)}${ext}`;
                            const tempPath = path.join(outputDir, tempName);
                            fs.writeFileSync(tempPath, buffer);
                            imageTempMap.set(reqUrl, { path: tempPath, extension: ext });
                            process.stdout.write("⚡"); 
                        }
                    } catch (err) {}
                }
            });
            try {
                let attempts = 0;
                let isSuccess = false;

                while (attempts < 3 && !isSuccess) {
                    try {
                        attempts++;
                        if(attempts > 1) console.log(`🔄 Attempt ${attempts}...`);
                        await page.goto(url, { waitUntil: 'domcontentloaded', timeout: 90000 });
                        
                        let title = await page.title();
                        if (title.includes("Just a moment") || title.includes("human") || title.includes("Cloudflare")) {
                            console.log("⚠️ Cloudflare detected! Waiting a bit longer...");
                            await new Promise(r => setTimeout(r, 8000)); 
                        }
                        isSuccess = true; 
                    } catch (e) {
                        console.log(`⚠️ Connection failed: ${e.message}`);
                        if (attempts >= 3) throw new Error("Failed after 3 attempts.");
                        console.log("☕ Waiting 5 seconds...");
                        await new Promise(r => setTimeout(r, 5000));
                    }
                }

                console.log("⏳ Waiting for content to stabilize...");
                await new Promise(r => setTimeout(r, 4000));

                console.log("📜 Starting Scroll...");
                let stuckCounter = 0;
                let lastScrollY = 0;

                while (true) {
                    try {
                        await page.evaluate('window.scrollBy(0, 800)');
                        await new Promise(r => setTimeout(r, 1000));  
                        const currentScrollY = await page.evaluate('window.scrollY');
                        if (currentScrollY === lastScrollY) {
                            stuckCounter++;
                            process.stdout.write("."); 
                            if(stuckCounter === 3) await page.evaluate('window.scrollBy(0, -300)');
                            if (stuckCounter >= 8) break; 
                        } else {
                            stuckCounter = 0;
                            lastScrollY = currentScrollY;
                        }
                    } catch (err) { break; }
                }

                console.log("\n📥 Processing images...");
                const orderedUrls = await page.evaluate(() => {
                    return Array.from(document.querySelectorAll('img')).map(img => img.src);
                });

                let savedCount = 0;
                for (const pageUrl of orderedUrls) {
                    if (imageTempMap.has(pageUrl)) {
                        savedCount++;
                        const tempInfo = imageTempMap.get(pageUrl);
                        const finalName = `${String(savedCount).padStart(3, '0')}${tempInfo.extension}`;
                        const finalPath = path.join(outputDir, finalName);
                        try {
                            if (fs.existsSync(tempInfo.path)) {
                                fs.renameSync(tempInfo.path, finalPath);
                                imageTempMap.delete(pageUrl);
                            }
                        } catch (e) {
                            console.log(`⚠️ Rename error: ${e.message}`);
                        }
                    }
                }
                for (const remaining of imageTempMap.values()) {
                    try {
                        if (fs.existsSync(remaining.path)) fs.unlinkSync(remaining.path);
                    } catch(e) {}
                }

                console.log(`✅ Success! ${savedCount} images saved.`);
                
            } catch (errLoop) {
                console.log(`❌ Failed on this link: ${errLoop.message}`);
            }

            if (i < links.length - 1) {
                console.log("☕ Resting for 3 seconds...");
                await new Promise(r => setTimeout(r, 3000));
            }
        } 
        console.log("\n🎉 BATCH DOWNLOAD COMPLETE!");

    } catch (e) {
        console.error("\n❌ GLOBAL ERROR:", e.message);
    } finally {
        if(browser) {
            console.log("🔒 Closing Browser...");
            await browser.close();
        }
    }
}
const C = {
    reset: "\x1b[0m",
    bright: "\x1b[1m",
    cyan: "\x1b[36m",
    green: "\x1b[32m",
    yellow: "\x1b[33m",
    red: "\x1b[31m",
    magenta: "\x1b[35m",
    white: "\x1b[37m",
    gray: "\x1b[90m",
    blue: "\x1b[34m"
};

function getTermWidth() {
    return process.stdout.columns || 80; 
}
function centerText(text) {
    const width = getTermWidth();
    const cleanText = text.replace(/\x1b\[[0-9;]*m/g, ""); 
    if (cleanText.length >= width) return text;
    
    const padding = Math.floor((width - cleanText.length) / 2);
    return " ".repeat(padding) + text;
}
function centerBlock(linesArray) {
    const width = getTermWidth();
    let maxLen = 0;
    const cleanLines = linesArray.map(line => {
        const clean = line.replace(/\x1b\[[0-9;]*m/g, "");
        if (clean.length > maxLen) maxLen = clean.length;
        return { original: line, cleanLen: clean.length };
    });
    const leftPad = Math.floor((width - maxLen) / 2);
    const padStr = " ".repeat(leftPad > 0 ? leftPad : 0);
    return linesArray.map(line => padStr + line).join("\n");
}

function printDivider() {
    const width = getTermWidth();
    const line = C.cyan + "=".repeat(width) + C.reset;
    console.log(line);
}
function showHeader() {
    console.clear();
    console.log("\n");
    
    const asciiArt = [
        C.cyan + C.bright + `  ██████╗ ██████╗ ███████╗███████╗████████╗ ██████╗  ██████╗ ███╗   ██╗` + C.reset,
        C.cyan + C.bright + ` ██╔════╝ ██╔══██╗██╔════╝██╔════╝╚══██╔══╝██╔═══██╗██╔═══██╗████╗  ██║` + C.reset,
        C.cyan + C.bright + ` ██║  ███╗██████╔╝█████╗  █████╗     ██║   ██║   ██║██║   ██║██╔██╗ ██║` + C.reset,
        C.cyan + C.bright + ` ██║   ██║██╔══██╗██╔══╝  ██╔══╝     ██║   ██║   ██║██║   ██║██║╚██╗██║` + C.reset,
        C.cyan + C.bright + ` ╚██████╔╝██║  ██║███████╗███████╗   ██║   ╚██████╔╝╚██████╔╝██║ ╚████║` + C.reset,
        C.cyan + C.bright + `  ╚═════╝ ╚═╝  ╚═╝╚══════╝╚══════╝   ╚═╝    ╚═════╝  ╚═════╝ ╚═╝  ╚═══╝` + C.reset
    ];

    console.log(centerBlock(asciiArt));
    console.log("\n" + centerText(`${C.yellow}⚡ COMIC DOWNLOADER by GREEVAL STUDIO ⚡${C.reset}`));
    printDivider();
}

(async () => {
    process.stdout.write('\x1b]0;GreeToon - Comic Downloader by Greeval Studio\x07');

    chromePath = findChrome();
    
    if (!chromePath) {
        console.error(centerText(C.red + "❌ CRITICAL ERROR: Google Chrome not found!" + C.reset));
        await askQuestion("\nPress ENTER to exit...");
        process.exit(1);
    }

    while (true) {
        showHeader();
        console.log("\n");

        const mainMenuLines = [
            `${C.green}[1]${C.reset}  📥  DOWNLOAD CHAPTER  ${C.gray}(Paste Link / Batch)     ${C.reset}`,
            `${C.green}[2]${C.reset}  🔑  LOGIN MODE        ${C.gray}(Naver / Kakao / Webtoon)${C.reset}`,
            `${C.green}[3]${C.reset}  ❤️  DONATE            ${C.gray}(Support Admin)          ${C.reset}`,
            `${C.red}[4]  🚪  EXIT${C.reset}                                          `
        ];

        console.log(centerBlock(mainMenuLines));

        console.log("\n");
        console.log(centerText(C.gray + "Copyright © 2026 Greeval Studio" + C.reset));
        printDivider();
        
        const promptText = centerText(C.bright + "👉 Select Menu (1-4): " + C.reset).trimEnd();
        const choice = await askQuestion("\n" + promptText + " ");

        if (choice === '1') {
            await runDownloadMode(); 
        } else if (choice === '2') {
            await runLoginMode();
        } else if (choice === '3') {
            await runDonateMode(); 
        } else if (choice === '4') {
            console.log(centerText(C.magenta + "\n👋 Thanks for using GreeToon! See you next time." + C.reset));
            await new Promise(r => setTimeout(r, 1500)); 
            rl.close();
            process.exit(0);
        } else {
            console.log(centerText(C.red + "⚠️  Invalid choice!" + C.reset));
            await new Promise(r => setTimeout(r, 1000));
        }
    }
})();

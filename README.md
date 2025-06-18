# x-communities-n8n-puppeteer
An automated workflow built with n8n and Puppeteer to post content into X.com (Twitter) Communities using login cookies. Includes structured input handling, cookie reuse, human-like behavior simulation, and dynamic community selection.

# 🐦 X Community Auto Poster (n8n + Puppeteer)

This repository provides a `n8n` workflow that automates the process of posting content to **X.com (formerly Twitter)** Communities using Puppeteer, with login session preserved via cookies.

Ideal for content creators, social media teams, or automation engineers looking to publish posts directly to Communities on X.

---

## 🚀 Features

✅ Post content to any X Community  
✅ Use stored cookies (no login required each time)  
✅ Simulate human behavior with randomized delays  
✅ Supports dynamic input (content + community)  
✅ Fully compatible with Puppeteer Node in n8n  
✅ Bypass interface challenges like multiple "Post" buttons  
✅ Minimal setup with cookie reuse and file handling

---

## 🧠 How It Works

1. You manually log into X.com using Puppeteer and save your cookies to a file.
2. The n8n workflow:
   - Reads the cookies from `x-cookies.json`
   - Accepts input: `content` + `community` name
   - Launches Puppeteer via n8n’s Puppeteer node
   - Navigates to the post box
   - Selects the correct community
   - Posts the content as if it were done by a human

---

## 🛠 Setup Instructions

### 1. Save your X.com login cookies

Run this Puppeteer script manually to log in and save cookies:

```js
// login-save-cookies.js (run outside n8n)
const puppeteer = require('puppeteer');
const fs = require('fs');

(async () => {
  const browser = await puppeteer.launch({ headless: false });
  const page = await browser.newPage();

  await page.goto('https://twitter.com/login');
  console.log('⏳ Please log in manually...');
  await new Promise(resolve => setTimeout(resolve, 30000)); // wait for manual login

  const cookies = await page.cookies();
  fs.writeFileSync('./x-cookies.json', JSON.stringify(cookies, null, 2));
  console.log('✅ Cookies saved to x-cookies.json');
  await browser.close();
})();


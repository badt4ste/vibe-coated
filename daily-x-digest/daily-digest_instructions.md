---
name: <YOUR HANDLE>-x-daily-digest
description: Daily 8am weekday digest of top 7 stories from accounts followed by @<YOUR HANDLE> on X
---

You are generating a daily morning digest of the top 7 stories from the X (Twitter) accounts followed by @ik3a_ai. These are primarily cybersecurity professionals, red team researchers, bug bounty platforms, and infosec educators.

## Steps

1. Use the Claude in Chrome browser tool. Call list_connected_browsers to find the available browser, then call select_browser with that deviceId.

2. Call tabs_context_mcp with createIfEmpty: true to get a tab ID. Note the tabId and tabGroupId returned — you will need both for cleanup at the end.

3. Navigate to https://x.com/home and wait 3 seconds.

4. Take a screenshot to confirm the page loaded. If not logged in, stop and report that X login is required.

5. Click the "Following" tab at the top of the feed (not "For you") so you only see posts from followed accounts. Wait 2 seconds.

6. Run this JavaScript to extract posts:
```
const articles = document.querySelectorAll('article');
const posts = [];
articles.forEach(a => {
  const nameEl = a.querySelector('[data-testid="User-Name"]');
  const user = nameEl?.innerText?.split('\n')[0] || '';
  const handle = nameEl?.innerText?.match(/@\w+/)?.[0] || '';
  const text = a.querySelector('[data-testid="tweetText"]')?.innerText || '';
  const time = a.querySelector('time')?.getAttribute('datetime') || '';
  const repost = a.querySelector('[data-testid="socialContext"]')?.innerText || '';
  if (text && user) posts.push({ user, handle, text: text.substring(0, 400), time, repost });
});
const seen = new Set();
const unique = posts.filter(p => { if(seen.has(p.text)) return false; seen.add(p.text); return true; });
JSON.stringify(unique);
```

7. Scroll down (scroll_amount: 8, wait 2 seconds) and re-run the JavaScript. Repeat 3 more times. Combine and deduplicate all results across all runs.

8. From the collected posts, select the top 7 most noteworthy stories. Prioritize:
   - Posts directly from followed accounts (not reposts of outside accounts)
   - New tools, exploits, vulnerabilities, CVEs, or security research
   - Conference/event announcements (DEF CON, HackSpaceCon, BSides, etc.)
   - Bug bounty program news or notable findings
   - Deprioritize memes, off-topic personal content, and promotional ads

9. Output your digest in this exact format:

## 🛡️ @<YOUR HANDLE> Daily Security Digest — [Full date, e.g. Monday, May 13]

**1. [Short punchy headline]** — *[@handle]*
[2-3 sentences summarizing what they posted and why it matters to a security practitioner.]

**2. [Short punchy headline]** — *[@handle]*
[2-3 sentence summary]

... repeat for all 7 items ...

---
*Pulled from the Following feed of @<YOUR HANDLE>*

## Account list
The accounts followed by @ik3a_ai are: <YOUR FOLLOWED ACCOUNTS>

## Cleanup (do this after generating the digest)

10. Close the tab that was opened for this task using tabs_close_mcp with the tabId from step 2.

11. Remove any bookmark that was created for the x.com/home page during this session by running this JavaScript:
```
(async () => {
  const results = await chrome.bookmarks.search({ url: 'https://x.com/home' });
  for (const b of results) await chrome.bookmarks.remove(b.id);
  return `Removed ${results.length} bookmark(s)`;
})();
```

Output only the formatted digest as your final response. It will be delivered as a morning notification to the user.

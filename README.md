## Problem statement
A major news publication has revealed sensitive private information about Daikibo Industrials, our client. A production problem has caused its assembly lines to stop, threatening the smooth operation of supply chains relying on Daikibo’s products. The client suspects the security of their new status board may have been breached.
In this task you will be joining our cyber security team. Your job is to:

Determine if the alleged breach could have happened from an attacker on the internet directly (i.e. no access to Daikibo's VPN).
Inspect a web_requests.log file (listing only data from a period when the alleged attack has to have happened):
Try to spot suspicious requests
Hint: In the Resources section, you can find a diagram example of how to read the logs file
Hint: Look for longer sequences of user requests
Hint: Notice the order of requests from Login → to requests for the dashboard page's resources (styles, scripts, images, etc.) → to API requests for the actual statuses of the machines
Hint: How would you recognise if an automated request to the API happens at an exact interval of time (assume no such functionality is available in the dashboard)?
If you've identified such requests make sure to write down the ID of the user (it's part of the requests)
Here is how the web_requests.log file is structured:

There is a sequence of blocks of text divided by empty lines
Each block represents the activity of a unique IP address (no 2 blocks have the same IP)
The block starts with the IP address followed by a table of the requests made to Daikibo's telemetry dashboard (the dashboard lives in Daikibo's intranet) by the device with this IP address, sorted by time
The IP addresses are from the internal Daikibo network and are static
1 block can represent 1 or multiple browsing sessions
Sessions made on different dates require new logins
There is no continuous polling/pushing of data between client and server - the users need to refresh the page to get the latest data
Hint: For an easier visual inspection, open up the file in a code editor like Sublime Text or Visual Studio Code, expand the window to the full width of your screen and decrease font size until no text breaks on a new line
When you believe you have completed the 2 tasks above, submit your work by taking a quick quiz to check your discoveries. Start the quiz by clicking 'Start your quiz' below. Good luck!
## Solution
## Step-by-Step Analysis of the Web Activity Logs

I analyzed the web activity logs to identify suspicious automated behavior that could indicate a security breach. Here's my systematic approach:

### Step 1: Understanding the Log Structure
I first examined the log format to understand:
- Each block represents activity from a unique internal IP address (192.168.x.x range)
- Requests include timestamps, HTTP methods, request paths, user IDs, and status codes
- Normal user behavior follows a pattern: unauthorized access → login → dashboard resources → API calls

### Step 2: Identifying Normal vs. Suspicious Patterns

**Normal user behavior typically shows:**
- GET "/" → 401 (UNAUTHORIZED) 
- GET "/login" → 200 (login page and resources)
- POST "/login" → 200 (successful login)
- GET "/" with authorizedUserId → 200 (dashboard access)
- Sporadic API calls to check factory/machine status

### Step 3: Detecting Automated Behavior

I searched for patterns indicating automated requests by looking for:
- API calls occurring at exact time intervals
- Multiple simultaneous requests to different endpoints
- Consistent timing patterns that humans wouldn't exhibit

### Step 4: Key Suspicious Discovery

**IP Address: 192.168.0.101**  
**Suspicious User ID: mdB7yD2dp1BFZPontHBQ1Z**

I identified highly suspicious automated behavior from this IP address:

**Pattern Analysis:**
Starting at 2021-06-25T17:00:48.000Z, this user makes **exactly 4 simultaneous API calls every hour at 48 seconds past the hour:**

```
2021-06-25T17:00:48.000Z GET "/api/factory/machine/status?factory=meiyo&machine=*"
2021-06-25T17:00:48.000Z GET "/api/factory/machine/status?factory=seiko&machine=*" 
2021-06-25T17:00:48.000Z GET "/api/factory/machine/status?factory=shenzhen&machine=*"
2021-06-25T17:00:48.000Z GET "/api/factory/machine/status?factory=berlin&machine=*"
```

This pattern repeats **exactly every hour** for multiple hours:
- 18:00:48, 19:00:48, 20:00:48, 21:00:48, 22:00:48, 23:00:48

### Step 5: Evidence of Session Management

At 2021-06-26T00:00:48.000Z, the automated requests start returning 401 (UNAUTHORIZED), indicating session expiration. The attacker then logs back in and resumes the automated polling.

## Conclusions

### Could the breach happen from the internet directly?
**No.** The suspicious activity originates from internal IP address 192.168.0.101, which is within Daikibo's internal network range. This indicates either:
- An insider threat using automated tools
- A compromised internal system
- Someone with legitimate VPN access acting maliciously

### Suspicious Activity Identified
- **User ID:** `mdB7yD2dp1BFZPontHBQ1Z`
- **IP Address:** `192.168.0.101` 
- **Behavior:** Automated hourly polling of all factory machine statuses
- **Pattern:** Exactly every hour at 48 seconds past the hour
- **Duration:** Multiple days with session management

Q 1/2: Is there a way for a hacker to access Daikibo's manufacturing status dashboard directly from the internet?
No, the attacker has no direct access to the status dashboard.
In the original scope of the project we have listed that the dashboard will be living in Daikibo's intranet. The only remote access to it would be through VPN tunnelling.
Q 2/2: Looking at the web_requests.log, what is the user ID with the most suspicious activity?
mdB7yD2dp1BFZPontHBQ1Z
It starts off with a regular login -> browsing of the dashboard. But then it turns into a regular, once-per-hour (see the time stamps) automated check of the statuses in all 4 factories with no page resources being loaded and with an obviously non-human punctuality


This systematic data extraction suggests someone is automatically harvesting sensitive production information from all factories, which could explain how private information about Daikibo's production problems was leaked to the news publication.

The precision and consistency of the timing (exactly at :48 seconds every hour) clearly indicates automated scripting rather than human interaction, representing a significant security breach of the status board system.

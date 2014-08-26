peoplewebapp
============
The People Web App is the online digital edition of the magazine, which  utilizes
the Adobe Digital Publishing Suite (DPS) Web Viewer Embedding Library (also called the Adobe API), and includes a library page, displaying all issues a user is entitled to, and content viewer page, displaying the issue content within an iframe hosted on Adobe's server. As the digital edition is for subscribers only, the web app requires authentication which is handled through People.com's Premium single sign on service.

My role on this project was as lead developer, developing the HTML, Javascript, and CSS for both the library and content viewer page, with some code contributed by other developers on my team. All code contributed by other developers on team are noted in the source.

DEMO

*To access the People Web App you must have People Magazine subscriber digital credentials*

1.  Go to www.people.com

2.  Click Sign In in top right corner

3.  Enter credentials
 
4.  Click Magazine or go here directly: http://www.people.com/people/static/digitalmagazine/index.html


RELEVANT FILES

index.html - library page, displays all magazine issues a subscriber is entitled to read

content.html - content viewer page for reading an issue


A full description of the functionality of each page is below

Library page

The library page is the page displaying all issues a user is entitled to read. A high level step by step description of the functionality is as follows:

1. Check if an article query string is appended, meaning a subscriber is trying to access a deep link (see Deep Links below for more info). If there is one appended, STOP and redirect to content page with query string appended, otherwise go to step 2.
2. Make an ajax call to php authentication file (https://auth.people.com/getTicket.php) which returns authentication token in JSON format. If there is no authentication token (meaning a status "401" or "503" is returned), STOP and redirect to subscription form, otherwise go to step 3
3. If there is an authentication token, use Adobe's API to check if there is already an entitlement token set (getEntitlementTicket()). If there is, use that token, otherwise set the token, using Adobe's API (setEntitlementTicket(authtoken)) to the one returned in step 2.
4. Check if there is a deep link cookie (see Deep Links below for more info). This cookie is set if a user tries to access an article via a deep link but is not logged in first. If there is a cookie present, STOP and redirect to content page with query string appended, otherwise go to step 5
5. Make an ajax call to php file (https://auth.people.com/ajaxEntitlements.php) which returns all entitlements, and other useful subscriber info, in JSON format.
6. Initialize Adobe's library service
7. Loop through Adobe's XML feed which returns all publications in XML format. If a publication's product ID matches the user's entitlement, and any additional conditions (based on title) display in the library. use Adobe's API to build the article URLs

Deep Links

A deep link is a URL that links to specific content in the web reader page. Deep links have the structure (using People as an example):

http://www.people.com/people/static/digitalmagazine/index.html?/PEOPLE/8632a117e7124ea988555c2dd41aa858/com.timeinc.people.ipad.inapp.10282013/278203.html

Deep Link Cookie

If a user tries to access a deep link, and the user is not logged in, the URL is stored in a cookie, named TCM_PE_WebAppDeepLink. This cookie expires at the end of the session.

Content Page

The content page is the page that displays the magazine content within an iframe, and must have a query string appended, representing the article URL. High level step by step description of the functionality is as follows:

1. Make an ajax call to php file (https://auth.people.com/getTicket.php) which returns auth token in JSON format. If there is no auth token (meaning a status "401" or "503" is returned), STOP and redirect to subscription form, otherwise go to step 2
2. If there is an auth token, use Adobe's API to check if there is already an entitlement token set (getEntitlementTicket()). If there is, use that token, otherwise set the token, using Adobe's API (setEntitlementTicket(authtoken)) to the one returned in step 2.
3. Build the outer frame around the content viewer, and call Adobe's create frame service (wvBridge), that loads the content viewer, and displays the content

There are several important events returned from Adobe's API, handled as follows

Paywall event: This event is returned on the content page, if a user is not logged in, or is logged in but trying to access content not entitled to. If this event is returned, and a user is not logged in, store the URL in a cookie, and redirect to a subscription form. If the user is logged in, and this event is returned, the user is redirected to the library page.
 
Navigation event: When the navigation event is returned, change the URL hash to the article path, especially to allow bookmarking and storage of the URL in a cookie
 

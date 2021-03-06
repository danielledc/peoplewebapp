peoplewebapp
============
Note: This description and source code was as of 2014.  

The People Web App is the online digital edition of the magazine, which  utilizes
the Adobe Digital Publishing Suite (DPS) Web Viewer Embedding Library (also called the Adobe API), and includes a library page, displaying all issues a user is entitled to, and content viewer page, displaying the issue content within an iframe hosted on Adobe's server. As the digital edition is for subscribers only, the web app requires authentication which is handled through People.com's Premium single sign on service.

My role on this project was as lead developer, developing the HTML, Javascript, and CSS for both the library and content viewer page, with some code contributed by other developers on my team. All code contributed by other developers on team are noted in the source.

DEMO:

People Web App image: http://daniellec.byethost12.com/portfolio/wp-content/uploads/2016/02/mc_pic12.png

RELEVANT FILES

index.html - library page, displays all magazine issues a subscriber is entitled to read

content.html - content viewer page for reading an issue


A full description of the functionality of each page is below

Library page

The library page is the page displaying all issues a user is entitled to read, after first checking that an auth token (from the People Premium single service) exists and logging in to Adobe's authentication service using that token (thus not requiring the user to sign in again). A step by step description of the functionality is as follows:

1. Check if an article query string is appended, meaning a subscriber is trying to access a deep link (see Deep Links below for more info). If there is one appended, STOP and redirect to content page with query string appended, otherwise go to step 2.
2. Make an ajax call to php authentication file which returns authentication token in JSON format. If there is no authentication token (meaning a status "401" or "503" is returned), subscriber is not logged in, so, STOP, and redirect to subscription form, otherwise go to step 3
3. If there is an authentication token, use Adobe's API to check if there is already an entitlement token set (getEntitlementTicket()). If there is, use that token, otherwise set the token, using Adobe's API (setEntitlementTicket(authtoken)) to the one returned in step 2.
4. Check if there is a deep link cookie (see Deep Links below for more info). This cookie is set if a user tries to access an article via a deep link but is not logged in first. If there is a cookie present, STOP and redirect to content page with query string appended, otherwise go to step 5
5. Make an ajax call to php file which returns all entitlements, and other useful subscriber info, in JSON format.
6. Initialize Adobe's library service
7. Loop through Adobe's XML feed which returns all publications in XML format. If a publication's product ID matches the user's entitlement, and any additional conditions (based on title) display in the library. use Adobe's API to build the article URLs

Deep Links

A deep link is a URL that links to specific content in the web reader page. 

Deep Link Cookie

If a user tries to access a deep link, and the user is not logged in, the URL is stored in a cookie, named TCM_PE_WebAppDeepLink. This cookie expires at the end of the session.

Content Page

The content page is the page that displays the magazine content within an iframe, and must have a query string appended, representing the article URL. Similar to the library page, the page checks that an auth token (from the People Premium single service) exists and logs in to Adobe's authentication service using that token (thus not requiring the user to sign in again).  A step by step description of the functionality is as follows:

1. Make an ajax call to php file which returns auth token in JSON format. If there is no auth token (meaning a status "401" or "503" is returned), STOP and redirect to subscription form, otherwise go to step 2
2. If there is an auth token, use Adobe's API to check if there is already an entitlement token set (getEntitlementTicket()). If there is, use that token, otherwise set the token, using Adobe's API (setEntitlementTicket(authtoken)) to the one returned in step 2.
3. Build the outer frame around the content viewer, and call Adobe's create frame service (wvBridge), that loads the content viewer, and displays the content

There are several important events returned from Adobe's API, handled as follows:

Paywall event: This event is returned on the content page, if a user is not logged in, or is logged in but trying to access content not entitled to. If this event is returned, and a user is not logged in, store the URL in a cookie, and redirect to a subscription form. If the user is logged in, and this event is returned, the user is redirected to the library page.
 
Navigation event: When the navigation event is returned, change the URL hash to the article path, especially to allow bookmarking and storage of the URL in a cookie
 
Orientation change: This event returns the orientation of the content, either portrait or landscape.  When this event is returned, the layout around the iframe is adjusted accordingly 

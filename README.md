# Chrome-Extension-for-Screenshots
build a chrome extension that can take specific screenshots of the page the customer is in and automatically send the data to a specific endpoint. The chrome extension needs to have some user authentication to make sure the user is authenticated.
----------
To build a Chrome extension that takes specific screenshots of the current page and automatically sends the data to a specified endpoint, including user authentication, you'll need to break the project down into a few steps:

    Create the Chrome Extension Manifest.
    Develop the background script for authentication and handling the screenshot functionality.
    Develop a content script to interact with the webpage and capture screenshots.
    Authenticate users via a secure authentication mechanism (OAuth or token-based).
    Send the screenshot data to the backend endpoint.

Below is the code structure to achieve this:
1. Create manifest.json

The manifest.json file defines the Chrome extension and its permissions.

{
  "manifest_version": 3,
  "name": "Screenshot Capture Extension",
  "version": "1.0",
  "description": "Capture screenshots and send them to the server.",
  "permissions": [
    "activeTab",
    "storage",
    "identity"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "host_permissions": [
    "http://*/*",
    "https://*/*"
  ],
  "oauth2": {
    "client_id": "YOUR_OAUTH_CLIENT_ID",
    "scopes": ["email"]
  }
}

2. Background Script (background.js)

This script is responsible for handling the authentication and capturing the screenshot.

chrome.runtime.onInstalled.addListener(() => {
  console.log("Extension installed");
});

// Handle user authentication
chrome.identity.getAuthToken({ interactive: true }, function (token) {
  if (chrome.runtime.lastError || !token) {
    console.error("Authentication failed");
    return;
  }
  console.log("User authenticated with token:", token);
  // Store the token for later use
  chrome.storage.local.set({ authToken: token });
});

// Listen for the command to capture screenshot
chrome.action.onClicked.addListener(async (tab) => {
  // Check if the user is authenticated
  chrome.storage.local.get("authToken", function (data) {
    const token = data.authToken;
    if (!token) {
      alert("Please log in first.");
      return;
    }

    // Capture screenshot
    chrome.tabs.captureVisibleTab(tab.windowId, { format: "png" }, function (imageUrl) {
      sendScreenshot(imageUrl, token);
    });
  });
});

// Function to send the screenshot to a specific endpoint
function sendScreenshot(imageUrl, token) {
  fetch("https://your-api-endpoint.com/upload", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${token}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      image: imageUrl,
      userId: "user123",  // You can replace this with dynamic user data
    }),
  })
    .then(response => response.json())
    .then(data => {
      console.log("Screenshot uploaded successfully", data);
    })
    .catch(error => {
      console.error("Error uploading screenshot", error);
    });
}

3. Content Script (content.js)

The content script can be used to manipulate the DOM or gather information about the page. In this case, it may not be required if we’re just capturing a screenshot of the page, but here’s an example of how you might interact with the page:

// Example content script code (could be expanded as per requirements)
console.log("Content script loaded");

// Example: Capture and log the page title
const pageTitle = document.title;
console.log("Page title:", pageTitle);

4. Popup HTML (popup.html)

A simple popup interface for the user to interact with the extension.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Screenshot Extension</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      width: 200px;
      padding: 10px;
    }
    button {
      width: 100%;
      padding: 10px;
      margin-top: 10px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h3>Take Screenshot</h3>
  <button id="captureBtn">Capture Screenshot</button>

  <script>
    document.getElementById("captureBtn").addEventListener("click", function () {
      chrome.runtime.sendMessage({ action: "capture_screenshot" });
    });
  </script>
</body>
</html>

5. Authentication Mechanism (OAuth)

In this example, we use Chrome's identity API for OAuth authentication. You'll need to replace the YOUR_OAUTH_CLIENT_ID in manifest.json with your actual OAuth client ID. This requires setting up OAuth credentials with Google or another authentication provider.

    OAuth with Google:
        Go to Google Developer Console.
        Create a new project and set up OAuth 2.0 credentials.
        Enable the Google+ API (for basic profile info).
        Add the OAuth Client ID to the manifest.json file.

    Other Authentication Providers:
        You can also integrate other authentication providers using the same method.

6. Icons

Add your icons (e.g., icon16.png, icon48.png, icon128.png) into the icons/ folder. These will be used in the Chrome extension interface.
7. Sending Screenshot to the Endpoint

The extension captures a screenshot using chrome.tabs.captureVisibleTab() and sends the image data to a specified endpoint via a POST request with the user's authorization token attached in the header.
8. Packaging the Extension

To install your extension in Chrome:

    Open chrome://extensions in Chrome.
    Enable Developer mode.
    Click Load unpacked and select your project folder.

Now, your extension should be active, and you can test capturing screenshots by clicking the extension icon.
Final Thoughts:

    Security: Ensure the API endpoint where you’re sending the screenshots is secure and uses HTTPS to prevent data leaks.
    User Authentication: Handle the OAuth token securely, and make sure it has an expiration mechanism.
    Error Handling: Add error handling for scenarios like failed uploads or unauthorized users.

This is a basic example that captures a screenshot and sends it to an endpoint. You can extend this to capture specific parts of the page, add more sophisticated user authentication, or further customize the interface.

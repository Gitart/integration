# How to Use Notion with Gmail and Google Sheets using Apps Script

[Home](https://www.labnol.org/)

July 20, 2021

![Amit Agarwal](https://www.labnol.org/static/amit-4e4f7e42a9450ced83c69dcf977cbb55.jpg)

[Amit Agarwal](https://www.labnol.org/about)

[@labnol](https://twitter.com/labnol)

[![Google Developer Expert](https://www.labnol.org/static/badge-3f4794bee3843a024e9393315e613f07.svg)](https://www.labnol.org/about "Google Developer Expert")

> How to use the Notion API with Google Apps Script to connect Gmail, Google Forms, and Google Sheets with your Notion workspace.

Notion, my absolute favorite tool for storing all sorts of things from web pages to code snippets to recipes, just got better. They’ve released a public API and thus it will be a lot easier for developers to read and write to their Notion workspace from external apps.

For instance, you can create a document in Google Docs and export it to Notion while staying inside Docs. Google Sheets users can pull pages from Notion database into their spreadsheet. Any new submissions in Google Forms can be directly saved to Notion and so on!

## [](https://www.labnol.org/gmail-to-notion-apps-script-210515#save-gmail-messages-in-notion)Save Gmail Messages in Notion

I have put together a [Gmail add-on](https://workspace.google.com/marketplace/app/save_to_notion/562294135760) that makes it easy for you to save email messages, or any other text content, from Gmail to your Notion workspace with a click. Here’s how the app works.

**Step 1:** Connect Gmail to Notion

 [![002815](https://www.labnol.org/static/df7ba0dbdcfdd8ea0d507ed1d65bfd24/16546/002815.png "002815")](https://www.labnol.org/static/df7ba0dbdcfdd8ea0d507ed1d65bfd24/c1eb0/002815.png)

**Step 2:** Allow Access to Notion pages - if you have multiple databases in your Notion workspace, you have an option to grant access to select databases and the rest will be inaccessible to the external app.

 [![Authorize Notion](https://www.labnol.org/static/65bfaeb5fa00155fb0d61edc7c68fb33/16546/003028.png "Authorize Notion")](https://www.labnol.org/static/65bfaeb5fa00155fb0d61edc7c68fb33/c1eb0/003028.png)

**Step 3:** Choose Email - open any email message in Gmail and you’ll have an option to edit the content of the email subject and body before sending the content to your Notion page. Please note that the app only supports plain text format at this time.

 [![Send Email to Notion](https://www.labnol.org/static/79c0485df96a8421835e0c843812ebae/16546/003309.png "Send Email to Notion")](https://www.labnol.org/static/79c0485df96a8421835e0c843812ebae/c1eb0/003309.png)

**Step 4:** Open Notion - As soon as you hit the `Send to Notion` button, the content of the currently selected email message is added to your Notion database. You can click the `All updates` link in your Notion sidebar to view to recently added page.

 [![Notion page](https://www.labnol.org/static/b76f603d5892b980a4139d46380b85c2/16546/003449.png "Notion page")](https://www.labnol.org/static/b76f603d5892b980a4139d46380b85c2/49d2b/003449.png)

### [](https://www.labnol.org/gmail-to-notion-apps-script-210515#how-to-use-notion-with-google-apps-script)How to Use Notion with Google Apps Script

If you would to integrate your own Google add-on with Notion API, here’s a brief outline of the steps involved.

1.  Go to [notion.so](https://www.notion.so/my-integrations) and click the `Create New Integration` button. You’ll be provided with a Client ID and Client Secret that you’ll need in a later step.

2.  Include the [OAuth2](https://github.com/googleworkspace/apps-script-oauth2) library in your Apps Script project and invoke the `getRedirectUri` method to get the OAuth2 redirect URL for the previous step.

```js
const getNotionService = () => {
  return OAuth2.createService('Notion')
    .setAuthorizationBaseUrl('https://api.notion.com/v1/oauth/authorize')
    .setTokenUrl('https://api.notion.com/v1/oauth/token')
    .setClientId(CLIENT_ID)
    .setClientSecret(CLIENT_SECRET)
    .setCallbackFunction('authCallback')
    .setPropertyStore(PropertiesService.getUserProperties())
    .setCache(CacheService.getUserCache())
    .setTokenHeaders({
      Authorization: `Basic ${Utilities.base64Encode(`${CLIENT_ID}:${CLIENT_SECRET}`)}`,
    });
};

const authCallback = (request) => {
  const isAuthorized = getNotionService().handleCallback(request);
  return HtmlService.createHtmlOutput(isAuthorized ? 'Success!' : 'Access Denied!');
};

const getRedirectUri = () => {
  console.log(OAuth2.getRedirectUri());
};
```

3.  Connect to Notion API - Make a `Get` [HTTP request](https://www.labnol.org/urlfetch) to the [/vi/databases](https://developers.notion.com/reference/get-databases) to fetch a list of all databases that the user has explicitly shared with authorized app.

```js
function getDatabasesList() {
  var service = getNotionService();
  if (service.hasAccess()) {
    const url = 'https://api.notion.com/v1/databases';
    const response = UrlFetchApp.fetch(url, {
      headers: {
        Authorization: `Bearer ${service.getAccessToken()}`,
        'Notion-Version': '2021-05-13',
      },
    });
    const { results = [] } = JSON.parse(response.getContentText());
    const databases = results
      .filter(({ object }) => object === 'database')
      .map(({ id, title: [{ plain_text: title }] }) => ({ id, title }));
    console.log({ databases });
  } else {
    console.log('Please authorize access to Notion');
    console.log(service.getAuthorizationUrl());
  }
}
```

### [](https://www.labnol.org/gmail-to-notion-apps-script-210515#download-gmail-to-notion)Download Gmail to Notion

The **Gmail to Notion** app is in beta. If you would like to use it with your Gmail or Google Workspace account, please install from here - [Gmail to Notion](https://workspace.google.com/marketplace/app/save_to_notion/562294135760)

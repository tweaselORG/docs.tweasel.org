{
    "title": "Viewing HAR files",
    "weight": 30,
    "description": "The tweasel tools and libraries make heavy use of HAR files, a standard format for HTTP(S) traffic recordings. Learn how to view the contents of a HAR file in Firefox and Google Chrome."
}

[HAR](http://www.softwareishard.com/blog/har-12-spec/) (HTTP Archive) is a standard file format for HTTP(S) traffic recordings. It is used by web browsers and HTTP(S) monitoring tools to export collected requests and responses. The tweasel tools and libraries use HAR files to allow for interoperability with other tools: You can analyze HAR files created with third-party tools using TrackHAR. Conversely, you can also view HAR files created by cyanoacrylate or tweasel CLI using your web browser, which is what this tutorial will focus on.

## Viewing HAR files in Firefox

In Firefox, you can view HAR files using the Developer Tools. Open a new tab and right-click on the page and select *Inspect* (or press <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>I</kbd>).

![Screenshot of the new tab page in Firefox. The right-click context menu is open and an arrow points to the last entry, "Inspect (Q)".](firefox-01-inspect.png)

In the Developer Tools section that opens, click on the *Network* tab.

![Screenshot of the Developer Tools section in Firefox. An arrow points to the Network tab.](firefox-02-network-tab.png)

Click on the cog icon in the top right corner of the tab. Then, click *Import HAR file*.

![Screenshot of the Network tab in the Firefox Developer tools. An arrow points to a cog icon in the top right, under which a context menu is open. Another arrow points to the "Import HAR file" entry.](firefox-03-import-har.png)

After you have selected your HAR file in the file picker, you will see a list of the requests recorded therein.

![Screenshot of the Network tab showing a tabular list of 22 requests, most of them to the api.airbnb.com domain.](firefox-04-har-requests.png)

If you click on an entry in the table, you will see a detailed view of the request. Using the tab bar at the top, you can switch between seeing the headers and cookies as well as the request and response payloads.

![Screenshot of the Network tab showing a detailed view of the headers in a request to https://api2.branch.io/v1/close. Both response and request headers are shown. At the top, a tab group with the entries Headers, Cookies, Request, Response, and Timings is highlighted with a box.](firefox-05-har-single-headers.png)

![Screenshot of the same request in the Network tab but this time the request tab is open, showing the beginning of the request payload. The payload is a JSON object, of which the device_fingerprint_id property is visible.](firefox-06-har-single-request.png)

# Captive Network Assistant

If you're on a pay-to-use wifi network or some dumbass corporate LAN with Internet auth hanging off of Active
Directory (hi guys!), then you'll have seen a popup appear on your device (IOS or macOS) that's trying to take you
to `captive.apple.com`. This is the captive network assistant at work, helping you get a login to the wifi network
you're attempting to connect to. Normally, you just punch in whatever creds you need and off you go.

Oddly, at work, where we auth out via a FortiGate firewall that asks Active Directory to validate the login,
`captive.apple.com` has stopped working. The site returns its `Success` message, but you don't actually get authenticated.
Additionally, if you attempt to get out to another website, you can't get the auth prompt unless the site you're visiting
doesn't use HTTPS, which is becoming increasingly rare.

Because of this, I now run `http://captive.xnode.co.za` in a normal browser window to help me break out (which I won't be applying HTTPS to). I'm
also looking for a way to change `captive.apple.com` to `captive.xnode.co.za`, to see if that would fix whatever issue
the FW has with the page at the end of Apple's link.

## Some research

I found this in `/System/Library/SystemConfiguration/CaptiveNetworkSupport.bundle/Contents/Resources/Settings.plist`
under macOS High Sierra:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>ProbeParameters</key>
	<dict>
		<key>ProbeURL</key>
		<string>http://captive.apple.com/hotspot-detect.html</string>
		<key>UserAgent</key>
		<string>wispr</string>
		<key>SSIDExceptions</key>
		<dict>
			<key>MOBILE</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>swisscom</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>dillardswifi</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>hhonors</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>Telekom_ICE</key>
			<dict>
				<key>UserAgent</key>
				<string>tmobile_wispr1</string>
			</dict>
			<key>Telekom</key>
			<dict>
				<key>UserAgent</key>
				<string>tmobile_wispr1</string>
			</dict>
			<key>tmobile</key>
			<dict>
				<key>UserAgent</key>
				<string>tmobile_wispr1</string>
			</dict>
			<key>T-Mobile_ICE</key>
			<dict>
				<key>UserAgent</key>
				<string>tmobile_wispr1</string>
			</dict>
			<key>attwifi</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>Wayport_Access</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>ATTMetroWiFi</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
			<key>ATTMETROWIFI</key>
			<dict>
				<key>ProbeURL</key>
				<string>http://attwifi.apple.com/library/test/success.html</string>
			</dict>
		</dict>
	</dict>
	<key>ScrapingParameters</key>
	<dict>
		<key>UserFields</key>
		<array>
			<string>user</string>
			<string>username</string>
			<string>account</string>
		</array>
		<key>PasswordFields</key>
		<array>
			<string>pass</string>
			<string>password</string>
		</array>
		<key>RealmFields</key>
		<array>
			<string>roamrealm</string>
			<string>realm</string>
			<string>suffix</string>
			<string>operator</string>
		</array>
		<key>DisabledRealms</key>
		<array>
			<string>@orange.fr</string>
		</array>
	</dict>
</dict>
</plist>
```

When Captive Network Assistant loads, it looks for

```
<key>ProbeURL</key>
<string>http://captive.apple.com/hotspot-detect.html</string>
```

which contains the following content:

```
curl -i http://captive.apple.com/hotspot-detect.html

HTTP/1.1 200 OK
x-amz-id-2: 73lrc9pQQHLB8W7zIbV5FzuD7/+ip0NOAzlDJvMKKkxoe5NPhbqcak7aral+OiOgJmwJJqQhvrI=
x-amz-request-id: F2A25F9199CCB313
Date: Wed, 29 Nov 2017 12:42:18 GMT
Last-Modified: Fri, 17 Feb 2017 20:36:28 GMT
Cache-Control: max-age=300
Accept-Ranges: bytes
Content-Type: text/html
Content-Length: 69
Server: ATS/7.1.1
Via: http/1.1 defra3-edge-lx-007.ts.apple.com (ApacheTrafficServer/7.1.1), http/1.1 defra3-edge-bx-037.ts.apple.com (ApacheTrafficServer/7.1.1)
CDNUUID: 0c8b394e-063f-4b19-a1f3-cef484a34b59-478113431
X-Cache: hit-fresh, hit-fresh
Etag: "41ba060eb1c0898e0a4a0cca36a8ca91"
Age: 250
Connection: keep-alive

<HTML><HEAD><TITLE>Success</TITLE></HEAD><BODY>Success</BODY></HTML>
```

I'm assuming that if you're on a specific provider network, you get bounced to a specific page so that you can punch
in your login creds, else you get served the default "Safari" authentication dialogue.

So, to disable SIP and edit the file, or not? Hmmm...

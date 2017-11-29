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

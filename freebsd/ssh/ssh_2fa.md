# FreeBSD SSH + 2FA using Google Authenticator

Adding 2FA to SSH on FreeBSD. First, install the Google Authenticator package:

`sudo pkg install pam_google_authenticator`

then run the command:

`google-authenticator`

This will result in a series of yes/no questions. Just hit a `y` until it shuts up (the defaults are sane):

```
> google-authenticator
https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/someone@box.domain.tld%3Fsecret%3DPKZ2JNIV6JXXXSEC
Your new secret key is: PKZ2JNIV6JXXX3SEC
Your verification code is 1266323
Your emergency scratch codes are:
61588958
11243750
63710794
52193634
23900338

Do you want me to update your "~/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) y

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
>
```

Copy the link generated under the initial command, and using your auth app (I like authy), create a new
account/login/whatever. This, obviously, is the source for the six digit token that comprises the something you have
part of 2FA.

Then, edit `/etc/pam.d/sshd` to look as follows:

```
#
# $FreeBSD: releng/11.1/etc/pam.d/sshd 197769 2009-10-05 09:28:54Z des $
#
# PAM configuration for the "sshd" service
#

# auth
auth		sufficient	pam_opie.so		no_warn no_fake_prompts
auth		requisite	pam_opieaccess.so	no_warn allow_local
#auth		sufficient	pam_krb5.so		no_warn try_first_pass
#auth		sufficient	pam_ssh.so		no_warn try_first_pass
auth		sufficient	pam_google_authenticator
auth		required	pam_unix.so		no_warn try_first_pass
```

You're adding in the following two lines specifically:

```
auth		sufficient	pam_google_authenticator
auth		required	pam_unix.so		no_warn try_first_pass
```

Because 2FA is effectively password-based authentication, you're going to need some stuff enabled in your SSH config
that many, myself included, normally switch off. The below config in `/etc/ssh/sshd_config` should help mitigate any
issues normally associated with enabling password based authentication via SSH:

```
Port 22
Protocol 2
LoginGraceTime 1m
PermitRootLogin no
MaxAuthTries 4
StrictModes yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication yes
UsePAM yes
TCPKeepAlive yes
UseDNS no
Subsystem       sftp    /usr/libexec/sftp-server
Ciphers aes128-ctr,aes192-ctr,aes256-ctr
MACs hmac-sha1,hmac-ripemd160,hmac-ripemd160@openssh.com
GSSAPIAuthentication no
GSSAPICleanupCredentials no

Match User someone
    AuthenticationMethods publickey,keyboard-interactive
    PasswordAuthentication yes
```

By default, we disable `PasswordAuthentication`, but enable `ChallengeResponseAuthentication`. Then, on a user by
user basis, we enable public key auth, followed by keyboard-interactive auth, with `PasswordAuthentication` enabled
to force the use of the 2FA verification code.

As a bonus, this configuration also removes insecure ciphers and MACs from your SSH daemon.

This is all that is needed. Now, when you login to a SSH+2FA session as a configured user, you'll see the following:

```
➜  ~ ssh shell.xnode.co.za
Verification code:
```

:boom:! SSH using 2FA on your FreeBSD server.

---
title: Tips for Linux on your own computer
...

# Setting up for UVa's  EduRoam setup

These instructions are current as of Summer 2018 (and still valid as of last update of this page, see the footer).
I've been at UVA since 2008 and have had to change how this works three times, leading me to expect about a 3--4 year lifespan for these instructions.
Hopefully I'm being pessimistic...

Using the Network Manager app (the default tool used for network connection in Cinnamon, MATE, Gnome, XFCE, LXDE, Ubuntu, etc.), most of the defaults should work; however, in Wi-Fi Security (which may be all you are shown depending on how you picked the eduroam SSID) you need

Security
:   WPA & WPA2 Enterprise

Authentication
:   TLS

Identity
:   mst3k`@virginia.edu`

Domain
:   *leave this blank*

CA certificate
:   This will need to be a file on your computer.
    UVA used to provide a list of 10 of these, but I can't seem to find it anymore.
    One of them that does work for me is the *US Higher Education Root (USHER V2)*:
    <http://h1.usherca.org/aia/ca.pem> (download link from <http://www.ushercs.org/>).
    
    Download that, as any name you want, somewhere you won't delete it, and then browse to its location for this field.
    
    Note, my copy of the certificate is set to expire in February 2026 (you can verify this with `openssl x509 -text -in ca.pem | grep 'Not After'`{.bash}) so I'll need to get a new one before then.
    Assuming, of course, that UVA doesn't change how it wants us to authenticate before then...
    
    The CA certificate file contains only a publicly-available signature and does not need to be protected in any particular way.

CA Certificate Password
:   *none; network manager should notice this automatically and may or may not show the field*

User certificate
:   This is the P12 [personal digital certificate](#cert) used for netbadge, etc.
    They last 13 months (if memory serves) so you'll be getting a new one every year.

User certificate password
:   The password you set when you downloaded your personal digital certificate.
    
    It is likely that this field will be disabled, defaulting to the user key password field's value instead

User private key
:   The same as the user certificate.

    It is likely that this field will be disabled, defaulting to the user certificate field's value

User key password
:   The password you set when you downloaded your personal digital certificate.
    

# Getting a personal digital certificate {#cert}

In theory, UVA provides instructions for this in multiple places.
In practice, they have a few holes.

1.  Go to any netbadge site, including <https://netbadge.virginia.edu>.
1.  Click on the "Get one now!" link, which currently goes to <https://in.virginia.edu/installcert>.
1.  Open up the Firefox tab (or others if they ever update that) to find the UVA Network Setup Tool (Limited), which currently goes to <https://cloud.securew2.com/public/82116/limited/>.
1.  Due to bad design, that page will auto-detect "Unknown" but not run the "you selected Unknown" event handlers. So, pick any other OS from the drop down, then select Unknown again to trigger the onchange event to get the rest of the Unknown OS page.
    - Incidentally, when I looked at the source of this page they actually detected Linux, then turned Linux into Unknown before finishing, which round-about pretending to not know your OS might be why the event doesn't trigger automatically.
1.  Sign in, follow the prompts, and save the file.
    - You may get prompts to make the certificate MAC-address specific. This can add security in that the certificate will be refused if used to authenticate on a different MAC address. It's not much help, though, as MAC spoofing is relatively easy. It may stop a casual hacker from impersonating you, but not a determined one.
        - Each network card has its own 48-bit MAC address. To find yours, run `ip address`{.bash} and look for the line beginning "number: w"something (e.g., mine is `3: wlp3s0`; the exact name varies by Linux distribution). The line after that should have `link/ether` followed by six bytes in hex separated by colons. That's the MAC address you want.
        - You likely also have a wired port, typically beginning with an "e" (e.g. mine is `2: enp0s25`); that one is what you'd need if you wanted eduroam to work for a cable plugged in to UVA network (I've not tried that though).
        - If you have more than one wireless card, you'll need to figure out which one is connecting for you (or skip the MAC-specific option in your certificate download).

This file gives anyone that owns it **power to impersonate you**.
You should definitely store it such that only you can read it; I recommend storing it in a hidden directory with owner-only permissions, such as can be created via

```bash
mkdir ~/.certificates
chmod 700 ~/.certificates
```


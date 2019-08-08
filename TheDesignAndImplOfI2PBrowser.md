# The Design and Implementation of the I2P Browser

This document should be considered a draft unless otherwise spesified.

## Table of Contents

* [Introduction](#intro)
  * [Browser Component Overview](#browsercomponents)
  * [Differences with TBB](#difftbb)
* [Design Requirements and Philosophy](#designreq)
  * [Security Requirements](#secreq)
  * [Privacy Requirements](#privreq)
  * [Philosophy](#philosophy)
* [The Counterparty Model](#counterpartymodel)
  * [The goals of any Counterparty](#counterpartygoals)
  * [Counterparty Capabilities - Positioning](#counterpartypositioning)
  * [Counterparty Capabilities - Attack](#counterpartyattack)
* [Implementation]()
  * [Strict I2P and Internet Separation]()
  * [State Separation]()
  * [Disk Avoidance]()
  * [Application Data Isolation]()
  * [Cross-Origin Identifier Unlinkability]()


## <a name="intro"></a>Introduction

This document describes the I2P model, design reuirements and roughly implementation of the I2P Browser. At the time of writing the browser is at version 2.0 Beta 4.

This document is also meant to serve as a set of design requirements and to describe a reference implementation of a Private Browsing Mode that defends against surveillance and actors on the internet which don't intend anything good for the end user.

For more practical information regarding I2P Browser development, please have a look at our [Browser Hacking Guide](https://trac.i2p2.de/wiki/I2P_Browser_develop_n_hacks), in general it might be worth reading the Tor equalent found at [Tor Browser Hacking Guide](https://trac.torproject.org/projects/tor/wiki/doc/TorBrowser/Hacking) as the products are quite alike in their build process and the documents describe different pharses of the process.

### <a name="browsercomponents"></a>Browser Component Overview

The I2P Browser is based on [Mozilla's Extended Support Release (ESR) Firefox branch](https://www.mozilla.org/en-US/firefox/organizations/), while we do also have some [patches in common with TBB](https://gitweb.torproject.org/tor-browser.git) as well as some of our own to enhance privacy and security. Browser behavior is altered through both direct Firefox patches and functionality from the [I2PButton extension](https://github.com/mikalv/i2pbutton). As TBB, we also [change a number of Firefox
preferences](https://github.com/mikalv/test-i2p-browser/blob/i2p-browser-60.7.0esr-9.0-1-build3/browser/app/profile/000-tor-browser.js) from their defaults.

The I2P process management and configuration is accomplished through the [I2PButton extension](https://github.com/mikalv/i2pbutton), and for now it will only be compatible with Firefox and not other Mozilla products like Thunderbird and XULRunner which also seems long forgotten by Mozilla.

To provide users with optional defense-in-depth against JavaScript and other potential exploit vectors, we include [NoScript](https://noscript.net/). We also modify some of [NoScript](https://noscript.net/) preferences from their defaults.

### <a name="difftbb"></a>Differences with TBB

The Tor project has done a good job with their browser, and we share a lot of common privacy and security patches, however we don't intend to be a direct copy or be fully based upon TBB as it has a lot of patches related to Tor spesific functionality that I2P won't need.

Working together with the people over at TBB isn't totally out of the question from our end, but as long as they pretend we don't exists - it's not possible.


## <a name="designreq"></a>Design Requirements and Philosophy

The I2P Browser Design Requirements are meant to describe the properties of a Private Browsing Mode that defends against both network and local forensic enemies.

There are two main categories of requirements: Security Requirements, and Privacy Requirements. Security Requirements are the minimum properties in order for a browser to be able to support I2P and similar privacy proxies safely. Privacy requirements are the set of properties that cause us to prefer one browser over another.

While we will endorse the use of browsers that meet the security requirements, it is primarily the privacy requirements that cause us to maintain our own browser distribution.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).


### <a name="secreq"></a>Security Requirements

The security requirements are primarily concerned with ensuring the safe use of I2P, violations in these properties typically result in serious risk for the user in terms of immediate deanonymization and/or observability. With respect to browser support, security requirements are the minimum properties in order for I2P to support the use of a particular browser.

1. **Proxy Obedience**<br>
    The browser MUST NOT bypass I2P proxy settings for any content.
2. **State Separation**<br>
    The browser MUST NOT provide the content window with any state from any other browsers or any non-I2P browsing modes. This includes shared state from independent plugins, and shared state from operating system implementations of TLS and other support libraries.
3. **Disk Avoidance**<br>
    The browser MUST NOT write any information that is derived from or that reveals browsing activity to the disk, or store it in memory beyond the duration of one browsing session, unless the user has explicitly opted to store their browsing history information to disk.
4. **Application Data Isolation**<br>
    The components involved in providing private browsing MUST be self-contained, or MUST provide a mechanism for rapid, complete removal of all evidence of the use of the mode. In other words, the browser MUST NOT write or cause the operating system to write any information about the use of private browsing to disk outside of the application's control. The user must be able to ensure that secure deletion of the software is sufficient to remove evidence of the use of the software. All exceptions and shortcomings due to operating system behavior MUST be wiped by an uninstaller. However, due to permissions issues with access to swap, implementations MAY choose to leave it out of scope, and/or leave it to the operating system/platform to implement ephemeral-keyed encrypted swap.

### <a name="privreq"></a>Privacy Requirements

The privacy requirements are primarily concerned with reducing linkability: the ability for a user's activity on one site to be linked with their activity on another site without their knowledge or explicit consent. With respect to browser support, privacy requirements are the set of properties that cause us to prefer one browser over another.

For the purposes of the unlinkability requirements of this section as well as the descriptions in the implementation section, a URL bar origin means at least the second-level DNS name. For example, for mail.google.com, the origin would be google.com. Implementations MAY, at their option, restrict the URL bar origin to be the entire fully qualified domain name.

1. **Cross-Origin Identifier Unlinkability**<br>
    User activity on one URL bar origin MUST NOT be linkable to their activity in any other URL bar origin by any third party automatically or without user interaction or approval. This requirement specifically applies to linkability from stored browser identifiers, authentication tokens, and shared state. The requirement does not apply to linkable information the user manually submits to sites, or due to information submitted during manual link traversal. This functionality SHOULD NOT interfere with interactive, click-driven federated login in a substantial way.
2. **Cross-Origin Fingerprinting Unlinkability**<br>
    User activity on one URL bar origin MUST NOT be linkable to their activity in any other URL bar origin by any third party. This property specifically applies to linkability from fingerprinting browser behavior.
3. **Long-Term Unlinkability**<br>
    The browser MUST provide an obvious, easy way for the user to remove all of its authentication tokens and browser state and obtain a fresh identity. Additionally, the browser SHOULD clear linkable state by default automatically upon browser restart, except at user option.

### <a name="philosophy"></a>Philosophy

In addition to the above design requirements, the technology decisions about I2P Browser are also guided by some philosophical positions about technology.

1. **Preserve existing user model**<br>
    The existing way that the user expects to use a browser must be preserved. If the user has to maintain a different mental model of how the sites they are using behave depending on tab, browser state, or anything else that would not normally be what they experience in their default browser, the user will inevitably be confused. They will make mistakes and reduce their privacy as a result. Worse, they may just stop using the browser, assuming it is broken.

2. **Favor the implementation mechanism least likely to break sites**<br>
    In general, we try to find solutions to privacy issues that will not induce site breakage, though this is not always possible.

3. **Plugins must be restricted**<br>
    Even if plugins always properly used the browser proxy settings (which none of them do) and could not be induced to bypass them (which all of them can), the activities of closed-source plugins are very difficult to audit and control. They can obtain and transmit all manner of system information to websites, often have their own identifier storage for tracking users, and also contribute to fingerprinting.<br><br>
    Therefore, if plugins are to be enabled in private browsing modes, they must be restricted from running automatically on every page (via click-to-play placeholders), and/or be sandboxed to restrict the types of system calls they can execute. If the user agent allows the user to craft an exemption to allow a plugin to be used automatically, it must only apply to the top level URL bar domain, and not to all sites, to reduce cross-origin fingerprinting linkability.

4. **Minimize Global Privacy Options**<br>
    From our failure with today's I2P console we have learned the lession about confused users. Each option that detectably alters browser behavior can be used as a fingerprinting tool. Similarly, all extensions should be disabled in the mode except as an opt-in basis. We should not load system-wide and/or operating system provided addons or plugins.<br><br>
    Instead of global browser privacy options, privacy decisions should be made per URL bar origin to eliminate the possibility of linkability between domains. For example, when a plugin object (or a JavaScript access of window.plugins) is present in a page, the user should be given the choice of allowing that plugin object for that URL bar origin only. The same goes for exemptions to third party cookie policy, geolocation, and any other privacy permissions.<br><br>
    If the user has indicated they wish to record local history storage, these permissions can be written to disk. Otherwise, they should remain memory-only.


## <a name="counterpartymodel"></a>The Counterparty Model

### <a name="counterpartygoals"></a>The goals of any Counterparty

1. **Bypassing proxy settings**<br>
    The counterparty's primary goal is direct compromise and bypass of I2P, causing the user to directly connect to an IP of the counterparty's choosing.
2. **Correlation of I2P vs Outproxy Activity**<br>
    If direct proxy bypass is not possible, the counterparty will likely happily settle for the ability to correlate something a user did via I2P with their outproxy activity. This can be done with cookies, cache identifiers, JavaScript events, and even CSS. Sometimes the fact that a user uses I2P may be enough for some authorities.
3. **History disclosure**<br>
    The counterparty may also be interested in history disclosure: the ability to query a user's history to see if they have issued certain censored search queries, or visited censored sites.
4. **Correlate activity across multiple sites**<br>
    The primary goal of the advertising networks is to know that the user who visited websiteA.com is the same user that visited websiteB.com to serve them targeted ads. The advertising networks become our counterparty insofar as they attempt to perform this correlation without the user's explicit consent.
5. **Fingerprinting/anonymity set reduction**<br>
    Fingerprinting (more generally: "anonymity set reduction") is used to attempt to gather identifying information on a particular individual without the use of tracking identifiers. If the dissident's or whistleblower's timezone is available, and they are using a rare build of Firefox for an obscure operating system, and they have a specific display resolution only used on one type of laptop, this can be very useful information for tracking them down, or at least tracking their activities.
6. **History records and other on-disk information**<br>
    In some cases, the counterparty may opt for a heavy-handed approach, such as seizing the computers of all I2P users in an area (especially after narrowing the field by the above two pieces of information). History records and cache data are the primary goals here. Secondary goals may include confirming on-disk identifiers (such as hostname and disk-logged spoofed MAC address history) obtained by other means.

### <a name="counterpartypositioning"></a>Counterparty Capabilities - Positioning

The counterparties can position themselves at a number of different locations in order to execute their attacks.

1. **Outproxy**<br>
    The counterparties might run outproxies which they will try force onto users.
2. **Ad servers and/or Malicious Websites**<br>
    The counterparties can also run websites, or more likely, they can contract out ad space from a number of different ad servers and inject content that way. For some users, the counterparties may be the ad servers themselves. It is not inconceivable that ad servers may try to subvert or reduce a user's anonymity through I2P for marketing purposes.
3. **Local Network/ISP/Upstream Router**<br>
    The counterparties can also inject malicious content at the user's upstream router when they have I2P disabled, in an attempt to correlate their I2P and Outproxy activity.
4. **Physical Access**<br>
    Some users face counterparties with intermittent or constant physical access. Users in Internet cafes, for example, face such a threat. In addition, in countries where simply using tools like I2P and other darknet/VPN services is illegal, users may face confiscation of their computer equipment for excessive I2P usage or just general suspicion.

### <a name="counterpartyattack"></a>Counterparty Capabilities - Attacks

The counterparties can perform the following attacks from a number of different positions to accomplish various aspects of their goals. It should be noted that many of these attacks (especially those involving IP address leakage) are often performed by accident by websites that simply have JavaScript, dynamic CSS elements, and plugins. Others are performed by ad servers seeking to correlate users' activity across different IP addresses, and still others are performed by malicious agents on the I2P network and at national firewalls.

1. **Read and insert identifiers**<br>
    The browser contains multiple facilities for storing identifiers that the counterparty creates for the purposes of tracking users. These identifiers are most obviously cookies, but also include HTTP auth, DOM storage, cached scripts and other elements with embedded identifiers, client certificates, and even TLS Session IDs.<br><br>
    An counterparty in a position to perform MITM content alteration can inject document content elements to both read and inject cookies for arbitrary domains. In fact, even many "SSL secured" websites are vulnerable to this sort of [active sidejacking](https://seclists.org/bugtraq/2007/Aug/70). In addition, the ad networks of course perform tracking with cookies as well.<br><br>
    These types of attacks are attempts at subverting our Cross-Origin Identifier Unlinkability and Long-Term Unlinkability design requirements.

2. **Fingerprint users based on browser attributes**<br>
    There is an absurd amount of information available to websites via attributes of the browser. This information can be used to reduce the anonymity set, or even uniquely fingerprint individual users. Attacks of this nature are typically aimed at tracking users across sites without their consent, in an attempt to subvert our Cross-Origin Fingerprinting Unlinkability and Long-Term Unlinkability design requirements.<br><br>
    Fingerprinting is an intimidating problem to attempt to tackle, especially without a metric to determine or at least intuitively understand and estimate which features will most contribute to linkability between visits.<br><br>
    The [Panopticlick study](https://panopticlick.eff.org/about) done by the EFF uses the [Shannon entropy](https://en.wikipedia.org/wiki/Entropy_%28information_theory%29) - the number of identifying bits of information encoded in browser properties - as this metric. Their [result data](https://wiki.mozilla.org/Fingerprinting#Data) is definitely useful, and the metric is probably the appropriate one for determining how identifying a particular browser property is. However, some quirks of their study means that they do not extract as much information as they could from display information: they only use desktop resolution and do not attempt to infer the size of toolbars. In the other direction, they may be over-counting in some areas, as they did not compute joint entropy over multiple attributes that may exhibit a high degree of correlation. Also, new browser features are added regularly, so the data should not be taken as final.<br><br>
    Despite the uncertainty, all fingerprinting attacks leverage the following attack vectors:<br>

      a. **Observing Request Behavior**<br>
          Properties of the user's request behavior comprise the bulk of low-hanging fingerprinting targets. These include: User agent, Accept-* headers, pipeline usage, and request ordering. Additionally, the use of custom filters such as AdBlock and other privacy filters can be used to fingerprint request patterns (as an extreme example).

      b. **Inserting JavaScript**<br>
          JavaScript can reveal a lot of fingerprinting information. It provides DOM objects such as window.screen and window.navigator to extract information about the user agent. Also, JavaScript can be used to query the user's timezone via the Date() object, [WebGL](https://www.khronos.org/registry/webgl/specs/1.0/#5.13) can reveal information about the video card in use, and high precision timing information can be used to [fingerprint the CPU and interpreter speed](https://cseweb.ucsd.edu/~hovav/dist/jspriv.pdf). JavaScript features such as [Resource Timing](https://www.w3.org/TR/resource-timing/) may leak an unknown amount of network timing related information. And, moreover, JavaScript is able to [extract](https://seclab.cs.ucsb.edu/media/uploads/papers/sp2013_cookieless.pdf) [available](https://www.cosic.esat.kuleuven.be/fpdetective/) [fonts](https://hal.inria.fr/hal-01285470v2/document) on a device with high precision.

      c. **Inserting Plugins**<br>
          The Panopticlick project found that the mere list of installed plugins (in navigator.plugins) was sufficient to provide a large degree of fingerprintability. Additionally, plugins are capable of extracting font lists, interface addresses, and other machine information that is beyond what the browser would normally provide to content. In addition, plugins can be used to store unique identifiers that are more difficult to clear than standard cookies. [Flash-based cookies](https://epic.org/privacy/cookies/flash.html) fall into this category, but there are likely numerous other examples. Beyond fingerprinting, plugins are also abysmal at obeying the proxy settings of the browser.
      d. **Inserting CSS**<br>
          [CSS media queries](https://developer.mozilla.org/En/CSS/Media_queries) can be inserted to gather information about the desktop size, widget size, display type, DPI, user agent type, and other information that was formerly available only to JavaScript.

3. **Website traffic fingerprinting**<br>
    Website traffic fingerprinting is an attempt by the counterparty to recognize the encrypted traffic patterns of specific websites. In the case of I2P, this attack would take place between the Outproxy and the website.

4. **Remotely or locally exploit browser and/or OS**<br>
    Last, but definitely not least, the counterparty can exploit either general browser vulnerabilities, plugin vulnerabilities, or OS vulnerabilities to install malware and surveillance software. An counterparty with physical access can perform similar actions.<br><br>
    For the purposes of the browser itself, we limit the scope of this counterparty to one that has passive forensic access to the disk after browsing activity has taken place. This counterparty motivates our Disk Avoidance defenses.<br><br>
    An counterparty with arbitrary code execution typically has more power, though. It can be quite hard to really significantly limit the capabilities of such an counterparty. [The Tails system](https://tails.boum.org/contribute/design/) can provide some defense against this counterparty through the use of readonly media and frequent reboots, but even this can be circumvented on machines without Secure Boot through the use of BIOS rootkits.











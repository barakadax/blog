# The differences between surface web, deep web and dark web
- [Background](#background)
- [Gopher vs Websites](#gopher-vs-websites)
- [Surface Web](#surface-web)
- [Deep Web](#deep-web)
- [Dark Web](#dark-web)
- [Sources](#sources)

## Background

The origins of the web trace back to the **ARPANET** (Advanced Research Projects Agency Network) in the late 1960s,
Initially, the network was a medium to share files and access system resources directly,
There were no "websites" in the modern sense; users navigated file systems of remote servers.

In 1989, **Sir Tim Berners-Lee**, a British scientist at **CERN**, proposed a "universal linked information system." By combining Hypertext (HTML) with the Internet (TCP/IP),
He created the World Wide Web. Early web architecture was predominantly static, consisting of publicly linked documents.

## Gopher vs Websites

Before the World Wide Web became the dominant way to share information, the Gopher protocol was a popular alternative,
Developed at the University of Minnesota, Gopher was designed for distributing, searching, and retrieving documents over the Internet,
The Web eventually won due to its flexibility and visual capabilities.

| Feature | Gopher (FTP-like) | Websites (HTTP/HTML) |
| :--- | :--- | :--- |
| **Structure** | Strict hierarchy of menus and files (tree-like). | Flexible, non-linear network of hyperlinked documents. |
| **Content** | Primarily text-based; binary files handled separately. | Multimedia intensive (Text, Images, Video, Audio). |
| **Navigation** | Menu-driven; navigating through folders. | Link-driven; clicking hyperlinks to jump between pages. |
| **User Interface** | Plain text interfaces; very limited styling. | Rich graphical interfaces with CSS and JavaScript. |

## Surface Web

The surface web also known as the indexed web or visible web is the part of the web that is accessible to the public,
Indexed by search engines via crawlers.

## Deep Web

The deep web is the part of the web that is not indexed by search engines or easily accessible to the public,
Everything that requires login, template pages that are filled with information from database, for example profile pages in social networks, private messages, paywall content, cloud storage etc.

## Dark Web

The dark web is a subset of the Deep Web that is intentionally hidden and inaccessible through standard browsers,
It operates on **Overlay Networks** that require specific authorization or software,
The Dark Web utilizes cryptographic protocols such as **Onion Routing** and **Garlic Routing**:

- **Onion Routing (Tor):** When a user accesses a `.onion` site, their traffic is wrapped in multiple layers of encryption and bounced through a series of volunteer nodes,
Each node only peels back one layer of encryption to know where to send the data next, ensuring that no single point in the path knows both the source and the destination.
- **Garlic Routing (I2P):** Utilized by the **I2P (Invisible Internet Project)** network, this is a variation of onion routing that encrypts multiple messages together (like cloves in a bulb of garlic)
It makes it even harder for an attacker to perform traffic analysis,
Unlike Tor's circuit-based approach, I2P uses "tunnels" which are unidirectional, meaning incoming and outgoing traffic follow different paths.

The dark web has both legal and illegal content,
Ethical reasons for using the dark web include whistle blowing, journalists, communication between countries and armies.

## Sources

- [TechQuickie youtube video](https://www.youtube.com/watch?v=nKrODPtVinw)
- [Surface web wiki](https://en.wikipedia.org/wiki/Surface_web)
- [Deep web wiki](https://en.wikipedia.org/wiki/Deep_web)
- [Dark web wiki](https://en.wikipedia.org/wiki/Dark_web)

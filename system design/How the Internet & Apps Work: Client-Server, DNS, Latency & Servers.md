### Client – Server

* **Client:** Your device (phone, laptop, watch, smart fridge) that asks for something—like opening Instagram to see your feed.
* **Server:** Just a regular computer owned by the company (e.g., Instagram's computer) sitting somewhere else that holds the data and serves it to you. Your phone only has ~128 GB storage; it can't store all of Instagram, so it asks the server.
* **Request & Response:** Every interaction on the internet is a two-way conversation:
* **Request:** Client asks ("Show me my feed").
* **Response:** Server sends it back ("Here is your feed").



---

### IP Address

* Every device connected to the internet has a unique virtual address, just like a physical home address—without a *From* and *To* address, you can't send a letter.
* **IPv4 Format:** Four numbers separated by dots, each ranging from `0` to `255` (256 possible values per slot).
* Example: `10.7.8.200` or `5.4.200.1`
* Range: `0.0.0.0` to `255.255.255.255` (~4.3 billion total addresses; IPv6 exists too, covered later).



---

### DNS (Domain Name System)

* **The Problem:** Humans are bad at remembering random numbers like `5.4.200.1` for every website or app, just like we don't memorize 10-digit phone numbers for every friend.
* **The Solution:** DNS is the **phone book of the internet**. Just like your phone's contacts map a name ("Raju") to a number, DNS maps human-friendly domain names (`instagram.com`, `openai.com`) to their actual IP addresses.

---

### DNS Resolver

* A special server whose IP address is already known/saved in your device's settings (e.g., Google's public DNS `8.8.8.8`).
* **How the lookup works:**
1. You type `instagram.com` in your browser/app.
2. Your phone silently asks the **DNS Resolver**: *"What is the IP address for `instagram.com`?"*
3. The resolver returns the IP address (ideally pointing you to the **nearest server**—e.g., Delhi or Mumbai instead of the US).
4. Now your phone knows both its own IP and Instagram's IP, so it sends the actual request.



---

### Latency & Packets (How Data Travels)

* **Packets:** Large data (like your feed or a 10-page book) isn't sent in one giant chunk. It gets broken into small pieces called **packets** (like 10 separate pages).
* **Routing & Reassembly:** Packets travel through different routes—wireless radio waves, routers, and undersea fiber-optic cables—and may arrive out of order (e.g., page 5 before page 2). Your phone puts them back in the right order before showing you the final content.
* **Latency (Why Server Location Matters):** Even over fiber-optic cables moving at the speed of light (~300,000 km/s), distance takes time—you can't beat physics.
* Requesting data from a US server (~14,000 km away from India) adds noticeable delay (~0.5–2 seconds).
* Companies place servers locally (e.g., in Delhi or Mumbai) so packets travel shorter distances and load in **less than 1 second**.



---

### Server Internals (CPU, RAM, Disk)

A server is just a computer with three core parts working together to handle your request:

| Component | Analogy | Role in a Server |
| --- | --- | --- |
| **CPU** | **Brain** | Executes code, processes logic, and decides what personalized feed/data to send you. |
| **RAM** | **Study Desk** | Fast, temporary workspace holding active data/books you need *right now*. Limited in size (e.g., 16 GB)—if it fills up, things slow down ("memory-intensive"). |
| **Disk** | **Bookshelf** | Large, permanent storage holding everything long-term. Slower to fetch from than RAM, so active items get loaded onto the "study desk" (RAM) while you're working on them. |

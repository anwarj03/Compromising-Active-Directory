<p align="center">
  <img src="assets/ad-logo.png" alt="Compromising Active Directory logo" width="180">
</p>

# Compromising Active Directory

<p align="center">
  <img src="https://img.shields.io/badge/Active%20Directory-0078D4?style=flat-square&logo=windows&logoColor=white" alt="Active Directory">
  <img src="https://img.shields.io/badge/TryHackMe-C11111?style=flat-square&logo=tryhackme&logoColor=white" alt="TryHackMe">
  <img src="https://img.shields.io/badge/Cybersecurity-111827?style=flat-square" alt="Cybersecurity">
  <img src="https://img.shields.io/badge/Red%20Team-991B1B?style=flat-square" alt="Red Team">
  <img src="https://img.shields.io/badge/Windows%20Security-0078D4?style=flat-square&logo=windows&logoColor=white" alt="Windows Security">
</p>

## Technologies and techniques

| Area | Covered in these notes |
| --- | --- |
| **Active Directory protocols** | LDAP, Kerberos, NTLM/NetNTLM/NTLMv2, SMB, DNS, LLMNR, NBT-NS, and WPAD |
| **Tools and scripts** | Responder, Hydra, Hashcat, PowerShell/PowerPXE, nslookup, and custom Python scripts |
| **Windows deployment and credentials** | PXE/MDT, configuration-file credential discovery, and McAfee SiteList password decryption |
| **Practice environment** | TryHackMe AttackBox and the Compromising Active Directory learning path |

The technologies table lists tools and protocols documented in the walkthrough.

## Learning path

### 1- Breaching Active Directory

[Full walkthrough](Walkthroughs/AD/1-%20Breaching%20AD.md)

#### The Big Picture: Why do attackers need a "first password"?

Imagine a company as a huge office building with a very strict security guard at the entrance. Inside, the guard keeps a master list of everyone who works there. This list is **Active Directory (AD)**. It decides who can open which doors.

- A **hacker outside the building** has no name on the list, so the guard won't let them in.
- Once they get **any employee's badge** (username + password), they can walk in and start looking around.

This room is about **how to get one employee's badge**. It doesn't matter if the badge belongs to a junior worker with almost no access. Just having a valid badge is enough to start exploring the building.

**Key idea:** getting _any_ valid login is the first goal. Getting _powerful_ access comes later.

---

#### The 5 Ways to Get a Badge

Think of each method as a different way to trick or find a way past the guard.

##### Method 1: Guessing passwords on a website (Password Spraying)

**Analogy:** Imagine a building with 100 employees. Instead of trying 100 passwords on one employee's door (the guard would lock that door after a few tries), you try **one common password** on **all 100 doors**. Eventually one door opens.

- Many companies set a default password like `Changeme123` for new employees.
- Some people never change it.
- The website says "wrong password" (HTTP 401) or "correct password" (HTTP 200), so the attacker knows which guesses worked.

**Why it works:** people forget to change default passwords.

---

##### Method 2: Stealing the password from an app's settings (LDAP Pass-back)

**Analogy:** A printer in the lobby has to check employee badges with the main office. The printer stores the office's "login" in its settings. The settings page is open with a weak admin password like `admin`.

The attacker changes the printer's settings so it sends its login to the **attacker's computer** instead of the office. When the printer tries to "test" the connection, it sends the login straight to the attacker, who reads it.

**Why it works:** the printer trusts whatever server address it's told to use, and it sends the password in plain text.

---

##### Method 3: Tricking computers on the network (Responder + Relay)

**Analogy:** In an office, when someone asks "Where is the file server?", people shout the answer to the whole room. An attacker in the room shouts back first: "It's over here, with me!" Everyone sends their login to the attacker.

The attacker can then do one of two things:

- **Record the login and try to crack it later** (slow, but it works if the password is weak).
- **Pass the login along to the real server immediately** (fast, and the attacker gets in right away). This is called **relaying**.

**Why it works:** Windows uses these "shouting" requests (LLMNR, NBT-NS) to find computers, and they don't check who is answering.

**The defense:** turn on **SMB signing**. This is like requiring a signature that the attacker can't copy, so the relay fails.

---

##### Method 4: Stealing the password from the setup system (PXE / MDT)

**Analogy:** When a company sets up a brand-new computer, it uses a "setup kit" that contains a temporary login so the computer can join the office network. That setup kit sometimes gets left in a shared folder where anyone can copy it.

The attacker:

1. Downloads a small config file that describes the setup kit.
2. Uses that file to find the full setup kit.
3. Downloads it and opens it.
4. Finds a text file inside (`bootstrap.ini`) that contains the temporary login in plain text.

---

##### Method 5: Finding passwords left on a computer (Config Files)

**Analogy:** You broke into one employee's desk. In the drawer, you find a sticky note with the password for a shared account. You don't know which door it opens yet, but it's a valid key.

On a computer, programs sometimes save their login information in files. The McAfee security program here saved its login in a small database file (`ma.db`). The password was scrambled, but with a scrambling method that everyone knows, so it's easy to unscramble.

**The lesson:** hiding a file is not the same as protecting it.

---

#### The 3 Things to Remember Most

1. **The goal is any valid login**, not a powerful one.
2. **Each method exploits a habit or a setting**: default passwords, plain-text sending, trusting network "shouting," leftover setup files, and passwords saved in files.
3. **The defenses are simple habits**: change default passwords, don't expose login pages to the internet, turn on SMB signing, and give accounts only the access they need.


### 2- Enumerating Active Directory

[Full walkthrough](Walkthroughs/AD/2-%20Enumerating%20Active%20Directory.md)

#### The Big Picture: Now that you have a badge, what can you learn about the building?

Back to the office analogy. In the last room you got **one employee's badge**. This room is about **reading the building's internal records with that badge**: who works there, what groups they belong to, what the password rules are, and where the paths to the top office (Domain Admin) run.

Think of it like this: you're a visitor with a guest badge. You can't open the executive office, but you _can_ read the directory in the lobby, the org chart on the wall, and the building's rulebook. That information tells you exactly where to go next.

**Key idea:** enumeration = gathering information with a valid login. It is a map-making step, not the break-in itself.

---

#### The 4 Ways to Read the Records

##### Method 1: Mmc (the visual org chart)

**Analogy:** A window that shows the company directory with folders for each department.

- You open the AD management tools on a Windows machine that is connected to the domain.
- You click through the departments, look at users, and check which groups each user belongs to.
- It's easy to use, but you need a desktop session (RDP) to use it.

##### Method 2: `net` commands (the quick phone call)

**Analogy:** Calling the front desk and asking short questions: "Who works here?" "Who is in the Admins group?" "What are the password rules?"

- Commands like `net user /domain`, `net group /domain`, `net accounts /domain`.
- Simple, built into Windows, and not usually monitored. Good when you only have a command prompt, for example through a phishing payload.
- Limitation: some output gets cut off, for example when a user is in more than ten groups.

##### Method 3: PowerShell (the detailed file request)

**Analogy:** Asking the records office for a precise report, with filters, instead of a quick question.

- Cmdlets like `Get-ADUser`, `Get-ADGroup`, `Get-ADGroupMember`, `Get-ADDomain`.
- Returns much more detail than `net`.
- Limitation: defenders watch PowerShell closely, and you may need to install the AD tools first.

##### Method 4: BloodHound (the map of routes)

**Analogy:** Instead of a list of rooms, you get a floor plan with arrows showing "if you get into room A, you can reach room B, then the executive office."

- **SharpHound** is the collector. It walks the domain and saves the data as a ZIP file of JSON.
- **BloodHound** is the viewer. It loads that data into a graph database (Neo4j) and draws the attack paths.
- Its famous idea: _"Defenders think in lists, attackers think in graphs."_
- Limitation: SharpHound is noisy and often caught by antivirus or EDR.

---

#### Concepts You Must Understand

**1. Why DNS matters.** Kerberos tickets are tied to hostnames, not IP addresses. If DNS is broken, your tools fail. When something doesn't work, check DNS first.

**2. Hostname vs IP authentication.** Using a hostname (`\\za.tryhackme.com\SYSVOL`) makes Windows use Kerberos. Using an IP (`\\<DC IP>\SYSVOL`) forces NTLM. Defenders may watch for certain Kerberos attacks, so switching to NTLM can be a stealth trick.

**3. Runas /netonly.** This injects your stolen credentials into the network side of a process without making your computer domain-joined. Local commands use your own account; network connections use the stolen one.

**4. Password policy matters for spraying.** `net accounts /domain` tells you the lockout threshold and the password rules. This tells you how many guesses are safe before accounts lock.

**5. Nested groups hide power.** A user in "Tier 0 Admins" is effectively a Domain Admin if that group is nested inside "Domain Admins." BloodHound finds these hidden paths automatically.

**6. Sessions change, structure doesn't.** The AD structure is fairly stable, but who is logged in changes all the time. Re-run session collection regularly.

---

#### The 3 Things to Remember Most

1. **Four tools, four levels of detail:** MMC (visual), `net` (quick, stealthy), PowerShell (detailed), BloodHound (attack paths).
2. **BloodHound shows paths, not lists.** Know that SharpHound collects and BloodHound visualizes.
3. **DNS and Kerberos vs NTLM** are the reasons behind many "it doesn't work" moments.

### 3- Lateral Movement and Pivoting

[Room notes](Walkthroughs/AD/3-%20Lateral%20Movement%20and%20Pivoting.md)

### 4- Exploiting Active Directory

[Room notes](Walkthroughs/AD/4-%20Exploiting%20Active%20Directory.md)

### 5- Persisting Active Directory

[Room notes](Walkthroughs/AD/5-%20Persisting%20Active%20Directory.md)

### 6- Credentials Harvesting

[Room notes](Walkthroughs/AD/6-%20Credentials%20Harvesting.md)



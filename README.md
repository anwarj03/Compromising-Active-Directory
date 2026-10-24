<p align="center">
  <img src="assets/ad-logo.png" alt="Compromising Active Directory logo" width="180">
</p>

# Compromising Active Directory

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

[Room notes](Walkthroughs/AD/2-%20Enumerating%20Active%20Directory.md)

### 3- Lateral Movement and Pivoting

[Room notes](Walkthroughs/AD/3-%20Lateral%20Movement%20and%20Pivoting.md)

### 4- Exploiting Active Directory

[Room notes](Walkthroughs/AD/4-%20Exploiting%20Active%20Directory.md)

### 5- Persisting Active Directory

[Room notes](Walkthroughs/AD/5-%20Persisting%20Active%20Directory.md)

### 6- Credentials Harvesting

[Room notes](Walkthroughs/AD/6-%20Credentials%20Harvesting.md)



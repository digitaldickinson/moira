# Moira - a lightweight newsroom computer system ##

<img width="1439" height="685" alt="image" src="https://github.com/user-attachments/assets/fec69e13-aaa1-478c-817c-4bea9f53fab8" />

## Core Principles.
Moira is designed to be a robust, multiuser, newsroom computer system. Think iNews, ENPS, CUEZ or Burli but simpler. Simpler to use and simpler to set up. 

It is built on a few core principles
- Lightweight - All it needs is one HTML file and an internet connection. No local dependencies, servers or anything else.
- Fast - it is designed to be quick and easy to set up.
- Free - it is designed to have no running costs (other than the internet). No subscriptions or dedicated hardware
- Industry adjacent - all the terminology, workflow, and (where possible) functionality should be as similar to professional practice as possible. You should recognise iNews functions in Moira and Moira functions in iNews

But those principles mean it's not perfect. They mean comprises and challenges. Perhaps the biggest is the use of the FIRESTORE database. This is both a vulnerability and a barrier - these things are never easy to set up for non-geeks. However, I hope the setup process is as easy as possible, given the functionality on offer. 

<img width="663" height="379" alt="image" src="https://github.com/user-attachments/assets/1c9eb518-8cdb-423a-a579-e035f8ab5d44" />

See [CHANGELOG.md](CHANGELOG.md) for version history.

## Changelog

### v2.6.0
**Presenter fired clips now work properly **
I noticed a real issue with clip buttons in the presenter view - they only worked locally.  So if you had a presenter view open, it would only trigger sound on the local device. Now it's set that whichever machine initiates TX mode, will become the master machine and playback will be limited to that machine. Outside of TX mode all machines connected to the database and the media folder will preview clips so you can rehearse.

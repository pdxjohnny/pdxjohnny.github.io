+++
date = 2021-02-10T06:00:00Z
lastmod = 2021-02-10T06:00:00Z
author = "default"
title = "Personal Digital Security"
subtitle = "How to secure your digital life"
+++

This post will cover steps you can take to better secure the digital aspects of
your life. We will cover some applications (software) and devices (hardware)
that you can use.

- [High Level Guidance](#high-level-guidance)
- [Applications](#applications)
- [Devices](#devices)
- [Personal Considerations](#personal-considerations)
- [Threat Considerations](#threat-considerations)
- [Rational](#rational)
- [Theory](#theory)

# High Level Guidance

This high level guidance is meant to be generic, since individuals have
different situations, hardware, and software. If you have a specific case you
need help with, reach out to me and I will try to give you more clarity.

## Update and Encrypt

These are two high level concepts we'll be following.

- Update

  - Make sure you're running the latest software. Ideally this also means
    running on hardware with the latest security features.

- Encrypt

  - Make sure that your talking over encrypted channels. To your friends, to
    your bank, wherever possible.

- Miscellaneous

  - Make long, unique passwords.

    - Use a password manager

### Update

- Make sure that auto updates are on on all your devices.

  - If there is a setting for frequency, make it as frequently as possible.

  - There is usually a device level setting found in the devices main settings
    application.

  - There may be a setting for the pieces of software on the device. Found in an
    app store or something like that.

- If you can afford it. Buy the latest versions of whatever devices you use.

  - This is particularly important if you are concerned your device might
    be physically accessible by someone you don't trust.

### Encrypt

- Use encryption wherever possible

  - You should always choose ways of communicating that are encrypted. You do
    not know what information, no matter how inconsequentiall it seems to you,
    might be of use to compromise the security of seemingly unrelated areas.

  - Communicating using encryption means that there has been some control placed
    on who can see the contents of your communication. The level of control over
    who can see communications is

- Use end-to-end encryption whenever possible

  - End-To-End encryption is used to describe a situation where only the
    sender and the recepieant can see a message. By definition, anything that
    is not end-to-end encrypted could possibly be seen by someone other than
    your intended recipient.

# Applications

Here are some applications you can use to increase your security.

## Signal

Signal is an end-to-end encrypted messaging application.

- [Install Signal](https://signal.org/install)

You can be confident that you are communicating securely if you have an updated,
recent phone, and you have verified the safety number of the person you are
talking to over signal. For the best possible assurance that you are talking to
who you think you are talking to, verify safety numbers in person by scanning
each others QR code safety numbers.

Do not set a PIN. This will make it so that your data is backed up on their
servers. This is convenient for you if you switch devices, but means that they
could potentially decrypt your messages and contacts if they guessed your PIN
(which is easy to do).

https://support.signal.org/hc/en-us/articles/360007059792-Signal-PIN#pin_disable

Make sure to read [Threat Considerations: Governments](#governments---police)
too.

# Devices

These are some devices which have reasonably good security, or can be used to
improve your security posture.

For any device, security is typically increased when the device is powered off.
If you are in a situation where you think your device might be taken from you,
**TURN IT OFF**.

## Chromebooks

A Chromebook is in my opinion the most secure consumer laptop available at the
moment. Their lack of ability to install software significantly decreases the
amount of attacks that can be done on them ([Theory: Attack
Surface](#attack-surface)).

# Personal Considerations

> This section is in progress

# Threat Considerations

> This section is in progress

## Apple

Apple does a very good job, they also tend to care about their users privacy
more than most other companies. This means they work hard than most others to
protect it by building secure products.

## Google

## Governments - Police

This goes for all government entities. If the government is in the list of
people you're worried about, do not use a fingerprint, face unlock, or any other
non-password based method to unlock your devices. There are laws protecting your
password in your head but not your fingers and face from being used to unlock
your devices.

If you think they are going to get your device, **TURN IT OFF**. This is the
best thing you can do to stop them from getting into it. Make sure it is off
before interacting with them.

## Governments - Three Letter Agencies

Using open source software is generally a good idea here. With Open Source, even if you dont know how to program, someone who does has looked at the code and you'd probably be able to find out through minimal searching if anyone has found any reason not to trust the open source project you're considering using. With closed source stuff, we usually only find out if it's doing something nefarious if a security researcher got bored in their free time and tried to hack a non-open source piece of software, which happens less often.

Related to the choice of using open source applications: https://www.vice.com/amp/en/article/akgkwj/operation-trojan-shield-anom-fbi-secret-phone-network

# Rational

Let's go into detail on why the above guidance holds true broad spectrum.

### Update

Updating is main way we respond to vulnerabilities. The software and hardware
our devices run on will enviably suffer from vulnerabilities. This is why we
MUST update them.

- Auto Updates

  - Hardware and software vulnerabilities are identified on a regular basis.
    Updates contain fixes to vulnerabilities. If your device has a
    vulnerability, an update may "patch" the vulnerability. A "patch" for a
    vulnerability is a fix which makes your device no longer vulnerable. If you
    do not update, you will not get the "patch"es that fix vulnerabilities, and
    your device will be vulnerable.

- If you can afford it. Buy the latest versions of whatever devices you use.

  - In a given product line, such as the Google or Apple phones, devices are
    made of hardware components. The product lines have versions, the components
    within those products also have versions. When you buy the latest version of
    a device, you're doing an update of your hardware. Hardware vulnerabilities
    usually a concern when someone might have physical access to your device.
    Typical examples of this include airport security, police, thieves, abusive
    partners, etc.

### Encrypt

- Use encryption wherever possible

  - You should always choose ways of communicating that are encrypted. You do
    not know what information, no matter how inconsequentiall it seems to you,
    might be of use to compromise the security of seemingly unrelated areas.

  - Communicating using encryption means that there has been some control placed
    on who can see the contents of your communication. The level of control over
    who can see communications is

- Use end-to-end encryption whenever possible

  - End-To-End encryption is used to describe a situation where only the
    sender and the recepieant can see a message. By definition, anything that
    is not end-to-end encrypted could possibly be seen by someone other than
    your intended recipient.

# Theory

> This section is in progress

## Attack Surface

The potential for a there to be a vulnerability is directly related to the
number of components in a system and the quality of those components.

## Threat Model

For a given situation.

- What are your assets?

  - Assets are things you may want to protect.

- Who are the actors?

  - Who are the potential people or entities involved in a situation? At a
    minimum there is you, and the "attacker".

## MITM

Man-in-the-middle or person-in-the-middle attacks are when someone inserts
themselves into a thought to be private conversation.

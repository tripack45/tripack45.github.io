---
title: Have I been pawned?
date: 2018-03-01 00:04:50
tags:
- Security Leaks
- Tools
- Password
categories:
- [Tech, Security]
---

![Screenshot of "Have I benn pawned" website at 'https://haveibeenpwned.com'](/images/haveibeenpwned.png)

I know I am supposed to make weekly updates. Sorry fellows. We both know something enourmous happenned this week. A comment on recent events is going to be published someday soon. I have almost done writing it. I need time to polish it, and more importantly make sure it will not cause any trouble. As a part of my trying to keep up with the weekly update, I'm introducing you to an interesting website, [Have I been pawned?](https://haveibeenpwned.com). 

<!-- More -->

# What is it?

Security breaches always happen. When a security breach happens, sometimes hackers may get access to sensitive information: usernames, passwords, etc. The hacker may be able to download the database from the breached website, and sell the database for a good price. In this case, if you are one of the users, you are pawned. 

Normally you will not have access to these databases. Selling these type of information is of course illegal, and those information is only sold at very secretive places. This website collects these databases for you, so that if you ever worried whether your username/password is leaked, you can go check it up. 

This service is brought to the world by [Troy Hunt](https://www.troyhunt.com/). Clicking this link takes to his technical blog, which is itself very interesting. For example, he wrote about how the website handles frequent searches in 500 million usernames/passwords. Many things you don't see in normal "technical" blogs (for intance, this one).

# Trial run

This website provides two usages, you can
- Input a username/email, see if it appears in any databse, or
- Input a password. If it is reported to appear in any of the databases, the password is leaked in **PLAIN TEXT**.

So I give it a trial run. I first tried out my recent account names, every thing is fine. Then I tried one of deprecated email, one that I registered in second year primary school. 4 Breaches pops up. 

![4 websites breached contains the account. Reminds of a lot things.](/images/haveibeenpwned-usr.png)

And when I searched the password I used.

![A password leaked in plain text.](/images/haveibeenpwned-pwd.png)

# How your password is protected?

OK. Now this is interesting. To cover the background, normally when websites do **NOT** save your password in plain text. It saves it's hashed form. When you logon to the website, your password is sent to the website in **Plain Text**, then hashed, and compared to the value stored in the database. If the website is careful enough, your password is hashed in your browser, before it reaches the server. This prevents transmitting plain text password over the Internet. The reason why passwords are stored in hashed forms, even when they are transmitted in plain text, has a multiple function:

- It prevents protection from staff. Websites hire database administrators (DBAs) to manage the database, and you certainly would want the person to know the plain text password of all its members. The DBA can manipulate these information (backup/restore, move, delete) without knowing the actual plain text password.
- In very unfortunate cases where the database is exposed, only user name is exposed, not the password. Hackers cannot retrieve the plain text from it, in which case they cannot log on to the website breached.
- By the way, this is also exactly the reason why you can only "reset your password" if you forgot the old one, instead of retreiving the old one. The website has no idea what the old one is itself.

Despite all these theory, hackers still have their ways of dealing with these "difficulties". First of all, it is very hard to create a secure hash function. So people have developed a few functions that are widely used, this incudes MD5, SHA1, etc. Since:

- People tends to use weak passwords such as "12345". Hackers can list these widely use password, hash them, and compare the hashed value with ones found from the database. This is usually refered a *dictionary attack*.
- Hackers developped what is called a [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table). The goal is to brute force the plain text from the hashed value, and the rainbow table is a very delicately constructed means of space-time trade off. Recall that two different strings might hash to the same function. A hacker does not need to find the exact original password, but merely a collision.

By the way, to defend these two types of attack, especially the second type, people developped *salt*. When you register for the website, a random string is generated for you. When you input your password, it is concatenated with the string, then hashed and stored. The salt itself is stored along with the password. That random string is called the *salt*. Note that add the salt does not really make your password safer. When a security breach happened, hackers WILL have access to the plain text of the salt. What id does it to prolong the time it takes for the hacker to find out the plain text. Essentially, for each different salt, there will be a different "dictionary" and different "rainbow table". A hacker most likely do not have the luxury to own a dictionay/table for every possible salt. 

# A look back at the situation

This password of mine is 10+ characters in length, contains digits, alphbetic characters and symbols. It does not contain any of the English word, or PinYin. So it is very unlikely that the password is used by others, or appear in any common diction. Since the rainbow table does not give back original plain text, this leaves us the only possiblity, that some website failed to do its job, and stored the password in plain text. 

Now we take a look back on the 4 leaks. The GFan and Tumblr leak all use hashed function, so they should be safe. Tumblr even used salt, which basically rule it out. 

The Adobe case is much more complicated yet interesting. There is a [nice post](https://nakedsecurity.sophos.com/2013/11/04/anatomy-of-a-password-disaster-adobes-giant-sized-cryptographic-blunder/) about this incident. Clearly, Adobe did **NOT** hash the password, since hash functions should return even length outputs. It used some sort of symmetric encryption scheme, which is very stupid. Further more, Adobe leaked password hints from the file. This is a huge issue, because now hackers are able to collect all the hints of passwords with identical "encrypted" values, and guess the plain text, making the entire leak the largest crosswords ever. 

![XKCD commic on the Adobe blunder, from 'https://xkcd.com/1286/'](/images/encryptic.png)

Despite the ridiculous encryption scheme, I doubt it is Adobe that leaked the password. A hacker still need to break the encryption to find the actual plaintext. Luckily it seems that the encryption is still in tact.

That leaves us with Netease. The website claims that Netease stores my password in Plain text. Now we found the murder.

# What's special about it

So you probably know a few websites that does exactly the same thing, or did they? The creator of this website cares a lot of authenticity. For instance, the authors not only collects the database, he also tries to [verify them as valid](https://www.troyhunt.com/heres-how-i-verify-data-breaches/). Companies sometimes release false stories, claiming that their competors have been hacked, to get an advantage. Some people made up database, so that they can sell them to others. The author of the website tries to make sure that the leak is actual. Read the post to find more. 

The website also cares about sensitive information. Sometimes just knowing you are on some website hurts you. Think about adult contents. 

> ... However, certain breaches are particularly sensitive in that someone's presence in the breach may adversely impact them if others are able to find that they were a member of the site. These breaches are classed as "sensitive" and may not be publicly searched. ... There are presently 18 sensitive breaches in the system including Adult Friend Finder, Ashley Madison, Beautiful People, Brazzers, CrimeAgency vBulletin Hacks, Fling, Freedom Hosting II, Fridae, Fur Affinity, HongFire, Mate1.com, Muslim Match, Naughty America, Non Nude Girls, Rosebutt Board, The Candid Board, The Fappening and YouPorn.

There is also the "unverified" breaches:

> Some breaches may be flagged as "unverified". In these cases, whilst there is legitimate data within the alleged breach, it may not have been possible to establish legitimacy beyond reasonable doubt. Unverified breaches are still included in the system because regardless of their legitimacy, they still contain personal information about individuals who want to understand their exposure on the web.

There are a lot of breaches in China are marked as unverified. The author [have a post explaining the reason](https://www.troyhunt.com/handling-chinese-data-breaches-in-have-i-been-pwned/). He contributes the reason mainly to language barriers, and mixed user replies.

> (Being not easy to verify a breach in China) is due to a combination of language barrier (there's Google translate, but that only takes you so far), breach origin (site domain names often don't match the name of the service) and a general lack of understanding about how some of the sites implicated in these breaches are used by the local population. 

There is also how he feels on the Internet in China:

> The point of all this is that China is a very different place to so much of the rest of the world, including neighbouring Asian countries. Yet one thing that's not different is that like everywhere else, data breaches are a serious ongoing problem. In fact in some ways it's worse in China due to their massive size combined with a very different social tolerance for privacy. In my experience ... there's just not the same outrage we'd have here knowing that others have access to our personal data. I want to be careful how I put this and caveat it with "my personal experiences", but where we'd be very unhappy ... they accept it as a more normal part of life.

I'm quoting it because I find the observation is right. It does seem that people here does not value "privacy" as much as Westerners do. 
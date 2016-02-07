# Introduction #

The hydraproject Sync API is RESTful, pull-driven and XML friendly.

Each server is solely responsible for keeping its data up to date by making requests to the fellow servers in its "federation".

The following will assume that we are an admin of a hydraproject server that will be running on the domain name 'mongoose.com'.  Any valid domain names on the Internet can of course run THP servers.

# Federated Sites List #

Implementors are free to implement the 'federation list' of trusted sites however they see fit.  In the spec rails app, this is a YAML-encoded file (see: config/federation.yml.example).

```
--- 
- domain: foo.org
  passkey: wafflesandjam69
  api_url: http://foo.org/api
  announce_url: http://foo.org/tracker/{{passkey}}/announce
- domain: bar.net
  passkey: pigsinablanket42
  api_url: https://www.bar.net/hydra/api.php
  announce_url: http://www.bar.net/hydra/announce.php/{{passkey}}
  ip_required: 66.29.132.45
```

Required fields:
  * domain - domain name of the remote server
  * passkey - secret password key used for authentication
  * api\_url - fully qualified URL of the remote API endpoint (note: secure connections "https" are allowed and encouraged if possible)
  * announce\_url - a template for the announce URL, which includes a template variable "{{passkey}}".  When the actual .torrent file is downloaded by a user, the bencoded 'announce-list' will contain these announce\_urls, only with the "{{passkey}}" template var substituted with the actual downloading user's passkey.

Optional field(s):
  * ip\_required - The IP Address of the remote client.  If a different IP than the one listed attempts to make a request, it should be denied.

In the above example, our mongoose.com Hydra server has 2 trusted sites in its network, running on domains foo.org and bar.net.  Requests coming from bar.net should originate from the IP 66.29.132.45; the API endpoint URL is secure (**https**) and appears to be running a PHP implementation of hydraserver.

# Function Calling #

Endpoint URLs can be any fully-qualified URL.  Parameters are specified directly after the endpoint URL or as HTTP POST variables.

Examples of acceptable API endpoint URLs:
  * http://foo.org/api
  * http://foo.org/hydra/pickles_and_jam/api.py
  * http://api.foo.org/api/index.php
  * https://foo.org/api

API endpoint URLs must **not** contain a question mark ("?") --  as this will be added by the client when building the request URL.

Which API function is being called is specified via the 'method' parameter passed via the URL string as HTTP GET or via HTTP Post.

For example, to call the 'time' method on each of the above examples, we'd call:
  * http://foo.org/api?method=time
  * http://foo.org/hydra/pickles_and_jam/api.py?method=time
  * http://api.foo.org/api/index.php?method=time
  * https://foo.org/api?method=time

# List of Allowed Methods #

The following is the current list of allowed API methods:

['time', 'list\_users', 'list\_transfer\_stats', 'list\_torrents', 'get\_torrent']

# Authentication #

To authenticate, clients must pass their passkey as an additional request parameter.  Note: passkeys must be unique within a network.

Over a secure, **https** connection, passkeys can be passed via HTTP GET, as in the following:
  * https://securefoo.org/api?method=time&passkey=bacon420

It is advised though that HTTP POST be used for submitting the passkey param to the remote API server.

The following example uses curl to call the time method, passing in the passkey via POST:
```
  curl -d "passkey=bacon420" http://foo.org/api?method=time
```

# Time Function (time) #

This function can be used for debugging and testing (to ensure authentication is working, for example).

Example request (passkey embedded in URL):
```
  curl https://foo.org/api?method=time&passkey=bacon420
```

Another example (passkey sent via POST):
```
  curl -d "passkey=bacon420" http://foo.org/api?method=time
```

Example response:
```
<time>Wed Nov 07 19:53:03 -0800 2007</time>
```

In the following examples, the passkey is sent via POST.

# List Users (list\_users) #

Used for both the initial pull of all users (in the case wherein one or more servers have been online for a long time), as well as periodic updates.

### List Users :: First Load ###

The optional parameter **first\_load** may be passed (value "true") to pull the entire list of users.  It should only be called once; subsequent loads can use time-based diffs.

List users (first load) request:
```
  curl -d "passkey=bacon420" http://foo.org/api?method=list_users&first_load=true
```

Sample response:
```
<users>
  <user>
    <login>pedro</login>
    <hashed_password>590f88589c44380f066c88fd4142716d28c2f3e76f0ba13ef9e6cb3872f81664</hashed_password>
    <salt>278448800.832002332338702</salt>
    <passkey>a472829819</passkey>
  </user>
  <user>
    <login>keith</login>
    <hashed_password>4738a1db458cab13e5e42c9578d21bed31a900930c179c44746ca8049c87b7a0</hashed_password>
    <salt>275374700.0221938363202334</salt>
    <passkey>59d7fc759b</passkey>
  </user>
  ...
</users>
```

Explanation of fields:
  * hashed\_password - The SHA256 hex digest of the (password + HARD\_SALT + user salt) - see below for more info.
  * salt - the user's unique salt.  used in addition to a hard salt to curb rainbow attacks
  * passkey - a random, 10-digit alphanumeric key used in tracker URLs to authenticate the user against the tracker's database of user passkeys


### List Users :: Since X Seconds Ago ###

The **list\_users** command also takes an optional **since** parameter which should restrict the server to sending only those users created in the last X seconds.

For example, if we wanted to only receive users created under an hour ago (60\*60 or 3600 seconds) we would do something like the following...

_Note: **since** is always relative to the API server's time, not the requesting client's.  It is up to the client to keep track of the last time (X seconds ago) that it ping'd neighboring servers for lists of newest added users.  In the event of one of the servers being down or missed requests for updates, a greater since value (1day=86,400 seconds) can always be passed to ensure all users have been added._

List users (first load) request:
```
  curl -d "passkey=bacon420" http://foo.org/api?method=list_users&since=3600
```

Sample response:
```
<users>
  <user>
    <login>abygale</login>
    <hashed_password>63a6a9c5515bc6dbb53ce2af68541b06d563ff6b6f82352af5fb6d909489dea7</hashed_password>
    <salt>277685200.628751501062818</salt>
    <passkey>8d31965e31</passkey>
  </user>
  <user>
    <login>brianna</login>
    <hashed_password>cf56544473827e8fde111a05caf30ce4ffd435b6e64ec5f378362ae8ad6e983b</hashed_password>
    <salt>279805600.660643620734607</salt>
    <passkey>4461b208d2</passkey>
  </user>
</users>
```

### How the Hashed Password is Calculated ###

Each network shares a common HARD\_SALT.  In the sample Rails app, this is defined as:
```
  HARD_SALT = 'TheHydraProject--123456789@!#%@^^#@'
```

The password hash is computed as follows (pseudocode).  Concatenate the following strings, in the order specified:
```
  str = raw_password + HARD_SALT + user_salt
```

Now take the SHA256 hex digest of the concatenation result:
```
  hashed_password = SHA256_hex(str)
```

The following is a working example in ruby:
```
require 'digest/sha1'
HARD_SALT = 'TheHydraProject--123456789@!#%@^^#@'
password = 'bacon420'
salt = 'salty.123456789'
hashed_password = Digest::SHA256.hexdigest(password + HARD_SALT + salt)
puts hashed_password # 'd0d305b4922a65d53d6a4add6233bb2d74b0c40b'
```

**Note**: Be sure to use **SHA256** not SHA1.  =)

# List Torrents (list\_torrents) #

This method works similarly to List Users; a **first\_load** param can be passed which will pull the entire database of torrents.

Note: the rest of the data for the torrent can be obtained via the bencoded data of the .torrent file itself, available via the **get\_torrent** method.  The 'name' and 'description' fields are user-submitted, and 'info\_hash' is a calculated value extracted from the original .torrent file.

### List Torrents :: First Load ###


List torrents (first load) request:
```
  curl -d "passkey=bacon420" http://foo.org/api?method=list_torrents&first_load=true
```

Example response:
```
<torrents>
  <torrent>
    <info_hash>38282be3b6168a42d9611eca74d27eb15a5ab0fa</info_hash>
    <name>Whoop Ass Remixes</name>
    <filename>whoop ass.torrent</filename>
    <category>Music</category>
    <description><![CDATA[A collection of remixes of the whipass.mp3 song]]></description> 
  </torrent>
  <torrent>
    <info_hash>671746d95b5a872cbaf50a9164b28750a98ed303</info_hash>
    <name>Games Programming FAQ</name>
    <filename>gamedev.pdf.torrent</filename>
    <category>eBooks - Development</category>
    <description><![CDATA[How to develop games, etc]]></description>
  </torrent>
  <torrent>
    <info_hash>69710247d0673106ff74d190a2c658de66e40a61</info_hash>
    <name>Tutorial on Rails</name>
    <filename>Rails_Tutorial.torrent</filename>
    <category>Tutorials - Programming</category>
    <description>Ruby on Rails development tutorial</description>
  </torrent>
</torrents>
```


### List Torrents :: Since X Seconds Ago ###

On subsequent loads, the **since** param can request only those torrents added within the previous X seconds.  (again, on the answering server's time)

Let's get new torrents added within the last hour. (3600 seconds ago)

List torrents (since) request:
```
  curl -d "passkey=bacon420" http://foo.org/api?method=list_torrents&since=3600
```

Example response:
```
<torrents>
  <torrent>
    <info_hash>af8b322d9a38d0e16672f19d86aece7a465cfc43</info_hash>
    <name>Crank That Mixtape</name>
    <filename>crankdat.orrent</filename>
    <category>Hip-Hop</category>
    <description>Crunk remixes</description>
  </torrent>
</torrents>
```

# Get Torrent (get\_torrent) #

This method takes a single param **info\_hash** which is the hex value of the info\_hash of the user-uploaded .torrent file.

The 'Content-Type' header is set to 'application/x-bittorrent'.

The data sent back is that of the raw .torrent file (a bencoded dictionary).

Curl example:
```
  curl --output foo.torrent -d "passkey=bacon420" http://foo.org/api?method=get_torrent&info_hash=69710247d0673106ff74d190a2c658de66e40a61
```

In the above, the output is written to a file called 'foo.torrent'.
# The Hydra Project #

## A Distributed, private BitTorrent tracker framework with goals of: ##

### ... user privacy, anonymity, survivability & distributed ratio maintainability. ###

![http://incredimazing.com/static/media/2007/12/09/9fbe45c99de8eb2/thpbig.png](http://incredimazing.com/static/media/2007/12/09/9fbe45c99de8eb2/thpbig.png)

Goals of THP:
  * Maintain user privacy & anonymity (as much as possible for a private BitTorrent tracker)
  * Survivability of the overall network in the face of a single or several (not 100%) of the network being attacked or raided
  * Distributed ratio and user management across multiple trusted domains (a bit like OpenID, for private trackers)
  * Development of the "spec" Hydra Project protocol (implemented in Ruby on Rails) which can be ported over to PHP, etc.

Anonymity and privacy precautions:
  * User IP addresses are never stored on disk or the database
  * Email addresses are _never_ collected
  * Password hashes are stored in the database with a random salt for each password hash (to curb the feasibility of widespread rainbow attacks)
  * Uploaded .torrent files are not associated with users
  * User information of who seeded/uploaded particular torrents is never stored
  * User share ratio information is, however, kept (bytes downloaded/uploaded)
  * One requirement of private trackers is to lookup IP Addresses to correlate username (last login from IP) with torrent client IPs
  * For this problem we use memcached, which only ever stores the IPs in system RAM: http://www.danga.com/memcached/
  * Web server configuration is up to the user; it is **highly** advised to disable IP and/or request logging altogether.

Survivability:
  * Multiple private THP domain owner/operators ("admins") grant trust rights to fellow private tracker operators, which both must use the THP sync protocol
  * At anytime, a server admin can remove a tracker from its list of trusted THP trackers (i.e. in the event of a raid or server compromise)
  * User accounts (including password hash information) are shared across all trusted domains and are synced periodically; thus allowing users to login to any known site in the network
  * Share ratio information (bytes upped/downloaded) is shared across all trusted servers;  note: which files were downed/upped are never stored for obvious privacy reasons.
  * Torrents are shared across servers. TBD: exactly how this works, including data synchronization and category push/pull mechanics, etc.
  * .torrent files are modified to include a list of all (or a specified subset) of the tracker announce URLs for the trusted trackers.  Thus if a single tracker goes down, the .torrent file should still be valid if at least one of its announce URLs is still operational. (Experimental)
  * TBD: how to propagate bans or other user management actions across trusted trackers.

Open protocol:
  * See the [HydraProjectSyncAPI](HydraProjectSyncAPI.md) for more details on the intra-server REST-based API
  * Feedback is welcomed and encouraged from the community

Credits:
  * Bram Cohen for BitTorrent itself: http://bitconjurer.org/
  * The TorrentStrike PHP private tracker, based on torrentbits & bytemonsoon code: http://sourceforge.net/projects/torrentstrike
  * William Morgan's Ruby BitTorrent library: http://rubytorrent.rubyforge.org/
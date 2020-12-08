---
title: Mac OS X hostname
---
To prevent my hostname from being set by DHCP, I have this:

```console
kenyon@devildog ~ % grep -i hostname /etc/hostconfig
HOSTNAME=devildog.kenyonralph.com
```

In System Preferences, Sharing, Computer Name should be set to the FQDN. The .local name will just be the first part.

`hostname` should return the FQDN (devildog.kenyonralph.com), and `hostname -s` should return just the first part (devildog).

## References

* <https://excitedcuriosity.wordpress.com/2007/08/24/mac-os-x-hostname-determination/>
* <https://web.archive.org/web/20141112155051/http://hintsforums.macworld.com/showthread.php?t=29712>
* <https://web.archive.org/web/20061022140935/http://www.macosxhints.com/article.php?story=20001224021638403>

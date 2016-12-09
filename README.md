grill
=====

`grill`, aka global rate-limiting in Linux, is a scanner for
CVE-2016-5696 (pure TCP off-path).

Install
-------
```
$ go get github.com/nogoegst/grill
```

Caveats
-------
*Don't ever use wireless links* on the way to the hosts. Constant packet loss and retransmisions drastically reduce scan accuracy.

*Use less NATs as possible* (down to 0), they introduce delays and change packets.

Currenly `grill` uses around avg. 400KBit/s and max. 500Kbit/s of output bandwidth (16 concurrent scans).

Kernel interference
-------------------
To avoid kernel interference during scan add a rule to your firewall to drop outgoing RST packets.

For PF (`/etc/pf.conf`):
```
block drop out quick proto tcp flags R/R
```
then `# pfctl -f /etc/pf.conf`.

For NetFilter:
```
# iptables -A OUTPUT -p tcp --tcp-flags RST RST -j DROP
```

Usage
-----
`grill` reads `stdin` and scans hosts from it (up to 16 concurrent scans). The input format is `host port\n`.

```
# cat probe | grill -i interface -dll gateway-MAC [-sll src-MAC] [-sip src-IP] > results 
```

The output format is `host:port,recievedChACKs,1stBurstSendingTime,2ndBurstSendingTime`.

To get human results, run results though `verdict` utility (is in `verdict` directory):
```
cat results | verdict
```

So it goes. Have fun and make love.


Scanning the Tor network
------------------------
To scan relays of the Tor network, just fetch and format last consensus:
```
curl https://collector.torproject.org/recent/relay-descriptors/consensuses/`date -u +'%Y-%m-%d-%H-00-00-consensus'` | grep '^r '| awk '{print $7" "$8}' > probe-consensus
```

And then just pass resulted file to `grill` input.
As of now, scanning whole Tor network should take less than 30m (16 concurrent scans).

I managed to scan whole net in 7m44s by using 127 concurrent scans and in 6m30s by reducing timeout to 1.7s further (this is probably not safe due to packet loss, congestion, etc).

Note that 127 is the maximum (and reasonable) number of open BPFs in OpenBSD. In Linux this limit is higher but it will make you kernel almost stuck. Anyway, good luck.

Acknowlegments
-------------
`grill` is hugely inspired by similar Scapy scanner by David Stainton [https://github.com/david415/scan_for_rfc5961]
and PoC by violentshell [https://github.com/violentshell/rover].


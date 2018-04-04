What & Why?
-----------

I usually use Shorewall as my firewall package of choice, unfortunately Shorewall 
assumes it's a sole source of `iptables` truth, which doesn't play well with
Kubernetes.

Basically, all rules entered by Kubernetes are dropped whenever Shorewall
is (re)started. The rules are refreshed by `kube-proxy` every 30 seconds (by default),
so it doesn't sound like a big deal, but it means that no cluster services are 
available for up to half a minute. 

Note: it only matters if Shorewall is restarted with Kubernetes (`kube-proxy`, to be exact) running. 
Kubernetes plays fair with `iptables` already configured, so if you can ensure Shorewall starts 
before `kube-proxy` and you never reload it, you're basically safe.

How?
----

The solution uses [Shorewall hooks](http://shorewall.net/shorewall_extension_scripts.htm) 
to save and restore Kubernetes chains (i.e. all chains named `KUBE-*` in `nat` and `filter` tables), as well
as re-add rules jumping from `INPUT`, `OUTPUT`, etc. chains to kubernetes-managed sub-chains.
Tested with Kubernetes v1.10.0.

It's inspired by this [Shorewall+Docker](https://blog.discourse.org/2015/11/shorewalldocker-two-great-tastes-that-taste-great-together/#) 
hack.

Note: this solution is far from perfect - there's a possible race condition between `kube-proxy` and 
these hooks, as `kube-proxy` uses several separate `iptables` and `iptables-restore` steps. It's a classic
shared mutable state problem. I suggest manaul inspection after Shorewall reconfiguration.

If you know better solution, let me know.


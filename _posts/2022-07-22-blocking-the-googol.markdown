---
layout: post
title: "Bloqueando sin querer un googol"
date:   2022-07-22
---

---

Algunos días después de configurar pf-badhost en mi router-gateway-firewall OpenBSD con bloqueo a varios miles de ASNs, tuve problemas con algo en Chrome.

Sospechando un IP bloqueado, revisé con tcpdump y efectivamente Chrome estaba intentando hacer un request a un host en el dominio 1e100.net.

Un dominio nuevo para mí que resulta ser [propiedad de Google](https://support.google.com/faqs/answer/174717?hl=en):

> What is 1e100.net?
> 
> 1e100.net is a Google-owned domain name used to identify the servers in our network.

> Following standard industry practice, we make sure each IP address has a corresponding hostname. In October 2009, we started using a single domain name to identify our servers across all Google products, rather than use different product domains such as youtube.com, blogger.com, and google.com. We did this for two reasons: first, to keep things simpler, and second, to proactively improve security by protecting against potential threats such as cross-site scripting attacks.
>
> Most typical Internet users will never see 1e100.net, but we picked a Googley name for it just in case (1e100 is scientific notation for 1 googol).









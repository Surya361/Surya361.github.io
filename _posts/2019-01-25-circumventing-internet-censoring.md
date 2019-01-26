---
layout: post
published: true
title: Circumventing internet censorship
tags:
  - internet-censorship
  - encryption
---

In complaince with direction given by [uttharakhand high court](https://www.indiatimes.com/technology/news/uttarakhand-high-court-asks-isps-to-block-porn-sites-for-rise-in-rape-cases-but-will-it-help-354022.html), Indian ISPs started blocking the pornographic sites. In this article I will discect the implementation used to enforce the blockade as well as techniques to circumvent. Finally we will discuss better implementations that address the current pitfalls.

A while back one of my friend recommended the [tv series 24](https://en.wikipedia.org/wiki/24_(TV_series)), but i couldnt find it on any of the streaming services, when enquired she reverted back with a link to 123movies.com, which for some reason was blocked in India. I started digging into the implementation of the block.

When i tried to resolve the domain name with [dig](https://en.wikipedia.org/wiki/Dig_(command)) it gave the following response

	$ dig 123moviestime.com  +noall +answer	
	; <<>> DiG 9.11.3-1ubuntu1.3-Ubuntu <<>> 123moviestime.com @8.8.8.8 +noall +answer
	;; global options: +cmd
	123moviestime.com.	10	IN	A	202.83.21.15

A quick whois on the ip revealed that it belonged to the ISP. There was no change in the result even when i forced dig to use googles dns resolver, confirming that the ISP was using [DPI](https://en.wikipedia.org/wiki/Deep_packet_inspection) technique on DNS.

To circumvent, i configured my local-dns to the original ip and requested the website using http and https, while the http request failed https worked. This indicated that the ISP is inspecting http request headers and proxying the request when it matches blocked sites, also confirming that the ISP is not inspecting [SNI header](https://en.wikipedia.org/wiki/Server_Name_Indication) to block https traffic.

DNS has been one of the weakest links on the internet suseptible to man in the middle attacks, and looks like the ISPs are exploiting it to enforce the ban. Its also a smart strategy as inspecting DNS traffic is way cheaper and more effictive than analysing other protocols(DNS traffic will atleast be 10x less than others). But this is gonna change soon, with the advent of [DoH](https://en.wikipedia.org/wiki/DNS_over_HTTPS) and [DoT](https://en.wikipedia.org/wiki/DNS_over_TLS) all DNS traffic will be encrypted while verifying the authenticity of the response. Mainstream browsers like [firefox](https://blog.nightly.mozilla.org/2018/06/01/improving-dns-privacy-in-firefox/) and chrome started integrating this feature in their nighty code. On the mobile front Google added support for the same on [Android Pie](https://blog.cloudflare.com/enable-private-dns-with-1-1-1-1-on-android-9-pie/). Cloudflare released a [dns daemon](https://developers.cloudflare.com/1.1.1.1/dns-over-https/cloudflared-proxy/) that runs on major platforms(Linux/Windows/MacOS), and proxies the dns request via DoH, inturn rendering the embargo ineffective. 

The above developments does not mean that ISPs will become incapable of enforcing the ban. Mind that above workaround helps only when the http traffic is encrypted i.e. when using https. ISPs could start [DPI](https://en.wikipedia.org/wiki/Deep_packet_inspection) on https and lookout for blocked hostnames in [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) to effectively enforce it for the time-being. I mention for the 'timebeing' because cloudflare annouced its support for [Encrypted SNI](https://blog.cloudflare.com/encrypted-sni/), as an extention of TLS/1.3, that will engender sni inspection fruitless.

As a philosophical foot note, i believe that ban on certain content on the internet can only be implemented by exploiting the existing vulnerabilities, while the academia and the industry works tirelessly to ensure that they are mitigated. [DoH](https://en.wikipedia.org/wiki/DNS_over_HTTPS),[DoT](https://en.wikipedia.org/wiki/DNS_over_TLS),[ESNI](https://blog.cloudflare.com/encrypted-sni/) have been created to alleviate attacks such as [DoS](https://en.wikipedia.org/wiki/Denial-of-service_attack). I also think that ban on content is by definition ineffective on the distributed internet and in some cases backfires, i learned about the documentary India's daughter only when the government tried to ban it. I will end this philosophical on a lighter  note with a quote by [mega reverand of the church Our Lady of Perpetual Exemption: John Oliver](https://www.youtube.com/watch?v=GrwOLITIe7U&t=771s) "[Internet is like quicksand](https://youtu.be/r-ERajkMXw0), the more aggresively you fight to remove from it, the deeper you sink down into it".

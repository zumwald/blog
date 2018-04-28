---
title: Hello World & SSL at Last
date: 2018-04-28 09:47:11
---

After an embarrassingly long time I have finally disabled HTTP for this site. Of course, the only way to go about disabling HTTP was to properly support HTTPS. _This_ is what's been causing the delay (at least that's what I'd been telling myself), and today I finally decided to show myself just how silly that excuse has been.

Turns out, it was _beyond_ silly. Acquiring a certificate and configuring SSL was easier than spinning up the website (which was easy - perhaps a retroactive post on that another day). It was easier than getting started with [Hexo](https://hexo.io/) to bootstrap the site's layout and authoring tooling. **It was less than an hour of work.**

I had heard about [Let's Encrypt](https://letsencrypt.org/), so naturally I did some online sleuthing and found [a tutorial for a plugin that "just works" for Azure WebApps](https://gooroo.io/GoorooTHINK/Article/16420/Lets-Encrypt-Azure-Web-Apps-the-Free-and-Easy-Way/20073#.WuSpftPwZ24). I found the tutorial easy to follow and made it to the very last step where I expected to see a magical "success!" screen but found this welcoming me instead.

![Error shown after attempting to set up the Let's Encrypt plugin](/2018/04/28/Hello-World-SSL-at-Last/lets-encrypt-plugin-error.jpg)

Don't get me wrong, the blog did its job; I invested very little time and it _nearly_ worked out. By all accounts, the plugin did work at one point (perhaps it still does and I just screwed up one of the few setup steps). I wasn't super keen on troubleshooting the issue, so I decided to poke around a bit more and ended up finding a slightly less opaque solution for generating free certificates at [ZeroSSL](https://zerossl.com/).

Using their [FREE SSL Certificate Wizard](https://zerossl.com/free-ssl/#crt) was straightforward. With a few clicks and a TXT entry at danzumwalt.com, I had a certificate. The process was less magical and clever, which in my opinion is a good thing for a dev tool like this - I learned a bit in the process, and I'll learn more when I automate the renewal of the cert sometime in [the next 90 days](https://letsencrypt.org/2015/11/09/why-90-days.html).

### Obtaining an SSL Certificate using ZeroSSL (powered by Let's Encrypt)

The Certificate Wizard is a breeze to use. In the first screen you'll accept terms of use and indicate what validation method you'll use to obtain the cert; I opted for the DNS verification option.

![ZeroSSL Certificate Wizard](/2018/04/28/Hello-World-SSL-at-Last/Free-SSL-1.jpg)

You can leave the [RSA](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29) and [CSR](https://en.wikipedia.org/wiki/Certificate_signing_request) fields blank and click 'next'. ZeroSSL with generate a RSA key and CSR for you. Be sure to save these somewhere private (i.e. _please_ don't check them in to source control, that's *not* a place for private/secret values).

![ZeroSSL Certificate Wizard with RSA and CSR populated](/2018/04/28/Hello-World-SSL-at-Last/Zero-ssl-save-csr.jpg)

The next step is very quick. The cert authority needs to know they can trust that you own the right to the domain you're trying to obtain the certificate for. The way you'll prove that is by performing a handshake of sorts by placing an agreed upon breadcrumb in the DNS record for your domain.

> Aside: I recently started using google domains instead of GoDaddy because Google lets you protect your private information (go ahead, run `whois danzumwalt.com` - but if you want to get to know me you're better off sending an invite on LinkedIn). 

Because I'm registering two hosts: `danzumwalt.com` and `*.danzumwalt.com` for good measure, I need to place two TXT entries at my domain, like so:

![ZeroSSL will ask you to place TXT records on your domain to verify you own it](/2018/04/28/Hello-World-SSL-at-Last/Zero-ssl-dns-verification.jpg)

After the validation is complete (it's only taken a few seconds for me, but _can_ take longer) ZeroSSL will provide an RSA key and the domain certificate. Save them both (again, somewhere private).

![](/2018/04/28/Hello-World-SSL-at-Last/Zero-ssl-save-rsa.jpg)

This is where you're left alone in the cold a bit. Your hosting provider is likely going to want a `pfx` file and secret, so you'll need to generate those. Luckily its pretty straightforward ([the Azure docs helped](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-ssl#export-certificate-to-pfx)).

As an example, if you had saved your RSA and CRT text as `domain-rsa-key.txt` and `domain-crt.txt` you would run the following command: `openssl pkcs12 -export -out myserver.pfx -inkey domain-rsa-key.txt -in domain-crt.txt`. You'll be prompted for a passphrase, remember it! You'll need it when you upload your certificate to your hosting provider.

![](/2018/04/28/Hello-World-SSL-at-Last/Generate-pfx.jpg)

The last part is to login to your website in the Azure portal and navigate to the _SSL Settings_ page. Toward the bottom of the page you'll see an option to upload your certificate. This is where you'll select the `.pfx` file you just generated, and enter the same passphrase you typed when running the command to generate the file.

![Uploading your shiny new cert to Azure](/2018/04/28/Hello-World-SSL-at-Last/Azure-Upload-Cert.jpg)

Next, you'll bind the certificate to your host:

![](/2018/04/28/Hello-World-SSL-at-Last/Azure-Bind-Certificate-to-Host.jpg)

And finally you'll see I took the measure of disabling HTTP support and bumping the minimum supported TLS to version 1.2. To confirm your site is up and running with the certificate, navigate to the site and click on the lock icon. If your browser is reporting everything as being OK and the cert matches what you generated (like in the screenshot below) you can go ahead and celebrate.

![](/2018/04/28/Hello-World-SSL-at-Last/Confirm-Cert-Details.jpg)


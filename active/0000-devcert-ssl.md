- Start Date: 2017-04-28
- RFC PR: (leave this empty)

# Summary

Add support for on-the-fly SSL certificates in development via devcert.

# Motivation

Many of the newest web platform features (i.e. service workers) require HTTPS transport. Generating
SSL/TLS certificates for local development is time consuming, and browsers will display scary
interstitials warning of untrusted certificates.

 [Devcert](https://github.com/davewasmer/devcert) is a library that automates the process of
 installing a root certificate authority and generating app certificates signed by this trusted
 root authority. This would significantly lower the barriers to entry for developing with these
 new web platform features.

# Detailed design

[Devcert](https://github.com/davewasmer/devcert) does most of the cross-platform heavy lifting
here. We would keep the existing command line flags as well, and simply tweak their existing
behavior: if `--ssl` is provided without corresponding `--ssl-key` and `--ssl-cert`, then the
CLI would invoke devcert to generate a certificate on the fly using the app name.

A simplified example implementation:

```js
if (options.ssl) {
  if (options.sslKey && options.sslCert) {
    // this is the current functionality
  } else {
    getDevelopmentCertificates(appName, { installCertutil: true }).then((ssl) => {
      https.createServer(ssl, handler);
    });
  }
}
```

# How We Teach This

Especially for frontend development, SSL in dev is typically motivated by wanting to unlock the
above-mentioned web platform features, so some people many discover this "along the way" to
implementing something like service workers. The Ember CLI docs would be updated to reflect the
new argument behavior, and tutorials might call out the SSL flags.

But overall, the goal here is to actually remove a step in existing workflows (manual management of
dev SSL certificates), so the need for learning changes here is limited.

# Drawbacks

* Introduces additional dependencies
* Devcert requires elevated permissions on first run (once per machine only). This might be a bit
scary/surprising, and should be treated with caution
* From our side, this

# Alternatives

I'm not aware of any other libraries like devcert. The alternative here seems to be the status quo:
leave SSL certificate management up to the developer.

# Unresolved questions

While I've done my best to ensure devcert's implementation is safe for use, I'm not an SSL/TLS
expert. We should be careful that nothing here exposes the developer or their machine to undue risks.
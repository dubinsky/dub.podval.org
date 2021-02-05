---
layout: post
title: 'Bintray is Dead!'
author: Leonid Dubinsky
tags: [bintray, gradle]
date: '2021-02-04'
---

* TOC
{:toc}
## Introduction ##

Yesterday JFrog [announced](https://jfrog.com/blog/into-the-sunset-bintray-jcenter-gocenter-and-chartcenter/)
that [Bintray/JCenter](https://bintray.com/beta/#/bintray/jcenter?tab=packages) will go away
in three months. Although they do provide free [JFrog Platform Cloud](https://jfrog.com/pricing/#sass)
subscriptions for Open Source developers, and I got [one](https://dub.jfrog.io/),
I do not see how I can use it to distribute my artifacts.
Besides, as I [wrote before](http://dub.podval.org/2020/06/29/bintray-gradle-plugin.html),
Bintray seems to be abandonware...

I am not ready to switch to the non-traditional [JitPack](https://jitpack.io/) (yet?),
so [Maven Central](central.sonatype.org) seems to be the way to go.

One reason I did not use it before because it has a reputation of being slow, and the way it
is struggling, returns `405 Not Allowed` and keeps being restarted 
right now, just a day after the JCenter announcement, seems to confirm that.
At least they report the [status](https://status.maven.org/) of things :)
I hope that with everybody moving there maybe Sombody Will Do Something about it and
eventually things will improve.

Another reason I didn't use it before is: it is supposed to be more difficult to use than
JCenter, with all their signing requirements and such. This turned out to be not that hard ;)

These are my notes on what did I have to do to switch to Maven Central.
Official (but somewhat outdated) documentation is available:
- [OSSRG Guide](https://central.sonatype.org/pages/ossrh-guide.html)
- [Working with PGP Signatures](https://central.sonatype.org/pages/working-with-pgp-signatures.html)
- [Publishing to Maven Central](https://github.com/chhh/sonatype-ossrh-parent/blob/master/publishing-to-maven-central.md)

## Account ##

I signed up for Sonatype at https://issues.sonatype.org/secure/Signup!default.jspa
with dub@podval.org email address.
The same credentials are used at https://oss.sonatype.org/ (OSSRH) and for deploying
artifacts. To make them available to Gradle, I put them into `~/.gradle/gradle.properties`:
```properties
mavenCentralUsername=dub
mavenCentralPassword=...
```

To the `publishing` block of the Gradle build file I added:
```groovy
repositories {
  maven {
    name = 'mavenCentral'
    url = version.endsWith('SNAPSHOT') ?
      'https://oss.sonatype.org/content/repositories/snapshots' :
      'https://oss.sonatype.org/service/local/staging/deploy/maven2'

    credentials {
      username = mavenCentralUsername
      password = mavenCentralPassword
    }
  }
}
```

## Namespace ##

I claimed the "namespace" (group id) `org.podval.tools` by opening a special 
kind of ticket via https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134

I had to verify that I own the domain `podval.org` by adding a DNS TXT record referencing
the ticket. I did ask if I should claim `org.podval` instead of the `org.podval.tools`,
but it seems that nobody read my question - probably because initial verification is handled by
robots :). But I then asked specifically to wide the scope
of my "staging profile" that resulted from this [ticket](https://issues.sonatype.org/browse/OSSRH-63919),
and [Joel Orlina](https://issues.sonatype.org/secure/ViewProfile.jspa?name=jorlina)
did it immediately!

In addition to `podval.org`, I control the domain (and GitHub organization) `opentorah.org`.
I want to sign artifacts in it with a key associated with an appropriate email address 
dub@opentorah.org. It seems that Maven Central verification does not require for the signing
key to be associated with email address of the Sonatype account, so I didn't have to create
another Sonatype account, and claimed the `org.opentorah` namespace under the same one!

## GPG Keys and Signing ##

I do not normally use (or have) a GPG key, so I had to make one (actually, two).
First, I made sure that permissions on the `~/.gnupg` are tight:
```shell
$ chown -R $(whoami) ~/.gnupg/
$ chmod 700 ~/.gnupg/*
$ chmod 700 ~/.gnupg
```
(Some advice on the Internet suggests `600`, but it is wrong: gpg2 won't be able to read
the secret keys...)

Generated GPG key for dub@podval.org:
```shell
$ gpg2 --generate-key ... whatwhen
```
Sent the public key to the keyserver so that Maven Central could verify it:
```shell
$ gpg2 --keyserver keys.gnupg.net --send-key EA493E02
```
That didn't seem to work, so I manually submitted at http://pool.sks-keyservers.net 
the output of
```shell
$ gpg2 --armor --export dub@podval.org
```

All the above was repeated for a dub@opentorah.org key.

By default, Gradle's [Signing Plugin](https://docs.gradle.org/current/userguide/signing_plugin.html)
uses properties (signing.keyId, signing.password, signing.secretKeyRingFile) to access
the key. I prefer to use explicit configuration, but more importantly, I need the flexibility
to use different keys to sign the artifacts of different projects.

I got the keys into a form suitable for `gradle.properties`: run
```shell
$ gpg2 --armor --export-secret-keys dub@podval.org | awk '1' ORS='\\n\\\n'
```
and removed the final `\n\`.

I then added to `~/.gradle/gradle.properties`:

```properties
gnupg.dub-podval-org.key=<see above>
gnupg.dub-podval-org.password=...
gnupg.dub-opentorah-org.key=<see above>
gnupg.dub-opentorah-org.password=...
```

To the Gradle build file, I added:
```groovy
signing {
  useInMemoryPgpKeys(
    getProperty('gnupg.dub-podval-org.key'),
    getProperty('gnupg.dub-podval-org.password')
  )
  sign publishing.publications.library
}
```

## Releasing ##

After artifacts are deployed to the staging repository:
- log into Nexus at https://oss.sonatype.org/
- "close" the staging repository (verify that their requirements are satisfied)
- "release" it
- if this is a first release in this namespace, comment on the Jira ticket so that Sonatype 
can start synchronizing the artifacts to Maven Central.
  
TODO multi-module projects

TODO nexus Gradle plugin(s)




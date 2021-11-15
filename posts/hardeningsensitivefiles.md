<p style="text-align: center; font-size: 40px;">Hardening Sensitive Files</p>

---

##### Metadata

###### Last revised - 11/15/21

###### Author       - Mason Sipe

---

## Preamble

This post is designed to set awareness of the individual changes for files that should be protected on your system in order to keep and maintain a hardened system.

## Background:

Some files on linux are by default readable by unprivileged users which is a problem. Throughout the evolution of linux, logs, core dumps, and other sorts of information has been stored on your system which contains sensitive information which should be protected.
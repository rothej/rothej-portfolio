---
layout: post
title: Setting Up a Vaultwarden Password Manager
featured: false
giscus_comments: true
authors:
  - name: Joshua Rothe
    url: "https://joshrothe.us"
excerpt: "Timing closure is a critical part of FPGA design, and verifying your firmware does not violate timing is mandatory if you want your implementation to perform as designed. Electrons move at a specific speed, and transistor switching speed is based on physical dimensions as well as internal capacitance. When you write RTL code you are designing hardware, flipping switches and filling Look-Up Tables (LUTs), and even environmental considerations such as temperature and humidity can affect the performance of programmable circuits. This guide goes over several methods for achieving timing closure on FPGA designs. Tradeoffs are discussed for each method, and design concerns are presented in a way that helps decide the best approach for the situation."
date: 2026-03-28
description: Guide for setting up a Vaultwarden password manager host on a Linux server, to handle clients on the local network or over VPN.
tags: [linux, vaultwarden, devops]
categories: [linux, devops]
toc:
  sidebar: left
citation: true
---

![Vaultwarden Logo](../../../assets/img/posts/2026-03-28-setting-up-a-vaultwarden-password-manager/vaultwarden_logo_auto.svg){: .img-fluid}
*Source: [Vaultwarden Github](https://github.com/dani-garcia/vaultwarden/tree/main)*

---

Password vaults are a convenient and secure way to manage multiple passwords. As data spills become more and more [common](https://www.statista.com/statistics/273550/data-breaches-recorded-in-the-united-states-by-number-of-breaches-and-records-exposed/), password complexity and even rules for regularly updating them leads to an inevitable mish-mash of credentials that are impossible to remember when not used daily. The logic behind constantly-evolving password guidelines is beyond the scope of this guide, but the latest word from NIST on password managers [recommends their use](https://pages.nist.gov/800-63-4/sp800-63b.html):

> Verifiers SHALL allow the use of password managers and autofill functionality. Verifiers SHOULD permit claimants to use the “paste” function when entering a password to facilitate password manager use when password autofill APIs are unavailable. Password managers have been shown to increase the likelihood that subscribers will choose stronger passwords, particularly if the password managers include password generators


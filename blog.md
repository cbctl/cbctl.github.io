---
layout: page
title: blog
permalink: /blog
menu: nav
---

<span class="italix">{{ "New to code? Check out my [resources page](/resources) for some free learning materials." }}</span>

<div class="home">

  <ul class="post-list">
  <li></li>
      {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
    <li>
      <span class="post-meta">{{ "2022-05-23" | date: date_format }}</span>
        <a class="post-link" href="/blog/how-to-review-candidates">
          {{ "How to review software engineering candidates' tech tests" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2022-05-23" | date: date_format }}</span>
        <a class="post-link" href="/blog/go-linting">
          {{ "Golangci linter lied to me?!" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2022-05-18" | date: date_format }}</span>
        <a class="post-link" href="/blog/baremetal-networking">
          {{ "DHCP, NAT, VLAN, oh my!" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2022-03-20" | date: date_format }}</span>
        <a class="post-link" href="/blog/salaries">
          {{ "Salary transparency scares me" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2021-12-18" | date: date_format }}</span>
        <a class="post-link" href="/blog/i-dont-know">
          {{ "Some things I don't know as of 2022" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2021-12-17" | date: date_format }}</span>
        <a class="post-link" href="/blog/custom-capi-provider">
          {{ "Building and Shipping a custom Cluster API provider" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2021-07-04" | date: date_format }}</span>
        <a class="post-link" href="/blog/hiking-cape-wrath-21">
          {{ "Hiking 100 miles alone on the Cape Wrath Trail" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2020-11-05" | date: date_format }}</span>
        <a class="post-link" href="/blog/hiking-in-cairngorms">
          {{ "8 Days Hiking Alone in the Cairngorms, Scotland" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2020-10-31" | date: date_format }}</span>
        <a class="post-link" href="/blog/how-to-tech-test">
          {{ "How to get the most out of your next tech test" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2020-04-10" | date: date_format }}</span>
        <a class="post-link" href="/blog/remote-pair-programming">
          {{ "How to pair-program remotely and not lose the will to live (or: there has never been a better time to learn Vim)" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2019-10-14" | date: date_format }}</span>
        <a class="post-link" href="/blog/inotify-release-deadlock-envoy">
          {{ "Envoy Proxy Deadlocked My Cloud" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2019-09-11" | date: date_format }}</span>
        <a class="post-link" href="/blog/hiking-in-norway">
          {{ "7 Days Hiking Alone from Voss to Dale, Norway" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2019-03-29" | date: date_format }}</span>
        <a class="post-link" href="https://www.cloudfoundry.org/blog/an-overlayfs-journey-with-the-garden-team/">
          {{ "Why is OverlayFS slow now?" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2018-09-08" | date: date_format }}</span>
        <a class="post-link" href="/blog/the-route-to-rootless-containers">
          {{ "The Route to Rootless Containers" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2018-05-09" | date: date_format }}</span>
        <a class="post-link" href="/blog/a-day-in-the-life-of-a-cf-engineer">
          {{ "A day in the life of a Cloud Foundry engineer" | escape }}
        </a>
    </li>
    <li>
      <span class="post-meta">{{ "2017-09-08" | date: date_format }}</span>
        <a class="post-link" href="/blog/container-rootfilesystems-in-prod">
          {{ "Container Root Filesystems in Production" | escape }}
        </a>
    </li>
  </ul>
</div>

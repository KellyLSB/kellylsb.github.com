---
layout: post
title: "Rails Structure Explained: Part 1"
description: "Explaining the importance of README files."
category: rails
tags: [ruby, rails, optimization]
---
{% include JB/setup %}

Over my years developing Ruby and Rails applications I have not seen a proper walkthrough of how Rails fits together from a base site perspective.
Most of these concepts will not be new to most Rails engineers; however it's always good for a refresher.

Let's start with README.md

# README.md

This file should give a brief explanation of what your application is/does. It should include things like how to setup in Dev, Staging and Production. It should include a list of application dependencies as well as how to install them on all the target systems. When writing this file, assume that the person reading has never used a Ruby application before especially not the one this file is in.

## Installation

The installation should be in-depth. You want to ensure that every new engineer installs their application the same way as everyone else. That does not necessarily mean that other engineers will break out of the box and choose their own path, but it's good to give a direction to all new engineers. A sample Installation blurb might look like.

<ol>
  <li>
    Install Rbenv and Ruby Build
    <ul>
      <li>
        Mac OS (If you already have homebrew skip to #2)
        <ol>
          <li>
            Install XCode (via App Store)
          </li>
          <li>
            Install Command Line Tools
            <ul>
              <li>Open XCode</li>
              <li>Open the Prefrences from the top bar menu</li>
              <li>Select the downloads tab</li>
              <li>Install the Command Line Tools</li>
            </ul>
          </li>
          <li>
            Install Homebrew (http://brew.sh/)
            <ul>
              <li>Open a terminal</li>
              <li><code>ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"</code></li>
            </ul>
          </li>
          <li>
            Install the latest Git
            <ul>
              <li><code>brew install git</code></li>
            </ul>
          </li>
          <li>
            Install the latest Rbenv and Ruby Build
            <ul>
              <li><code>brew install rbenv</code></li>
              <li><code>brew install ruby-build</code></li>
            </ul>
          </li>
        </ol>
      </li>
      <li>
        Ubuntu (LTS 12.04)
        <ol>
          <li>
            Update your machine
            <ul>
              <li><code>sudo apt-get update</code></li>
              <li><code>sudo apt-get upgrade</code></li>
            </ul>
          </li>
          <li>
            Install dependencies
            <ul>
              <li><code>sudo apt-get install nodejs libc6-dev bison openssl libreadline6 libreadline6-dev zlib1g zlib1g-dev libssl-dev libyaml-dev libxml2-dev autoconf</code></li>
            </ul>
          </li>
          <li>
            Install Rbenv and Ruby Build
            <ul>
              <li><code>git clone https://github.com/sstephenson/rbenv.git ~/.rbenv</code></li>
              <li><code>echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc</code></li>
              <li><code>echo 'eval "$(rbenv init -)"' >> ~/.bashrc</code></li>
              <li><code>git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build</code></li>
            </ul>
          </li>
        </ol>
      </li>
    </ul>
  </li>
</ol>

etc... You want to give the most detailed instructions possible. Because you never know what level of expertise your new engineer is. Also it makes it easier to reinstall if you need to. It's a lot faster to get started if you already know everything you need to do. Don't forget to include things like installing your database software (i.e. MySQL) and creating, migrating, and loading your database dump.

## Dependencies

If your application does image processing it's very likely you might need a dependency like ImageMagick. If you include this dependency in your README.md it will lessen the amount of work, questions and hassle when debugging application issues later. An example of how this might look might be.

- Ruby 2.0
- ImageMagick

We know the application is ruby, but always specify the intended version in the README.md!

## Deploy Instructions

Don't forget to include deploy instructions to Staging, QA, Production, etc...
If your deploys are automatic by CI don't forget to note that here as well.

## Continuous Integration

If you are using continuous integration, see if a CI build status badge is available for the primary branch. If so place it at the top of the readme along with CI access instructions.

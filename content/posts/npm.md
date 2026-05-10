---
date: '2026-05-09T20:47:24+05:30'
draft: false
title: 'npm'
description: 'Intro to npm'
tags: ['html', 'css', 'js']
categories: ['web']
showToc: true
weight: 1
cover:
  image: 'cover.jpg'
  alt: 'This is a post image'
  caption: 'This is the caption'
  relative: false
---

## Introduction

> **What's the problem its trying to solve?**

>To automate the process of downloading and upgrading libraries from a central repository.

**npm** is a package manager, i.e. it manages third party packages that you use in your application. It also helps you in managing dependencies by tracking the version of packages your app needs through a `package.json` file.

## package.json

A **JSON** file containing information about our project such as, name and version of external packages on which our project depends upon. This is useful because anyone using our project can run `$ npm install` to let **npm** grab the project dependencies for them.

### Creating a new `package.json` file

You can create a `package.json` file by running a CLI questionnaire or creating a default `package.json` file.

**Running a CLI questionnaire**:
- `$ npm init`
- answer the questions in the command line questionnaire.

**Creating a default `package.json` file**:
- `$ npm init -y`

## Downloading and installing packages with npm

To install a public package, on the command line, run

`$ npm install <package-name>`

This will create the `node_modules` directory in your current directory and will download the package to that directory. By default **npm** will install the latest version of a package.

---

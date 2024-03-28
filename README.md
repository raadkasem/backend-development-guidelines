# Backend development guidelines

Welcome to Backend Development Guidelines! This repository contains some of the best practices, tools and guidelines for backend applications gathered from different sources.

==================================================================================================

**Table of Contents**

- [N Commandments](#n-commandments)
- [General points on guidelines](#general-points-on-guidelines)
- [Development environment setup in README.md](#development-environment-setup-in-readmemd)
- [Data persistence](#data-persistence)
  - [General considerations](#general-considerations)
  - [SaaS, cloud-hosted or self-hosted?](#saas-cloud-hosted-or-self-hosted)
- [Environments](#environments)
  - [Local development environment](#local-development-environment)
  - [Testing environment](#testing-environment)
  - [Staging environment](#staging-environment)
  - [Production environment](#production-environment)
- [Bill of Materials](#bill-of-materials)
- [Security](#security)
  - [Docker](#docker)
  - [Credentials](#credentials)
  - [Secrets](#secrets)
  - [Login Throttling](#login-throttling)
  - [User Password Storage](#user-password-storage)
  - [Log](#audit-log)
  - [Anonymized Data](#anonymized-data)
  - [Temporary file storage](#temporary-file-storage)
- [Checklists](#checklists)
  - [Responsibility checklist](#responsibility-checklist)
  - [Release checklist](#release-checklist)
- [General questions to consider](#general-questions-to-consider)
- [Generally proven useful tools](#generally-proven-useful-tools)

<br>
<br>

### We do not want to limit ourselves to certain tech stacks or frameworks. Different problems require different solutions, and hence these guidelines are valid for various backend architectures.
<br>
<br>

# N Commandments

1. README.md in the root of the repo is the docs
2. Single command run
3. Single command deploy
4. Use [UTC as the timezone](http://yellerapp.com/posts/2015-01-12-the-worst-server-setup-you-can-make.html) all around

<br>

# Development environment setup in README.md

Document all the parts of the development/server environment. Strive to use the same setup and versions on all environments, starting from developer laptops, and ending with the actual production environment. This includes the database, application server, proxy server (nginx, Apache, ...), SDK version(s), gems/libraries/modules.

Automate the setup process as much as possible. For example, [Docker Compose](https://docs.docker.com/compose/) could be used both in production and development to set up a complete environment.

<br>

# Data persistence

## General considerations

Independent of the persistence solution your project uses, there are general considerations that you should follow:

* Have backups that are verified to work
* Have scripts or other tooling for copying persistent data from one env to another, e.g. from prod to staging in order to debug something
* Have plans or tooling for managing schema changes
* Have monitoring in place to verify health of the persistence solution

## SaaS, cloud-hosted or self-hosted?

An important choice regarding any solution is where to run it.

* SaaS -- fast to get started, easy to scale up, some infrastructure work required to allow access from everywhere etc.
* Self-hosted in the cloud -- allows tuning database more than SaaS and probably cheaper at scale in terms of hosting, but more labor-intensive
* Self-hosted on own hardware -- able to tweak everything and manage physical security, but most expensive and labor intensive


# Environments

This section describes the environments you should have, at a minimum. It might sound like a lot, [but there is a purpose for each one](http://futurice.com/blog/five-environments-you-cannot-develop-without).

- [Local development](#local-development-environment)
- [Testing](#testing-environment)
- [Staging](#staging-environment)
- [Production](#production-environment)

## Local development environment

This is your local development environment. You probably should not have a shared external development environment. Instead, you should work to make it possible to run the entire system locally, by stubbing or mocking third-party services as needed.


## Testing environment

This is a shared environment where code is deployed to as often as possible, preferably every time code is committed to the mainline branch. It can be broken from time to time, especially in the active development phase. It is an important canary environment and is as similar to production as possible. Any external integrations are set up to use staging-level versions of other services.

## Staging environment

Staging is set up exactly like production. No changes to the production environment happen before having been rehearsed here first. Any mysterious production issues can be debugged here.

## Production environment

The big iron. Logged, monitored, cleaned up periodically, squared away and secured.

# Bill of Materials

This document must be included in every build artifact and shall contain the following:

1. What version(s) of an SDK and critical tools were used to produce it
1. Which dependencies have been included
1. A globally unique revision number of the build (i.e. a git SHA-1 hash)
1. The environment and variables used when building the package
1. A list of failed tests or checks


# Security

Be aware of possible security threats and problems. You should at least be familiar with the [OWASP Top 10 vulnerabilities](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project), and you should of monitor vulnerabilities in any third party software you use.

Good generic security guidelines would be:

## Docker

**Using Docker will not make your service more secure.** Generally, you should consider at least following things if using Docker:

- Don't run any untrusted binaries inside Docker containers
- Create unprivileged users inside Docker containers and run binaries using unprivileged user instead of root whenever possible
- Periodically rebuild and redeploy your containers with updated libraries and dependencies
- Periodically update (or rebuild) your Docker hosts with latest security updates


## Credentials

Never send credentials unencrypted over public network. Always use encryption (such as HTTPS, SSL, etc.).

## Secrets

Never store secrets (passwords, keys, etc.) in the sources in version control

Probably the easiest way to handle secrets is to put them in a separate file on the servers that need them, and to be ignored by version control. You can keep e.g. a `.sample` file in the version control, with fake values to illustrate what should go there in the real file.

## Login Throttling

Place limits on the amount of login attempts allowed per client per unit of time. Lock a user account for specific time after a given number of failed attempts (e.g. lock for 5 minutes after 20 failed login attempts).
The aim of these measures is make online brute-force attacks against usernames/passwords infeasible.

## User Password Storage

> Never EVER store passwords in plaintext!

Never store passwords in reversible encrypted form, unless absolutely required by the application / system.


## Audit Log

For applications handling sensitive data, especially where certain users are allowed a relatively wide access or control, it's good to maintain some kind of audit logging—storing a sequence of actions / events that took place in the system, together with the event/source originator (user, automation job, etc). This can be, e.g:

    2012-09-13 03:00:05 Job "daily_job" performed action "delete old items".
    2012-09-13 12:47:23 User "admin_user" performed action "delete item 123".
    2012-09-13 12:48:12 User "admin_user" performed action "change password of user foobar".
    2012-09-13 13:02:11 User "sneaky_user" performed action "view confidential page 567".
    ...

Avoid logging personally identifiable information, for example user’s name.

If your logs contain sensitive information,  make sure you know how logs are protected and where they are located also in the case of cloud hosted log management systems.

If you must log sensitive information try hashing before logging so you can identify the same entity between different parts of the processing.

## Temporary file storage

 * Make sure you are aware where your application is storing temporary files.
 * If you are using publicly accessible directories like `/tmp` and `/var/tmp`, make sure you create your files with mode 600 readable only.
 * Alternatively, have a protected directory for storing temporary files.

# Checklists

## Responsibility checklist

In bigger projects, especially when multiple parties are involved, it is crucial to keep track of all different aspects and its responsibilities. The following table illustrates how a go-live checklist for releasing a website could look like:

| Aspect    | Task                              | Responsible person / party | Deadline     | Status            |
|---        |---                                |---                         |---           |---                |
| Frontend  | Website wireframes                | e.g. Company B / Person X  | e.g. 17.6.   |  e.g. in progress |
| Frontend  | Website design                    | e.g. Company A / Person Z  | e.g. 23.7.   |  e.g. waiting     |
| Frontend  | Website templates                 |   |   |   |
| Frontend  | Content creation and population   |   |   |   |
| Backend   | Setup CMS                         |   |   |   |
| Backend   | Setup staging environment         |   |   |   |
| Backend   | Setup production environment      |   |   |   |
| Backend   | Migrate hosting services to client accounts |   |   |   |
| Backend   | DNS configuration                 |   |   |   |
| Backend   | Setup website analytics           |   |   |   |

## Release checklist

When you are ready to release, remember to check off everything on your release checklist! The resulting peace of mind, repeatability and dependability is a great boon.

You *do* have one, right? If you don't, here is a good generic starting point for you:

* [ ] Deploying works the same no matter which environment you are deploying to
* [ ] All environments have well defined names, and they are referred to using those names
* [ ] All environments have the same underlying software stack
* [ ] All environment configuration is version controlled
* [ ] The product has been tested from the networks from where it will be used (e.g. public Internet, customer LAN)
* [ ] The product has been tested with all of the targeted devices
* [ ] A versioning scheme has been defined
* [ ] Rolling back a deployment is possible
* [ ] Backups are running
* [ ] Restoring from a backup has been tested
* [ ] No secrets are stored in version control
* [ ] Logging is turned on
* [ ] Logging includes exceptions and stack traces where appropriate
* [ ] Release notes have been written
* [ ] Server environments are up-to-date
* [ ] A plan for updating the server environments exists
* [ ] The product has been load tested
* [ ] A method exists for replicating the state of one environment in another (e.g. copy prod to QA to reproduce an error)
* [ ] All repeating release processes have been automated

# General questions to consider

* What is the expected/required life-span of the project?
* Is the project one-off, or will there be continuous development?
* What is the release cycle for a version of the service?
* What environments (dev, test, staging, prod, ...) are going to be set up?
* How will downtime of the production service impact the value of the service?
* How mature is the technology? Is major changes that break backward compatibility to be expected?

# Generally proven useful tools

* [Postman](https://www.postman.com) Postman is an API platform for building and using APIs. Postman simplifies each step of the API lifecycle and streamlines collaboration so you can create better APIs—faster.


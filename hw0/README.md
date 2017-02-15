# Homework 0


**Due date for registering your GitHub account**: Sunday, 19 February
  at 11:59 PM SAST

**Due date for submission**: Wednesday, 22 February at 11:59 PM SAST

The purpose of this assignment is for you to learn the tools that you
will be using when working on and submitting your assignments for
COMS4037.

## Your task

1. If you don't have a GitHub account, create one.

2. Register your GitHub username with this course by filling out [this
form](https://goo.gl/0N53Wg).  This is due on Sunday, 19 February --
**you will not be able to submit any assignments, including this one,
if this is not done**.

3. On your local machine, create a repository that you will be using
for your COMS4037 assignments. Create a file `done.txt` in a
subdirectory 'hw0' of that repository and commit it.

4. Once notified that your private repository for this course has been
created, push your local repository to your private GitHub repository
for this course.

5. You are done -- you have submitted your assignment by having
completed the previous step.

**Good luck with your first assignment!**

## `git` and GitHub

To be able effectively work on your assignments and submit them, you
need to learn to use both `git` and GitHub.

`git` is a version control tool that helps you to keep track of
different versions of your files, synchronize them across different
machines, and -- though not for the purposes of this course --
collaborate with others while working on
them. [GitHub](https://github.com) is a hosting service for git
repositories with a convenient web interface.

It is worth your while to invest time and effort in learning to use
git and GitHub effectively -- you will be using them throughout this
course, and possibly for a very long time afterwards. A good place to
start learning about them is [this](http://git-scm.com/book/en/v2)
book. Additional resources are suggested at the [course's
web-page](http://www.cs.wits.ac.za/~dmitry/coms4037/).  GitHub's help
page also contains numerous links to resources for learning about both
`git` and GitHub.

## Creating and registering your GitHub account

If you don't yet have a GitHub account, create one by following the
instructions [here](https://help.github.com/articles/set-up-git/).

Once you have an account, register it with this course by filling out
[this form](https://goo.gl/0N53Wg).  After you've registered your account with
us, you will receive an email message with a link to your personal
GitHub repository for this course; this message will be sent to the
email address associated with your GitHub account.

## Configuring `git` to by-pass Wits proxy server

Before you can fetch/pull repositories from or push repositories to
GitHub from a local machine on Wits campus, you will have to set up
`git` to by-pass the university proxy server.  To do that, execute the
following command:

git config --global https.proxy
"http://students\student_number:password@proxyss.wits.ac.za:80"

## Submitting assignments in COMS4037

All assignments will be submitted by pushing to your private
repository associated with this course.

For the duration of the course, you will have access to two
repositories in the `WITS-4037` GitHub organization:

1. [A homework repository](https://github.com/WITS-COMS4037/hw)
containing public information about the homework assignments. You only
have the read access to this repository.  All the assignments and
updates will be posted here. You should regularly check the course's
web-site and Wit-e for announcements related to homework assignments.

2. A personal repository whose name will coincide with your student
number.  This repository is private to you -- only you and the course
staff can read from this repository: you should keep the contents of
this repository private and secure.  You will be using this repository
to submit your assignments.

Each assignment will reside in a separate top-level directory. Each
assignment has a `README.md` containing instructions, as well as the
files you are going to need to complete the assignment. Right now, you
are reading a `README.md` for hw0.

At the start of the course, your personal repository will be empty,
except for the `README.md` file at its root.

## Suggested workflow

In this section, we are going to suggest a workflow for completing
your assignments; if you find an alternative workflow more convenient,
you are welcome to use it.

We suggest that you create a repository on your local machine and link
it up with both of your remote repositories for this course; you might
want to tag the public repository containing the assignments as
"course" and your private repository as "origin". Assuming you have
adopted this naming convention, before starting work on your
assignment, you will pull (or fetch and merge) from "course" into your
local repository, and once you've completed your work, you will push
it to "origin". To make pulling from "course" easier, you might want
to set up a branch in your local repository that tracks
"course/master".

Working out the details of how the tasks described in the preceding
paragraph are done is left to you as part of your first
assignment. Your are welcome to seek help on this both from your
fellow students and from the course staff. A brief presentation on
using `git` and GitHub will be given at the first tutorial.



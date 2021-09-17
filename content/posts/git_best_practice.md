---
title: "Project Best Practice"
date: 2021-09-08T14:55:21+05:30
Summary: "Their are serveral pratice like commit conventions, issues, pr, merge templates, branch naming conventions and mantaining changelogs"
---
In most of the projects, we use Git or a version control tool to keep track of our development process. Even though we use a tool like Git, it's not enough to keep track of the project changes. If you look at projects where more than three people are contributing in a day, then the commit section will be messy. When you have more developers and every one of them is pushing their code to your project, how will you be able to know what kind of things are pending and which issues are fixed? 

This is where the commit convention comes in. It means that we are following some rules to name our commit messages. These rules will help us to maintain our commits message, and it will be much easier to understand for the whole team collaborating on the project. For example, imagine a commit message phrased as camera access added, contact access fixed on IOS. These messages fall short of conveying what has been done. On the first message, camera access added.  It doesn't tell if it is fully implemented or not also it is hard to know if it's a feature or not.  

We can solve these issues by using a commit convention. First of all, we need to have the rule to format our message. For this example, I will be going with a widely used format.  
> **type**: commit message in short. [description]

In the above format, type means what kind of thing you achieved with the commit. Use a descriptive message like style change, issues fixed, implement a new feature. But instead of giving a full description, we will be using short words. 
For example, `feat: camera services add`


> It will be good if you add a short description of what you did on the code in the body. Using a body is optional. For example, `git commit -m"feat: camera services add" -m"maded a camera services to be used with any module"`. The second **m** params contain body.  


We talked about commit convention and type. But we didn't discuss commonly used different types in commit messages. feat, fix, style, refactor, test, docs, chore are some commonly used ones.  

* feat: This type is used if you are committing a code that is used to develop a feature. For example, implementing a camera is a part of the image capture of the user feature.
* fix: This type is used when your commit is used to fix issues.
* style: This is used when your commit code is deals with style updates
* refactor: This is used when your committed code deals with refactoring your code for better readability.
* doc: This is used when you have created or updated docs in your project. It can be a simple readme file or comments for better understanding. (But I highly recommend you to give comments when you are coding the feature).
* chore: This is used when changing something on your project for better maintainability of the project. Like splitting functions or making services or changing file structure.


Even though we are using this convention we are not sure that everyone in our team will use it. Because these things can be easily forgotten if we are doing in a hurry. That is why need a tool to check that if we are following this rule or not.

That's where [commitlint](https://commitlint.js.org/#/) comes into play. Just like other linting packages, it is used to set rules in our git commit message.

To add commitlint to your project you need a [node](https://nodejs.org/en/)  and [npm](https://www.npmjs.com/package/npm) installed. Now run the command at the root of your project.

```bash
# for linix and mac
npm install --save-dev @commitlint/{config-conventional,cli}

# for windows
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```
Now we need to craete a config file. Configuration of the commitlint is done in this file.  
Create a file named `.commitlintrc.json` at the root of your project.

```json
{
    "extends" : ["@commitlint/config-conventional"]
}
```
If you want to know more about above rule we used. https://github.com/conventional-changelog/commitlint/tree/master/@commitlint/config-conventional

But the only thing is commitlint will not automatically work when you commit a change in your project. How it to work we need another tool named husky. Husky is a git hook that runs when you run git commands.  

To install husky  
```bash
npm install husky --save-dev

# install husky it will create .husky folder
npx husky install
```
Now add commitlint to husky when we give commit messsage.
```bash
npx husky add .husky/commit-msg "npx commitlint --edit $1"
```
Now when ever you commit a change then this command will work and throw out error if your don't follow commit convention.

Now to try committing the changes

```bash
git commit -m"feat: commitlint add" -m"Added new commit convention"
```

## Changelog

We talk about commit convention but it's not enough to maintain the project. Think like this when you're doing a project you will have so many kinds of version releases like 1.01, 1.02. In this case, how will you know what all feature is implemented and what all fixes are done?  

This is where changelog comes into play. You can use changelog to create version logs.

```bash
npm i --save-dev standard-version
```

Now you need to add a script you can

```json

{
  "scripts": {
    "release": "standard-version"
  }
}
```

Now run the script to make a changelog

```bash
rpm run release
```

For example, https://gitlab.com/evolvingkid/testing-react-native/-/blob/test/CHANGELOG.md

## Merge Request

We talk about commit messages and changelog. Like this, we even have templates for merge requests. It is used because like commit messages people will write unwanted or not well-mannered descriptions in merge requests or pull requests. That is why we need to set a rule or template that enforce contributors to follow a well-mannered description in our merge request or pull request.

To make a template we need to create a dir as `.gitlab/merge_request_templates` in our root dir.


```md

# Description

Please include a summary of the changes and issues that are fixed.

# Dependencies

please note down the dependencies that use in this code change.

For example 
* react-native : 1.02

# Native Dependencies

please note down the native dependencies that use in this code change. Which can have issues with code push

For example 
* react-native : 1.02

# checklist

- [ ] used convection commits

```
This template was designed for a react native project in mind. That is why I added dependencies. Your merge request needed to be descriptive. You can use different types of merge requests according to your need. 

Now you can find the template in the merge request. Example : https://gitlab.com/evolvingkid/testing-react-native/-/blob/master/.gitlab/merge_request_templates/merge_request_default.md

Merge request will look like : https://gitlab.com/evolvingkid/testing-react-native/-/merge_requests/1

In the case of GitHub, If you want to make a pull request like a merge request you need to create a file on `.github/pull_request.md `. Now need to add the same MD file like on merge request file.  

Like example : https://github.com/evolvingkid/github_templates/blob/main/.github/pull_request_template.md  

Merge request will look like: https://github.com/evolvingkid/github_templates/pull/2  

## Issues  

When you are doing a project with two more contributors it will be hard to maintain issues and features. That is why we use issues templates like merge templates. In the issues template, we can create multiple issues like a feature request for implementing new features or create issues for mobile, web, or server as you want.  

To create issues templates create file as `.gitlab/issue_templates/bug_report.md`  

Then fill the code below.  

```md

# name

Keep it brief and use correct terms

# Description

Give a brief description of how the bug occurred.

# Environment

Tell us about what env used like qa, dev, prod

# device

Tell us about the device used to check like web, android, ios

# logs

If there are any logs, paste them.

# Steps to reproduce

steps to reproduce the issues

# Expected

Please explain the expected output

# Actual

Please explain the actual output
```

This will create a template for your issues.  

Example: https://gitlab.com/evolvingkid/testing-react-native/-/blob/master/.gitlab/issue_templates/bug_report.md  

This issues will look like : https://gitlab.com/evolvingkid/testing-react-native/-/issues/1  

If you want to make the same kind of feature in Github it’s much simpler. Just do the **project settings** then scroll down. You will see the **issues checkbox** and you will see the setup template option just below. Click that button now you will see options to choose different types of templates. You can choose a built-in template or a custom one. The custom one is the same as the GitLab MD file

For example the issues will be looking like: https://github.com/evolvingkid/github_templates/issues/1

## Branches

Like, commit message convections we need to follow on naming branch in our project too. If you are making a project then there will be a **master** branch and a **release** branch. The release branch will only contain code that is released after testing. Then there will be a dev branch where all your development goes on. It will be good if you do commit to the dev branch instead of committing to it.  

If you’re doing feature work like accessing a camera. Then it will be good to make a branch name like **feat-camera** or if you are changing a style or refactor a code then use a prefix like **style-** or **refactor-**.  

If you making a branch to fix an issue then it will be good to make the prefix as you issue id or number then followed by a short name of the issue. Like my project has an issue with accessing storage in ios and I have an opened issues with ID **321**. Then my branch name will be like `321-camera-ios`. This will help us to know what issues we are working on. Like when you commit the code we can easily mention the issues ID as **#321** without looking through our issues list.  

This approach is also good if you have a developer who is in charge of code review where he will be the one to check your merge request and accept it to the dev branch. Also if your are in GitHub and using milestone to track your progress then it will easy for tester, project manager and devs to know the progress of our development.  

Commitlint, brach naming, issues and merge templates are used to make your project works more organized. It's not mandatory that you must use these things. But it will be good to be well maintain your project. 
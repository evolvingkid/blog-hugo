---
title: "Git Best Practice"
date: 2021-09-08T14:55:21+05:30
Summary: "Their are serveral pratice like commit conventions, issues, pr, merge templates, branch naming conventions and mantaining changelogs"
tags:
    - git
---
In any development using a git or source tracking is essential. But even though we will be using something like git it's not enough to keep our code or program in a well-mannered way. So that others can contribute to it. This is where certain practice comes into play. Like, writing good commit messages. There are several conventions that we follow to have a well mannered commit message.
<br/>
For example :
```  type: commit-msg ```

## Commit Message
The type means what kind of scope this code change we did. Some usual types we use are feat, fix, style, refactor, test, docs, chore. 

* fix: means something related to fixing
* feat: means something related to a feature we are working on.
* style: means something related to the styling of the app.
* refactor: means something related to refactoring of the code
* test: means something related to testing of the project.
* docs: means something related to documentation on the project.
* chore: means something related to code maintainability.

In a team project maintaining this standard is hard. That's why we can use tools like commitlint to set up rules in our commit.

To install and setup the tool.  
<br />
Install commitlint (This is for Linux and mac)  
```bash
npm install --save-dev @commitlint/{config-conventional,cli} 
```  
If you are using windows: x
```bash
 npm install --save-dev @commitlint/cli @commitlint/config-conventional
```
Now create a `.commitlintrc.json` on our project home dir to define our rules.  
now add this line to the `.commitlintrc.json` file  
```json
{
    "extends" : ["@commitlint/config-conventional"]
}
```
This will give us basic commit rules.  
Now install husk.  
```bash
npm install husky --save-dev
```
Now add a script in your `package.json` file
```json
 "scripts": {
    "postinstall": "husky install"
  },
```

> You dont need this the above step you can skip it

now run command  
```bash
npx husky install
```
now run command  
```bash
npx husky add .husky/commit-msg "npx commitlint --edit $1"
```
After these try to commit like
```bash
 git commit -m "this is my 1st commit"
```

This will surely fail. After the rule is setup you need to follow the commit convention like `type: msg`. For example  
```bash
git commit -m "feat: initial project setup" -m "The project is set up to do collaborative work with our teammates"
```

After following good commit convection. It will be easy to know which commits means what like which commit is done for a fix, implement a new feature, or refactoring of your code. 

Having a good commit message is good but think like If your app has a ton of release versions then how you are going to know what kind of things you implement on those releases. This is where changelog comes in.

## changelog

In every project maintaining the changelog file is also important. So that newer and other devs can see that all are done and fixed in a version release.
For this purpose, we will use another npm package called standard-version.

```bash
npm i --save-dev standard-version
```

Now add script

```json
{
  "scripts": {
    "release": "standard-version"
  }
}
```

now run the script 


```bash
rpm run release
```

Now as for a demo how the changelog file look like 
https://gitlab.com/evolvingkid/testing-react-native/-/blob/test/CHANGELOG.md

Doing npm release manually is not that good. It will be better you connect an action (CI/CD) that triggers when you push to the master branch or a tag is pushed.

## Merge Request
Like the commit message in the merge request, we must have a well-defined description. For this, we can make templates. In GitLab, all the templates are store in `.gitlab/ `    dir.

We can make a md file like default_merge_request.md and fill this code.

I made this merge request template for react native project. You can create the template based on what you want to reflect. When you are doing a merge request template it must be descriptive of what you did in the code. It will be better to add a checklist if you need your companion to check some kind of rule or anything before committing the code.

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

Now you can find the template in the merge request. Example : https://gitlab.com/evolvingkid/testing-react-native/-/blob/master/.gitlab/merge_request_templates/merge_request_default.md

Merge request will look like : https://gitlab.com/evolvingkid/testing-react-native/-/merge_requests/1


Now in the same case if you want to make a pull request template on Github. It's kind of similar. You can create a `.github/pull_request.md` file in your project then add the above md file. 

Like example : https://github.com/evolvingkid/github_templates/blob/main/.github/pull_request_template.md

Merge request will look like:
https://github.com/evolvingkid/github_templates/pull/2

## Issues

When you are doing a project with more than 2 people. You need a way to track all the bugs, feature requests, or issues of your project. So for these kinds of different types of issues, it's easy to make a template for each of these use cases. Like, merge request issues must be descriptive.


Like the merge request template, issues description can also have a template. Create a file name `.gitlab/issue_templates/bug_report.md`. 

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

If you want to make the same kind of feature in Github it's much simpler. Just do the project settings then scroll down. You will see the issues checkbox and you will see the setup template option just below. Click that button now you will see options to choose different types of templates. You can choose a built-in template or a custom one. The custom one is the same as the GitLab MD file 

For example the issues will be looking like:
https://github.com/evolvingkid/github_templates/issues/1

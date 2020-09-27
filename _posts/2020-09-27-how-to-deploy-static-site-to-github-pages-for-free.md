---
title: How To Deploy Static Site to GitHub Pages for Free!
layout: post
post-image: https://i.ytimg.com/vi/Zndd7Tp16PA/maxresdefault.jpg
description: How to deploy a Static Site to GitHub Pages for free.
tags:
- GitHub
- GitHub Pages
- Deploy
- CI/CD
---

In this post is part of the video series where we deploy a static site to different cloud providers. In this post we will look at how to deploy to GitHub Pages.

you can find all the code in: https://github.com/coding-flamingo/BlazorStaticDeploy
## Video Version
<iframe width="560" height="315" src="https://www.youtube.com/embed/Zndd7Tp16PA" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
## Text Version
### Prerequisites
1. Have a Static Site in a GitHub repo. 

### Tutorial
#### Setting Up the GH Action
First we are going to create the workflow file for the GitHub Action, we first start with setting up the name for the Action and the trigger for the action (change this to the name you want and the trigger you want, this sample uses the name DeployToGHPages and it gets triggered when we push to the 'dev' branch):
```
name: DeployToGHPages
 
on:
  push:
    branches:
      - dev
```
Then if you have to compile your site (in this example we use a Blazor Site that needs to be compiled) you add your compiling steps and publish the artifacts for the deploy action to download (if you don't have to compile your site, you can skip this but note the deploy action grabs from artifacts so you would have to change that).
```
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    
    - name: Set up .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '3.1.102'

    - name: Build with dotnet
      run: dotnet build --configuration Release

    - name: dotnet publish
      run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/blazorapp

    - name: dotnet changehtml
      run: mv ${{env.DOTNET_ROOT}}/blazorapp/wwwroot/GHIndex.html  ${{env.DOTNET_ROOT}}/blazorapp/wwwroot/index.html
     
    - name: Publish artifacts
      uses: actions/upload-artifact@master
      with:
        name: blazorStaticSite
        path: ${{env.DOTNET_ROOT}}/blazorapp
```
#### Index.html Side Note
If you notice, in the build task, I change the regular index.html for my github version this is because Blazorin GH Pages has trouble redirecting some of the pages so I had to add this script inside the body:
```
 <script type="text/javascript">
        // Single Page Apps for GitHub Pages
        // https://github.com/rafrex/spa-github-pages
        // Copyright (c) 2016 Rafael Pedicini, licensed under the MIT License
        // ----------------------------------------------------------------------
        // This script checks to see if a redirect is present in the query string
        // and converts it back into the correct url and adds it to the
        // browser's history using window.history.replaceState(...),
        // which won't cause the browser to attempt to load the new url.
        // When the single page app is loaded further down in this file,
        // the correct url will be waiting in the browser's history for
        // the single page app to route accordingly.
        (function (l) {
            if (l.search) {
                var q = {};
                l.search.slice(1).split('&').forEach(function (v) {
                    var a = v.split('=');
                    q[a[0]] = a.slice(1).join('=').replace(/~and~/g, '&');
                });
                if (q.p !== undefined) {
                    window.history.replaceState(null, null,
                        l.pathname.slice(0, -1) + (q.p || '') +
                        (q.q ? ('?' + q.q) : '') +
                        l.hash
                    );
                }
            }
        }(window.location))
    </script>
```
And if you do not have a custom URL (keep an Eye out for a post on that ;)) your site will start on "YOUR GITHUB URL/YOUR REPO NAME" so you have to also change the base href to start on "/your repo name" for example mine was:
`<base href="/BlazorStaticDeploy/" />`
#### Back to GH Action
Going back to the GitHub Action, we then have to deploy the site to gh-pages. To do this I download the Artifacts created in the build stage, and then using crazy-max's github action I deploy to the gh-pages brach (which is the default branch for github pages) here is the rest of the YAML file and then I will go through each of the Variables:
```
 deploy:
    needs: build
    name: Deploy
    runs-on: ubuntu-latest
    steps:
 
    # Download artifacts
    - name: Download artifacts
      uses: actions/download-artifact@master
      with:
        name: blazorStaticSite
 
    - name: Deploy to GitHub Pages 🚀
      uses: crazy-max/ghaction-github-pages@v2.1.1
      with:
        # Build directory to deploy
        build_dir: wwwroot 
        # Allow Jekyll to build your site
        jekyll: false
      env:
        GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
```

1. First I download the artifacts from "blazorStaticSite" which is the folder I published the artifacts in the previews step. 
2. Then I use[ crazy-max/ghaction-github-pages@v2.1.1](https://github.com/crazy-max/ghaction-github-pages) and I set the `build_dir` to wwwroot which is where my site is. 
3. You can set a `target_branch` but I am using the default that is 'gh-pages' but if you are going to use a different branch you set the name here. 
4. I set jekyll to false since this is not a jekyll site. 
5. I pass the `GITHUB_TOKEN` from the github secrets (in the next section I will show how to set it)
#### Getting a GitHub Token
1. In GitHub, Click on your user picture on the top right and click on settings.
2.  Click on Developer settings:

![GitHub Settings](/assets/images/DeveloperSettings.jpg)

3. Now Click on Personal Access Tokens.
4. In the top right there is a generate new token button.
5. Enter you password.
6. Enter a name for your token.
7. Give select the full control of private repositories (**note:** this will give it permission to manage all your public and private repos so make sure you treat this token as a password)
8.  Click the Generate button. 
9.  Copy the token created. 
#### Saving the GitHub Token
1.  Go back to your repo and click on the settings tab:

![GitHub Settings](/assets/images/GitHubSettings.jpg)

2. Then on the left click on Secrets.
3. Click on new Secret:
![GitHub Settings](/assets/images/RepoSecrets.jpg)
4. Create a new Secret called `GH_TOKEN` **Note:** the name is the same as we specified in the GitHub Action. 
5. Paste the token you copied before as the secret and save it. 
#### Commit and Push
1. commit and push to the branch you set as the trigger in the GitHub Action to trigger the action. 
2. in your repo go to Actions and make sure the action ran successfully. 
#### Enabling GH Pages on Your Repo
1. In your repo go to settings:

![GitHub Settings](/assets/images/GitHubSettings.jpg)

2. Scroll down to the GitHub Pages section:

![GitHub Pages](/assets/images/ghPagesEmpty.jpg)

3.  Select your GitHub pages branch (default is gh-pages) and click save. 
4.  Done! go To de site and enjoy your new Site :)

![GitHub Pages](/assets/images/ghPages.jpg)
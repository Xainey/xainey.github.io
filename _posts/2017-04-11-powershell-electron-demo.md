---
layout: post
title:  Back to the Frontend | A PowerShell-Electron Demo
date:   2017-04-11 21:46:00
comments: false
description: Creating a GUI app with PowerShell and Electron
categories:
    - PowerShell
tags:
    - PowerShell
    - Electron
    - NodeJS
    - jQuery
    - Bootstrap4
thumbnail: /images/posts/powershell-electron-demo/back-to-the-frontend.png
---

![Img](/images/posts/powershell-electron-demo/back-to-the-frontend.png)

## Abstract

The following article demonstrates using PowerShell and Atom Electron to create and package an application.
In the following pages, we will cover creating the frontend and backend of a simple disk utility called *Diskr*.
This demo project was inspired by Stephen Owen's XAML based GUI demo.
If you want to work along or view the source the project can be found on Github at [xainey/powershell-electron-demo].

-----

* TOC
{:toc}

-----

## Introduction

I first started using PowerShell to build simple tooling and automation scripts.
Since all of our PCs already had PowerShell installed, we could hand over scripts that could run on any system.
The problem was getting others to learn how to use PowerShell.
The average CLI experience for our target user didn't go much further than `ping` or `ipconfig`.

*Here guys, I made a simple script, run it like this:*

```console
./ConfigureDelorean.ps1 -Month 8 -Day 26 -Year 1985 -Hour 9
```

![JackieChan](/images/posts/powershell-electron-demo/jackie-chan-meme.png)
*Average Response*

After seeing that our scripts were not getting used, we started wrapping them in [Freestyle Jobs in Jenkins][Hodge-PowerShell-Jenkins] to give the user a GUI.
It looked like a great idea; not only did users get an easy form, but we also generated an audit each time the job was run.
None of our Ops guys wanted to take the time to learn Jenkins.

*Okay, we just need to put this on their desktop and give them a GUI wrapper.*

A quick search and we found [Stephen Owen's GUI ToolMaking Series].
Generating XAML in Visual Studio was easy enough and with some modest effort, we created a few simple apps.
However, it started to become tedious managing the layout and styling as the projects grew.
When it comes to designing frontend interfaces, I've made apps in C# using WPF as well as Java for Desktop and Android.
To this day, I still find HTML/CSS to be the absolute easiest way to control layout.

In *Back to the Frontend &trade;*, we are going to use [Atom Electron] to create our desktop application.
We will be creating a small app inspired by Stephen Owen's ToolMaking Series.

## Getting Started

As may of you may know, frontend development can be overwhelming.
You start out looking to create a project that is ultimately CSS, HTML, and Javascript.
Later you find yourself trying to figure out how to use SASS, Gulp, Webpack, Babel, Browserify, NodeJS, React, Flux, Redux, etc.
Check out the [Developer-Roadmap] for a quick bird's eye view.

In this article, we will stick to using some of the most common and basic tools.

### Installing Tools

We will need to pull in everything though git and NodeJS (npm). 
The optional tools are by preference only. If you haven't written PowerShell in VSCode do yourself a solid and go try it out.
Install these tools manually or with Chocolatey.

- Required
    - [Git]
    - [NodeJS]
- Optional
    - [VSCode]
    - [Yarn]

![Doctocat](/images/posts/powershell-electron-demo/doctocat-brown-by-jonrohan.jpg)
*Great Scott! You don't have Git installed?!? It is 2017!!!1one*

#### Using Chocolatey

For **personal** use and the lazy alike, you may want to use chocolatey to install the tools.

```powershell
# Set your PowerShell execution policy
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Force

# Install Chocolatey
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# Install Packages
choco install git visualstudiocode nodejs yarn -y
```

For an **organization**, you may want to consider your own internally hosted packages as those packages
won't be subjected to distribution rights like the community repository is (being in the public domain).
Rob ([@ferventcoder]) told me this, trust him, he made Chocolatey.

### Scaffolding a New Project

Go to the directory you want to make your project and clone the [electron-quick-start] repo.
I've chosen to use this project as our boilerplate since it is incredibly simple and official.

In your PowerShell console clone the project:

```console
cd ~/Github/
git clone https://github.com/electron/electron-quick-start diskr
cd diskr
```

Remove the existing git folder reinitialise it. There may be a more eloquent approach such as [git-clone-init].

```console
rm .\.git -Force -Recurse
git init
```

Install the node dependencies and start the app.
If you are using **yarn** instead of **npm** simply use `yarn` or `yarn install`.

```console
npm install
npm start
```

![FirstRun](/images/posts/powershell-electron-demo/first-run.png)
*First Run*

### Getting Oriented with Electron

Before we start writing any code, let's take a quick glance at the hotkeys, tools, and files we have to work with.

#### Initial Menu / Hotkey Items

Command                       | Hotkey
------------------------------|---------------------
View > Toggle Developer Tools | `Ctrl-Shift-I`
View > Reload                 | `Ctrl-R`
View > Force Reload           | `Ctrl-Shift-R`
{:.table-striped}

#### DevTools

If you have ever used Chrome DevTools before, rejoice here they are.
If you are a complete beginner relax, this tool is very user-friendly.
You should start by getting familiar with the `Elements` and `Console` Tabs.

* The `Elements` tab shows you HTML and CSS.
* The `Console` tab shows console logs and errors.

![DevTools](/images/posts/powershell-electron-demo/chrome-dev-tools.png)
*Chrome DevTools*

Both of these tabs can be used to "play" with the HTML, CSS, and JavaScript.

Try it out. Go to the console tab and enter in this ES2015 Expression:

```javascript
[1, 5, 14, 130, 9].filter(val => val > 10)
```

#### Files

Open the project in [VSCode] or your editor of choice.
Since my console (ConEmu/PowerShell) is still in the project directory I can type `code .` to open the project in VSCode.
I started doing this with sublime `subl .` on *OS X* a few years ago. The time I've saved since is huge.

![DevTools](/images/posts/powershell-electron-demo/electron-files.png)
*Electron Files*

`.gitignore`, `license.md`, `readme.md` are standard to most git projects.

The main files are:

* `index.html`
    * Initial frontend page loaded by main.js
* `main.js`
    * Main entry point into our Electron app
* `package.json`
    * Manages node dependencies, packaging meta, CLI scripts
* `renderer.js`
    * JavaScript for index.html

## Importing Frontend Dependencies

### HTML/CSS

For our front-end HTML/CSS we are going to use the [Bootstrap v4] framework.
At this time, v4 is still in alpha so feel free to fall back to v3 if needed.

To pull in bootstrap we have a few options:

1. Download it manually
2. Reference it by CDN
3. Use a [package manager][bootstrap-packages] such as npm, bower, rubygems, nuget, or composer

Since we already have NodeJS installed we are going to use npm to make it easier to follow along.

#### Bootstrap from NPM

In your project root run the following command to save bootstrap to our dependencies.

```console
npm install bootstrap@4.0.0-alpha.6 -S
```

> **Note:** In the future, you will want to save these to your dev-dependencies and use something like `gulp` or `webpack` to package them.
{:.pull .blue}

We can now edit our `index.html` file and make sure bootstrap is working.

**For the index.html we are going to:**

1. Update the `<title>` (this is also the title bar on the app)
2. Include a `<link>` reference to bootstrap.min.css in our `node_modules` directory
3. Add a `div.container` to wrap the body content
4. Use the [alert message][bootstrap-alert] component to make sure bootstrap is working

**Edit index.html**

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Diskr</title>
    <link rel="stylesheet" href="node_modules\bootstrap\dist\css\bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <div class="alert alert-success" role="alert">
        <strong>Well done!</strong> You successfully read this important alert message.
      </div>
    </div>
  </body>
  <script>
    require('./renderer.js')
  </script>
</html>
```

Start the app again with `npm start` or refresh it with `Ctrl-R`.

![TestingBootstrap](/images/posts/powershell-electron-demo/testing-bootstrap.png)
*An Alert is Born!*

### Javascript

For our JavaScript requirements we are going to use [jQuery].
Most people have seen, used, or tasted jQuery.
It tastes okay at times, but for larger form intensive projects frameworks like `Angular` or `React` start to taste better.
Personally, I love `VueJS` -- it's the cat's pajamas (this is how my grandfather let me know something is "Hip").

#### jQuery from NPM

Again, from your project root, run the following command:

```console
npm install jquery -S
```

Now in `renderer.js`, we will import jQuery and test to make sure it works.

```javascript
// Require jQuery
const $ = require('jquery');

// Test jQuery: ES2015 Arrow Syntax
$(document).ready( () => console.log("Page is loaded!") )
```

Start the app again with `npm start` or refresh it with `Ctrl-R`.

![TestingJQuery](/images/posts/powershell-electron-demo/testing-jquery.png)
*Console.log() shows are message in DevTools*

## Setting up PowerShell

After a quick search I found two decent NodeJS projects for PowerShell:

- [rannn505/node-powershell]
- [IonicaBizau/powershell]

### Node-PowerShell From NPM

We are going to be using the one by `rannn505`.
I chose the one with the better documentation and most stars.

```console
npm install node-powershell -S
```

Looking at this nice [API][node-powershell-api], we can use PowerShell with JavaScript [Promise] based methods.

### Running PowerShell in Javascript

That's right Daddy-O, it's time to use some PowerShell with JavaScript.

We will now update `renderer.js` to:

```javascript
// Require Dependencies
const $ = require('jquery');
const powershell = require('node-powershell');

// Testing PowerShell
$(document).ready(() => {

    // Create the PS Instance
    let ps = new powershell({
        executionPolicy: 'Bypass',
        noProfile: true
    })

    // Load the gun
    ps.addCommand("Roads? Where we're going, we don't need roads.")

    // Pull the Trigger
    ps.invoke()
    .then(output => {
        console.log(output)
    })
    .catch(err => {
        console.error(err)
        ps.dispose()
    })

})
```

Here is a quick breakdown:

- `line: 3`  - Require the dependency
- `line: 9`  - Create a PS Instance
- `line: 15` - Add a Command to be executed
- `line: 18` - Invoke the Command
- `line: 19, 22` - Handle the response with promises

Start the app `npm start` or refresh it with `Ctrl-R`.

![PowershellError](/images/posts/powershell-electron-demo/powershell-error-example.png)
*Crap, we broke it already.*

Looks like we need to escape our string a wee bit.

```javascript
ps.addCommand("\"Roads? Where we're going, we don't need roads.\"")
```

![StringEscapeHell](/images/posts/powershell-electron-demo/doc-holding-lightning.jpg)
*Marty, It looks like we are going to string escape hell!*

*Save* and *Refresh* again.

![PowershellResponseGood](/images/posts/powershell-electron-demo/powershell-response-good.png)
*Now we are cooking with plutonium!*

Now instead of just printing a string, let's inline a PowerShell expression to get system information:

```javascript
ps.addCommand("Get-Process -Name electron")
```

![PowershellSystemCommand](/images/posts/powershell-electron-demo/powershell-system-command.png)
*Hash Table is passed from PowerShell to JavaScript as a string*

### Running a PS1 script

Next, we will test running a `.ps1` script from *node-electron*.
This is more practical for managing and passing arguments.

In the project root create the file `Test-Power.ps1`:

```powershell
param (
    [Parameter(Mandatory = $true)]
    [double] $GigaWatts
)

if ($GigaWatts -ge  1.21) {
    $canWarp = $true
} else {
    $canWarp = $false
}

@{
    GigaWatts = $GigaWatts
    CanWarp = $canWarp
}
```

In this contrived example, we take some input, do some logic, and output a hashtable.

To run *Test-Power.ps1*, we use `addCommand` and provide:

- The path to the file
- An array of JavaScript objects for each PowerShell parameter

Update `ps.addCommand()` in `renderer.js` to:

```javascript
    // Load the gun
    ps.addCommand("./Test-Power", [
        { GigaWatts: 1.0 }
    ])
```

Give the app a refresh and look at the DevTools Console.

![TestHashTable](/images/posts/powershell-electron-demo/test-power-hashtable.png)
*No Warping Today :[*

At this point, you can see how our PowerShell hashtable is just a big string on the JS side -- not much we can do with that.

To fix this we will use `ConvertTo-Json` in PowerShell and supply the `-Compress` flag to minify it.

Update `Test-Power.ps1`:

```powershell
@{
    GigaWatts = $GigaWatts
    CanWarp = $canWarp
} | ConvertTo-Json -Compress
```

In `rendered.js` use `JSON.parse()` to convert the JSON into a JavaScript object.

```js
.then(output => {
    console.log(output)
    console.log(JSON.parse(output))
})
```

![PowershellToJson](/images/posts/powershell-electron-demo/powershell-to-json.png)
*PowerShell hashtable -> JSON -> JavaScript Object*

## Creating the Front-End: Diskr

In `index.html` we add a small form using the [Bootstrap v4] scaffolding.
We will also add and inline CSS class `.top-buffer` to give us some buffer space between the div `.rows`.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Diskr</title>
    <link rel="stylesheet" href="node_modules\bootstrap\dist\css\bootstrap.min.css">
    <style>
      .top-buffer {
        margin-top: 1em;
      }
    </style>
  </head>
  <body>
    <div class="container">

    <!-- Form -->
      <div class="row top-buffer">
        <div class="col-lg-6 mx-auto">
        <div class="input-group">
          <input id="computerName" type="text" class="form-control" placeholder="Computer Name">
          <span class="input-group-btn">
            <button id="getDisk" class="btn btn-primary" type="button">Get Disk Info!</button>
          </span>
        </div>
        </div>
      </div>

      <!-- Output -->
      <div class="row justify-content-center">
        <div id="output" class="col-lg-6 mx-auto"></div>
      </div>

    </div>
  </body>
  <script>
    require('./renderer.js')
  </script>
</html>
```

With the form in place, we want to do a few things:

1. When the user clicks on the `<button>` grab the HTML `<input>` value
2. Pass the `<input>` value as arguments to a PowerShell script
3. Output the results on the page instead of in the DevTools console

Create `Get-Drives.ps1` in the project root.

```powershell
param (
    [Parameter(Mandatory = $false)]
    [string] $ComputerName = 'localhost'
)

 $drives = [System.IO.DriveInfo]::GetDrives() |
    Where-Object {$_.TotalSize} |
    Select-Object   @{Name='Name';     Expr={$_.Name}},
                    @{Name='Label';    Expr={$_.VolumeLabel}},
                    @{Name='Size(GB)'; Expr={[int32]($_.TotalSize / 1GB)}},
                    @{Name='Free(GB)'; Expr={[int32]($_.AvailableFreeSpace / 1GB)}},
                    @{Name='Free(%)';  Expr={[math]::Round($_.AvailableFreeSpace / $_.TotalSize,2)*100}},
                    @{Name='Format';   Expr={$_.DriveFormat}},
                    @{Name='Type';     Expr={[string]$_.DriveType}},
                    @{Name='Computer'; Expr={$ComputerName}}

$drives | ConvertTo-Json -Compress
```

Here I started using `dotNet` classes instead of `GWMI` since I wrote this on my MacBook Pro.

![doc-thinking](/images/posts/powershell-electron-demo/doc-thinking-hat.jpg)
*Since Electron is cross-platform, we can write our PowerShell to work cross OS as well.*

Now back in `renderer.js`, we are going to change our `$(document).ready()` event to `$("#getDisk").click()`.

**Note:** `#getDisk` is the jQuery selector for the ID we placed on our button in the HMTL.

Inside the `$("#getDisk").click()` event, we get the form input at the top:

```js
// Get the form input or default to 'localhost'
let computer = $('#computerName').val() || 'localhost'
```

And change our script to run `Get-Drives.ps1`:

```js
// Load the gun
ps.addCommand("./Get-Drives", [
    { ComputerName: computer }
])
```

To show the response on the page, we add the following *jQuery* command to our `.then()` function.

```js
.then(output => {
    console.log(output)
    console.log(JSON.parse(output))
    $('#output').html(output) // Show the results
})
```

Now if we run this, we can see that we are now passing `PC-Delorean` from our form to PowerShell and showing the output in the HTML.

![FormInputGetDisk](/images/posts/powershell-electron-demo/form-input-get-disk.png)
*Just pretend of the console logs uses "PC-Delorean" I forgot to fix the screenshot :)*

### Outputting the Data to a Table

Printing the JSON directly to the HTML was the first step, but now we want to output it to an HTML `<table>`.
You should be able to find table plugins for just about any JavaScript framework or library.

#### DataTables from NPM

For this example, we are going to use a framework called [DataTables] to create a simple table from our JSON reponse.

- [DataTables-Install]
- [DataTables-BootStrap4]

First, we need to pull them in using *npm*:

```console
npm install datatables.net -S
npm install datatables.net-bs4 -S
```

In the HTML `<head>` of `index.html`, reference the DataTables CSS for Bootstrap v4:

```html
<link rel="stylesheet" href="node_modules\datatables.net-bs4\css\dataTables.bootstrap4.css">
```

In `renderer.js` we want to require them in:

```js
const dt = require('datatables.net')();
const dtbs = require('datatables.net-bs4')(window, $);
```

Next, in our `.then()` promise, we can have the data create a DataTable.

```js
    .then(output => {
        console.log(output)
        let data = JSON.parse(output)
        console.log(data)

        // generate DataTables columns dynamically
        let columns = [];
        Object.keys(data[0]).forEach( key => columns.push({ title: key, data: key }) )

        // Create DataTable
        $('#output').DataTable({
            data: data,
            columns: columns
        });
    })
```

**DataTable Note**: For every column returned from PowerShell we would need to make a columns array (see [example][DataTables-JS-Array]).

We use `line 7-8` above to avoid manually adding these:

```js
    var columns [
        {title: "Name",  data: "Name"},
        {title: "Label", data: "Label"},
        // ... etc
    ]
```

![ConvertKeysAuto](/images/posts/powershell-electron-demo/convert-keys-auto.png)
*Auto Create columns from Object.Keys()*

Update `index.html` to support a *DataTable*:

```html
<!-- Output -->
<div class="row justify-content-center top-buffer">
    <table id="output" class="table table-striped table-bordered" cellspacing="0"></table>
</div>
```

Start or Refresh the app and click `Get-DiskInfo!`.

![DataTablesAll](/images/posts/powershell-electron-demo/data-tables-all.png)
*DataTables with default options*

Since, we really don't need to `search`, `paginate`, or the `info` on a small list of drives we can just configure *DataTables* to omit
these options.

In `renderer.js` update the *DataTable*:

```js
$('#output').DataTable({
    data: data,
    columns: columns,
    paging: false,
    searching: false,
    info: false,
    destroy: true  // or retrieve: true
});
```

> **Note**: Use `destroy` or `retrieve` to allow the Table to be recreated on subsequent button clicks.
{:.pull .blue}

Give the app a quick refresh:

![DataTablesSimple](/images/posts/powershell-electron-demo/data-tables-simple.png)
*DataTable after options are configured*

## Using PowerShell Remoting

Now that our app has taken shape, we need to make it work on remote machines on the network.
Since we are not using `GWMI` we need to update our script to use PowerShell remoting by using `Invoke-Command`.

Update `Get-Drives.ps1`:

```powershell
param (
    [Parameter(Mandatory = $false)]
    [string] $ComputerName = 'localhost'
)

$out = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
    [System.IO.DriveInfo]::GetDrives() |
    Where-Object {$_.TotalSize} |
    Select-Object   @{Name='Name';     Expr={$_.Name}},
                    @{Name='Label';    Expr={$_.VolumeLabel}},
                    @{Name='Size(GB)'; Expr={[int32]($_.TotalSize / 1GB)}},
                    @{Name='Free(GB)'; Expr={[int32]($_.AvailableFreeSpace / 1GB)}},
                    @{Name='Free(%)';  Expr={[math]::Round($_.AvailableFreeSpace / $_.TotalSize,2)*100}},
                    @{Name='Format';   Expr={$_.DriveFormat}},
                    @{Name='Type';     Expr={[string]$_.DriveType}}
}

$out | ConvertTo-Json -Compress
```

Refresh the app and click `Get Disk Info!`:

![PSRemotingAll](/images/posts/powershell-electron-demo/ps-remoting-all-fields.png)
*The Invoke-Command response as a variable adds a few additional fields*

Back in `Get-Drives.ps1` let's exclude the flux out of the extra fields.

```powershell
$out | Select-Object * -ExcludeProperty PSComputerName, RunspaceId, PSShowComputerName  ConvertTo-Json -Compress
```

Much cleaner. I probably don't need the PC name in the table considering that I just typed in the above box.

*Hello, I'm Dory.*

![PSRemotingAll](/images/posts/powershell-electron-demo/ps-remoting-excluded.png)
*Updated with filtered $out*

### Font-End Error Messages

Right now, if any PowerShell error was to occur we are using `console.error()` to log a message.
In a production app, we will want to emit messages on the GUI.
To do this we will use the [bootstrap-alert] component for our warning messages.

Add the following in `index.html` between our **Form** and **Output** components.

```html
<!-- Error -->
<div class="row justify-content-center top-buffer">
    <div class="alert alert-danger" role="alert" style="display: visible">
        <strong>Whoops!</strong>
        <div class="message">Flux capacitor is not Fluxing.</div>
    </div>
</div>
```

Give it a quick refresh.

![FluxNotFluxing](/images/posts/powershell-electron-demo/whoops-flux-capacitor-not-fluxing.png)
*The message will appear below our form*

Now, lets leave the message blank and set `visible` in `style="display: visible"` to `none` to hide the component.

To show the alert whenever an error occurs, add some *jQuery* to our `.catch()` function in `renderer.js`.

- Set the `div.message` inner HTML to the error
- Show the alert

```javascript
.catch(err => {
    console.error(err)
    $('.alert-danger .message').html(err)
    $('.alert-danger').show()
    ps.dispose()
})
```

Now, let's give it a refresh and try to connect to a PC that doesn't exist.

![ErrorBoxTardis](/images/posts/powershell-electron-demo/error-box-tardis.png)
*Chameleon circuit appears to still function somewhat*

Now, if you query a good computer after the error is shown, the error will not go away when the table is displayed.
We will want to **clear** and **hide** the alert each time the form is submitted.

At the top of our `click()` event in `renderer.js` we will add:

```javascript
// Clear the Error Messages
$('.alert-danger .message').html()
$('.alert-danger').hide()
```

### Improving Errors

From the `TARDIS` example above, node-powershell just takes the PowerShell error and all its messy glory and hands it off to JavaScript as a string.
Unfortunately, It's not as friendly as the `$_` hashtable you see in a PowerShell catch block.
Once you have the error in JavaScript you can't access the properties individually.

For the time being, I filed an issue: [node-powershell-issue-22].

#### Simple Workaround

As a workaround, we can update `Get-Drives.ps1` to form a custom error block.

1. `Invoke-Command` can fail in the script block or at the remoting call if it cannot connect or authenticate.
2. To make sure we actually catch both cases of this we specify `ErrorAction = "Stop"`
3. We form our own error object `$myError` to be converted to JSON.
    - Here I've added `Type`, which is nice if you throw your own custom errors as seen in [#22][node-powershell-issue-22]

```powershell
param (
    [Parameter(Mandatory = $false)]
    [string] $ComputerName = 'localhost'
)

$parms = @{
    ComputerName = $ComputerName
    ErrorAction = "Stop"
}

try {
    $out = Invoke-Command @parms {
        [System.IO.DriveInfo]::GetDrives() |
        Where-Object {$_.TotalSize} |
        Select-Object   @{Name='Name';     Expr={$_.Name}},
                        @{Name='Label';    Expr={$_.VolumeLabel}},
                        @{Name='Size(GB)'; Expr={[int32]($_.TotalSize / 1GB)}},
                        @{Name='Free(GB)'; Expr={[int32]($_.AvailableFreeSpace / 1GB)}},
                        @{Name='Free(%)';  Expr={[math]::Round($_.AvailableFreeSpace / $_.TotalSize,2)*100}},
                        @{Name='Format';   Expr={$_.DriveFormat}},
                        @{Name='Type';     Expr={[string]$_.DriveType}}

    } | Select-Object * -ExcludeProperty PSComputerName, RunspaceId, PSShowComputerName
} catch [System.Management.Automation.RuntimeException] {
    $myError = @{
        Message = $_.Exception.Message
        Type = $_.FullyQualifiedErrorID
    }
    $out = @{ Error = $myError }
}

ConvertTo-Json $out -Compress
```

Next, go to `renderer.js` in the `.then()` block and add our custom error code.
If the response object contains Error we can now display an error and exit the `then()` statement early.

```javascript
// Catch Custom Errors
if (data.Error) {
    $('.alert-danger .message').html(data.Error.Message)
    $('.alert-danger').show()
    return
}
```

Now we have full control over our cross-language errors.

![CustomErrorFrontend](/images/posts/powershell-electron-demo/custom-error-front-end.png)
*PC Not Found*

![AccessDeniedEnterprise](/images/posts/powershell-electron-demo/access-denied-enterprise.png)
*Captain's Log, Stardate 94881.74. Access was Denied*

## Authentication
If you noticed the second error message above, we have a computer which we do not have access use with remoting.

- `Invoke-Command` uses the security context for the current user running the electron app.

How can we change this context? Let's take a look at a few of our options.

1. `RunAS`: the electron *exe* can be run as a different user
2. `ADHOC`: we can inline `Get-Credential` into our PowerShell Script
3. `ADHOC/Saved`: we can save the credential either in memory, a secure file, or use a module like [BetterCredentials].

### ADHOC

Test by editing our hashtable in `Get-Drives.ps1`

```powershell
$parms = @{
    ComputerName = $ComputerName
    ErrorAction = "Stop"
    Credential = Get-Credential
}
```

Refresh the app and click `Get Disk Info!`:

![adhocGetCredential](/images/posts/powershell-electron-demo/adhoc-get-credential.png)
*Each time you click on `Get Disk Info!` the script will prompt for credentials*

### ADHOC/Saved

For saved credentials, we will explore 2 new concepts: Electron global variables, and passing JSON to PowerShell.

- Please check with your local security expert before you start saving Domain Admin credentials.
- Only you can prevent ~~forest fires~~ security breaches.

#### Passing Data

Create a new file in the project root called `Convert-CredToJson.ps1`:

```powershell
param (
    [Parameter(Mandatory = $false)]
    [PSCredential] $Cred = (Get-Credential)
)

@{
    user = $Cred.UserName
    pass = $Cred.Password | ConvertFrom-SecureString
} | ConvertTo-Json -Compress
```

In `index.html` add an extra button to our form where the user can specify a different account to use for the `Invoke-Command` call:

```html
    <!-- Form -->
      <div class="row top-buffer">
        <div class="col-lg-6 mx-auto">
        <div class="input-group">
          <input id="computerName" type="text" class="form-control" placeholder="Computer Name">
          <span class="input-group-btn">
            <button id="getDisk" class="btn btn-primary" type="button">Get Disk Info!</button>
          </span>
          <span class="input-group-btn">
            <button id="changeUser" class="btn btn-warning" type="button">Change User</button>
          </span>
        </div>
        </div>
      </div>
```

In `main.js` under *path* and *url*, declare a global variable for `cred`:

```javascript
const path = require('path')
const url = require('url')

global.sharedObj = {
  cred: null
};
```

In `renderer.js`:

1. Require in `remote` (it's how we are going to get/set variables in main.js)
2. Add a new `click()` event for our `#changeUser` `<button>`
3. Call `Convert-CredToJson.ps1` to generate our cred
4. Set the global Variable from the response

```javascript
// Get Global Variables
let remote = require('electron').remote;

$('#changeUser').click(() => {
    let ps = new powershell({
        executionPolicy: 'Bypass',
        noProfile: true
    })

    ps.addCommand('./Convert-CredToJson.ps1', [])
    ps.invoke()
    .then(output => {
        console.log(output)
        // Set the global Variable
        remote.getGlobal('sharedObj').cred = JSON.parse(output)
        // Read the global variable
        console.log(remote.getGlobal('sharedObj').cred)
    })
    .catch(err => {
        console.dir(err);
        ps.dispose();
    })
})
```

Refresh the app and click `Change User`:

![customCredential](/images/posts/powershell-electron-demo/login-prompt-custom.png)
*Get Custom Credential*

Here is the response JSON which gets saved to our global electron variable `cred`.

![customCredentialJSObject](/images/posts/powershell-electron-demo/cred-as-js-obj.png)
*Credential as JS Object*

### Using the Credential

Now that we have a global credential, we want to pass it to our `Get-Drives.ps1` function (if it is not null).

First, we add the input param `JsonUser` to `Get-Drives.ps1`.

- I used `JsonUser` since the word `Cred` causes *PSScriptAnalzer* to complain (I'm too lazy to add a suppress rule).

```powershell
[Parameter(Mandatory = $false)]
[String] $JsonUser
```

Next, after our `$parms` hashtable we inject our credential into the commands we intend to `splat`.

If `slatting` is new to you, be sure to go read up on it. For starters, take a look at [PSCookieMonster's Splat Guide].

This example also uses the `::new()` class method instead of `New-Object`.
I'm using PowerShell v5, moving forward, and Boe Prox showed [it was faster][BoeProx].

```powershell
$parms = @{
    ComputerName = $ComputerName
    ErrorAction = "Stop"
}

# Using IsNullOrEmpty instead of advanced function validators in case JS passes a blank/null
# Get-Drives -Computer $PC -JsonUser ''
if ( ! [string]::IsNullOrEmpty($JsonUser) ) {
    $hash = $JsonUser | ConvertFrom-Json
    $hash.pass = $hash.pass | ConvertTo-SecureString
    $parms.Credential = [PSCredential]::new($hash.user, $hash.pass)
}
```

Now, when we call `Get-Drives.ps1` from JavaScript, we will want to serialize and pass along our global cred if it exists.

In `renderer.js` we can update the following:

```javascript
// Load the gun
ps.addCommand("./Get-Drives", [
    { ComputerName: computer }
])
```

To:

```javascript
let commands = [{ ComputerName: computer }]
let cred = remote.getGlobal('sharedObj').cred

// If global cred exists, seralize and push it to commands
if (cred)
    commands.push({ JsonUser: JSON.stringify(cred) })

// Load the gun
ps.addCommand('./Get-Drives', commands)
```

If we refresh the app and try this out, we will get an error. If you look at the command that is being passed, you will see that the JSON string needs to be wrapped with quotes.

![JsonToPsError](/images/posts/powershell-electron-demo/json-to-powershell-error.png)
*Parmameter string is not wrapped when node-powershell calls script*

This could probably be handled a little better in `node-powershell`. I've added a ticket which helps illustrate the problem ([node-powershell-issue-21]).

#### Workaround: String Wrap

To avoid needing to manually wrap or escape every string we pass like:

```javascript
"'" + var + "'"
```

We will add a *prototype* to the native string construct.

In the global scope at the top of the `renderer.js` add:

```javascript
// Helper to wrap a string in quotes
String.prototype.wrap = function () {
    return `'${this}'`;
}
```

Now we can just use `.wrap()` when needed.

```javascript
let commands = [{ ComputerName: computer.wrap() }]
let cred = remote.getGlobal('sharedObj').cred

// If global cred exists, seralize and push it to commands
if (cred)
    commands.push({ JsonUser: JSON.stringify(cred).wrap() })

// Load the gun
ps.addCommand('./Get-Drives', commands)
```

![messageDialog](/images/posts/powershell-electron-demo/hover.png)
*Prototypes make for a smoother ride*

## Customize Toolbar

We are going to customize the menu bar a bit. Check out the [official docs][electron-menu] for a full breakdown.

In `main.js` let's update our Electron imports using the [Object Destructuring] syntax.
I've also included [dialog][Electron-dialog] component to demonstrate a click action in our custom menu.

```javascript
// const app = electron.app
const {app, Menu, dialog} = electron
```

In `main.js` we will make our own function to keep things organized:

```javascript
function createMenu() {
  const template = [
    {
        label: 'View',
        submenu: [
          {
            role: 'reload'
          },
          {
            role: 'forcereload'
          },
          {
            role: 'toggledevtools'
          }
        ]
    },
    {
      label: 'Tools',
      submenu: [
        {
          label: 'Check Cred',
            click () {
                let user = (global.sharedObj.cred) ? global.sharedObj.cred.user : "Default"
                dialog.showMessageBox({
                    type: "info",
                    title: "Current Cred",
                    message: `The current user is: ${user}.`
                })
            }
        }
      ]
    }
  ]

  const menu = Menu.buildFromTemplate(template)
  Menu.setApplicationMenu(menu)
}
```

Add `createMenu()` to the `createWindow()` function in `main.js`.

```javascript
function createWindow () {
  createMenu()
  //...
```

To see the changes take effect, close your app and restart it with `npm start`.

![customMenu](/images/posts/powershell-electron-demo/custom-menu.png)
*Custom Menu example*

**Note:** I left the DevTool options and bindings in the custom menu. Feel free to remove them.

![messageDialog](/images/posts/powershell-electron-demo/message-dialog.png)
*`Check Cred` click event opens a dialog*

## Cleanup and Packaging

### Paths

A lot of node projects recommend using absolute paths, be it for OS compatibility or whatnot, don't get me lying lol.
Keep in mind, any of of `.ps1` script references could be updated with something like:

```javascript
let scriptPath = require("path").resolve(__dirname, './Get-Drives.ps1')
```

### Package.json

Since we used the electron-quick-start we are still using some of their default config in our package.json.
Be sure to visit this file and update it.

### Packaging

Once you have completed your app and start searching for packaging instructions you will land on these official pages:

- [Electron-packaging]
- [Electron-distribution]

My eyes glazed over when I read the documented process. Luckily Electron mentions two projects to help automate this:

- [Electron-builder]
- [Electron-packager]

I'm just using `electron-packager` here since It had more stars... (that's how instantaneous decision making works).

Install it globally or as a dev-dependency:

```console
# for use in npm scripts
npm install electron-packager --save-dev

# for use from cli
npm install electron-packager -g
```

If you installed it globally, check out the help docs with:

```console
electron-packager --help
```

While in the project root, `electron-packager` will create a package intended for your systems **OS** and **Architecture** if you provide no arguments.
Running the following command will also name your `exe` using the `project name` in `package.json`

```console
electron-packager .
```

#### Adding a Custom Icon

Now we can add an icon to the project root. In this example I found a quick NodeJS looking [disk-icon].

Since this app is intended for Windows I went ahead and supplied a few more options.

```console
electron-packager . --platform=win32 --arch=x64 --icon=icon.ico --overwrite
```

Running this command from the project root generates the following files:

![BuildWin](/images/posts/powershell-electron-demo/electron-packager-build-win.png)
*Electron-Packager Build for Windows*

To make it easier to exclude builds in source version, we want to change the command to output to a `dist` folder.

Add the command as a simple npm task in `package.json`. Then run in from the project root using `npm build`.

```json
"scripts": {
    "start": "electron .",
    "build": "electron-packager . --platform=win32 --arch=x64 --icon=icon.ico --out=dist --overwrite"
}
```

Add `dist` to `.gitignore`

```text
node_modules
dist
```

#### ASAR

If you inspect the dist folder, you will find that your source files are all easily availiable in plain text.

- Supplying the `-asar` flag to `electron-packager` will throw them all in an `.asar` file instead.
- The `.asar` file can still be opened with a text editor and seen in plain text.
- The `.asar` file may cause path issues with loading the `.ps1` files with `node-powershell`.
    - I haven't used `.asar` much. Feel free to send a PR on proper path workarounds.

## Conclusion

Hopefully, this quick demo helps others get starting with Electron and Powershell.
This started as a quick proof-of-concept I made after talking to few guys about using PowerShell with NodeJS.
With PowerShell steadily becoming a more solid cross platform product, the pairing with Electron could potentially 
make some very powerful tools that could be distributed on any OS. If you have any thoughts or questions let me know and
as always, feel free to file and issue or PR if you find any errors.


![messageDialog](/images/posts/powershell-electron-demo/outatime.jpg)
*Outatime...Thanks, for reading*

## References

1. [Xainey/powershell-electron-demo]
2. [Stephen Owen's GUI ToolMaking Series]
3. [Git]
4. [VSCode]
5. [NodeJS]
6. [Yarn]
7. [electron-quick-start]
8. [git-clone-init]
9. [Bootstrap v4]
10. [bootstrap-packages]
11. [bootstrap-alert]
12. [jQuery]
13. [rannn505/node-powershell]
14. [IonicaBizau/powershell]
15. [Promise]
16. [node-powershell-api]
17. [DataTables]
18. [DataTables-Install]
19. [DataTables-Bootstrap4]
20. [DataTables-JS-Array]
21. [node-powershell-issue-21]
22. [node-powershell-issue-22]
23. [BetterCredentials]
24. [@ferventcoder]
25. [electron-menu]
26. [object destructuring]
27. [Electron-dialog]
28. [Electron-packaging]
29. [Electron-distribution]
30. [electron-builder]
31. [electron-packager]
32. [disk-icon]
33. [PSCookieMonster's Splat Guide]
34. [BoeProx]
35. [storing-powershell-credentials-in-json]
36. [Hodge-PowerShell-Jenkins]
37. [Developer-Roadmap]
38. [Atom Electron]

[Xainey/powershell-electron-demo]: [https://github.com/Xainey/powershell-electron-demo]
[Stephen Owen's GUI ToolMaking Series]: https://foxdeploy.com/2015/04/16/part-ii-deploying-powershell-guis-in-minutes-using-visual-studio/
[Git]:    https://git-scm.com/downloads
[VSCode]: https://code.visualstudio.com/Download
[NodeJS]: https://nodejs.org/en/download/
[Yarn]:   https://yarnpkg.com/en/docs/install
[electron-quick-start]: https://github.com/electron/electron-quick-start
[git-clone-init]: https://www.npmjs.com/package/git-clone-init
[Bootstrap v4]: https://v4-alpha.getbootstrap.com/
[bootstrap-packages]: https://v4-alpha.getbootstrap.com/getting-started/download/#package-managers
[bootstrap-alert]: https://v4-alpha.getbootstrap.com/components/alerts/#examples
[jQuery]: https://jquery.com/
[rannn505/node-powershell]: https://github.com/rannn505/node-powershell
[IonicaBizau/powershell]: https://github.com/IonicaBizau/powershell
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[node-powershell-api]: http://cdn.rawgit.com/rannn505/node-powershell/236b6c3a/docs/docs.html
[DataTables]: https://datatables.net/
[DataTables-Install]: https://datatables.net/manual/installation
[DataTables-Bootstrap4]: https://datatables.net/examples/styling/bootstrap4.html
[DataTables-JS-Array]: https://datatables.net/examples/data_sources/js_array.html
[node-powershell-issue-21]: https://github.com/rannn505/node-powershell/issues/21
[node-powershell-issue-22]: https://github.com/rannn505/node-powershell/issues/22
[BetterCredentials]: https://github.com/Jaykul/BetterCredentials
[@ferventcoder]: https://twitter.com/ferventcoder
[electron-menu]: https://electron.atom.io/docs/api/menu/
[object destructuring]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Object_destructuring
[Electron-dialog]: https://github.com/electron/electron/blob/master/docs/api/dialog.md
[Electron-packaging]: https://electron.atom.io/docs/tutorial/application-packaging/
[Electron-distribution]: https://electron.atom.io/docs/tutorial/application-distribution/
[electron-builder]: https://github.com/electron-userland/electron-builder
[electron-packager]: https://github.com/electron-userland/electron-packager
[disk-icon]: http://www.iconarchive.com/show/polygon-icons-by-graphicloads/disk-2-icon.html
[PSCookieMonster's Splat Guide]: https://ramblingcookiemonster.wordpress.com/2014/12/01/powershell-splatting-build-parameters-dynamically/
[BoeProx]: https://learn-powershell.net/2014/09/07/more-new-stuff-in-powershell-v5-a-new-way-to-construct-things/
[storing-powershell-credentials-in-json]: http://jdhitsolutions.com/blog/powershell/5396/storing-powershell-credentials-in-json
[Hodge-PowerShell-Jenkins]: https://hodgkins.io/automating-with-jenkins-and-powershell-on-windows-part-1
[Developer-Roadmap]: https://github.com/kamranahmedse/developer-roadmap
[Atom Electron]: https://electron.atom.io/

*[npm]: node package manager
*[CDN]: Content Delivery Network
*[CSS]: Cascading Style Sheet
*[JSON]: JavaScript Object Notation
*[CLI]: Command Line Interface
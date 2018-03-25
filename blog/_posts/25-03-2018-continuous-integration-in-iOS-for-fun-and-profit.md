# Continuous integration in iOS for fun and profit

![](https://cdn-images-1.medium.com/max/1600/1*LlIIJM9rFniQuu3Yqpzs1g.jpeg)

> They say a picture says more than a thousand words.

Looking at the picture above, you see an illustration of my preferred road to
setup continuous integration. In this article, I am going to be explaining how
to properly setup CI process for your iOS apps, also we will see how to debug
common issues that you might face.

Important to know is that you should install Ruby Version Manager
([RVM](https://rvm.io/)) to make sure that you have consistent ruby version
installed on your system and are able to manage multiple installations of Ruby
on the same device. Once that is done I will talk about how to install Fastlane:
A tool that help us automatinging development and release process for iOS and
Android apps. Fastlane handles all tedious tasks like dealing with code signing,
beta deployment, App Store deployment, automate screenshots…etc<br> Once we are
done with that, we will take a look into setting up Jenkins, which will be the
continuous delivery hub, allowing us to create different jobs for beta and App
Store deployment.

#### So why RVM?

[Fastlane](https://fastlane.tools/) is written in Ruby, which means installing
it could be as easy as running this command:

    sudo gem install fastlane

Using this command to install Fastlane will result in using Apple’s system
version of Ruby, and having it placed in the`/usr/bin` directory. Let me explain
why that’s not a good thing.

OS X comes with various open source packages pre-installed. The packages are
only updated to their latest versions as part of major OS X Updates and some
packages are used elsewhere by other parts of OS X. In general, Apple assumes
(and so should you) that everything under`/System/Library` and `/usr/` (except
`/usr/local/` of course) is part of the operating system, and therefore
administered by Apple.<br> This is why any attempt to remove or modify files in
these directories will result in problems along the way. That applies to all
open sources packages that come with OS X, including Ruby.

The right approach is having new versions installed in a seperate location,
using the RVM.

Note that this will not only help you with having proper setup for Fastlane, but
also for any other ruby gems you install.

### 1. RVM 💎

To check where your Ruby installation is located, run the below command:

    which ruby

If you are using the pre-installed Ruby version, it might respond with the below
line:


### INSTALLLATION

Now I will mention simple steps to install RVM but you can always check the
[installation page of RVM.](https://rvm.io/rvm/install)

1.  Install [gpg](http://en.wikipedia.org/wiki/GNU_Privacy_Guard) using Homebrew:


2. Install the **mpapis** public key:


3. Install RVM with the latest stable version of Ruby:


After the installation is completed, close the terminal window and open a new
one.

Then run this command:


You will see the list of available rubies:

    rvm rubies

    =* ruby-2.4.1 [ x86_64 ]

    # => — current
    # =* — current && default
    # * — default

Now it’s time to start using the RVM’s Ruby by running the command:

    rvm use ruby-2.4.1

You can verify that you are using RVM’s Ruby by running `which ruby`. If all is
set up correct, you will see a similar output as below:

    /Users/mkalatrash/.rvm/rubies/ruby-2.4.1/bin/ruby

That’s it for the first step 🎉.

*****

### 2. Fastlane 🚀

Next up is installing Fastlane and integrating it in the project.

Run the below command, to install fastlane tools and the necessary dependencies:

    gem install fastlane

Once it’s done make sure that fastlane is installed in **rvm/gems** directory by
running:

    gem which fastlane

The output of this command should be similar to:

    /Users/mkalatrash/.rvm/gems/ruby-2.4.1/gems/fastlane-2.81.0/fastlane/lib/fastlane.rb

**Integrating Fastlane into your project 🚀**

*For our example I created new project named ***iOSAutomationTutorial.**

Navigate to your project’s root folder and run:

     

Once you have run the command, a new folder will be created in the project
directory named fastlane. Now Fastlane will continue with the setup through
couple of possible steps. In our case we will choose the **4th option** which is
**Manual setup**.

Continue by pressing Enter until Fastlane is done.

Now we will add new **lane** to our fastfile in order have it build the iOS
project for us, but before that we will need to make sure the project scheme is
**shared**:

* Open the project.
* Select scheme followed by manage scheme.
* Make sure “**Shared**” is checked.

<span class="figcaption_hack">Manage Schemes</span>

<span class="figcaption_hack">Make scheme shared</span>

Let’s proceed. Open fastfile located in`./iOSAutomationTutorial/fastlane`(I
prefer to use Sublime Text) and add the below:

    default_platform(:ios)

    platform :ios do

      desc "Build iOS App"

        lane :build do

          xcodebuild(build_settings: {

              "CODE_SIGNING_REQUIRED" => "NO",

              "CODE_SIGN_IDENTITY" => ""

              })

         end

    end

Save and run this command to make sure your setup is working fine:

    bundle exec fastlane build

*Please note that I used ***bundle*** to run fastlane which is recomended. In
case you don’t have *[bundler](https://bundler.io/)*, then you need to install
it first by running *`gem install bundler`*.*

If everything is fine you should see a success message from Fastlane.

More information on this you can find [Fastlane
docs](https://docs.fastlane.tools/).

*****

### 3. Jenkins 🚘

You have reached the final step in the setup. Now all that’s left is installing
Jenkins on your server machine.

The recommended way to install [Jenkins](http://jenkins-ci.org/) is through
[homebrew](http://brew.sh/):

    brew update && brew install jenkins

*If you have a fresh version of Mac OS then download the latest version of the
JDK from
*[here](http://www.oracle.com/technetwork/java/javase/downloads/index.html)* or
get it by running this command :*


From now on start `Jenkins` by running:

    jenkins

Or you could run it as a background service, using the below command:


Now we will have to go through Jenkins initial setup. An admin user has been
created and a password is automatically generated (password will be prinited in
your terminal).

Go to [http://localhost:8080](http://localhost:8080/), which is the local
address of your Jenkins.

<span class="figcaption_hack">Unlock Jenkins</span>

Provide your administrator password and press continue.

Select **Install suggested plugins**, which will cover all the important
plugins.

<span class="figcaption_hack">Installing plugins</span>

After installing all plugins, Jenkins will ask you to provide credentials for a
new admin user. Just select **Continue as admin** for now.

Your Jenkins setup is done and ready for it’s first job! 🎉

*In order to change the initial password or create a new user go to ***admin >
Configure,*** ***Jenkins > Manage Jenkins > Manager Users***.*

Before creating your first job we will need an extra plugin to get installed. Go
to **Manage jenkins > Manage Plugins > Available**.

Now search and install the RVM plugin, which will help specifiying which ruby
version to use. This way we won’t run into situations where Jenkins can’t find
fastlane or other ruby gems we use, as I described before.

After you install RVM plugin, restart Jenkins and then on the homepage click on
**Create new jobs**.

<span class="figcaption_hack">Enter job name and select Freestyle project</span>

We will now create a job to build the iOS project.<br> Proceed by entering the
job name and selecting freestyle project, followed by clicking OK (see snapshot
above).

Now we clone the project from the public repo using git plugin that comes with
Jenkins:

<span class="figcaption_hack">Source code management</span>

At this point, make sure you have **Xcode** and **command line developer tools**
installed.

Secondly, check the RVM plugin and mention Ruby version you will use (While
writing this it’s 2.4.1).

Third and last is to add an “Execute shell” step as your first build step and
call add:

    bundle update

    bundle exec fastlane build

<span class="figcaption_hack">Specifying Ruby version and adding Execute shell</span>

Click **Save** and on **Build Now**. Your job will start executing and the start
building. When the build finishes you will see a success message like the one
below:

<span class="figcaption_hack">Build success</span>

*****

### Where to go from here

This is just the beginning of the automation journey. It’s a walkthrough to
setup the required tools your on CI machine to get started.

There is still a lot to learn and to improve depending on how you have defined
your process:

* Fetching your project form private repo.
* Signing project and download latest certificates.
* Auto increment build number and version number.
* Setup Dev/Live configurations.
* Download latest localizations from your server or a third party.
* Build the project.
* Upload build to S3.
* Upload build to Crashlytics and iTunes connect.
* Notify people in slack.
* ….And much more

*****

Any comments or feedback is appreciated! 
Share this articke if you enjoyed this article and found it useful. This will encourage me to
write more!

Feel free to add me on [github](https://github.com/mohammad19991),
[twitter](https://twitter.com/mkalatrash),
[linkedin](https://www.linkedin.com/in/mohmmadalatrash/),
[medium](https://medium.com/@matrash)

#### Thank you for reading!





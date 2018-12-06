# CIS 4076/6076 - Technical Skills Writeup

## Skill Descriptions

(Note: This is for a class, disregard this.)

* AWS
	* AWS, or Amazon Web Services, is the IaaS service provided by Amazon that lets companies host content in the cloud with readily available access all over the world. The goal of the AWS Cloud Practitioner certification is to certify that an administrator is proficient in both configuration and deployment of AWS products and services according to the environments in which they are suited.
* GitHub
	* As GitHub has become the most popular source on the planet to host publicly available code and content, it makes a lot of sense to be learning the basics of GitHub. Using the GitHub Labs, one will become proficient in a basic knowledge of GitHub, including Jekyll CI, GitHub Pages, and Markdown.
* Powershell
	* Powershell is a task automation and configuration management framework that is both a command line shell and scripting language available initially for Windows, then as an open-source cross-platform component. Powershell's ubiquity and ease of learning and use in large deployment environments make it an ideal system to learn. Powershell takes a lot after Linux in that it has a very extensive help system not unlike manpages, making learning this powerful framework easy.
* PowerBI
	* PowerBI is a business analytics service created by Microsoft as a sort of Tableau for Microsoft products and services. edX's PowerBI training will allow one to become proficient with basic data-warehousing capabilities as well as data prep, data discovery, and customizable dashboards for interaction with large amounts of data in an easy-to-view format.


## Github

So, Github. Officially the biggest codebase on the planet. Operating as a web based version control for Git, it provides very popular features such as bug tracking, feature requesting, task management, and wikis. The software, Git, is originally a brain child of Linus Torvalds, and can interact with the site's Ruby on Rails software to allow easy access to repositories and access control features.

The amount of features available to the average user, even without premium persona accounts, provides a very easy-to-use and quick-learning-curve are heavily tutorial-based via Github Labs. I was able to complete most of the labs in about 6 hours, although there was one lab where the bot looking for Github Pages was not working properly. The only way I could get around this was to simply ignore it, as the remaining steps were easily replicated without the bot watching them.

I personally greatly enjoyed this tutorial, as it was very good training. Well formatted, not very intrusive or annoying, and I ended up learning a couple things from it, like that Jekyll is actually Ruby-based, so my site looks really cool since I know some offhand Ruby stuff and how to stage some Jekyll sites. I would highly recommend this training to ANY Business student, because of the popularity of Github and the ease of running Github-like services such as Gogs or GitLab, and because this would be a very good way for people to track their own changes. Perhaps a class that supplements Business's Programming 1 or something would be a good addition, but I still highly recommend that the Github training is included.

## Amazon AWS

Amazon AWS is a cloud service provider that provides on-demand cloud computing to anyone on a paid subscription basis. This allows a user (be it personal or enterprise) to have a virtual computing cluster available 24/7 through the internet, as well as the emulation of a typical computer including hardware, choice of OS, networking, and application software such as web servers, databases, CRM, anything the user may require.

The tech is implemented for mostly larger-scale enterprises, as the fees are designed to be, instead of a flat-rate, dynamic with a combination of usage, the running state such as OS and software, availability, security, redundancy required, and any service options available with it. Spanning a global presence, this incredibly popular service system has encompassed more than 90 services and is used by nearly 34% of all the cloud services.

After having gone through the training, I have some complaints.

* The training content doesn't feel like training content. It feels like a 6-9 hour sales pitch and/or super-basic user training.
* The SCORM content is...powerful boring. I had to go through a lot of SCORM content at Targa Resouces for user training, and that was pretty boring.
* The content certainly does not provide the basic level of skill that would even give one a slight familiarity with AWS. Certainly not taking this training and saying you could go in and manage a company's x-amount of services.

I cannot in good consciousness (yes, consciousness) recommend this to future class-takers, as this is content I don't feel is as particulary useful as Github. Something that I think would be more appropriate is something with a greater return, such as maybe even expansion on the Git software itself. I'm a major proponent of "learn the command line first".

![AWS Proof](/img/class/AWS.png)

## Microsoft PowerBI

Microsoft PowerBI is business analytics software that is provided by Microsoft. Providing visualizations with business intelligence not all that dissimilar to Tableau, PowerBI allows large amounts of raw data can be prettified without having a significant IT or database administration requirement.

Basically a unified version of, like, all of Microsoft Office, PowerBI allows cloud based services as PowerBI Services and a desktop based interface, avaliable for all platforms and via the UWA Windows Store, PowerBI Desktop. The application is designed to offer data warehousing services such as discovery, preparation, and dashboarding. Recently, Microsoft offered an additional service called PowerBI Embedded on Azure, which allows for a greater user audience, as well as being able to create custom visualizations for certain complex kinds of data.

This teaching was provided by edX, and I have to say, it wasn't bad, but it wasn't good either. Some of the training content doesn't seem to match up with the results that one may get with the PowerBI examples (on top of me having to inexplicably drag out an old Windows tablet because my newer stuff had trouble running it...), although that may have been my frustration with some of the results. I'm way more used to using Tableau so this was a bit of a fish out of water experiences.

While I think this training was good, I think that it would be more apt to teach Tableau based software, because Tableau's web services provide much more detailed and different kinds of data to work with. Learning in conjunction would be optimal, but this class is not a data analytics and presentation class. Otherwise I would recommend both, as data visualization will be a major player in the MIS (It's not CIS, pbbbbt) community, especially when it comes to data visualization, as not everyone will understand what they're looking at when you select two tables and have them projected using the MySQL command-line client or something.

![PowerBI Proof](/img/class/edx.png)

## Microsoft Powershell

Powershell is a task and configuration management system from Microsoft, gone full open source and cross platform, built on .NET Framework and .NET Core. Powershell provides a fantastic interface for administrative tasks via commandlets (cmdlets as they say), which enable particlar operations on local or remote machines, allowing operations on differing data stores, with nearly everything in the OS being treated as a datastore.

Powershell has a huge influence from popular systems like Bash, Manpages, and some of Microsoft's older software such as Home Server. The software is intensely powerful and can be used to easily automate tasks on just about any software platform. It's effectively the UNIX tools made API'd into Windows.

What makes Powershell even greater is the powerful pipelining and scripting capabilities, not unlike that of Bash on most UNIX/Linux based systems. Users can easily create functions and capabilities that can be integrated into Powershell, and Powershell provides a great security framework for preventing operation of possibly undesired or unwanted scripts.

The coolest feature is probably Desired State Configuration, which ensures a system gets exactly the state described in the software configuration. This is EXTREMELY useful in an AD environment, and I got to play with it some at Targa Resources. It makes deployment or re-deployment a breeze because the clients are able to easily reconfigure themselves at a convenient opportunity.

Our training was provided via the Microsoft Virtual Academy, but honestly, I can't say I would recommend it to a beginner user, as the content goes from basic, beginner-level stuff, to hardcore Desired State Configuration training, which is a leap and bound for those that don't have at least a little experience with something like SCCM or SCOM, regular deployment, or a little bit of Hyper-Suck (excuse me, I think it's called Hyper-V here).

The content is also quite dated, as I believe some of the features being used are reminiscent of Powershell 3.0, with Powershell currently in its 6th iteration. Either some fresh online content, or a good PDF eBook would be better, as Microsoft's content just jumps right into the DSC without any warning or explanation, at least I feel.

[PowerShell Certificate](/docs/Certificate.pdf)
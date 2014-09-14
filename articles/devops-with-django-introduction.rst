How-to: Start a Django project and integrate it into a scalable Dev/Ops workflow
================================================================================
This is a start of a series on applying developer/operations techniques to a Django project. In this series we will cover the following:

  - Getting started with Django
  - Defining Dev/Ops and establishing a process
  - Setting up the host
  - Simple, automated deployments with git
  - Basic Continuous Integration w/Jenkins
  - Enterprise Deployment Patterns
  - Operationalizing Django
  - Advanced Continuous Integration and Continuous Delivery
  - Configuration Management and Orchestration

There is lots of information available on Dev/Ops. It means many things to many practitioners but to me it's simply automating various aspects of the software development life-cycle. Practically, this means automated testing and deployment. Another important aspect of devops is telemetry, which we cover much later in the series.

In **part 1, Getting Started with Django**, we will dive into Django and setup a real content management system. We will introduce various tools in the PyDev ecosystem and setup the developer environment. Then, add functionality, demonstrating database migrations -- a very attractive and powerful Django abstraction, well suited for dev/ops. In tandem we will build out the html templates and modify the look and feel of the site. Finally, we will discuss implementation caveats which will arise when we move the app to production.

In **part 2, Defining Dev/Ops and Establishing a Process** we step back from coding and define Dev/Ops, loosely, for the purposes of establishing a clean plan. We will talk about what Dev/Ops is and how to apply such practices to Django in a way that not only makes sense but, more importantly, makes our lives easier.

In **part 3** of the series, entitled **Setting up the Host** we will wear the hat of the system admin. We will setup a VPS using CentOS because Dev/Ops is most valuable in an Enterprise setting and CentOS is an Enterprise Linux. We will introduce the review the top choices for serving our Django app. Then we will move forward with performing the actual implementation. By the end of the article, we will have our new CMS deployed to the cloud and accessible via a domain name.

In **Part 4, Simple Automated Deployments with Git** we will get back to the practical application of Dev/Ops practices as they pertain to Django. We will setup a simple method for pushing out updates which we can build upon when the time comes to scale the workflow so support scaling the team. Often times, adding developers to a project is a major hassle. Not only because they have to ramp up the codebase and subject matter, but also because often times the developer workflows are a mess. By the end of this article we will have implemented deployment automation practices in accordance with the processes previously defined.

**Part 5, Basic Continuous Integration w/Jenkins** we begin to take the automation to the next level. We give a short introduction to Jenkins and Continuous Integration, then move on to setup jobs which allow us to perform basic automated testing each time a developer commits a changeset. We will dig deeper into testing Django Python and demonstrate how minimal adaptation to our workflow can help to yield higher quality code, higher developer productivity, less maintenance hassles, and offer project managers more insight into the development process.

In **part 6, Enterprise Deployment Patterns** various ways software is deployed in big, high-volume environments where small amounts of downtime can mean a substantial loss of revenue. In environments such as this, often the best tool is chosen for the job, regardless of the cost or sophistication. In terms of deployments often times this means leveraging the native tools of the target infrastructure to manage the installation and updates of software. In simple terms, this means rolling native OS packages for your deploy target and applying shell scripts as appropriate to massage the upgrade process when necessary. In this article we will discuss mainly RPM packaging as Enterprise Linux is best suited for these types of setups. As a final note, compliance adherence practices, such as maintaining a software inventory, are often best met when leveraging native software installation management tools.

**Part 7, Operationalizing Django** is all about moving your app out of it's development environment and into production. Often times this process is a hassle, compounded by multiple issues. In this section, we will cover all the essentials for moving a Django app to a live, mission-critical environment. We will cover creating RPM packages, discuss suitable environment settings, and review best practices in regards to system administration. We will define troubleshooting processes and remediation procedures. 

In **part 8, Advanced Continuous Integration and Continuous Delivery** we automate our "Operationalization" process using shell scripts and jobs configured in Jenkins. We will discuss topics such as Automated Acceptance Testing and demonstrate how to design a robust test harness which will afford peace of mind and provide assertions that the required functionality has been implemented correctly. Another important concept, "Continuous Delivery" is introduced. We will show how to graduate software updates from pre-release stages to production release using native OS tools and triggers in Jenkins.

**Part 9, Configuration Management and Orchestration** introduces another critical Enterprise need: Configuration Management. This concept is another one of those critical to regulatory compliance, especially FedRamp certification. In this article we will introduce the top player in this market, review their offerings and evaluate our top two choices. After demonstrating the evaluation, we will move forward with our top choice and begin to establish an orchestration framework capable of managing vast infrastructure, reliably.

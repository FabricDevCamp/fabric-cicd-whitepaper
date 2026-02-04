# Part 1 - Overview of Fabric CI/CD

Microsoft Fabric is a unified analytics platform designed to bring
together different data and analytics tools into a single, integrated
software-as-a-service (SaaS) solution. Microsoft Fabric caters to data
engineers, data scientists, and analysts who need to collaborate on
analytics and AI projects. The Fabric platform adds value by reducing
complexity while still making it possible to build, deploy and manage
enterprise-scale projects focused on analytics and AI.

The Microsoft Fabric platform attracts people with different technical
backgrounds. For example, there are data engineers tasked with creating
notebooks to ingest and transform data to populate lakehouse tables.
There are data modelers tasked with building and refining semantic
models on top of lakehouse tables. There are report designers tasked
with authoring and updating Power BI reports that are built on top of
those semantic models. Once you understand how CI/CD works in Fabric,
you will be able enable collaboration which allows all these people to
continually update their part of a Fabric project without stepping on
anyone else’s work.

This article is written to provide guidance to data engineers, release
managers, Fabric administrators and professional developers building
end-to-end solutions on the Fabric platform. This article explains how
to setup the CI/CD infrastructure for analytics and AI projects in
Fabric as well as how to deploy and manage the CI/CD lifecycle of
workspace items such as lakehouses, notebooks, pipelines, semantic
models and reports. This article starts with an overview of the Fabric
CI/CD landscape covering essentials concepts and features. After this
overview, the article will move through the steps of setting up a
real-world Fabric CI/CD project which enables collaboration and provides
a structured release process.

### Quick Review of Traditional CI/CD Fundamentals
CI/CD represents a set of core principles and best practices that have
been widely adopted across the software industry. The goal of CI/CD is
to streamline and accelerate the lifecycle of software development.
CI/CD focuses on two different aspects of managing the application
lifecycle which are continuous integration (CI) and continuous
deployment (CD).

**Continuous integration (CI)** is the practice of merging code changes
into a shared repository on a regular basis. Each time developers merge
their changes, it can trigger automated workflow processes that ensure
the application source code as a whole still works together by running
validation tests. Continuous integration makes it possible to detect
bugs early on. This, in turn, makes it possible to maintain a codebase
that's always in a working state and ready to deploy to production.

Continuous integration is popular because it allows a team of developers
to work on the same application at the same time. CI/CD solves the
challenging problem of integrating the work of multiple developers into
a shared codebase in an ongoing basis. Once you have designed and
implemented a GIT branching strategy, you’ll be able to merge
everybody’s changes together in a structured process that maintains code
quality while also getting updates and new features into production as
quickly as possible.

<img src="./images/Part1/media/image1.png" style="width:30%" />

**Continuous deployment (CD)** is the practice of automating the
deployment of code. Deployment is often preceded by some type of
interactive human approval process. Once a proposed set of changes has
been approved, a continuous deployment process is automatically
triggered to deploy these changes to a target environment.

With continuous deployment, you can build automated processes to route
the lifecycle of application code through a sequence of environments.
Promoting code changes through environments provides opportunities for
enhanced testing and for interactive approval processes that ensure only
high quality code reaches production. The release process in a CI/CD
workflow is often built using a standard set of environments such as
**dev**, **test** and **prod**.

<img src="./images/Part1/media/image2.png"  style="width:38%" />

## Fabric CI/CD Capabilities
The Fabric platform provides the following capabilities and features to
assist with CI/CD.
- GIT Integration
- Deployment pipelines
- Variable libraries
- Workspace Item Types
- Item Definitions

> This section will drill into each of these capabilities to build your
understanding of how they fit together into the big picture.

### GIT Integration
The Fabric CI/CD infrastructure starts with its support for GIT
integration. Imagine you’re on a team building a data analytics solution
based on a Fabric workspace that includes a lakehouse, notebook,
semantic model and report. Fabric makes it possible to maintain the
source code for all four workspace items as a single source of truth in
a GIT repository. Any updates made to workspace items can be versioned,
tracked and compared to other versions.

> Fabric's list of supported GIT providers includes **Azure DevOps**,
**GitHub** and **GitHub Enterprise**. Microsoft is currently planning to
add other popular GIT providers in the future. You can find the most
up-to-date information about Fabric's support for GIT providers and
their limitations by reading [**Supported Git
providers**](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/intro-to-git-integration?tabs=azure-devops#supported-git-providers).

Fabric’s support for GIT integration is enabled at workspace scope. More
specifically, you enable GIT integration by connecting a workspace to a
branch in a GIT repository. If you navigate to the **Git integration**
tab of the **Workspace settings** dialog, you can work through the steps
to connect a workspace to a branch in a GIT repository. The first step
is to select a GIT provider such as Azure DevOps or GitHub.

<img src="./images/Part1/media/image3.png"  style="width:65%" />

Before connecting a workspace to a branch, you need to create a Fabric
connection to the target GIT repository. Fabric provides one connector
for creating Azure Dev Ops source control connections and another
connector for creating GitHub source control connections. When creating
a connection to an Azure DevOps repository, you can configure connection
credentials using a service principal. When creating a connection to
GitHub, you must configure the connection credentials using a personal
access token (PAT).

<img src="./images/Part1/media/image4.png"  style="width:80%" />

> It is a best practice to create GIT source control connections using
service principal credentials. It is also recommended to create GIT
source control connections using the Fabric REST APIs or a tool such as
Terraform. This best practice will be revisited in the walkthrough in
**Part 2 - Deploying a Fabric CI/CD Project**.

If you select Azure DevOps as the GIT provider and create a source
control connection to the target Azure DevOps repository, you can then
select an **Organization**, **Project**, **Git repository** and
**Branch**. Optionally, you can include a **Git folder** setting.

<img src="./images/Part1/media/image5.png" style="width:50%" />

Specifying a **Git folder** setting allows you to avoid storing the
folders containing workspace item definitions in the root folder of the
target repository. Using a **Git folder** setting is recommended in
scenarios where you need to add other project files to the same GIT
repository. In a Fabric CI/CD project, it's common to add YAML files for
defining workflows as well as other types of files used to implement
workflow logic such as shell files, PowerShell files and/or Python
files. By configuring a **GIT folder** setting, you avoid the confusion
mixing the workspace item definition files together with other types of
project files.

> This examples shown in this article will use **Git folder** setting of
**workspace**. However, you can use any name you’d like.

Now imagine you have deployed a data analytics project to a workspace
which includes a lakehouse, a notebook, a semantic model and a report.
When you first connect the workspace to a GIT repository, Fabric will
generate an item definition for each workspace item which it commits to
the target GIT branch as a named folder containing definition files. If
you used a **GIT folder** setting of **workspace**, you should see a
top-level folder named **workspace** with a child folder for each
workspace item. The Fabric workspace UI provides a **GIT status** column
that displays **Synced** indicating all workspace items are in sync with
the item definitions stored in the GIT repository.

<img src="./images/Part1/media/image6.png" style="width:94%" />

Fabric uses a naming convention for workspace item folders which includes item
display name, a period and the item type in the format of **\[Item
Display Name\].\[Item Type\].** You can see from the previous screenshot
that GIT synchronization creates folders using this naming convention
for folder names such as **sales.Lakehouse** and **Product Sales
Summary.Report**.

The **GIT folder** setting also makes it possible to connect multiple
workspaces to a single GIT branch. Consider a scenario where a Fabric
solution is spread out across two workspaces with a staging workspace
and a presentation workspace. The staging workspace can be connected to
a branch with the **Git folder** setting of **workspace/staging** while
the presentation workspace can be connected to the same branch with a
**Git folder** setting of **workspace/presentation**. Even though this
type of solution is spread out across multiple workspaces, the source
code for the entire solution is tracked in a single branch.

<img src="./images/Part1/media/image7.png" style="width:35%" />

### GIT Synchronization

In the simplest use case, you can connect a workspace to a GIT
repository with a single branch. This minimal setup allows you to start
taking advantage of the capabilities of Fabric GIT integration. For
example, you have the ability to version workspace items. You also have
a backup mechanism giving you the ability to revert to previous versions
of a workspace item if something goes wrong with an update.

Let's say you have a workspace and you have connected it to a GIT
repository with a single branch named **main**. At first, the items in
the workspace are in sync with the item definitions in GIT. Next, you
open a report in the browser, navigate to edit mode and make a few
changes. Once you save your changes, the updated report is no longer in
sync with the item definition in GIT. The Fabric workspace UI displays a
**Source control** button in the upper right shows that indicates there
is a workspace item with uncommitted changes. You can also see the **Git
status** column for the report displays **Uncommitted**.

<img src="./images/Part1/media/image8.png" style="width:50%" />

> If you update a report in the browser, the **Git status** for that item
will continue to display **Synced** until you first save your changes by
invoking the **Save** command. Other types of workspace items exhibit
auto-save behavior and don't require an explicit save operation before
they begin to show a **Git status** of **Uncommitted**.

If you click the **Source control** button, that will open the **Source
control** pane on the right side of the page. The **Source control**
pane has two tabs named **Changes** and **Updates**. The **Changes** tab
shows the workspace items which have changes that have not yet been
committed to GIT.

<img src="./images/Part1/media/image9.png" style="width:75%" />

> What happens if you made a mistake while editing the report and
accidently saved your changes? You can click the **Undo** button to
return the report to its previous state before you started editing.
Clicking the **Undo** button will discard any changes made to the
workspace item and return it to its previous state by synching with the
underlying GIT branch.

Once you have made updates to the report and saved the changes, you can
click the **Commit** button. Clicking the **Commit** button runs a
**Commit To GIT** operation which pushes the workspace item changes to
the underlying GIT branch. You should take note that there is a small
textbox at the top of the **Changes** tab. This textbox allows you to
enter a commit message which gets recorded in GIT. While the Fabric UI
does not enforce adding a commit message, you should always have the
discipline to enter a useful and descriptive message about what is being
committed. If you leave the textbox empty, Fabric generates a generic
commit message which doesn’t really indicate what was updated or why.

<img src="./images/Part1/media/image10.png" style="width:35%" />

You have seen that you can run a **Commit to GIT operation** to push
workspace item changes to a GIT branch. However, you can also
synchronize changes in the other direction. Think about a scenario in
which a workspace is synchronized with a GIT branch. After that, new
changes are merged into the branch from a source other than the
workspace item. You can use an **Update from GIT** operation to
synchronize workspace items with changes merged into the underlying GIT
branch.

<img src="./images/Part1/media/image11.png" style="width:35%" />

Consider a scenario in which you have updated a report and committed
your changes to GIT. After that, you realize that there is a problem
with the update and you'd like to roll it back. You can go to the GIT
repository and revert the commit that contains the undesirable changes.
Reverting a commit is often accomplished using a pull request which
makes the reverted commit appear as new changes within the branch. When
Fabric detects new changes in the underlying GIT branch, it determines
that the report is no longer in sync and the **Git status** column
displays **Update Required**.

<img src="./images/Part1/media/image12.png" style="width:50%" />

You can update the report by navigating to the **Updates** tab of the
**Source control** pane and clicking the **Update all** button. Clicking
**Update all** will run an **Update from GIT** operation to synchronize
the report by applying the changes merged into the branch. This will
effectively roll back the changes to the report that were previously
commit to GIT.

<img src="./images/Part1/media/image13.png" style="width:25%" />

You have now seen that GIT integration allows you to synchronize updates
in two different directions. After you've updated a workspace item, you
can commit those changes to GIT. You can also update from GIT to keep
workspace items in sync with changed merged into the underlying GIT
branch. However, GIT integration in Fabric allows you to go on step
further. You can create an empty workspace and initialize it using an
**Update from GIT** operation which uses the items definitions in a GIT
branch to create a matching set of workspace items.

<img src="./images/Part1/media/image14.png" style="width:35%" />

Consider a scenario in which the **prod** workspace is connected through
GIT integration to a repository with a single branch named **main**. You
have just received a request from the business team to perform updates
on a specific report. However, you don't want to perform these updates
by editing the report in the **prod** workspace where business users
will continue to access the current version. Instead, a better approach
is to create a new, isolated workspace for development. This will allow
you to perform updates and test your changes without affecting anything
user can access in the **prod** workspace.

To set up an isolated development environment, you can create a new
workspace named **dev1** and connect it to the same GIT branch named
**main**. When you connect the **dev1** workspace to the same branch
with the same **Git folder** setting as the **prod** workspace, Fabric
will react by automatically creating a set of workspace items in the
**dev1** workspace to match the set of workspace items in the **prod**
workspace.

<img src="./images/Part1/media/image15.png"  style="width:25%" />

When you initialize an empty workspace using a **Update from GIT**
operation, the workspace usually requires additional configuration
before it's ready for development purposes. This is an important topic
that will be revisited shortly.

A key observation is that the **dev1** workspace provides a development
environment that is isolated from the **prod** workspace. You can make
edits to the report in the **dev1** workspace without affecting anything
in the **prod** workspace. After completing the updates on the report,
you can save your work. After that, you can run a **Commit to GIT**
operation to commit the changes from the report item in the **dev1**
workspace to the report definition in the **main** branch.

<img src="./images/Part1/media/image16.png"  style="width:25%" />

Once changes are committed to the **main** branch, the report item in
**prod** workspace will show a **GIT status** of **Update required**.
Now, you can click the **Update all** button to run an **Update from
GIT** operation which will synchronize changes from the report
definition the **main** branch to the report item in the **prod**
workspace.

> Instead of synchronizing these changes by hand, you can develop a
workflow that is triggered automatically whenever changes are merged to
**main** which automates the process of synchronizing changes to the
**prod** workspace.

If you are new to working with GIT, you must be mindful of avoiding
merge conflicts. A merge conflict occurs in Fabric when changes are made
independently to both a workspace item and its corresponding item
definition in GIT. You can update one or the other, but you can't update
both without causing a merge conflict. When Fabric detects a workspace
item has a merge conflict, the **Git status** column displays
**Conflict** and the **Commit** button is disabled for that workspace
item. When you experience merge conflicts, you must take some type of
corrective action to resolve them before you can commit your changes.

<img src="./images/Part1/media/image17.png" style="width:55%" />

**Resolving merge conflicts in Fabric:**
> - [Resolve conflict in a Fabric workspace](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/conflict-resolution)

When you have multiple workspaces connected to a single GIT branch,
there is an increased risk of merge conflicts. You must have the
discipline to ensure changes flow in only one direction. As an example,
let's say you've updated the report in the **dev1** workspace but
haven't yet committed your changes to GIT. What happens if someone else
updates the same report in the **prod** workspace and commits those
changes to the **main** branch? This will create a merge conflict which
restricts you from being able to commit the changes you've made in the
**dev1** workspace.

<img src="./images/Part1/media/image18.png"  style="width:25%" />

> A development model which involves merging all changes into a single GIT
branch is known as **trunk-based development**.

Trunk-based development provides the ability to conduct development work
in isolated workspaces while avoiding the complexity and overhead of a
multi-branch strategy and pull requests. Since there is only one branch
in trunk-based development, it eliminates the risk of merge conflicts
between branches. Trunk-based development offers the benefit of
providing the quickest and most direct way to publish changes into
production.

Trunk-based development also has its drawbacks. It requires a team of
disciplined and experienced people that know what they are doing. Anyone
can merge whatever changes they want directly into the **main** branch
at any time. With trunk-based development, there isn't much control over
what changes get merged into the production codebase. The alternative
approach to trunk-based development is to adopt a multi-branch strategy
based on feature branches and pull requests.

### Feature Workspaces

The use of features branches has become widely adopted in software
development due to popular branching strategies such as Gitflow or
GitHub Flow. A branching strategy which includes feature branches allows
developers to work and commit changes in isolation. You can also
configure a feature branch with workflows that are triggered whenever
changes are committed. These workflows can automate running tests with
linters, validation checks and security scans as part of the continuous
integration process to improve the quality of code that reaches
production.

After committing changes to a feature branch, the next step is to create
a **pull request** to merge those changes to the shared branch. You can
think of a pull request as a proposal that allows collaborators and
release managers to review a proposed set of changes. GIT providers such
as Azure DevOps and GitHub make it possible to configure pull requests
with interactive approval processes that are as simple or as complex as
required for a given scenario. When the approval process completes
successfully, the changes are automatically merged to the shared branch
which is known as the release branch.

<img src="./images/Part1/media/image19.png" style="width:35%" />

The **release branch** is an essential concept in continuous
integration. The release branch is the shared branch where everyone's
changes are merged together. It's called the *release branch* because
its contents represent the single source of truth that is always ready
for deployment to test and production environments.

> The release branch is a concept not a name. A release branch is
typically given a name such as **main** or **dev**.

**Creating and managing pull requests:**
> - [Create pull requests in Azure Dev Ops](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests?view=azure-devops&tabs=browser)
> - [Creating a pull request in GitHub](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)

For developers migrating to Fabric with experienced in traditional
CI/CD, some aspects of Fabric CI/CD will be familiar while other aspects
will not. For example, developers with traditional CI/CD experience are
already familiar with feature branches and pull requests. From the
perspective of traditional CI/CD, creating a feature branch is a quick,
inexpensive operation that allows you to get up and running right away.
Things are different in a Fabric CI/CD project because each feature
branch must be paired up with a feature workspace.

Think about the requirements to create a new feature workspace for
development. In addition to permissions in the GIT repository, the
developer requires permissions in Fabric to create a new workspace and
to assign that workspace to a Fabric capacity. There's also a
requirement to create or reuse a GIT source control connection which is
required to connect the feature workspace to the target feature branch.

When you create a GIT source control connection to connect a workspace
to GIT, the connection is created with a path to the repository.
However, this path does not include anything specific about the branch.
That means you can create a single source control connection to Azure
DevOps or GitHub and then use that connection to connect multiple
workspaces to any number of branches in a target GIT repository.

Consider a scenario in which you have connected the **prod** workspace
to the **main** branch in GIT repository. The **main** branch acts as
the release branch tracking the single source of truth. If you create a
new branch from **main** named **feature1**, this new branch will
initially contain an identical copy of each workspace item definition in
**main**. Next, you can create an empty workspace named **featue1** and
connect it to the **feature1** branch. This triggers Fabric to run an
**Update from GIT** operation which automatically populates the feature
workspace with a matching set of workspace items.

<img src="./images/Part1/media/image20.png" style="width:35%" />

When you initialize a feature workspace with an **Update from GIT**
operation, you're often required to complete additional configuration
steps before it's ready for development purposes. This topic will be
revisited in the discussion of automation.

The following diagram shows a high-level view of a development process
which is based on multiple feature workspaces where each feature
workspace is connected to its own feature branch. Changes to a feature
workspace are first committed to the underlying feature branch. After
that, you need to create a pull request to merge the changes into the
release branch.

<img src="./images/Part1/media/image21.png"
style="width:4.5633in;height:1.66901in" />

Keep in mind that feature branches should be short-lived. You don't want
a feature branch hanging around too long because that increases the risk
of merge conflicts with the release branch. It's a common practice to
delete a feature branch as soon as a pull request completes and its
changes have been merged into a release branch.

When it comes to managing the lifecycle of feature workspaces, you have
a few options. One option is to manage the feature workspace lifecycle
to coincide with feature branches. When using this approach, you delete
both the feature branch and the feature workspace after a pull request
completes and pull request's changes have been merged.

A second option is to delete the feature branch but to keep the feature
workspace around for use with other feature branches. For example, you
can create a feature workspace for a specific person or team working in
a particular area. If you are working with several teams, you can create
one feature workspace for the Spark team working with notebooks and
lakehouses. You can create a second feature workspace for the data
modeling team working with semantic models. You can create a third
feature workspace for the analytics teams who will be continually
updating reports.

The Fabric UI provides built-in commands for creating and managing
feature workspaces and their connection's to feature branches. When you
navigate to the **Source control** pane, it displays the **Update and
Commit** panel by default. If you click the **Branches** tab in this
navigation menu, the **Source control** pane displays the **Branches**
panel.

<img src="./images/Part1/media/image22.png"
style="width:2.7394in;height:1.24636in" />

After navigating to the **Branches** panel, the **Source control** pane
displays a dropdown menu with branching commands.

<img src="./images/Part1/media/image23.png"
style="width:3.06143in;height:2.81477in" />

When you run the **Branch out to workspace** command, Fabric prompts you
with a dialog with options for creating a new feature branch and
connecting this branch to a new or exiting feature workspace. Remember
that creating a new feature workspace requires that the user has
permissions to create a workspace and to assign that workspace to a
Fabric capacity.

The **Switch branch** command allows you to switch the connection for
the current workspace from one branch to another in the same repository.
Before switching to another branch you must either commit or discard any
uncommitted changes in the workspace. When switching to a different
branch, Fabric runs an **Update from GIT** operation which can involve
adding, updating and/or deleting workspace items in the current
workspace.

The **Check out new branch** command runs a set of operations that can
assist with resolving merge conflicts between a feature branch and the
release branch. This command allows the user to switch from the current
branch to a new branch without having to discard any changes in the
current branch. This command is used to resolve merge conflict
scenarios.

More info on branching in the Fabric UI

- [Branch out to
  workspace](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/manage-branches?tabs=azure-devops#scenario-2---develop-using-another-workspace)

- [Switch
  branch](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/manage-branches?tabs=azure-devops#switch-branches)

- [Resolve conflict in git using Checkout new
  branch](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/conflict-resolution#resolve-conflict-in-git)

### Deployment Pipelines

The Fabric platform provides Deployment Pipelines as a continuous
deployment mechanism to help manage the lifecycle of workspace items.
The key concept behind Deployment Pipelines is that changes to workspace
items can be deployed through a set of stages where each stage
represents a specific environment implemented using a Fabric workspace.

<img src="./images/Part1/media/image24.png"
style="width:3.01381in;height:1.03502in" />

Deployment pipelines are designed to assist with continuous deployment
in building a release process. You must keep in mind that release
process only covers the second half of full CI/CD lifecycle. The use of
Deployment Pipelines should be combined together with Fabric GIT
integration to fully implement a complete end-to-end solution.

For example, you can design the development process using feature
workspaces and feature branches where changes are merged back to a
release branch. Updates in the release branch can be synchronized up to
workspace items in the **dev** workspace which represents the first
stage of a Deployment Pipeline. Once changes from the development
process have been propagated to the **dev** workspace, the Deployment
Pipeline is used to implement a release process to deploy changes to the
**test** workspace and then on to the **prod** workspace.

<img src="./images/Part1/media/image25.png"
style="width:2.72919in;height:1.59607in" />

Deployment pipelines were designed to enhance productivity by hiding
many of the low-level details of pushing workspace item changes across
workspaces. Deployment Pipelines provide one out of several options you
have for building a release process. This is an import topics that will
be revisited after you learn a little more about Fabric CI/CD
fundamentals.

### Variable Libraries

An essential aspect of the CI/CD lifecycle involves promoting changes
through a sequence of environments. The canonical example is a release
workflow which moves changes through environments such as **dev**,
**test** and **prod**. When you need to test and validate code for a
project across multiple environments, it’s essential to find an
effective parameterization strategy for environment-specific settings.

The Fabric platform provides the **variable library** to assist with
parameterization. The variable library is a creatable type of workspace
item in which you can define a set of variables. Other workspace items
such as notebooks, pipelines, copy jobs and dataflows can be defined to
read variable values from a variable library. This makes it possible to
avoid hardcoding configuration settings into your code that change
across environments.

<img src="./images/Part1/media/image26.png"
style="width:3.39373in;height:1.78743in"
alt="A diagram of a work space AI-generated content may be incorrect." />

Consider the classic scenario in which you need to build a release
process that moves changes through a sequence of workspaces each of
which represents an environment such as **dev**, **test** and **prod**.
The problem is that code in different workspaces must be configured with
different settings to connect to a different database. To solve this
problem, each workspace requires access to its own unique connection
settings.

<img src="./images/Part1/media/image27.png"
style="width:3.07219in;height:1.22546in" />

The variable library was designed to solve the problem of parameterizing
these types of settings across workspaces. A variable library allows you
to avoid hardcoding the configuration settings required to connect to a
database. Consider a scenario in which you must write Python code in a
Fabric notebook to connect to a SQL database. The thing you want to
avoid is hardcoding these configuration settings into your code as
literal strings.

database_server = 'devcamp.database.windows.net'

database_name = 'ProductSalesDev'

The obvious problem is that these hardcoded values are specific to a
single environment. As an alternative, you can leverage a variable
library to enable support for parameterization. You start by creating a
new variable library with a display name such as
**environment_settings**. Next, you can add two string variables named
**database_server** and **database_name**.

<img src="./images/Part1/media/image28.png"
style="width:4.88895in;height:1.59072in" />

Once you have created the variable library with the two variables, you
can remove the hardcoded configuration values from the notebook and
replace them with code to read the variable values at run time.

database_server =
notebookutils.variableLibrary.get("\$(/\*\*/environment_settings/database_server)")

database_name =
notebookutils.variableLibrary.get("\$(/\*\*/environment_settings/database_server)")

You have learned that a variable library allows you to add variables.
That's probably not too surprising. However, variable libraries have
another important dimension used to provide parameterization support
across workspaces. More specifically, Fabric allows you to extend a
variable library using ***value sets***.

Let's take a step back to build your understanding of why value sets are
important. You've learned that a variable library is a collection of
variables where each variable is defined with a name, a type and a
default value.

<img src="./images/Part1/media/image29.png"
style="width:4.30636in;height:0.89744in" />

The purpose of a value set is to provide an alternate set of values for
each of the variables in the variable library. Consider an example of a
variable library in which the default values have been configured for
the **dev** environment. You can extend the variable library by adding a
value set for the **test** workspace and a second value set for the
**prod** workspace.

<img src="./images/Part1/media/image30.png"
style="width:4.41862in;height:1.17912in" />

The item definition for a variable library includes variables and values
sets. However the item definition does not contain anything to indicate
which value set is active. That's because Fabric maintains a separate
workspace-level setting that tracks which value set is active in the
context of that workspace. That means three different workspaces could
each contain a variable library with an identical item definition, yet
each workspace can be configured to use a different value set.

<img src="./images/Part1/media/image31.png"
style="width:5.53186in;height:1.13147in" />

### Workspace Item Types

You can think of workspace items as the building blocks used to build a
**Fabric solution.** The workspace items that compose a Fabric solution
can be deployed to one or more workspaces to provide specific data
analytics or AI functionality to its users.

<img src="./images/Part1/media/image32.png"
style="width:2.77945in;height:0.8099in" />

In Fabric, each workspace item has an underlying **workspace item type**
which defines its capabilities and the user experience it provides in
the Fabric service. Examples of workspace item types include
**Lakehouse**, **Notebook**, **DataPipeline**, **Semantic Model** and
**Report**. However, this is just a small sampling of the workspace
items types made available when you are building solutions on the Fabric
platform.

The core abstraction of the workspace item type in Fabric is what allows
artifacts from different Fabric workloads to behave in a similar and
consistent manner. For example, each workspace item type provides the
GIT synchronization capability to generate an item definition for a
workspace item and to store that item definition in GIT. Likewise, each
workspace item type provides the complimentary behavior to respond to
**Update from GIT** operation by either creating or updating a workspace
item from using an item definition in GIT.

It is important to understand that Fabric is an evolving platform. The
behavior of workspace item types is updated on a regular basis.
Moreover, Fabric is continually adding new workspace item types to the
platform. As a result, some workspace item types are more mature than
others with respect to their CI/CD capabilities.

There are four primary CI/CD capabilities that are supported by some but
not all workspace item types.

- Support for reading variables from a variable library

- Support with Fabric REST APIs to create and update items using item
  definitions

- Support for calling Fabric APIs using a Service Principal identity

- Support for auto-binding workspace item dependencies

Whenever possible, you should use variable libraries to manage
parameterization across environments. However, not all workspace item
types support variable libraries. So, what should you do when you are
building a Fabric solution and you encounter a workspace item type that
does not support variable libraries? To achieve the same goal with
respect to parameterization, you might be required to make direct edits
to an item definition either in GIT or through a Fabric REST API call.

There can also be differences between workspace items types in their
support for the Fabric REST APIs. While some workspace item types
support creating and updating workspace items using item definitions,
others do not. You also might encounter workspace item types do not
support calling the Fabric REST APIs as a service principal. This is
especially true as workspace item types are initially released in public
preview. The required workaround for this limitation is to call Fabric
REST APIs using a user identity instead of a service principal identity.

It is also noteworthy that some but not all workspace item types are
able to self-manage their relations to other workspace items. More
specifically, some workspace item types can auto-bind to the other
workspace items on which they depend. This auto-binding behavior occurs
as part of the GIT synchronization process. Workspace item types that do
not support auto-bind behavior require your attention because you might
need to run scripts to correctly rebind workspace item dependencies
after an **Update from GIT** operation completes.

Take an example of a Fabric solution composed of a lakehouse, a notebook
and a pipeline. Assume that the notebook has a dependency on the
lakehouse and that the pipeline has a dependency on both the lakehouse
and the notebook. After you have built out the solution in the **dev**
workspace, there are three established dependencies between these
workspace items.

<img src="./images/Part1/media/image33.png"
style="width:2.97734in;height:1.56154in" />

Now think about what happens when you use Fabric GIT synchronization to
replicate these three workspace items from the **dev** workspace to the
**prod** workspace. The outcome you want to avoid is for workspace items
created in the **prod** workspace to retain their dependencies pointing
back to an item in the **dev** workspace. Instead, the desired outcome
is for each workspace item to properly resolve its dependencies to its
related items in the same workspace. You can see from the following
diagram that the notebook and pipeline in the **prod** workspace were
both able to self-manage their relations.

<img src="./images/Part1/media/image34.png"
style="width:5.40093in;height:2.2874in" />

This example demonstrates how notebooks and pipelines support
auto-binding behavior. Keep in mind that not all workspace item support
auto-binding. If you're building a Fabric solution with workspace item
types that lack support for auto-binding, you might be required to write
a script for post-sync job to correctly reestablish workspace item
relations.

Notebooks did not support auto-binding throughout 2025. However,
Microsoft updated the **Notebook** workspace item type in Q1 of 2026
with support for auto-binding. If you encounter a workspace item type
that lacks auto-binding support, it is likely that adding auto-binding
support is a work-in-progress that will be released as the workspace
item type matures.

### Item Definitions

Item definitions represent an essential component type in the Fabric
platform's support for CI/CD. At its core, an item definition is a set
of files that contain the metadata for an instance of a specific
workspace item type. Every item definition requires a file named
**.platform** which is known as the **platform file**. The platform file
contains metadata such as the item type and display name.

<img src="./images/Part1/media/image35.png"
style="width:2.78117in;height:0.85392in" />

The platform file contains a **logicalId** value which is used by Fabric
internally to track logical instances of workspace items as they are
propagated across GIT branches and workspace. For example, what happens
when you use GIT synchronization to replicate a lakehouse from the
**dev** workspace to the **prod** workspace? The two lakehouses have
different ids but the same logical id.

In addition to the platform file, each workspace item type defines its
own unique schema for the files required and/or allowed in an item
definition. For example, the file structure for a notebook item
definition is simple. It just requires one additional definition file
named **notebook-contents.py**.

<img src="./images/Part1/media/image36.png"
style="width:1.65764in;height:0.5363in" />

Note that Fabric supports working with **Notebook** item definitions
using either the **.py** format or the **.ipynb** format.

The item definition for a lakehouse will contain a different set of item
definition files.

<img src="./images/Part1/media/image37.png"
style="width:1.20774in;height:0.63544in" />

Other workspace item types can have item definitions that are far more
complex. For example, the item definition for a semantic model can
include dozens or even hundreds of TMDL files.

<img src="./images/Part1/media/image38.png"
style="width:1.62425in;height:1.93735in" />

While an item definition with a large number of files is more
complicated, it also allow for collaboration in a much more granular
fashion. Consider the benefit of splitting out each table into its own
file in the item definition for a semantic model. This file structure
allows one developer to work on the measures in the **Sales** table
while another developer is making changes to the **Calendar** table.
This extra granularity is essential when multiple developers are working
on a large semantic model at the same time.

In general, you should gain a basic understanding of item definition
files for the workspace items types you are working with in a Fabric
CI/CD project. An easy way to get started is to walk through item
definition files created by GIT synchronization.

Fabric Item Definitions

- [Item management
  overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/item-management-overview)

- [Item definition
  overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/item-definition-overview)

- [GIT Repository with public item definition
  schemas](https://github.com/microsoft/json-schemas/tree/main/fabric/item)

## Automation in Fabric CI/CD

Deploying an end-to-end Fabric solution with CI/CD support requires many
steps. The ability to automate the setup process for a Fabric CI/CD
project infrastructure is essential. The alternative approach (known as
ClickOps) involves completing tasks by hand using a browser in the
Fabric service. Relying on ClickOps is not recommended because it can
lead to problems caused by human error and unnecessary delays. This is
especially true in a scenario involving disaster recovery.

The Fabric platform provides public APIs which make it possible to write
and version programming logic that automates all aspects of the Fabric
CI/CD setup process. For example, the **Fabric REST APIs** provides the
ability to create workspaces, connections and workspace items. The
**Azure Microsoft Fabric REST API** provides the ability to create and
manage Fabric capacities. The **Power BI REST API** is also important to
Fabric developers because it fills some gaps in scenarios in which you
need update semantic models and reports.

This section will begin by discussing the common scenarios in which a
Fabric CI/CD project requires automation. You will learn what needs to
be automated during the setup phase of a project. You will also learn
why and when you need to add automation logic into workflows integrated
into both the development process and the release process.

Once you understand what you need to automate, you will learn about your
options for writing and versioning automation logic. As you will see,
you can leverage developer tools that boost your productivity by
handling the interaction with Fabric APIs behind the scenes. You will
also learn about the alternative approach of writing code that directly
calls the Fabric REST APIs. The section will conclude by introducing the
**fabric-cicd** library and explaining how it's used to build a release
process.

Microsoft APIs used in Fabric development

- [The Fabric REST
  APIs](https://learn.microsoft.com/en-us/rest/api/fabric/articles/)

- [Azure Microsoft Fabric REST
  APIs](https://learn.microsoft.com/en-us/rest/api/microsoftfabric/fabric-capacities?view=rest-microsoftfabric-2023-11-01)

- [Power BI REST
  APIs](https://learn.microsoft.com/en-us/rest/api/power-bi/)

### Determining What Needs To Be Automated

When starting a Fabric CI/CD project, you need to begin by setting up
the project's infrastructure. This is accomplished by creating
tenant-level Fabric items such as workspaces, capacities, connections,
gateways and deployment pipelines. You also need to create a GIT
repository and configure GIT integration support between Fabric
workspaces and GIT branches. This is accomplished by creating a GIT
source control connection to the repository and then by using that
connection to connect one or more workspaces to specific GIT branches.

After a Fabric CI/CD project is up and running, there is often an
ongoing need to create and configure feature workspaces. In many
scenarios it makes sense to automate the creation of feature workspaces
so that you can run an **Update from GIT** operation immediately
followed by programming logic that prepares the workspace for
development. For example, the **Update from GIT** operation will create
a lakehouse, but it will not automatically populate the lakehouse with
data. The **Update from GIT** operation will create a semantic model,
but it will not automatically create and bind a connection to the
semantic model's datasource.

The key observation here is that there is often additional work that
needs to be completed after a workspace has been initialized or updated
with an **Update from GIT** operation. This provides a motivation for
writing custom scripts to automate whatever tasks are required to
prepare the feature workspace for development. As you begin developing
these types of scripts, you should consider two different types of jobs.

- **Post-deploy jobs** run only once after an empty workspace is first
  initialized with an **Update from GIT** operation.

- **Post-sync jobs** run each time after an **Update from GIT**
  operation completes on a workspace.

You can think of a **post-deploy job** as a set of one-time tasks that
are part of the workspace initialization process. An example of common
tasks automated in a post-deploy job are setting the active value set
for a variable library and running a notebook with ETL logic to populate
a lakehouse with data. Another common task is preparing a semantic model
by creating and binding a connection to its datasource. If you have a
semantic model built using import, you should also automate the process
of running a refresh operation to ingest and store the imported data.

You should avoid adding ETL logic directly into the scripts that you
write for post-deploy jobs. Instead, it's recommended that you factor
out ETL logic and add it into workspace items designed for ETL purposes
such as notebooks, pipelines, copy jobs, user-defined functions and
dataflows. If you write the logic in a post-deploy script to discover a
notebook by name and then run it, you can then update the ETL logic in
the notebook without requiring any updates to the script.

A **post-sync job** contains logic that must run every time after an
**Update from GIT** operation completes. One common scenario which
requires post-sync logic when is automating a release process using GIT
synchronization. Consider a development process like the one shown in
the following diagram. Changes to items in the feature workspace are
first committed to a feature branch and then merged into the **main**
branch using a pull request.

<img src="./images/Part1/media/image39.png"
style="width:2.90898in;height:2.09668in" />

Now it's time to think about the release process. How should you
automate synchronizing changes from the **main** branch to the **prod**
workspace? You can start by creating a workflow that is triggered by a
pull request whenever changes are merged to **main**. This workflow can
start by running an **Update from GIT** operation to sync changes from
the **main** branch to the **prod** workspace. However, some workspace
items might require additional modification after GIT synchronization
completes. This can be the case when you are working with workspace
items that do not support auto-bind behavior. It can also be the case
when you re working with workspace items which require
environment-specific settings but do not yet support variable libraries.

### Options for Writing Automation Logic

The previous section examined common Fabric CI/CD scenarios in which you
are required to write automation logic. You learned about what needs to
be automated during project setup and in workflows integrated into the
development process and the release process. Now it's time to move ahead
and examine your options for writing and maintaining automation logic in
a Fabric CI/CD project.

This section begins by introducing two essential developer tools that
simplify automation by hiding the low-level details of authenticating
and programming against Microsoft APIs. First, you will learn about
Terraform and its ability to automate the deployment process which
involves creating the infrastructure for a Fabric CI/CD project. After
that, you will learn about the Fabric CLI which provides the easiest
path to scripting automation logic required in Fabric CI/CD workflows.

The Fabric CLI provides a façade over the Fabric REST APIs to provide an
ease-of-use experience at the expense of some degree of control.
Experienced developers who prefer control over simplicity have the
option of programming directly against the Fabric REST APIs. This
section will cover a few essential Fabric REST API topics including how
to create and update workspace items using item definitions. You will
also learn about the challenges in developing an API-driven release
process using Fabric REST APIs followed by a discussion of how the
**fabric-cicd** library assists to overcome thee challenges.

Whether you use a tool such as Terraform or the Fabric CLI or you call
Microsoft APIs directly, you must consider the identity used to run your
automation logic. Every call to Fabric REST APIs executes under the
identity of a specific Entra Id security principal. Fabric REST APIs
support two types of identity which are **user principals** and
**service principals**. It is a best practice to execute API calls as a
service principal. This is especially true when code with CI/CD workflow
automation logic executes in a cloud-based platform such as Azure
pipelines or GitHub Actions workflows.

### Creating Project Infrastructure using Terraform

Terraform is an open-source tool built on the principles of
**infrastructure as code (IaC)**. IaC is based on the pattern of
defining the infrastructure for a software project using configuration
files as opposed to using manual processes or procedural programming
logic. For most organizations managing cloud based deployments, the use
of IaC has become a well-established best practice in DevOps and CI/CD.

When using Terraform, you start by defining a set of resources in one or
more configuration files. After that, you can run the Terraform **plan**
command to review a planned lists of the steps required to deploy the
configuration. However, running the **plan** command doesn’t actually do
anything. It just shows you the list of steps that need to be completed.
When you're ready to deploy, you run the Terraform **apply** command to
run the job that creates the resources needed to match your
configuration.

When you first run the **apply** command to deploy a configuration,
Terraform creates a state file which tracks the properties for each
resource as it's created or updated. By tracking the state of resources
in the current deployment, Terraform can compare what's in your
configuration against what's been deployed. This allows Terraform to
determine what resources need to be created, updated or destroyed each
time you run the **apply** command.

Terraform is widely adopted because it provides a quick and reliable way
to create the required resources during project setup. However,
Terraform also plays a valuable role in managing resources over the
lifetime of a project. Imagine a scenario in which you need to update a
resource after the initial deployment. You just need to update the
resource and its properties in your configuration and then run the
**apply** command again.

Terraform simplifies Fabric CI/CD project setup because you only need to
define the configuration you want. You don't have to worry about
figuring out what steps are required to get there.

Terraform also offers the advantage of building a deployment process
that is both versionable and repeatable. For example, you can version a
Terraform configuration simply by adding its configuration files to a
GIT repository just as you would version any of type of source files. A
Terraform deployment process is also repeatable. That means you can run
it again and again and it will produce the same outcome. A repeatable
process provides consistency and the fastest path to disaster recovery.

From an architectural perspective, Terraform plays the role of an
orchestrator which completes its work by calling APIs behind the scenes.
But Terraform doesn't call these APIs directly. Instead, it delegates
this work to **Terraform providers** which are plugin components
designed to interact with a specific set of cloud-based APIs. One reason
that Terraform has become so popular is due to its registry which
contains 1000s of available providers allowing you to manage resources
in platforms such as Azure, AWS, and Google cloud.

Microsoft released the **Fabric provider for Terraform** in 2025. Using
Terraform together with Fabric provider enables you to deploy Fabric
CI/CD project infrastructure by creating and assigning permissions to
tenant-level items in Fabric such as workspaces, connections, gateways
and deployment pipelines. In addition to Fabric items, a Terraform
configuration can also be designed include the provisioning of other
types of Azure resources such as an Azure storage account.

Let's look at an example. Image you need to set up a Fabric CI/CD with
three environments as shown in the following diagram. In this scenario,
each environment requires its own instance of an Azure storge account, a
Fabric capacity, a workspace and an ADLS Gen2 connection to the
environment's storage account. Automating this type of deployment
process using an IoC-based approach is exactly what Terraform was
designed for.

<img src="./images/Part1/media/image40.png"
style="width:5.21457in;height:1.64694in" />

When using Terraform, it is recommended that you create a separate
configuration for each environment. After all, you don't want a problem
in the configuration of the **dev** environment or the **test**
environment to affect the **prod** environment. Fortunately, Terraform
makes it possible to define configuration which is reusable across
configurations by creating a **Terraform module** that can serve as a
template for provisioning a consistent set of resources across multiple
environments.

Let's walk through a simple example. You can start by creating a
Terraform module with a reusable configuration designed to provision a
standard set of resources including a storge account, a capacity, a
workspace and a connection. After that, you can reference that module in
the three configurations you create for the **dev**, **test** and
**prod** environments.

By parameterizing your Terraform module with input variables, you can
configure the deployment process to create the resources for each
environment with unique property settings for things like display names
and the SKU size of the Fabric capacity. For example, you can provision
the capacities for your project with an **F4** for **dev**, an **F16**
for test and **F64** for prod.

Now think about the scenario a few months later when you determine the
**test** environment capacity isn't as powerful as it needs to be and
you need to dump it up to a larger SKU size such as **F32** or **F64**.
This example demonstrates how Terraform can make life easier. You just
need to update the **test** environment configuration with the new SKU
size value and run the **apply** command. That's it. Now your problem is
figuring out what to with the all free time you're afforded since
adopting Terraform.

The walkthrough in **Part 2 - Deploying a Fabric CI/CD Project** will
demonstrate the technique of creating a Terraform module as an
environment template for use across three configurations to create
environments for **dev**, **test** and **prod**.

Resources for Terraform

- [Terraform
  Documentation](https://developer.hashicorp.com/terraform/docs)

- [Terraform on Azure
  documentation](https://learn.microsoft.com/en-us/azure/developer/terraform/)

- [Microsoft Fabric
  Provider](https://registry.terraform.io/providers/microsoft/fabric/latest/docs)

Terraform and the Fabric provider also provide capability to create
workspace items such as lakehouses and notebooks. However, you will find
that working with item definitions in a Terraform configuration is
tricky because making dynamic updates to item definition files requires
advanced Terraform templating syntax. Many experienced Fabric CI/CD
practitioners prefer using Terraform exclusively for creating and
managing tenant-level items in Fabric while using either Fabric CLI or
programming the Fabric REST APIs methods for creating and managing
workspace items.

### Automation using the Fabric CLI

Fabric CLI is a developer tool that offers the easiest path to writing
automation logic to complete tasks in Fabric for DevOps and CI/CD. At a
high level, Fabric CLI acts as a façade which abstract away the
underlying Microsoft APIs to provide a single, focused experience to
assist with Fabric automation. Fabric CLI adds value by shielding you
from having to worry about Entra Id authentication, access token
acquisition and the execution of HTTP requests. Fabric CLI also
simplifies running automation logic under the identity of a service
principal.

Fabric CLI is a is a command-line interface tool that supports both an
interactive mode and a scripting mode. The interactive mode allows you
to type and execute a sequence of commands. For example, you can use
Fabric CLI in interactive mode to quickly create a new workspace and
then to create a lakehouse inside that workspace. Fabric CLI also
provides commands to load data into lakehouse tables and to automate
running notebooks and pipelines which contain ETL logic.

Fabric CLI scripting mode makes it possible to write and version
automation logic for a Fabric CI/CD project. For example, you can use a
shell script or a programming language such as PowerShell or Python to
execute a sequence of Fabric CLI command. If you decide not to use
Terraform, you can write scripts which call Fabric CLI commands to
create and manage tenant-level items such as workspaces, capacities,
connections, domains and gateways.

Fabric CLI offers rich support for creating and managing workspace
items. For example, Fabric CLI provides commands such as **create**,
**get**, **set** and **rm** to perform CRUD operations on workspace
items. There are also two Fabric CLI commands that allow you to work
with an item definition. Fabric CLI provides an **import** command used
to create or update an workspace item using an item definition. There is
also a complimentary **export** command used to export a workspace item
to an item definition as a set of definition files in a local folder.

Links for more information about Fabric CLI

- [Fabric CLI](https://microsoft.github.io/fabric-cli/)

- [CLI Modes](https://microsoft.github.io/fabric-cli/essentials/modes/)

- [Fabric CLI
  Commands](https://microsoft.github.io/fabric-cli/commands/)

- [Fabric CLI Usage
  Examples](https://microsoft.github.io/fabric-cli/examples/)

Another important capability of Fabric CLI is its ability to integrate
with cloud-based GIT providers such as Azure DevOps and GitHub. If
you're developing workflows using Azure pipelines or GitHub workflow
actions, it's relatively straight forward to install and load Fabric CLI
executable which enables you develop workflows with logic written to
call Fabric CLI commands.

### Programming the Fabric REST APIs

Developer tools like Terraform and Fabric CLI boost your productivity by
hiding low-level details associated with authentication, acquiring
access tokens and executing HTTP requests. However, you might encounter
scenarios in which you need a greater degree of control. This can be
achieved by programming the Fabric REST API to execute API calls using a
familiar programming language such as Python, C# or PowerShell. For
developers already experienced in calling REST APIs, programming the
Fabric REST API might provide a more natural fit than using developer
tools such as Terraform or Fabric CLI.

You have learned that Terraform is recommended for creating
infrastructure in larger Fabric CI/CD projects. But there could be
factors that make you decide against using Terraform in a particular
Fabric CI/CD project. As an alternative, you can write automation
scripts which call the Fabric REST APIs to create tenant-level Fabric
items such as workspaces, connections, gateways and deployment
pipelines. Using Fabric REST APIs also makes it possible to fully
automate the process setting up GIT integration for a Fabric CI/CD
project. This is achieved by creating a GIT source control connection
and then using that connection to bind workspaces to GIT repository
branches.

Programming the Fabric REST APIs provides the richest support for
managing the lifecycle of workspace items. This is due to the ability to
work directly with items definitions when calling CRUD APIs to create
and update workspace items. This makes it possible to deploy a Fabric
solution to a target workspace using a repeatable process which ensure
workspace items are created with the correct relationships to other
workspace items in the same workspace. For example, you can create a
lakehouse and a notebook in such as way that the notebook is configured
to use the new lakehouse as its default lakehouse.

The Fabric REST APIs can be used to control GIT synchronization. There
is an **Update From Git** API which can be used to initialize or update
a feature workspace from an underlying GIT branch. It's also common to
call **Update From Git** in a release process workflow to synchronize
changes from GIT branch to workspace items. There is a complimentary
**Commit To Git** API used in scenarios in which you need to automate
the commit operation to push workspace item changes to a GIT branch.

If you decide to program directly against the Fabric REST APIs, there
are a few fundamental you need to learn right away. The Fabric REST APIs
are built upon three essential concepts which are **paginated results**,
**long-running operations (LRO)** and **throttling**. Developers who
choose to program the Fabric REST APIs must understand these concepts in
order to write code that is correct and reliable. Fortunately, this is
something you don't have to worry about when using developer tools such
as Terraform and Fabric CLI which deal with paginated results,
long-running operations and throttling behind the scenes.

Fabric REST API Fundamentals

- [Pagination](https://learn.microsoft.com/en-us/rest/api/fabric/articles/pagination)

- [Long running
  operations](https://learn.microsoft.com/en-us/rest/api/fabric/articles/long-running-operation)

- [Throttling](https://learn.microsoft.com/en-us/rest/api/fabric/articles/throttling)

Microsoft offers two Software Development Kits (SDKs) for the Fabric
REST APIs. Microsoft provides a **Microsoft Fabric .NET SDK** for C#
developers and a **Microsoft Fabric Python SDK** for Python developers.
These SDKs provide a productivity boost by hiding low-level details of
executing HTTP requests, transmitting access tokens and converting back
and forth between JSON and strongly-typed objects. The two SDKs also
provide the convenience of dealing with paginated results, long running
operations and throttling errors behind the scenes.

Unlike the Fabric CLI and Terraform, the Fabric REST APIs SDK do not
provide the convenience of authenticating or acquiring access tokens for
you. Regardless of whether you using Fabric REST APIs SDK or calling
Fabric REST API endpoint directly, you need custom code to authenticate
with Entra Id and to acquire access token. The best practice for writing
this type of code is to use the **Microsoft Authentication Library
(MSAL)** which offers versions for several platforms and programming
languages.

Fabric REST API SDKs

- [Microsoft Fabric .NET
  SDK](https://www.nuget.org/packages/Microsoft.Fabric.Api)

- Python SDK (coming soon)

Let's examine a scenario in which you might prefer programming the
Fabric REST APIs as an alternative to using Fabric CLI. Imagine you need
to deploy a Fabric solution to a target workspace which involves
creating a lakehouse and a notebook. You have an existing item
definition for a notebook containing Python code with the ETL logic
needed to populate the lakehouse with data. As a requirement, you need
to create the notebook in such a way that it references the new
lakehouse as its default lakehouse. This means you need to update the
item definition file named **notebook-contents.py** with the lakehouse
id before you can use the item definition to create the notebook**.**

There is a timing issue involved in this scenario. You cannot determine
the new lakehouse Id until after the lakehouse has been created. Once
the lakehouse has been created, you must then update the item definition
file **notebook-contents.py** with the Id of the new lakehouse. This
might lead you to automate your deployment process with Fabric CLI using
the following steps.

1.  Run a script which calls the Fabric CLI **import** command to create
    the lakehouse

2.  Examine new lakehouse in the Fabric service using a browser to
    discover its Id

3.  Open **notebook-contents.py** in a text editor and update the
    metadata which references the default lakehouse

4.  Save your changes to **notebook-contents.py**

5.  Run a script which calls the Fabric CLI **import** command passing
    the updated item definition to create the notebook

The problem here is that the Fabric CLI **import** command depends on
files persisted in the item definition folder. The **import** command
provides no ability to modify an item definition on the fly. Instead,
you must make changes to an item definition file and save your changes
before calling **import**.

There's a more flexible approach available if you're willing to program
using the Fabric REST APIs or one of its available SDKs. More
specifically, you can load a item definition into memory and update it
dynamically on the fly before calling **Create Item** or **Update Item
Definition**. The ability to dynamically update an item definition file
provides a more seamless experience when developing a Fabric solution
where you're required to create a set of workspace items with
dependencies between them.

The Fabric REST APIs offer three primary APIs which involve programming
with item definitions. First, the **Create Item** API allows you to pass
an item definition when creating a workspace item. Second, the **Update
Item Definition** API allows you to pass an item definition when
updating a workspace item. Third, the **Get Item Definition** API allows
you to retrieve the item definition for an existing workspace item.

<img src="./images/Part1/media/image41.png"
style="width:5.48534in;height:1.61868in" />

Let's walk through a scenario in which creating a workspace item
requires dynamically updating an item definition file. Imagine you have
just used the Fabric REST API to create a workspace and a lakehouse
inside that workspace. Now it's time to create the notebook. However,
you have to create a notebook in such a way that it's configured to use
the lakehouse as its default lakehouse.

As discussed earlier, the item definition file **notebook-content.py**
used to create a notebook contains metadata which tracks the
configuration of its default lakehouse. The metadata is hidden from
users when the notebook is opened in the web-based notebook editor
provided by the Fabric service. However, you can view this metadata by
examining the raw file contents of **notebook-content.py** file as shown
in the following screenshot.

<img src="./images/Part1/media/image42.png"
style="width:5.09616in;height:3.37715in" />

Before calling **Create Item**, you first need to update the contents of
**notebook-content** to include the correct workspace id and lakehouse
id. This means your code must track the ids of the workspace and the
lakehouse as they are created. Once you have determined what the ids
are, you can perform some type of find-and-replace operation to
substitute the workspace id, lakehouse id and lakehouse display name
into the contents of **notebook-content.py**.

\# Synapse Analytics notebook source

\# METADATA \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

\# META {

\# META "synapse": {

\# META "lakehouse": {

\# META "default_lakehouse": "{LAKEHOUSE_ID}",

\# META "default_lakehouse_name": "{LAKEHOUSE_NAME}",

\# META "default_lakehouse_workspace_id": "{WORKSPACE_ID}",

\# META "known_lakehouses": \[{ "id": "{LAKEHOUSE_ID}" }\]

\# META }

\# META }

\# META }

Directly manipulating item definition files is a powerful technique that
assumes you know what you are doing. If you update an item definition
file with invalid syntax, you will experience errors when calling
**Create Item** and **Update Item Definition**.

Now let's walk through the programming steps required to call **Create
Item** using an item definition. First, you need to enumerate through
all the files in the item definition folder and load their contents and
their relative files paths into memory. At this point you can updated
the contents of **notebook-content.py** in memory with the metadata for
the default lakehouse.

The next step is to parse together a JSON payload that allows you to
pass the item definition across the network. Things get a bit tricky
because there's a need to pass the contents of multiple files across the
network in a single HTTP request. The way to accomplish this is to
convert the contents of each file into a base64 encoded format. By
formatting file contents using base64 encoding, you can add the content
of any file into a JSON element as a standard string value. This makes
it possible to transmit all the files required for an item definition
across the network in a call the **Create Item** API.

The following JSON listing shows an example of an HTTP request body that
can be passed across the network in a call to the **Create Item** API.
The **definition** element contains a **parts** collection which
contains a **part** element for each file in the item definition. Each
**part** element includes a relative **path** value and a payload for
the file contents in a base64 encoded format.

{

"displayName": "Create Lakehouse Tables",

"type": "Notebook",

"definition": {

"parts": \[

{

"path": ".platform",

"payload":
"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*BASE64-ENCODED-FILE-CONTENT\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"

"payloadType": "InlineBase64"

},

{

"path": "notebook-content.py",

"payload":
"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*BASE64-ENCODED-FILE-CONTENT\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"

"payloadType": "InlineBase64"

}

\]

}

}

At this point, you should be able to make an observation about the
difference between using Fabric CLI compared to programming the Fabric
REST APIs. Programming the Fabric REST APIs provides more control but at
the expense of significantly more complexity. When you use the Fabric
CLI **import** command, all the work of formatting file contents back
and forth between base64 encoding and clear text is handled for you
behind the scenes. Fabric CLI doesn't provide as much control, but it
sure makes things a lot easier.

Consider another convenience provided by the Fabric CLI **import**
command. You don't have to worry about whether the target workspace item
already exists or not. Fabric CLI has a way to determine whether the
item already exists. If the item does not exist, the call to **import**
command results in an API call to **Create Item**. If the item does
exist, Fabric CLI calls **Update Item Definition** instead. To implement
the same logic with the Fabric REST APIs requires multiple steps. First,
you need to call an API such as **Get Items** to determine whether the
target item exists. You must follow that with conditional logic which
determines whether to call either **Create Item** or **Update Item
Definition**.

Let's examine another common scenario in which you need to program the
Fabric REST APIs to update an existing workspace item. Perhaps you'd
like to update the metadata for an existing notebook to switch its
default lakehouse from one lakehouse to another. You can accomplish this
using the following steps.

- Call **Get Item Definition** to retrieve JSON response with the
  current item definition

- Extract the **notebook-content.py** file content and convert it from
  base64 to plain text

- Perform a find-and-replace operation on **notebook-content.py** to
  update the workspace id and lakehouse id

- Convert the updated file contents for **notebook-content.py** back
  into the base64 encoded format

- Update the JSON element for the item definition with the base64
  encoded content for **notebook-content.py**.

- Call **Update Item Definition** passing the JSON with the updated item
  definition.

This section examined programming the Fabric REST APIs using item
definitions to create and update workspace items. When you compare this
approach to using the Fabric CLI **import** command, there is more
control yet also a greater level of complexity. You have to decide which
approach is best for you. If you have seen the movie *The Matix*, you
can make the analogy that the Fabric CLI represent the blue pill while
Fabric REST API programming represents the red pill.

Resources for working with item definitions

- [Fabric item definitions
  overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/item-definition-overview)

- [Create Item
  API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/create-item?tabs=HTTP)

- [Get Item Definition
  API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/get-item-definition?tabs=HTTP)

- [Update Item Definition
  API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/update-item-definition?tabs=HTTP)

When you begin to program the Fabric REST APIs, you might notice that it
provides item-generic APIs as well as item-specific APIs. You've already
seen examples of using item-generic APIs such as **Create Item** and
**Update Item Definition**. However, the Fabric REST APIs also offer
item-specific APIs such as **Create Lakehouse** and **Create Notebook**.
This begs the question should you use item-generic APIs or item-specific
APIs.

With respect to creating, updating and deleting workspace items, it
doesn’t really matter whether you use the item-generic APIs or
item-specific APIs because the result will be the same. For this reason,
many developers prefer using item-generic APIs to create and update
workspace items because it allows them to write generic code which can
be reused with any type of workspace item.

The only time where it's essential to use item-specific APIs is when you
need to a query a workspace item to retrieve its properties. As an
experiment, let's compare the results of calling **Get Item** versus
**Get Lakehouse**. A call to **Get Item** results in an HTTP GET request
to a URL which targets the **items** endpoint for a specific workspace.

https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/items/{itemId}

The response from **Get Item** returns a JSON element which contains
properties common to all types of workspace items.

{

"displayName": "sales",

"description": "Contain for sales data",

"type": "Lakehouse",

"workspaceId": "{LAKEHOUSE_ID}",

"id": "{WORKSPACE_ID}"

}

Now let's compare this to a call to **Get Lakehouse** which uses an URL
targeting the **lakehouses** endpoint for a workspace.

https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/lakehouses/{lakehouseId}

The response from **Get Lakehouse** includes a **properties** element
with additional properties that are specific to lakehouses.

{

"displayName": "sales",

"description": "Contain for sales data",

"type": "Lakehouse",

"workspaceId": "{LAKEHOUSE_ID}",

"id": "{WORKSPACE_ID}"

"properties": {

"oneLakeTablesPath":
"https://onelake.dfs.fabric.microsoft.com/{workspaceId}-{ItemId}/Tables",

"oneLakeFilesPath":
"https://onelake.dfs.fabric.microsoft.com/{workspaceId}-{ItemId}/Files",

"sqlEndpointProperties": {

"connectionString":
"{SQL_ENDPOINT_UNIQUE_PATH}.datawarehouse.fabric.microsoft.com",

"id": "{SQL_ENDPOINT_ID}",

"provisioningStatus": "Success"

}

}

}

There is a common scenario in which you need to create a lakehouse along
with a semantic model which connects to that lakehouse using the OneLake
URL or the SQL endpoint. After creating the lakehouse, you need to call
the **Get Lakehouse** API to discover its properties so that you can
update the semantic model with the correct connection settings.

### Building a Release Process

Now it's time to revisit the topic of building a release process for a
Fabric CI/CD project. Earlier you learned that Deployment Pipelines
provide a low-code approach for building a release process. However,
Deployment Pipelines have limitations which limits their use in certain
scenarios. For example, a Deployment Pipeline and all of its associated
workspaces must exist inside the same Entra Id tenant. If you have a
requirement create the **dev** workspace in one Entra Id tenant and the
**prod** workspace in another, you must choose a different approach for
building your release process.

A second option for building a release process in Fabric is using GIT
synchronization. When using this approach, you can call the **Update
from GIT** API to push changes from a release branch to workspace items
in a target workspace. However, there are issues to consider when
deciding whether to build a release process based on Fabric GIT
synchronization.

First, there is typically extra work required after a call **Update from
GIT** completes. For example, you might be required to write code for a
post-sync job to update workspace items to reestablish item relations or
to replace environment-specific settings.

<img src="./images/Part1/media/image43.png"
style="width:4.00224in;height:0.90786in" />

A second issue with using GIT synchronization is that requires a
connection between the target workspace and a GIT branch. When a
workspace is connected to a GIT branch, the Fabric service lights up the
workspace with indicators and messages to control and monitor GIT
synchronization. For example, the workspace summary page displays the
**Status** column with values such as **Synced** or **Update required**
which you might prefer to hide from users in a production workspace.

<img src="./images/Part1/media/image44.png"
style="width:3.13963in;height:1.13625in" />

The third option for building a release process is use Fabric REST APIs
to automate pushing changes from GIT to a target workspace. Implementing
a release process using an API-driven approach has become popular
because it's able to address and overcome the limitations and issues
involved with using Deployment Pipelines or GIT synchronization.

The goal of an API-driven release process is to update the items in a
target workspace using item definitions retrieved from a GIT branch. The
release process must contain logic to determine if target items already
exist so it can decide whether to call **Create Item** or **Update Item
Definition**. The release process should also be designed with the
ability to update item definitions on the fly to replace
environment-specific settings and to reestablish relationships between
workspace items

<img src="./images/Part1/media/image45.png"
style="width:3.24856in;height:1.07443in" />

In theory, you could spend the time and energy required to implement a
release process using the Fabric REST APIs. However, this approach would
require a non-trivial coding effort because it requires custom logic
that differs across workspace item types. Fortunately, there's a much
better option. You can fully implement your release process without
incurring significant technical debt by leveraging the **fabric-cicd**
library.

### The fabric-cicd Python Library

**fabric-cicd** is an open-source Python library designed by Microsoft
to enable a code-first approach for deploying and updating Fabric
solutions. Code-first deployment in Fabric is based on a development
methodology in which you begin by creating a set of item definitions.
After that, you use these item definitions as templates to deploy
workspace items to a target workspace.

If you have a folder in GIT containing item definitions, **fabric-cicd**
provides an easy way to deploy your code to a target workspace. Behind
the scenes, **fabric-cicd** handles authentication with Entra Id as well
as all the required interaction with Fabric REST APIs. By acting as a
façade, the **fabric-cicd** library provides a significant productivity
boost by hiding all the grungy details involved with Fabric REST API
programming and performing dynamic updates on item definition files.

While the **fabric-cicd** library is an open source project, it is
officially supported by the Fabric product team. You can have confidence
that **fabric-cicd** will continue to evolve by adding support for new
workspace item types add to the platform.

The **fabric-cicd** library is available in the **Python Package Index
(PyPI)** which enables you to install it using **pip install**.

pip install fabric-cicd

The **fabric-cicd** library is commonly used to build release processes
that run in Azure DevOps or in GitHub. The ability to install on demand
using **pip install** makes it easy to integrate **fabric-cicd** with
Azure pipelines or GitHub Custom Actions.

To get started with **fabric-cicd**, you first need to create a
**FabricWorkspace** object initialized with the three properties.

- **repository_directory:** path to top-level folder which contains item
  definitions

- **workspace_id:** target workspace id

- **item_type_in_scope**: string array containing workspace item types
  to include in deployment processing

After initializing a **FabricWorkspace** object, you can pass it as a
parameter in a call to a function named **publish_all_items.** The call
to **publish_all_items** starts the **fabric-cicd** deployment process
and blocks synchronously until the job completes.

from fabric_cicd import FabricWorkspace, publish_all_items,
unpublish_all_orphan_items

\# Initialize the FabricWorkspace object with the required properties

target_workspace = FabricWorkspace(

repository_directory = "{PATH_TO\_ FOLDER_WITH_ITEM_DEFINITIONS}",

workspace_id = "{TARGET_WORKSPACE_ID}",

environment = "PROD",

item_type_in_scope = \["Lakehouse", "Notebook", "SementicModel",
"Report", "VariableLibrary"\],

)

\# run the deployment process

publish_all_items(target_workspace)

When you call **publish_all_items**, **fabric-cicd** runs a deployment
process which enumerates through each item definition and calls either
**Create Item** or **Update Item Definition** depending on whether the
target item already exists. Keep in mind that **fabric-cicd** will
ignore any workspace items whose type is not explicitly included in the
**item_type_in_scope** property.

<img src="./images/Part1/media/image46.png"
style="width:4.81712in;height:1.41257in" />

You've just learned how to use **fabric-cicd** to push changes from a
source folder of item definitions to a target workspace. The next thing
you need to learn is how to dynamically modify item definition files as
part of the **fabric-cicd** deployment process. This is accomplished by
leveraging **fabric-cicd** support for parameterizing
environment-specific settings.

Before walking through how to set up parameterization, let's first
discuss when it's necessary. Imagine you're working with a Fabric
solution designed with a lakehouse and a semantic model. The semantic
model connects to the lakehouse as its datasource using a DirectLake on
OneLake connection. You are currently at a phase where you have this
Fabric solution up and running in the **dev** workspace.

<img src="./images/Part1/media/image47.png"
style="width:0.66676in;height:1.06034in" />

Your next step is to set up GIT integration between the **dev**
workspace and a **release** branch. You connect the **dev** workspace to
the **release** branch and run a **Commit to GIT** operation to persist
item definitions to GIT. The thing to focus on in this example is the
semantic model definition in GIT which has a datasource setting which
points to the lakehouse in **dev**. The problem you need to avoid is
propagating these types of environment-specific settings from **dev** to
**prod**.

<img src="./images/Part1/media/image48.png"
style="width:1.74715in;height:1.10325in" />

Think about what happens when you deploy the lakehouse and semantic
model from the **release** branch to the **prod** workspace. Without
parametrization, your deployment will be flawed because the semantic
model in **prod** will be configured to connect to the lakehouse in
**dev**. The motivation for using parameterization is that you can
dynamically update the item definition for the semantic model before its
used to create or update the target semantic model item in **prod**. By
using parameterization, you can deploy a semantic model with datasource
settings to successfully reference the lakehouse in **prod**.

<img src="./images/Part1/media/image49.png"
style="width:3.14847in;height:1.22207in" />

In general, parameterization is required whenever you need to deal with
environment-specific settings committed to GIT.

Now it's time to discuss how to enable parametrization with
**fabric-cicd**. It requires two steps. The first step is to add a
parameter file. The second step is to initialize the **FabricWorkspace**
object the **environment** property. Let's start by adding a parameter
file named **parameter.yml** which is located in the root folder
containing the item definitions.

<img src="./images/Part1/media/image50.png"
style="width:2.40532in;height:1.48583in" />

To address the deployment problem with the semantic model, you must
configure **parameter.yml** to run a find-and-replace operation to
redirect the semantic model's datasource from the lakehouse in **dev**
to the lakehouse in the target workspace.

The following YAML listing demonstrates the configuration required to
run a find-and-replace operation. There is a top-level key named
**find_replace** with nested keys named **file_value** and
**replace_value**. The **find_value** key is assigned a string value
with the datasource path from the **dev** workspace that you want to
replace. The **replace_value** key contains a nested key-value pair for
each target environment. In this example, the configuration defines two
target environments named **TEST** and **PROD**.

find_replace:

\- find_value:
"https://onelake.dfs.fabric.microsoft.com/{dev-workspace-id}-{dev-lakehouse-id}"
\# \[dev\]

replace_value:

TEST:
"https://onelake.dfs.fabric.microsoft.com/{test-workspace-id}-{test-lakehouse-id}"
\# \[test\]

PROD:
"https://onelake.dfs.fabric.microsoft.com/{prod-workspace-id}-{prod-lakehouse-id}
\# \[prod\]

By default, **fabric-cicd** processes a find-and-replace operation by
inspecting every file in each item definition who type is include in the
**item_type_in_scope** property setting. This can lead to longer
processing times in scenarios in which you have a large number of items
or you have item definitions with a large number files.

To optimize performance, **fabric-cicd** parameterization allows you to
define filters to control which items and files are examined during a
replacement operation. For example, you can extend the **find_value**
key by adding filtering keys such as **item_type**, **item_name** and
**file_path**. The following configuration demonstrates how to filter a
find-and-replace operation to restrict processing so it only inspects
the **expressions.tmdl** file found in the item definitions for semantic
models.

find_replace:

\- find_value:
"https://onelake.dfs.fabric.microsoft.com/{dev-workspace-id}-{dev-lakehouse-id}"
\# \[dev\]

replace_value:

TEST:
"https://onelake.dfs.fabric.microsoft.com/{test-workspace-id}-{test-lakehouse-id}"
\# \[test\]

PROD:
"https://onelake.dfs.fabric.microsoft.com/{prod-workspace-id}-{prod-lakehouse-id}
\# \[prod\]

Item_type: "SemanticModel"

file_path:

\- "/definition/expressions.tmdl"

The second step required to get parameterization working is to
initialize the **FabricWorkspace** object with one additional property
named **environment**. By specifying a value for **environment**, you
are able to select which set of environment settings you'd like to use
for the current deployment. Once you have configured **parameter.yml**
with target environments such as **TEST** and **PROD**, it's trivial to
write the code to deploy to either the **test** workspace or the
**prod** workspace

test_workspace = FabricWorkspace(

environment = "TEST",

repository_directory = "{PATH_TO_FOLDER_WITH_ITEM_DEFINITION}",

workspace_id = "{TEST_WORKSPACE_ID}",

item_type_in_scope = \["Lakehouse", "Notebook", "SementicModel",
"Report", "VariableLibrary"\],

)

publish_all_items(test_workspace)

prod_workspace = FabricWorkspace(

environment = "PROD",

repository_directory = "{PATH_TO_FOLDER_WITH_ITEM_DEFINITION}",

workspace_id = "{PROD_WORKSPACE_ID}",

item_type_in_scope = \["Lakehouse", "Notebook", "SementicModel",
"Report", "VariableLibrary"\],

)

publish_all_items(prod_workspace)

The parameterization support in **fabric-cicd** is powerful because it
allows you to build a release process with a one-to-many mapping between
a single release branch and multiple target workspaces. This flexibility
provides another reason why you might select **fabric-cicd** over
Deployment Pipelines and GIT synchronization as the best choice for
building a release process.

<img src="./images/Part1/media/image51.png"
style="width:1.78234in;height:1.4671in" />

The rich parameterization support available with **fabric-cicd** goes
far beyond what you saw in this simple example. There is support for
configuring replacement operations which use regular expressions and
which use key-based lookups to modify the contents of JSON files and
YAML files. There is also parameterization support for setting the Spark
settings in the target workspace and for binding a semantic model to a
pre-existing connection.

Resource for learning more about fabric-cicd

- [Home page](https://microsoft.github.io/fabric-cicd/0.1.7/)

- [Code
  Samples](https://microsoft.github.io/fabric-cicd/0.1.7/code_sample/)

- [Code
  Reference](https://microsoft.github.io/fabric-cicd/0.1.7/code_reference/)

- [Parameterization](https://microsoft.github.io/fabric-cicd/0.1.33/how_to/parameterization/)

## Developing Workflows for a Fabric CI/CD Project

Now that you've seen your options for how to write the code to implement
Fabric CI/CD tasks, the next question is where to deploy that code. The
answer to this question is simple. You want to deploy and run your code
in the same GIT repository that holds the item definitions. The next two
sections will introduce the topic of hosting custom code in Azure DevOps
and GitHub.

### Developing Azure Pipelines

Azure DevOps is a Microsoft cloud platform that provides a suite of
development tools for software teams to plan, build, test, and deploy
applications. It's designed with CI/CD features to support the entire
software development lifecycle. When you create a new project in Azure
DevOps, the project is automatically created with a GIT repository of
the same name. For example, creating a project named **Project1** will
automatically create a repository named **Project1**. In many scenarios,
the GIT repository that's created along with the project is all you
need. However, you can always add other repositories to a project if
it's needed.

<img src="./images/Part1/media/image52.png"
style="width:3.3222in;height:0.87792in" />

Azure Pipelines is the CI/CD workflow service that's built into Azure
DevOps which provides features to integrate, test and deploy code using
the principals of CI/CD. An Azure pipeline can be configured to run
on-demand or on a schedule such as every night at midnight. Alternately,
you can define an Azure pipeline with a trigger to run automatically in
response to a code commit or the completion of a pull request.

To create an Azure pipeline, you must first add a YAML file into your
repository. Once you have added the YAML file, you must then explicitly
register it with the Azure DevOps project. This can be accomplished
through the Azure DevOps user interface or it can be automated using the
Azure DevOps CLI or the Azure REST APIs. Once a pipeline has been
registered, you should be able to see it in the **Pipelines** page in an
Azure DevOps project.

<img src="./images/Part1/media/image53.png"
style="width:2.53555in;height:1.10719in" />

You can write the code for an Azure pipeline in a programming language
such as PowerShell or Python. This is accomplished by referencing the
script with the PowerShell or Python code from the YAML file. The
following screenshot shows an example of three YAML files for three
Azure pipelines in the **.pipelines** folder and their associated Python
scripts in the **src** folder. Below these two folders you can also see
the **workspace** folder which contains the item definitions for a
specific workspace.

<img src="./images/Part1/media/image54.png"
style="width:1.79409in;height:1.75842in" />

As you begin to develop Azure pipelines for a Fabric CI/CD project, you
will find there's a need to track environment settings such as the ids
for workspaces, capacities, users and groups. You might also need
authentication credentials for an Entra Id application so your code can
authenticate as a service principal. The Azure pipelines service allows
you to create variables to track these types of environment settings.

In order to create variables for use in Azure pipelines, you must first
create a variable group. Once you have created a variable group, you can
then add a set of named variables along with their values. The following
screenshot shows the typical set of variables created for Azure
pipelines in a Fabric CI/CD project.

<img src="./images/Part1/media/image55.png"
style="width:3.39496in;height:1.1021in" />

When you create a variable that contains sensitive data, you can mark
that variable as a secret. For example, your pipeline might need to
access to a client secret used to authenticate as a service principal.
When you mark a variable as a secret, the Azure DevOps service does
several important things to protect its value. First, the variable is
persisted in the cloud using an encrypted format rather than plain text.
Second, the value cannot be viewed in the Azure DevOps user interface
after saving. Third, the secret value is masked in logs. If the secret
value appears in pipeline output, Azure Pipelines replaces it with
\*\*\* so it won't be visible in build logs.

Variable groups in Azure DevOps can also be integrated Azure Key Vault.
That means you can configure access to secrets in Azure Key Vault from
code running in an Azure pipeline. This makes it possible to benefit
from Azure Key Vault features such as conditional access policies,
auditing, and secret rotation.

By default, an Azure pipeline does not have access to the variables in a
variable group. Instead, you must configure permissions for a variable
group to ensure the code in your Azure pipelines has access to its
variables and secrets. These permissions can be configured using the
**Pipeline permissions** dialog in the Azure Devops user interface or by
using the Azure DevOps CLI or the Azure REST APIs.

<img src="./images/Part1/media/image56.png"
style="width:3.58499in;height:1.71429in" />

As you begin to develop with Azure pipelines, you should become familiar
with monitoring pipeline runs in order to test and debug your code. It's
important to add logging to your code to report on successful operations
and to provide diagnostic information about any errors that occur. The
following screenshot shows how an Azure pipeline can be written to log
its progress in a Fabric CI/CD workflow.

<img src="./images/Part1/media/image57.png"
style="width:4.21993in;height:1.65711in" />

Azure DevOps Pipeline Resources

- [Azure Pipelines
  documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops)

- [Key
  concepts](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops)

- [YAML pipeline
  editor](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/yaml-pipeline-editor?view=azure-devops)

- [Manage variable
  groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=azure-pipelines-ui%2Cyaml)

- [Use Azure Key Vault secrets in Azure
  Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=managedidentity%2Cyaml)

### Developing GitHub Workflow Actions

GitHub is different from Azure DevOps with respect to how you work with
projects and repositories. GitHub allows you to create repositories
without first creating a project. While GitHub supports creating
projects to assist with organizing repositories, the use of projects in
GitHub is optional. When using GitHub as the GIT provider for a Fabric
CI/CD project, you just need to create a repository to get started.

<img src="./images/Part1/media/image58.png"
style="width:2.34809in;height:1.03316in" />

GitHub Actions is the workflow platform in GitHub that allows you to
automate CI/CD tasks. You can create GitHub Actions workflows that can
be run on-demand or run on a schedule. You can also configure workflows
to run in response to events such as a code commit or the completion of
a pull request.

With GitHub Actions, you create a new workflow by copying a YAML file
into a special repository folder named **.github/workflows**. Unlike
Azure pipelines, there is no need to register the YAML file with GitHub.
Just coping to YAML file into the **.github/workflows** folder is all
you need to do. Inside the YAML file, you can reference a script written
in a language such as PowerShell or Python. The following screenshot
shows a typical example of the file structure of a Fabric CI/CD project
in a GitHub repository.

<img src="./images/Part1/media/image59.png"
style="width:2.06807in;height:1.67883in" />

As you begin to develop Git Actions workflows for a Fabric CI/CD
project, you need a way to track environment settings such as the ids
for workspaces, capacities, users and groups. Your code might also need
authentication credentials to authenticate as a service principal.
GitHub allows you to create secrets and variables to track these types
of environment settings.

You can create secrets in a GitHub repository by navigating to
**Settings** and selecting the **Secrets and variables \> Actions**.
When you create a secret, GitHub will encrypt its value and redact it
from logs. A GitHub Actions workflow decrypts a secret at runtime,
injects it into your workflow and then immediately discards it from
memory when the workflow completes.

<img src="./images/Part1/media/image60.png"
style="width:3.90423in;height:1.02703in" />

You can also add variables to a GitHub repository for environment
settings that are not sensitive. The following screenshot shows an
example of variables created for Azure pipelines in a Fabric CI/CD
project.

<img src="./images/Part1/media/image61.png"
style="width:2.55979in;height:1.49392in" />

Unlike Azure pipelines, there is no need to configure permissions
because GitHub Actions workflows automatically have access to secrets
and variables in the same repository by default.

While creating secrets and variables at the repository level is adequate
for some projects, you might find you need more granularity. GitHub
supports a feature which allows you to create environments in a GitHub
repository and then to create secrets and variables at environment
scope.

Consider a CD release process in which workspaces don't exists within
the same Entra Id tenant. For example, imagine the **dev** workspace,
the **test** workspace and the **prod** workspace all exist in different
Entra Id tenants. In this scenario, you will need different
authentication credentials for each Entra Id tenant. You can start by
creating three different environments in your GitHub repository named
**dev**, **test** and **prod**.

<img src="./images/Part1/media/image62.png"
style="width:3.56465in;height:1.23178in" />

After that, you can configure each environment with the authentication
credentials specific to its Entra Id tenant.

<img src="./images/Part1/media/image63.png"
style="width:2.21279in;height:1.4922in" />

Once you have added the YAML file for a GitHub Actions workflow into the
**.github/workflows** folder, you can view that workflow in the left
navigation of the **Actions** page. You can also select a workflow see
its run history.

<img src="./images/Part1/media/image64.png"
style="width:3.74109in;height:1.44447in" />

After selecting a workflow in the left navigation, you can drill into a
specific workflow run and view its logs. You can also run a workflow on
demand by dropping down the **Run workflow** menu and clicking the **Run
workflow** button. If you need to collect information from the user
before running the workflow, you can define input fields in the
workflow's YAML file. The following screenshot shows an example of a
workflow with input parameters that prompt the user with a textbox and
two checkboxes.

<img src="./images/Part1/media/image65.png"
style="width:1.65368in;height:1.55828in" />

As you begin to develop GitHub Actions workflows, you should become
familiar with monitoring workflow runs in order to test and debug your
code. It's important to add logging to your code to report on successful
operations and to display diagnostic information about any errors that
occur. The following screenshot shows an example of an Azure pipeline
written to log its progress in a Fabric CI/CD workflow.

<img src="./images/Part1/media/image66.png"
style="width:3.26471in;height:1.89776in" />

Resources for GitHub Actions Workflows

- [Understanding GitHub
  Actions](https://docs.github.com/en/actions/get-started/understand-github-actions)

- [Workflows](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows)

- [Variables](https://docs.github.com/en/actions/concepts/workflows-and-actions/variables)

- [Deployment
  environments](https://docs.github.com/en/actions/concepts/workflows-and-actions/deployment-environments)

## Summary

You have now completed an overview of the Fabric platform's support for
CI/CD. You learned that Fabric GIT integration and the ability to
connect a workspace to a GIT branch provide the foundation for
continuous integration and continuous deployment. Fabric GIT integration
is based on serializing workspace items into item definitions which can
be written to a GIT branch as a set of item definitions. Fabric is also
able to initialize an empty workspace using a set of item definitions in
GIT.

You learned that deployment pipelines provide the easiest way to build a
release process in Fabric for continuous deployment. You also know that
GIT integration and deployment pipelines can be used together to build
an end-to-end CI/CD workflow.

This overview introduced variable libraries which play an essential role
in Fabric CI/CD. Variables libraries allow you to design a
parameterization strategy by creating a set of variables and value sets.
When used correctly, variable libraries help you to avoid writing code
with hardcoded environment settings that change across environments and
workspaces.

You learned there are several different approaches you can take for
automating tasks in a Fabric CI/CD project. You can leverage tools and
libraries like Fabric CLI, Semantic Link Labs and Terraform. You also
have the ability to call Microsoft APIs directly if you feel the extra
degree of control is worth the effort.

This overview introduced the workflow development environments for Azure
DevOps and GitHub. The development experience in Azure DevOps is based
on Azure pipelines while development experience in the GitHub is based
GitHub Action workflows. Both development environments provide a similar
ability to create workflows using YAML-based files and to provide your
code with access to variables and secrets. You should become comfortable
with writing, testing and debugging code in one of these development
environments if you are the one responsible for automation in a Fabric
CI/CD project.

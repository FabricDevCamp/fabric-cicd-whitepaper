# Part 1 - Overview of Fabric CI/CD

CI/CD represents a set of core principles and best practices that have
been widely adopted across the software industry. The goal of CI/CD is
to streamline and accelerate the lifecycle of software development.
CI/CD focuses on two different aspects of managing the application
lifecycle which are Continuous Integration (CI) and Continuous
Deployment (CD)**.**

**Continuous Integration (CI)** is the practice of frequently merging
code changes into a shared repository. Each time developers merges their
changes, it triggers an automated workflow process to ensure the
application code as a whole still works together and can pass validation
tests. CI makes it possible to detect bugs and other problems early on.
This, in turn, makes it possible to maintain a codebase that's always in
a working state and ready to deploy to a production environment.

CI is popular because it allows a team of developers to work on the same
application at the same time. CI/CD solves the challenging problem of
integrating the work of multiple developers into a shared codebase in an
ongoing basis. Once you have designed and implemented a branching
strategy, you’ll be able to merge everybody’s changes together in a
structured process that maintains code quality while also getting new
features into production as quickly as possible.

<img src="./images/Part1/media/image1.png"
style="width:2.28267in;height:0.96045in" />

**Continuous Deployment (CD)** is the practice of automating the
deployment of code. Deployment is typically preceded by some type of
interactive human approval process. Once changes have been approved, a
CD process is automatically triggered to deploy these changes to a
target environment.

With CD, you can build automated processes to route the lifecycle of
application code through a set of stages. Moving code changes through a
set of stages provide opportunities for enhanced testing and approval
processes to ensure only high quality code reaches the production
environment. The release process in a CI/CD workflow is often built
using stages to simulate different environments such as **dev**,
**test** and **prod**.

<img src="./images/Part1/media/image2.png"
style="width:2.98295in;height:0.90051in" />

## Fabric CI/CD Features

The Microsoft Fabric platform attracts people with different technical
backgrounds. For example, there are data engineers tasked with creating
and testing notebooks to ingest and transform data to populate lakehouse
tables. There are data modelers tasked with building and refining
semantic models on top of lakehouse tables. There are report designers
tasked with authoring and updating Power BI reports that are built on
top of those semantic models. The goal of CI/CD is to enable
collaboration which allows all these developers to continually update
their part of a Fabric project without stepping on anyone else’s work.

### Fabric GIT Integration

The Fabric platform's CI/CD infrastructure starts with its support for
GIT integration. Imagine you’re on a team building a data analytics
solution based on a workspace that includes a lakehouse, notebook,
semantic model and report. Fabric makes it possible to maintain the
source code for all four workspace items as a single source of truth in
a GIT-based repository. Any changes made to workspace items can be
versioned, tracked and compared to other versions.

Fabric's list of supported GIT providers includes **Azure DevOps**,
**GitHub** and **GitHub Enterprise**. Microsoft is currently planning to
add other popular GIT providers in the future. You can find the most
up-to-date information about Fabric's support for GIT providers and
their limitations by reading [**Supported Git
providers**](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/intro-to-git-integration?tabs=azure-devops#supported-git-providers).

Fabric’s support for GIT integration is enabled at workspace scope. More
specifically, you enable GIT integration by connecting a workspace to a
branch in a GIT repository. If you navigate to the **Git integration**
tab of the **Workspace settings** dialog, you can create a connection
between the workspace and a GIT repository using a GIT provider such as
Azure DevOps or GitHub.

<img src="./images/Part1/media/image3.png"
style="width:2.32202in;height:1.29109in"
alt="A screenshot of a computer AI-generated content may be incorrect." />

If you select Azure DevOps as the GIT provider, you must then select an
**Organization**, **Project**, **Git repository** and **Branch**.
Optionally, you can also include a value for the **Git folder** setting.

<img src="./images/Part1/media/image4.png"
style="width:2.21017in;height:1.87943in" />

Specifying a **Git folder** is a best practice because it prevents the
source code for workspace items from being stored in the root folder of
the target repository. Storing Fabric item definitions in the root
directory is problematic because you typically need to add other project
files to that repository such as YAML files, Python files and PowerShell
files used to implement workflows with CI/CD logic. By configuring a GIT
folder, you avoid the confusion mixing the definitions files for
workspace items together with other project files.

This examples shown in this article will use **Git folder** setting of
**workspace**. However, you can use any name you’d like.

Consider the scenario in which you have deployed a data analytics
project to a workspace which includes a lakehouse, a notebook, a
semantic model and a report. When you first connect the workspace to a
GIT repository, Fabric will generate a serialized definition for each
workspace item which writes to the target GIT branch as a set of item
definition files. After syncing the workspace to GIT, you should see a
top-level folder named **workspace** with child folders for each
workspace item.

<img src="./images/Part1/media/image5.png"
style="width:5.25065in;height:1.35788in" />

Fabric uses a naming convention for workspace items which includes item
display name, a period and the item type in the format of **\[Item
Display Name\].\[Item Type\].** You can see from the previous screenshot
that GIT synchronization creates folders using this naming convention
for folder names such as **sales.Lakehouse** and **Product Sales
Summary.Report**.

Fabric GIT integration also makes it possible to sync changes in the
other direction. That means you can sync changes from a GIT repository
to a target workspace. This makes it possible to initialize an empty
workspace from a branch which contains a set of workspace item
definitions. As it turns out, initializing an empty workspace from a GIT
branch is very common task in the Fabric development process.

Consider a scenario in which you have connected a workspace to the
**main** branch of a GIT repository. The **main** branch holds item
definitions for each workspace item. If you create a new branch from
**main** named **feature1**, this new branch will initially contain an
exact copy of each item definition in **main**. Now you can create an
empty workspace and connect it to the **feature1** branch. This triggers
Fabric to automatically populate the new workspace by rehydrating a
matching set of workspace items.

<img src="./images/Part1/media/image6.png"
style="width:5.89795in;height:1.51981in" />

After an empty workspace has been initialized from a GIT branch to serve
as a feature workspace, there is often additional work required to
prepare that workspace for development purposes. For example, the GIT
integration process recreates semantic models but it does not
automatically create connections to the underlying datasources.

In a Fabric CI/CD project, it's usually required to set up automated
processes which run after GIT synchronization occurs. As you develop
these types of automated processes, you need to differentiate between
two different kinds of tasks. There are post-deploy tasks and post-sync
tasks.

A **post-deploy task** is a process that must be run only once after an
empty workspace has been initialized from a GIT branch. Examples of
post-deploy tasks are running ETL jobs to populate lakehouse tables with
data and creating and binding connections to semantic models. You can
think of these as one-time tasks that are part of the workspace setup
process.

A **post-sync task** is a process that must be run each time after
Fabric GIT integration pushes changes from a GIT branch to a target
workspace. Examples of post-sync tasks include updating the definition
of a semantic model so that it continues to point to a lakehouse in the
same workspace.

### Team-based Development

Team-based development presents the challenge of merging together the
work of multiple developers into a single location. This challenge is
addressed by building a continuous integration process with an
integration branch. The **integration branch** is a core concept in the
CI/CD development process. This is the place where changes from multiple
developers are merged together in a single branch.

<img src="./images/Part1/media/image7.png"
style="width:4.00663in;height:1.42384in" />

As you design a CI workflow for the development process, one option is
to use the **main** branch as the integration branch.

<img src="./images/Part1/media/image8.png"
style="width:3.05565in;height:1.65893in" />

Alternately, you could design a more intricate workflow that uses the
**dev** branch as the integration branch.

<img src="./images/Part1/media/image9.png"
style="width:4.31588in;height:1.44582in" />

The key point is that you need an integration branch for the development
process. However, which branch you chose as the integration branch will
depend upon the GIT branching strategy you are using.

### Deployment Pipelines

The Fabric platform provides deployment pipelines as a CI/CD mechanism
to help manage the lifecycle of workspace items. The key concept behind
deployment pipelines is that changes to workspace items are deployed
through a set of stages where each stage is implemented using a
workspace.

<img src="./images/Part1/media/image10.png"
style="width:2.92703in;height:1.00522in" />

There are two different ways to utilize deployment pipelines in a Fabric
CI/CD project. The first option is to keep things as simple as possible
and use deployment pipelines as an alternative to using Fabric GIT
integration. The second option is to use deployment pipelines together
in combination with Fabric GIT integration.

One key audience for deployment pipelines is less technical users.
Deployment pipelines were designed to hide many of the low-level details
of pushing changes to workspace items across workspaces. Using
deployment pipelines instead of using GIT integration is a good choice
for organizations that want to avoid the complexity of working with
GIT-based repositories and getting everyone on the team up to speed with
GIT-based development and branching strategies.

Using deployment pipelines without Fabric GIT integration has a major
disadvantage. You lose the ability to track the source code for a Fabric
project as a single source of truth in a GIT repository. Not having a
single source of truth violates one of the key principles of CI/CD. This
makes the option of using deployment pipelines without GIT integration a
non-starter for organizations who have embraced CI/CD and all its
benefits.

A more viable option for leveraging deployment pipelines in a Fabric
CI/CD project is to combine them together with Fabric GIT integration.
For example, the development process can be built using feature
workspaces and feature branches where changes are merged back to the
**main** branch. In this scenario, the main branch is acting as the
integration branch. You can use GIT integration to synchronize changes
from the **main** branch to the **dev** workspace. A deployment pipeline
is used to complete the end-to-end workflow with a CD release process to
push approved changes to **test** and then on to **prod**.

<img src="./images/Part1/media/image11.png"
style="width:4.39066in;height:1.71561in" />

Using deployment pipelines is discussed further in **Chapter 3 -
Building a Release Process with Continuous Deployment.**

### Variable Libraries

An essential aspect of the CI/CD lifecycle involves promoting changes
through a sequence of stages. The canonical example is a release
workflow which moves changes through environments such as **dev**,
**test** and **prod**. When you need to test and validate code for a
project across multiple environments, it’s essential to find an
effective parameterization strategy.

The Fabric platform provides the **variable library** to assist with
parameterization. The variable library is a creatable type of workspace
item in which you can define a set of variables. Other workspace items
such as notebooks, pipelines, copy jobs and dataflows can be defined to
read variable values from a variable library. This makes it possible to
avoid hardcoding configuration values that change across environments
into your code.

<img src="./images/Part1/media/image12.png"
style="width:2.65539in;height:1.39855in"
alt="A diagram of a work space AI-generated content may be incorrect." />

Consider the classic scenario in which you need to build a CD release
process that moves changes through a sequence of environments such as
**dev**, **test** and **prod**. The problem is that code in different
environments must be configured to connect to different databases.

<img src="./images/Part1/media/image13.png"
style="width:3.79383in;height:1.51331in" />

The variable library was created to solve the problem of parameterizing
configuration settings across environments. A variable library allows
you to avoid hardcoding the configuration settings required to connect
to a database. Consider a scenario in which you must write Python code
in a Fabric notebook to connect to a SQL database. The thing you want to
avoid is hardcoding these configuration settings into your code as
literal strings.

database_server = 'devcamp.database.windows.net'

database_name = 'ProductSalesDev'

The obvious problem is that these hardcoded values are specific to one
environments. As an alternative, you can leverage a variable library
along with its support for parameterization. You can start by creating a
new variable library with a display name such as
**environment_settings**. Next, you can add two string variables named
**database_server** and **database_name**.

<img src="./images/Part1/media/image14.png"
style="width:4.04734in;height:1.31688in" />

Once you have created the variable library with the two variables, you
can remove the hardcoded configuration values from the notebook and
replace them with code to read the variable values at run time as shown
in the following code listing.

database_server =
notebookutils.variableLibrary.get("\$(/\*\*/environment_settings/database_server)")

database_name =
notebookutils.variableLibrary.get("\$(/\*\*/environment_settings/database_server)")

You have learned that a variable library allows you to add variables.
That's probably not too surprising. However, variable libraries have
another important dimension used to provide parameterization support
across multiple workspaces. More specifically, Fabric allows you to
extend a variable library using ***value sets***.

A variable library is a collection of variables where each variable is
defined with a name, a type and a default value.

<img src="./images/Part1/media/image15.png"
style="width:3.73871in;height:0.77914in" />

The purpose of a value set is to provide an alternate set of values for
each of the variables in the variable library. Consider an example of a
variable library in which the default values have been configured for
the **dev** environment. You can extend the variable library by adding a
value set for the **test** environment and a second value set for the
**prod** environment.

<img src="./images/Part1/media/image16.png"
style="width:3.91769in;height:1.04182in" />

The item definition for a variable library includes variables and values
sets. However the item definition does not contain anything to indicate
which value set is active. That's because Fabric maintains a separate
workspace-level setting that tracks which value set is the active value
set for that workspace. That means three different workspaces could each
contain a variable library with an identical item definition, yet each
workspace can be configured to use a different value set.

<img src="./images/Part1/media/image17.png"
style="width:5.19496in;height:1.06256in" />

## Understanding Workspace Items

Microsoft created Fabric by merging together several existing products
including Azure Data Factory, Azure Synapse and Power BI. Microsoft's
motivation to merge these products has been to create a unified platform
for a wide variety of projects related to data analysis and AI. In this
merging process, Microsoft was presented with the challenge of making
artifacts, components and features from different products work
together. This led Microsoft to design the Fabric platform on top of a
core abstraction. This is abstraction is that of the **workspace item**.

The Fabric platform is built upon a set of backend services known as
workloads. For example, there is a **Data Factory** workload which
provides workspace item types used in ETL processes such as
**DataPipeline** and **Dataflow**. There is a **Data Engineering**
workload which provides workspace item types familiar to Azure Synapse
developers such as **Lakehouse** and **Notebook**. There is a **Power
BI** workload which provides item types for analytics such as
**SemanticModel** and **Report**.

<img src="./images/Part1/media/image18.png"
style="width:5.14225in;height:1.71721in" />

This is just a small sample of the workloads and workspace items types
available in Fabric. Currently, Fabric supports over 30 creatable
workspace item types and the list is constantly growing. Two years from
now there will likely be new creatable workspace item types that nobody
even knows about yet.

The core abstraction of the workspace item in Fabric is what allows
artifacts from different workloads to behave in a similar manner. The
Fabric platform defines the following set of requirements that each
workspace item type must implement to fully support all the Fabric CI/CD
features.

- Workspace items must support serialization to item definitions for GIT
  integration

- Workspace items must support deserialization from item definition
  pulled from GIT branch

- Workspace items must support relations management during
  deserialization

- Workspace items must support deployment pipelines

- Workspace items must support parameterization using variable library

- Workspace items must support CRUD APIs with item definition for
  create/update

- Workspace items must support service principal when calling CRUD APIs

Not every workspace item type supports all of these requirements. This
can be the case for workspace item types that are still in a preview
release. There is also a number of workspace item types that have
reached general availability (GA) such as warehouses, but do not yet
meet these requirements. While Microsoft is working towards meeting
these requirements in all workspace item types, items that do not meet
these requirements require various workarounds.

### Workspace Item Definitions

Item definitions represent an essential component type in the Fabric
platform's support for CI/CD, As you've seen, an item definition can be
represented as a set of item definition files. Every item definition
requires a common file named **.platform** which is known as the
**platform file**. The platform file contains metadata such as the item
type and display name.

<img src="./images/Part1/media/image19.png"
style="width:5.08154in;height:1.56022in" />

In addition to the platform file, each workspace item type defines its
own unique set of requirements for the files required and/or allowed in
an item definition. For example, the item definition for a notebook is
simple. It just requires one additional item definition file named
**notebook-contents.py**.

<img src="./images/Part1/media/image20.png"
style="width:2.47069in;height:0.79934in" />

The item definition for a lakehouse requires a different set of files.

<img src="./images/Part1/media/image21.png"
style="width:2.09781in;height:1.10375in" />

Other types of workspace items can have item definitions that are far
more complex. For example, the item definition for a semantic model can
include dozens or even hundreds of TMDL files.

<img src="./images/Part1/media/image22.png"
style="width:2.06704in;height:2.46551in" />

While an item definition with a large number of files is more
complicated, it also allow for CI/CD and collaboration in a much more
granular fashion. Consider the benefit of splitting out each table in
the definition of a semantic model into its own file? It allows one
developer to work on the measures in the **Sales** table while another
developer is making changes to the **Calendar** table. This extra
granularity is essential when a team of developers are all working on
the same semantic model.

In general, you should gain a basic understanding of item definition
files for the workspace items types you are working with in a Fabric
CI/CD project. The easiest way to do this is to examine the files in the
item definitions that have been written to a GIT branch using Fabric GIT
integration support.

Additonal information About Fabric Item Definitions

- [Item management
  overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/item-management-overview)

- [Item definition
  overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/item-definition-overview)

### The Role of the LogicalId

Using Fabric GIT integration, you are able to replicate a set of
workspace items across multiple workspaces. As an example, imagine you
have created a lakehouse named **sales** and then you have replicated
the lakehouse across three different workspaces. Each workspace contains
its own distinct instance of the **sales** lakehouse. All three item
instances share the same type and display name. Yet each item instance
has its owns unique i**d**.

<img src="./images/Part1/media/image23.png"
style="width:7.5in;height:1.13403in" />

Fabric integration needs a way to track that these three item instances
are all associated with a single logical item. This is the reason why
Fabric GIT integration includes a **logicalId** property in the platform
file for each workspace instance.

<img src="./images/Part1/media/image24.png"
style="width:5.1708in;height:1.54071in" />

Fabric GIT integration uses **logicalId** to associate multiple item
instances that all represent the same logical item. While each the item
instance has a unique **id**, all item instance share the same
**logicalid**. <img src="./images/Part1/media/image25.png"
style="width:5.39357in;height:0.9269in" />

The **logicalId** is important because it allows workspace items to
abstract away item instance ids that are specific to a single workspace.
This level of indirection makes it possible for workspace items to
automatically resolve their intra-workspace dependencies as part of the
GIT synchronization process.

Take an example of a Fabric solution built using a lakehouse, notebook
and a pipeline. Assume that the notebook has a dependency on the
lakehouse and that the pipeline has a dependency on both the lakehouse
and the notebook. After you have built out the solution in the **dev**
workspace, there are three established dependencies between these
workspace items.

<img src="./images/Part1/media/image26.png"
style="width:1.62417in;height:0.83224in" />

Now think about what happens when you use Fabric GIT integration to
replicate these three workspaces items from the **dev** workspace to the
**prod** workspace. The outcome you do not want is for workspace items
created in the **prod** workspace to retain their dependencies pointing
back to an item in the **dev** workspace. Instead, the desired outcome
is for each workspace item to resolve its own dependencies to its
related items in the same workspace.

<img src="./images/Part1/media/image27.png"
style="width:3.022in;height:1.32576in" />

It is not likely that you will ever need to work directly with the
**logicalId**. It is an internal property which is updated and read by
Fabric behind the scenes. Fabric automatically generates the
**logicalId** for a workspace item the first time it is serialized to an
item definition and written to a GIT branch. As you have seen, workspace
items can manage and reestablish relations as they are replicated across
workspaces. A key point is that workspace items are designed with
behavior to reestablish relations to other workspace items. The
implementation of this behavior in workspace items relies on the
**logicalId** to reestablish intra-workspace dependencies.

You should never modify a **logicalId** value as this is an unsupported
action that can lead to unpredictable behavior.

## Automating Fabric CI/CD Tasks

Automation becomes an important aspect setting up and managing a Fabric
CI/CD project. At the start of a project, there is a need to provision
tenant-level items such as workspaces, capacities, connections and
deployment pipelines. There will be an ongoing need to create new
feature branches, feature workspaces and GIT connections. In some
scenarios, there could be a need to automate synchronizing changes back
and forth between GIT branches and workspaces. There will is also be a
need to update items and their underlying item definitions when writing
the code for post-deploy tasks and post-sync tasks.

While many of the tasks for setting up a Fabric CI/CD project can be
completed by hand, they can also be automated by writing scripts or
programming logic which call public Microsoft APIs. Whenever possible,
it's best to build CI/CD processes that don't require manual
intervention because that can lead to unnecessary delays and problems
caused by human error. The ability to automate an end-to-end setup
process using code which can be tested and versioned leads to greater
levels of reliability and scale.

There are several public APIs from Microsoft that are used to automate
Fabric CI/CD tasks. The Fabric REST APIs provides the ability to create
workspaces, connections and workspace items. However, you must use the
Azure Microsoft Fabric REST API to create and manage Fabric capacities.
The Power BI REST API provides the ability to automate tasks on Power BI
items such as refreshing semantic models and binding them to
connections. The Tabular Object Model is another powerful API that
provides extensive coverage for reading and modifying semantic models.

You can also take advantage of the Azure Data Lake Storage (ADLS) Gen2
APIs and their ability to access content in OneLake. This makes it
possible to read and write files in the **Files** section of a
lakehouse. You also might need to automate tasks within a GIT repository
such as creating a new feature branch or creating and completing a pull
request. To automate tasks with Azure DevOps repositories, you can use
the Azure DevOps Services REST APIs. To automate tasks with GitHub
repositories, you can use the GitHub REST APIs.

<img src="./images/Part1/media/image28.png"
style="width:4.73627in;height:1.8853in" />

As you can see, there are quite a few APIs that make up the complete
landscape for automating tasks in Fabric.

Every call to a Microsoft API executes under the identity of a specific
security principal. Microsoft APIs support two types of principal
identity which are **user principals** and **service principals**. In
general, it is a much better choice to execute API calls for CI/CD tasks
as a service principal. This is especially true when code executes in a
cloud-based environment such as with Azure pipelines or GitHub Actions
workflows.

Links to documentation for Microsoft APIs used in Fabric development

- [Azure Microsoft Fabric REST
  APIs](https://learn.microsoft.com/en-us/rest/api/microsoftfabric/fabric-capacities?view=rest-microsoftfabric-2023-11-01)

- [The Fabric REST
  APIs](https://learn.microsoft.com/en-us/rest/api/fabric/articles/)

- [Power BI REST
  APIs](https://learn.microsoft.com/en-us/rest/api/power-bi/)

- [Tabular Object
  Model](https://learn.microsoft.com/en-us/analysis-services/tom/tom-pbi-datasets?view=sql-analysis-services-2025)

- [ADLS Gen2
  APIs](https://learn.microsoft.com/en-us/rest/api/storageservices/data-lake-storage-gen2)

- [Azure DevOps Services REST
  API](https://learn.microsoft.com/en-us/rest/api/azure/devops)

- [GitHub REST API](https://docs.github.com/en/rest)

There are several approaches to automating Fabric CI/CD tasks using
public Microsoft APIs. Over the next few sections you will see several
different options which make things easier including the Fabric CLI,
Semantic Link Labs and Terraform. There is always the option of writing
code that calls these Microsoft APIs directly using the programming
language of your choice.

### Automating Tasks using the Fabric CLI

The easiest approach for automating tasks in Fabric is to leverage the
Fabric CLI which hides many of the low-level details for authenticating
with Entra Id and executing HTTP requests against Microsoft API
endpoints. Fabric CLI also makes it easy to follow the best practice of
authenticating as a service principal when developing CI/CD workflows.

Fabric CLI is a is a command-line interface tool that supports both an
interactive mode and a scripting mode. The interactive mode allows a
developer or workspace administrator to type in and execute a sequence
of commands. For example, a developer can use Fabric CLI in interactive
mode to quickly create a new workspace and then to create a lakehouse
inside that workspace. Fabric CLI also provides commands to load data
into lakehouse tables and to automate running notebooks and pipelines
which contain ETL logic.

Fabric CLI support for scripting mode can be used to develop CI/CD
workflows that run as an Azure pipeline or a GitHub Actions workflow.
You can develop Fabric CLI scripts in PowerShell or in Python to create
workspaces, capacities, connections and workspace items. Fabric CLI
provides an **export** command to export an item in a workspace to an
item definition which consists of a set of item definition files in a
local folder. Likewise, Fabric CLI provides a complimentary **import**
command used to create or update an item in a target workspace using an
item definition in a local folder.

The Fabric CLI doesn't provide built-in commands to cover every possible
call to the Fabric REST APIs. However, Fabric CLI provides a backdoor
with the **api** command. The **api** command allows an advanced user to
specify the target URL and any JSON to be passed on the request body.
This opens up the possibility of using the Fabric CLI to call any APIs
made available through the Fabric REST APIs and the Power BI REST APIs.

Behind the scenes, Fabric CLI calls to several Microsoft APIs to
complete its work. This includes the Fabric REST APIs, the Power BI REST
API, ADLS Gen2 APIs and the Azure Microsoft Fabric REST APIs. Keep in
mind that all of these APIs have different endpoints and different
requirements for acquiring access tokens. However, that is something you
don't have to worry about when using Fabric CLI. Fabric CLI provides a
single set of commands which abstract away the underlying APIs to
provide a single, focused experience for Fabric automation.

Links for more information about Fabric CLI

- [Fabric CLI](https://microsoft.github.io/fabric-cli/)

- [CLI Modes](https://microsoft.github.io/fabric-cli/essentials/modes/)

- [Fabric CLI
  Commands](https://microsoft.github.io/fabric-cli/commands/)

- [Fabric CLI Usage
  Examples](https://microsoft.github.io/fabric-cli/examples/)

### Automating CI/CD Tasks using Semantic Link Labs and Python

Semantic Link is a feature in Microsoft Fabric that bridges the gap
between data science and data analytics by connecting Power BI semantic
models with Python and Spark environments. Semantic Link allows data
scientists to load the data from a semantic model into a
**FabricDataFrame** object which can be used directly with many popular
data science libraries such as pandas and scikit-learn.

Semantic Link Labs is a Python library designed to provide access to
Semantic Link for developers working in Fabric. However, Semantic Link
Labs goes far beyond just wrapping Semantic Link. It adds a significant
amount of functionality for working with workspace item such as semantic
models, reports, lakehouses, notebooks and variable libraries. Beyond
the scenes, Semantic Link Labs completes its work by calling to other
APIs including Fabric REST APIs, Power BI REST APIs and the Tabular
Object Model. Developers using Semantic Link Labs benefits from the
simplicity of a single API surface without having to think about the
details of acquiring access tokens and dispatching calls to multiple
APIs.

Semantic Link Labs wraps the Tabular Object Model (TOM) to provide
in-depth support for reading and updating semantic model definitions.
For example, Semantic Link Labs makes it simple to redirect the
datasource for a semantic model from one lakehouse to another. Semantic
Link Labs can be used to run refresh operations on semantic models and
to update incremental refresh policy. Furthermore, Semantic Link Labs
includes support for migrating semantic models created using DirectQuery
mode or Import mode over to use DirectLake mode instead.

Semantic Link Labs support updating and analyzing Power BI reports that
use the new PBIR format for their item definitions. This makes it
possible to analyze report metadata and to find broken visuals. This can
help to build your understanding of the high-level structure of a report
definition and to troubleshoot errors.

Semantic Link Labs provides the ability to work with workspace items
from the Data Engineering workload such as lakehouses, notebooks and
environments. Semantic Link Labs simplifies creating lakehouses and
notebooks and configuring a notebook to use a specific lakehouse as its
default lakehouse. There is also support for running jobs for notebooks
and pipelines and monitoring job execution to determine whether the job
completed successfully.

Semantic Link Labs can provide assistance setting up Fabric CI/CD
projects that use deployment pipelines. There are functions you can call
to automate the provisioning of a deployment pipeline with a specific
number of stages and the assignment of workspaces to deployment pipeline
stages. Semantic Link Labs additionally provides the ability to automate
the deployment of workspace item changes from one stage to the next.

Semantic Link Labs can be used in CI processes to improve code quality.
For example, you can create an automated process which uses Best
Practice Analyzer to inspect the item definition for a semantic model or
report. When a developer makes updates to a semantic model or report and
then tries to commit those changes, that can trigger a workflow. The
workflow can inspect the updated item to determine whether it follow all
the best practice rules. If the workflow detects that the updates to the
item violate best practices, it can place responsibility on the
developer to fix the problem before any changes can be committed to a
feature branch.

An important aspect of using the Semantic Link Labs library is that is
only works inside the Fabric environment. You cannot load Semantic Link
Labs in a non-Fabric environment such as Azure pipelines or GitHub
Actions workflows. In order to leverage Semantic Links Labs from a
non-Azure environment, you must maintain code which uses Semantic Link
Labs in a Fabric notebook and then call the Fabric REST APIs to automate
running the notebook.

<img src="./images/Part1/media/image29.png"
style="width:4.98119in;height:1.75956in" />\\

Links for Semantic Link Labs

- [What is semantic
  link?](https://learn.microsoft.com/en-us/fabric/data-science/semantic-link-overview)

- [Semantic Link Labs](https://github.com/microsoft/semantic-link-labs)

- [Semantic Link Labs Code
  Examples](https://github.com/microsoft/semantic-link-labs/wiki/Code-Examples)

### Automating Infrastructure CI/CD Tasks with Terraform

Terraform is an open-source tool built on the CI/CD principle of
infrastructure as code (IaC). Terraform lets you define and set up the
infrastructure for a Fabric CI/CD project using configuration files
rather than ad hoc code or manual processes. Terraform makes it possible
to define the infrastructure setup as code for resources such as
servers, networks and databases. Once the infrastructure setup has been
defined in code, it can be version-controlled, automated, and managed
consistently across different environments.

Terraform has become an industry standard for provisioning Azure
resources such as storage accounts, Azure SQL database and Azure Key
Vault. Given that a Fabric capacity is an Azure resource, Terraform
provides a good fit for provisioning capacities at the start of a Fabric
CI/CD project.

Microsoft has created a Fabric Provider for Terraform that makes it
possible to script out provisioning instructions to create tenant-level
items in Fabric such as workspaces, connections, gateways and deployment
pipelines. The ability to define and version code to provision
tenant-level items provides a valuable compliment to Fabric GIT
integration which only provides support at the level of the workspace
item.

Resources for Terraform

- [Terraform
  Documentation](https://developer.hashicorp.com/terraform/docs)

- [Terraform on Azure
  documentation](https://learn.microsoft.com/en-us/azure/developer/terraform/)

- [Microsoft Fabric
  Provider](https://registry.terraform.io/providers/microsoft/fabric/latest/docs)

### Getting Started with the Fabric REST APIs

Fabric CLI and Fabric Link Labs were both created to simplify automation
in Fabric . Both have become very popular because they provide a
productivity-oriented façade on top of multiple Microsoft APIs. However,
some developers and development teams prefer full control over
simplicity. This is especially true for experienced developers who are
familiar with Entra Id authentication and executing HTTP requests
against REST API endpoints. Calling directly to public APIs such as the
Fabric REST API provides the great amount of flexibility and control.

The Fabric REST APIs were designed with open standards which make it
accessible from a wide variety of development platforms and programming
languages. The architecture of the Fabric REST APIs is built upon three
fundamental concepts which are paginated results. long-running
operations and throttling. Developers who begin working with the Fabric
REST APIs must understand these three essential concepts in order to
write code that is correct, efficient and reliable.

Fabric REST APIs that return list-based results implement a pattern
known as **paginated results**. The motivation for paginated results is
the need to avoid passing too much data across the network at once. For
example, an API call might request a list that is too large to pass back
to the caller in a single response body. The paginated results pattern
allows an API endpoint to pass data to the caller in smaller chunks
(i.e. pages). The use of paginated results improves the performance and
efficiency of API calls, especially when dealing with a large amount of
data.

Fabric REST APIs that run asynchronously implement a pattern known as
**long-running operations**. An API call that runs as a long-running
operation returns a status code of **202 Accepted** to indicates that a
job (i.e operation ) to be completed is queued up and will run sometime
in the future. The caller who receives a **202 Accepted** response is
responsible to take additional steps to monitor the operation’s progress
and to retrieve the final result when a long-running operation
completes.

Fabric REST APIs enforce throttling to maintain optimal performance and
reliability. Fabric throttling behavior limits the number of API calls
which can be executed by a caller within a 60-second time window. Once a
caller has reached the throttling limit in a specific time window,
future calls in the same window will be rejected with a status code of
**429** **Too many requests**. Fabric REST API calls which return a
**429** error will include the **Retry-After** response header. The
value of the **Retry-After** header tells the caller how many seconds to
wait before a new time window begins when the call can be resubmitted.

Microsoft offers Software Development Kits (SDKs) for the Fabric REST
APIs to .NET developer and to Python developers. These SDKs provides a
significant productivity boost to developers by hiding many of the
tedious and low-level details of HTTP executing HTTP requests such as
passing access tokens and converting back and forth from JSON to
strongly-typed .NET objects. These SDK also provide support to
seamlessly deal with paginated results, long running operations and
throttling error behind the scenes.

Fabric REST API Fundamentals

- [Pagination](https://learn.microsoft.com/en-us/rest/api/fabric/articles/pagination)

- [Long running
  operations](https://learn.microsoft.com/en-us/rest/api/fabric/articles/long-running-operation)

- [Throttling](https://learn.microsoft.com/en-us/rest/api/fabric/articles/throttling)

- [.NET SDK](https://www.nuget.org/packages/Microsoft.Fabric.Api)

- Python SDK (no link yet - hoping these will be a link by January 2026)

### Programming with Item Definitions

Programming directly against the Fabric REST APIs is powerful because it
provides full control when creating and updating workspace items. This
style of programming might be tricky at first because you must learn how
to update item definition files and how to pass an item definition
across the network. Once you get over these hurdles, you will find that
this style of programming yeilds the greatest level of control.

There are three primary scenarios in which you program directly with
item definitions. First, you can pass an item definition when calling
the **Create Item** API. Second, you can update an existing item by
passing an item definition in a call to the **Update Item Definition**
API. Third, you retrieve the item definition for an existing item by
calling the **Get Item Definition** API.

<img src="./images/Part1/media/image30.png"
style="width:3.45984in;height:1.36151in" />

Let's walk through a scenario in which creating a workspace item
requires updating an item definition file. In this scenario, imagine you
have just used the Fabric REST API to create a workspace and a lakehouse
inside tht workspace. Now it's time to create the notebook. However, you
have to create a notebook in such a way that it's configured to use the
lakehouse as its default lakehouse.

A notebook item definition requires one file named
**notebook-content.py** in additon to the platform file.

<img src="./images/Part1/media/image20.png"
style="width:2.43631in;height:0.78822in" />

The **notebook-content.py** file has metadata at the top which tracks
the configuration of its default lakehouse. This metadata is hidden from
the user when a notebook is opened in Fabric notebook editor. However,
you can view this metadata by examining the **notebook-content.py** file
as shown in the following screenshot.

<img src="./images/Part1/media/image31.png"
style="width:4.26787in;height:2.82825in" />

The problem is that you need to update **notebook-content.py** with the
correct workspace id and lakehouse id. First, you need to track the ids
that are generated by Fabric when you create the workspace and the
lakehouse. After that, you need to perform some type of
search-and-replace operation to substitute the workspace id, lakehouse
id and lakehouse name into the content of **notebook-content.py**.

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

Once you have updated the contents of **notebook-content.py**, the next
thing you need to do is to pass item definition files across the network
when calling to the **Create Item** API. The next problem you encounter
is how to pass multiple files across the network in an HTTP request
body. The answer to this problem is to format the content of each file
using base64 encoding.

When you format the contents of a item definition file with base64
encoding, you can add that encoded content in a JSON element as a string
value. This makes it possible to transmit all the files for an item
definition in a call the **Create Item** API.

{

"displayName": "Create Lakehouse Tables",

"type": "Notebook",

"definition": {

"parts": \[

{

"path": ".platform",

"payload":
"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*BASE64-ENCODED-FILE-CONTENT"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"

"payloadType": "InlineBase64"

},

{

"path": "notebook-content.py",

"payload":
"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*BASE64-ENCODED-FILE-CONTENT"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"

"payloadType": "InlineBase64"

}

\]

}

}

Now consider another scenario in which you want to update an existing
workspace item. Perhaps you'd like to redirect the default lakehouse for
existing notebook from one lakehouse to another. You can accomplish this
using the following steps.

- Call **Get Item Definition** to retrieve current item definition

- Extract the **notebook-content.py** file content and convert it from
  base64 to plain text

- Perform substitution on the **notebook-content.py** file contents to
  update the workspace id and lakehouse id

- Convert the updated file contents for **notebook-content.py** back
  into the base64 encoded format

- Call **Update Item Definition** passing updated item definition.

Directly manipulating item definition files is a powerful technique that
assumes you know what you are doing. If your update to an item
definition file results in invalid syntax, you will experience errors
calling **Create Item** or **Update Item Definition**.

Resources for working with item definitions

- [Fabric item definitions
  overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/item-definition-overview)

- [Create Item
  API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/create-item?tabs=HTTP)

- [Get Item Definition
  API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/get-item-definition?tabs=HTTP)

- [Update Item Definition
  API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/update-item-definition?tabs=HTTP)

## Developing Fabric CI/CD Workflows

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

<img src="./images/Part1/media/image32.png"
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

<img src="./images/Part1/media/image33.png"
style="width:3.00899in;height:1.31393in" />

You can write the code for an Azure pipeline in a programming language
such as PowerShell or Python. This is accomplished by referencing the
script with the PowerShell or Python code from the YAML file. The
following screenshot shows an example of three YAML files for three
Azure pipelines in the **.pipelines** folder and their associated Python
scripts in the **src** folder. Below these two folders you can also see
the **workspace** folder which contains the item definitions for a
specific workspace.

<img src="./images/Part1/media/image34.png"
style="width:2.05788in;height:2.01697in" />

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

<img src="./images/Part1/media/image35.png"
style="width:4.00772in;height:1.30102in" />

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

<img src="./images/Part1/media/image36.png"
style="width:3.58499in;height:1.71429in" />

As you begin to develop with Azure pipelines, you should become familiar
with monitoring pipeline runs in order to test and debug your code. It's
important to add logging to your code to report on successful operations
and to provide diagnostic information about any errors that occur. The
following screenshot shows how an Azure pipeline can be written to log
its progress in a Fabric CI/CD workflow.

<img src="./images/Part1/media/image37.png"
style="width:6.40177in;height:2.51388in" />

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

<img src="./images/Part1/media/image38.png"
style="width:3.51955in;height:1.5486in" />

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

<img src="./images/Part1/media/image39.png"
style="width:3.13839in;height:2.5477in" />

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

<img src="./images/Part1/media/image40.png"
style="width:4.59134in;height:1.20778in" />

You can also add variables to a GitHub repository for environment
settings that are not sensitive. The following screenshot shows an
example of variables created for Azure pipelines in a Fabric CI/CD
project.

<img src="./images/Part1/media/image41.png"
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

<img src="./images/Part1/media/image42.png"
style="width:3.56465in;height:1.23178in" />

After that, you can configure each environment with the authentication
credentials specific to its Entra Id tenant.

<img src="./images/Part1/media/image43.png"
style="width:3.025in;height:2.03991in" />

Once you have added the YAML file for a GitHub Actions workflow into the
**.github/workflows** folder, you can view that workflow in the left
navigation of the **Actions** page. You can also select a workflow see
its run history.

<img src="./images/Part1/media/image44.png"
style="width:3.74109in;height:1.44447in" />

After selecting a workflow in the left navigation, you can drill into a
specific workflow run and view its logs. You can also run a workflow on
demand by dropping down the **Run workflow** menu and clicking the **Run
workflow** button. If you need to collect information from the user
before running the workflow, you can define input fields in the
workflow's YAML file. The following screenshot shows an example of a
workflow with input parameters that prompt the user with a textbox and
two checkboxes.

<img src="./images/Part1/media/image45.png"
style="width:1.65368in;height:1.55828in" />

As you begin to develop GitHub Actions workflows, you should become
familiar with monitoring workflow runs in order to test and debug your
code. It's important to add logging to your code to report on successful
operations and to display diagnostic information about any errors that
occur. The following screenshot shows an example of an Azure pipeline
written to log its progress in a Fabric CI/CD workflow.

<img src="./images/Part1/media/image46.png"
style="width:4.56973in;height:2.65636in" />

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

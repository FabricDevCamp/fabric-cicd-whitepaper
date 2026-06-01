---
title: Best Practices with Fabric CI/CD 
description: Conceptual guide to Microsoft Fabric CI/CD covering background fundamentals, Fabric-specific capabilities such as Git integration and variable libraries, and automation tools including Terraform, Fabric CLI, and Semantic Link Labs.
ms.reviewer: NimrodShalit
ms.topic: concept-article
ms.date: 05/28/2026
#customer intent: As a Fabric administrator or developer responsible for planning a CI/CD project, I want to understand how Fabric's CI/CD capabilities and automation tools fit together so that I can design a scalable development process and a fast, reliable release process for my solution.
---

# Best Practices with Fabric CI/CD

Microsoft Fabric is a unified analytics platform that brings together different data and analytics tools into a single, integrated software-as-a-service (SaaS) solution. Fabric serves data engineers, data scientists, and analysts who collaborate on analytics and AI projects, reducing complexity while still making it possible to build, deploy, and manage enterprise-scale solutions.

The Fabric platform attracts people with different technical backgrounds. Data engineers create notebooks to ingest and transform data into lakehouse tables. Data modelers build and refine semantic models on top of lakehouse tables. Report designers author and update Power BI reports that consume those semantic models. Once you understand how CI/CD works in Fabric, you can enable collaboration that lets all these people continually update their part of a Fabric project without stepping on each other's work.

This guidance targets technical professionals who plan end-to-end solutions on the Fabric platform, including data engineers, release managers, Fabric administrators, and professional developers. The goal is to build your conceptual understanding of the Fabric CI/CD infrastructure so you can design a scalable development process for continuous integration and a fast, reliable release process for continuous deployment.

## Fabric CI/CD background

Before learning about Fabric's built-in CI/CD capabilities, you should understand a few background topics. This section reviews the core concepts and terminology of traditional CI/CD, and explains the difference between a Fabric *solution* and a Fabric *CI/CD project*.

### Traditional CI/CD fundamentals

CI/CD represents a set of core principles and best practices widely adopted across the software industry. Its goal is to streamline and accelerate the lifecycle of software development. CI/CD focuses on two aspects of managing the application lifecycle: continuous integration (CI) and continuous deployment (CD).

***Continuous integration (CI)*** is the practice of merging code changes into a shared repository on a regular basis. Each merge can trigger automated workflows that validate the application still works as a whole by running tests. CI detects bugs early, which keeps the codebase in a working state and ready to deploy to production. It also lets a team of developers work on the same application simultaneously by structuring how each developer's work is merged into the shared codebase.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices1.png" alt-text="This is a diagram of developers merging code changes into a shared code base through continuous integration." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices1.png":::

***Continuous deployment (CD)*** is the practice of automating the deployment of code. Deployment is often preceded by a manual approval process in which one or more approvers must sign off on a release. Once a release is approved, a continuous deployment process automatically deploys those changes to a target environment.

With continuous deployment, you can build automated processes that route the lifecycle of application code through a sequence of environments. Promoting code changes through environments enables enhanced testing and manual approval gates so only high-quality code reaches production. The release process in a CI/CD lifecycle is often built to promote changes through a standard set of environments such as **dev**, **test**, and **prod**.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices2.png" alt-text="This is a diagram of continuous deployment promoting changes through dev, test, and prod environments." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices2.png":::

### Fabric solutions

You deliver value to users on the Fabric platform by building ***Fabric solutions***. A Fabric solution is a composition of workspace items such as variable libraries, lakehouses, notebooks, pipelines, semantic models, and reports. The Fabric platform currently offers more than 35 distinct workspace item types, and that number continues to grow.

One important decision you must make when designing a Fabric solution is how many workspaces it requires to run. In many scenarios, a Fabric solution can be designed to run inside a single workspace. The following screenshot shows a simple example of a Fabric solution with five workspace items designed to run within a single workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices3.png" alt-text="This is a screenshot of a Fabric workspace containing a simple solution with variable library, lakehouse, notebook, semantic model, and report items." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices3.png":::

When building a Fabric solution with a large number of items, you can partition items using ***workspace folders***. Workspace folders can assist with organizing items by role or by type. The use of workspace folders also helps to declutter the top-level folder which is the default view for business users.

There are scenarios where it's either impractical or impossible to deploy all the items for a Fabric solution to a single workspace. For example, Fabric imposes a limit of 1,000 items per workspace. If you build a solution with more than 1,000 items, you must design it to span two or more workspaces. There can also be design factors related to security, least privilege, and item ownership that lead you to a multi-workspace design.

Consider a Fabric solution that uses a medallion architecture with a security requirement preventing business users from accessing staging data in the bronze and silver lakehouses. Workspace folders inside a single workspace can't be used to configure security. When you add a workspace role assignment, the user gets access to everything inside the workspace. The best way to enforce the security requirement is to split the solution into a **staging workspace** and a **presentation workspace**.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices4.png" alt-text="This is a diagram of a medallion Fabric solution split between staging and presentation workspaces for security isolation." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices4.png":::

The staging workspace contains the items used to run ETL jobs and store staging data. The presentation workspace contains items intended for business users. After deploying this solution, you configure access to each workspace independently. Business users who need access to reports, the semantic model, and the gold lakehouse get a role assignment on the presentation workspace, with no access to the staging workspace.

### Fabric CI/CD projects

Imagine you've prototyped a Fabric solution using the platform's analytics and AI capabilities, shared it with a few peers for review, and received positive feedback. You then present it to the leadership team, who decide to *productionize* the solution so it can be rolled out to a large audience. That moment is when a Fabric CI/CD project begins.

A ***Fabric CI/CD project*** applies CI/CD concepts to a Fabric solution. While the term *application lifecycle management (ALM)* is widely used, it can be helpful to think about Fabric CI/CD as *solution lifecycle management (SLM)*. You stand up multiple environments and get an instance of the Fabric solution running in each one. You also build the CI/CD processes that propagate solution updates from one environment to the next.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices5.png" alt-text="This is a diagram of a Fabric CI/CD project propagating solution updates across dev, test, and prod environments." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices5.png":::

One essential aspect of planning a Fabric CI/CD project is defining the requirements for each environment. An ***environment*** is a set of resources that gives you a place to deploy and run the Fabric solution. At a minimum, an environment for a Fabric solution requires a workspace assigned to a Fabric capacity. If the solution requires two workspaces, then each environment needs two workspaces.

Another important consideration is whether each environment requires its own separate capacity. While you can assign workspaces from all environments to a single shared capacity, the recommended practice is to isolate environments by creating a separate Fabric capacity for each one. The Fabric CI/CD project examples in this guidance use three environments named **dev**, **test**, and **prod**. Your project can use other common names such as **qa**, **uat**, or **ppe**.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices6.png" alt-text="This is a diagram of dev, test, and prod environments each assigned to a separate Fabric capacity." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices6.png":::

When planning environments, you might decide to include other types of Azure resources. For example, a Fabric solution might access data files from an Azure Storage account. In that case, plan to create a separate Storage account for each environment. A primary motivation for environments is the ability to run a Fabric solution against different datasources for development, testing, and production. During environment planning, map out all URLs and paths used to connect to external datasources so you can build a parameterization strategy that lets items in each environment connect to their environment-specific datasources.

## Fabric CI/CD capabilities

The Fabric platform provides the following capabilities to support CI/CD processes:

- Git integration
- Branched workspaces
- Client tools
- Variable libraries
- Workspace item types
- Item definitions
- Deployment pipelines

The rest of this section drills into each of these capabilities, along with a few closely related topics. The goal is to build your understanding of how these pieces fit together before you move on to Fabric automation and workflow development.

### Git synchronization

The Fabric CI/CD infrastructure starts with its support for Git integration. Imagine you're on a team building a data analytics solution in a Fabric workspace that includes a lakehouse, notebook, semantic model, and report. Fabric lets you maintain the source code for all four workspace items as a single source of truth in a Git repository. Any updates made to workspace items can be versioned, tracked, and compared.

Fabric's supported Git providers include Azure DevOps, GitHub, and GitHub Enterprise. For the most current information about Git provider support and limitations, see [Introduction to Git integration](./git-integration/intro-to-git-integration.md). For the typical first-time setup, see [Get started with Git integration](./git-integration/git-get-started.md).

Fabric's Git integration is enabled at workspace scope. You enable it by connecting a workspace to a branch in a Git repository. There are two steps: first, create a Fabric connection to the target Git repository; second, configure a workspace-level setting that connects the workspace to a specific branch.

Fabric provides the **Azure DevOps - Source control** connector for connecting to Azure DevOps repositories. When you > create a connection using this connector, you configure credentials using a Microsoft Entra identity, which can be  either a user principal or a service principal. For GitHub and GitHub Enterprise, Fabric provides the **GitHub - Source control** connector, which uses a personal access token (PAT) for credentials.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices7.png" alt-text="This is a screenshot of the Fabric connections page showing connections created with source control connectors for Azure DevOps and GitHub." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices7.png":::

An essential aspect of Fabric Git integration is its ability to dynamically generate item definitions from workspace items. When you connect a workspace to a branch, Fabric runs a **Commit to Git** operation that generates an item definition for each item and persists it to the target branch as a folder containing item definition files.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices8.png" alt-text="This is a screenshot of item definition folders generated in a Git repository by a Fabric Commit to Git operation." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices8.png":::

> [!NOTE]
> Fabric uses a naming convention for item definition folders that combines the item display name, a period, and the item type in the format of **`[Item Display Name].[Item Type]`**. For example, you'll see folder names such as **`sales.Lakehouse`** and **`Product Sales Summary.Report`**. 

Once a workspace is connected to and synchronized with a Git branch, the workspace **List** view displays a **Git status** column that shows **Synced** to indicate all items are in sync with the underlying item definitions in Git.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices9.png" alt-text="This is a screenshot of a Fabric workspace list view with the Git status column showing items marked as Synced." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices9.png":::

Fabric Git integration also supports synchronization in the other direction. Consider a scenario where all items in a workspace are synchronized with a Git branch, and a peer's pull request later merges new changes into that branch from another branch. You can run an **Update from Git** operation to synchronize those changes from the item definitions in Git back to the items in the workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices10.png" alt-text="This is a diagram of Update from Git synchronizing item definition changes from a Git branch into a Fabric workspace." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices10.png":::

When you run **Update from Git**, Fabric checks whether a target item already exists for each item definition by matching display name and type. If Fabric finds a matching item, it updates the existing item to keep it in sync. If Fabric can't find a matching item, it creates a new item from the definition.

Consider what happens when you connect a Git branch containing item definitions to an empty workspace and run **Update from Git**. Fabric creates a new item from each definition in Git. This leads to an important observation: if you have a Git branch that contains the item definitions for a Fabric solution, you can use that branch to deploy the solution to a target workspace. This is the foundation of Git-based deployment patterns discussed later in this article.

### Git folder settings

When you use the Fabric UI to configure a connection between a workspace and a Git branch, you can specify an optional **Git folder** setting. If you leave **Git folder** at its blank default, Git synchronization creates the item definition folders in the root folder of the target branch. By specifying a value, you direct Git synchronization to write its output to a child folder in the Git branch instead.

There are two important scenarios for configuring a **Git folder** setting.

**Scenario 1: A Fabric CI/CD project that mixes workspace items with other project files.** If your Git repository also needs to hold workflow files such as YAML or Python scripts, configuring a Git folder setting (for example, **`/workspace`**) keeps the item definitions for workspace items separate from those other project files and avoids confusion. The [Git - Connect](/rest/api/fabric/core/git/connect) REST API provides the **`directoryName`** parameter for the same purpose.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices11.png" alt-text="This is a screenshot of a Git repository where Fabric item definition folders are grouped under a workspace folder." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices11.png":::

**Scenario 2: A Fabric solution spread across multiple workspaces.** Revisit the medallion-architecture solution from earlier — a staging workspace with notebooks and lakehouses for bronze and silver data, and a presentation workspace with the gold lakehouse, a semantic model, and a report. While you *could* connect each workspace to its own Git repository, that's a bad idea: you need a single source of truth in Git for the solution as a whole, even when the solution spans multiple workspaces.

The recommended practice is to synchronize both workspaces to the same Git branch in the same repository, with a different **Git folder** setting for each workspace. For example, connect the staging workspace with a **Git folder** setting of  **`/workspace/staging`** and the presentation workspace with **Git folder** setting of **`/workspace/presentation`**. This design is essential for multi-workspace solutions because it effectively creates a single source of truth in Git for the entire solution.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices12.png" alt-text="This is a screenshot of the staging workspace and presentation workspace synchronized to separate folders in the same Git branch." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices12.png":::

### Feature workspaces

Feature branches are widely adopted in modern software development. A development process based on feature branches lets developers work and commit changes in isolation. You can configure feature branches with workflows that automatically run linters, validation checks, and security scans whenever changes are committed, improving the quality of code that reaches production.

After committing changes to a feature branch, the next step is to create a pull request to merge those changes to the shared branch. Git providers such as Azure DevOps and GitHub let you configure pull requests with manual approval processes that are as simple or as complex as your scenario requires. When the approval process completes, the changes are automatically merged into the *integration branch* — the shared branch where all developers' changes are merged together. The integration branch is the single source of truth that's always ready for deployment to test and production environments. Remember that *integration branch* is a concept, not a name. It typically has a name like `main` or `dev`.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices13.png" alt-text="This is a diagram of a feature branch pull request merging approved changes into the integration branch." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices13.png":::

For developers migrating to Fabric with experience in traditional CI/CD, some aspects of Fabric CI/CD will be familiar while other aspects will not. For example, these developers may already be familiar with creating feature branches and pull requests. However, things are different in Fabric CI/CD because each feature branch must be matched up and connected to a feature workspace.

Let's walk through the high-level steps to build a development process for Fabric CI/CD project using feature workspaces. The first step is to create a single source of truth in GIT. For this you need to create a GIT repository and you need to determine which branch will serve as the integration. After that, you can create the single source of truth by creating a set of item definitions in the integration branch.

While there are several approaches you can use to create the initial set of item definitions in the integration branch, a common approach involves getting a version of the Fabric solution up and running in a **dev** workspace. This makes it possible to connect the **dev** workspace to the integration branch and run a **Commit to GIT** operation. The **Commit to GIT** operation creates an item definition in the integration branch for each item in the dev workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices14.png" alt-text="This is a diagram of a Commit to Git operation pushing changes from workspace items to the integration branch." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices14.png":::

When you run a **Commit to GIT** operation to create item definitions in the integration branch, this should be considered a one-time operation. Once you have created the item definitions which represent the single source of truth, you should refrain from directly updating items or running **Commit to GIT** operation in the **dev** workspace. Instead, future changes should be merged from feature branches to the integration branch. This disciple can be enforced by configuration the integration branch in GIT with a branch policy that only allows updates through pull requests.

Once the integration branch has been configured as a single source of truth, you can begin to create feature workspaces. This process involves two steps. First, you create a new feature branch based on the integration branch. After that, you create a new workspace, connect it to the feature branch and run an **Update from GIT** operation. This flow will automatically populate the feature workspace with a matching set of workspace items.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices15.png" alt-text="This is a diagram of creating a feature branch from the integration branch and running a **Update from GIT operation** to synchronize items in a feature workspace." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices15.png":::

> [!NOTE]
> When you initialize a feature workspace with an **Update from GIT** operation, you're often required to complete additional configuration steps before it's ready for development. This topic will be revisited shortly in the discussion of automation.

Once you standardize on a process for creating feature workspaces, you can begin to build what's required for continuous integration. The following diagram shows a development process built using feature branches and feature workspaces. This is an approach which can scale to accommodate a larger number of developers or development teams.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices16.png" alt-text="This is a diagram of a development process that uses a shared dev workspace, an integration branch, feature branches, feature workspaces and pull requests" lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices16.png":::

Keep in mind that feature branches should be short-lived. You don't want a feature branch hanging around too long because that increases the risk of merge conflicts with the integration branch. It's a common practice to delete a feature branch as soon as a pull request completes and merges its changes to the integration branch.

When it comes to managing the lifecycle of feature workspaces, you have two options. The first option is to manage the feature workspace lifecycle to coincide with feature branches. When using this approach, you delete the feature workspace and the feature branch at the same time after the pull request completes and its changes have been merged.

A second option is recycle feature workspaces by reusing them across pull requests. For example, you can create a feature workspace for a specific developer or development team working in a particular area. As an example, you can create a dedicated feature workspace for the Spark team writing ETL logic in notebooks to build lakehouses tables. You can create a second feature workspace for the data modeling team working with semantic models. You can create a third feature workspace for the analytics teams who will be continually updating reports.

> [!NOTE]
> For day-to-day branch operations from inside Fabric, see [Manage branches](./git-integration/manage-branches.md).

### Branched workspaces

Fabric offers a capability known as *branched workspaces* to assist with creating and managing feature workspaces. Fabric support for branched workspaces offers a streamlined user experience which simplifies creating feature workspaces and switching between feature branches. Building out the development process using branched workspaces enhances productivity because Fabric is able to create new GIT branches and managing GIT connections behind the scenes.

Consider a scenario in which the **dev** workspace is connected to an integration branch named **main** in a GIT repository. If you navigate in the **dev** workspace to the **Branches** panel of the **Source control** pane, you can find a set of branching commands which include **Branch out to workspace**.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices17.png" alt-text="This is a screenshot of the Fabric Source control pane showing the Branch out to workspace command." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices17.png":::

When you run the **Branch out to workspace** command, Fabric prompts you with a dialog to enter a new branch name. The **Branch out to workspace** dialog also allows you to choose between creating a new feature workspace or connecting to an existing feature workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices18.png" alt-text="This is a screenshot of the Fabric Source control pane showing the Branch out to workspace command." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices18.png":::

The **Branch out to workspace** dialog provides an option to **Select items individually** to enable a capability known as selective branching. If you select this option and click the **Branch out** button, you will be prompted with another dialog which allows you to select which items to include in the branch out process.

Selective branching is particularly useful in scenarios which include workspaces containing a large number of items. It can take a long time to run a full branch out process when the **dev** workspace that contains 100s of items. Selective branching allows you to complete the branch out process much faster in a case where you only need to work on a small subset of items. After the initial branch out, you can also incrementally add other items to the branched workspace as needed.

When you branch out to a feature workspace, Fabric creates a relationship between the branched workspace and the shared **dev** workspace connected to the integration branch. This makes it easier to visualize the high-level development process for a Fabric CI/CD project. If you navigate the **Related Branches** view in the **Source control** pane, you can see the top navigation node in the hierarchy displays the integration branch named **main** and the name of the shared **dev** workspace. Each child node displays the name of each feature workspace along with its underlying GIT branch name.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices19.png" alt-text="This is a screenshot of the Fabric Source control pane showing the Branch out to workspace command." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices19.png":::

Within each branched workspace, Fabric also provides a workspace breadcrumb menu that allows you to visualize that this feature workspace is related to the shared dev workspace connected to the integration branch.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices20.png" alt-text="This is a screenshot of the Fabric Source control pane showing the Branch out to workspace command." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices20.png":::

An important consideration when working with the **Branch out to workspace** command involves the permissions required in the GIT repository and in Fabric. To branch out to a new workspace, you must have permissions to create new branches in the GIT repository. You also need permissions in Fabric to create new workspaces and to assign workspaces to a Fabric capacity. For more information about permissions see [Required Git permissions for popular actions](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/git-integration-process?tabs=Azure%2Cazure-devops#required-git-permissions-for-popular-actions)

> [!NOTE]
> Microsoft is currently working to enhance branched workspace capabilities to address issues that customers face with permissions. Fabric plans to introduce a new delegated model to enable self-service branch out operations for users that do not posses the permissions to create workspaces or assign a workspace to a Fabric capacity.

In addition to the **Branch out to workspace** command, there are several other commands in the **Source control** pane that used to manage branched workspaces and their connections to feature branches. The **Switch branch** command allows you to switch the connection for the current workspace from one branch to another in the same GIT repository. Before switching to another branch you must either commit or discard any uncommitted changes in the workspace. When switching to a
different branch, Fabric runs an **Update from GIT** operation which can involve adding, updating and/or deleting workspace items in the current workspace.

The **Check out new branch** command runs a set of operations that can assist with resolving merge conflicts between a feature branch and the integration branch. This command allows the user to switch from the current branch to a new branch without having to discard any changes in the current branch. This command is used to resolve merge conflict scenarios.

For more information about permissions the branch-out experience, see [Development process using branched workspace](./git-integration/ranched-workspace.md) and [Git integration process](./git-integration/git-integration-process.md). To review changes before committing or updating, see [Compare changes with granular compare](./git-integration/granular-compare.md).

### Variable libraries

An essential aspect of the CI/CD lifecycle involves promoting changes through a sequence of environments. The canonical example is a Fabric solution lifecycle which moves changes through environments such as **dev**, **test** and **prod**. When you need to test and validate code for a project across multiple environments, it's essential to find an effective parameterization strategy for environment-specific settings.

The Fabric platform provides the *variable library* to assist with parameterization. The variable library is a creatable type of workspace item in which you can define a set of variables. Other workspace items such as notebooks, pipelines, copy jobs and dataflows can be defined to read variable values from a variable library. This makes it possible to avoid hardcoding configuration settings that change across environments.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices27.png" alt-text="This is a diagram of workspace items reading environment settings from a Fabric variable library at run time." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices27.png":::

Consider the classic scenario in which you need to build a release process that moves workspace item updates through a sequence of workspaces representing environments such as **dev**, **test** and **prod**. The problem is that items in different workspaces must be configured with different settings to connect to a different database or to some other type of datasource. To solve this problem, items in each workspace requires access to their own unique connection settings.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices28.png" alt-text="This is a diagram of dev, test, and prod workspaces requiring separate configuration values for their data sources." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices28.png":::

The variable library was designed to solve the problem of parameterizing these types of settings across workspaces. A variable library allows you to avoid hardcoding the configuration settings required to connect to a database. Consider a scenario in which you must write Python code in a Fabric notebook to connect to a SQL database. The thing you want to avoid is hardcoding these configuration settings into your code as literal strings.

``` python
database_server = 'devcamp.database.windows.net'
database_name = 'ProductSalesDev'
```

The obvious problem is that these hardcoded settings are specific to a single environment. As an alternative, you can leverage a variable library to enable support for parameterization. You start by creating a new variable library with a display name such as **environment_settings**. Next, you can add two string variables named **database_server** and **database_name**.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices29.png" alt-text="This is a screenshot of a variable library with two variables used to parameterize configuration values for connecting to a database." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices29.png":::

Once you have created the variable library with the two variables, you can remove the hardcoded configuration values from the notebook and replace them with code to read the variable values at run time.

``` python
database_server = notebookutils.variableLibrary.get("$(/**/environment_settings/database_server)")
database_name = notebookutils.variableLibrary.get("$(/**/environment_settings/database_server)")
```

You have learned that a variable library allows you to add variables. That's probably not too surprising. However, variable libraries have another important dimension used to provide parameterization support across workspaces. More specifically, Fabric allows you to extend a variable library using *value sets*.

Let's take a step back to build your understanding of why value sets are important. You've learned that a variable library is a collection of variables where each variable is defined with a name, a type and a default value.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices30.png" alt-text="This is a tables showing the name, type and default value for two variables." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices30.png":::

The purpose of a value set is to provide an alternate set of values for each of the variables in the variable library. Consider an example of a variable library in which the default values have been configured for the **dev** environment. You can extend the variable library by adding a value set for the **test** workspace and a second value set for the **prod** workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices31.png" alt-text="This is a diagram showing the variable library extended with value sets for test and prod." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices31.png":::

The item definition for a variable library includes variables and values sets. However the item definition does not contain anything to indicate which value set is active. That's because Fabric maintains a separate workspace-level setting that tracks which value set is active in the context of that workspace. That means three different workspaces could each contain a variable library with an identical item definition, yet each workspace can be configured to use a different value set.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices32.png" alt-text="This is a diagram showing an example of value set activation across the dev, test, and prod workspaces." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices32.png":::

You can create variables in a variable library based on standard types including **String**, **Number**, **Integer**, **DateTime** and
**Guid**. Variable libraries additionally support two important reference variable types which are **Connection reference** and **Item reference**. Using these reference variable types makes it easier to edit variable values through the Fabric UI. When you edit the value of a reference type variable, Fabric presents with an item picker showing a list of candidate items from which to choose.

**Connection reference** variables are commonly used to parameterize connections for items involved with ETL such as pipelines and shortcuts in a lakehouse. It's a best practice to design items which connect to external datasources using a variable library which includes a connection reference variable and other variables which parameterize datasource paths. Using a connection reference variable is considered a best practice because it removes anything from the item definition that is environment specific.

**Item reference** variables are useful for managing dependencies between items in the same solution. This is especially true when you need to deal with an item dependency that spans across workspace boundaries. Let's revisit the scenario with the multi-workspace solution introduced earlier. There is a notebook in the staging workspace that needs to write its output to a lakehouse in the presentation workspace with gold layer tables. You can use an item reference variable to parameterize the target lakehouse. This, in turn, provides the ability to remove any code from the notebook which is environment specific because the notebook can use the item refence to determine where to write its output.

> [!NOTE]
> For more on variable libraries in CI/CD, see [What is a variable library?](./variable-library/variable-library-overview.md), [CI/CD and variable libraries](./variable-library/variable-library-cicd.md), [Variable types](./variable-library/variable-types.md), [Value sets](./variable-library/value-sets.md), and [Variable library permissions](./variable-library/variable-library-permissions.md). For details on the two reference variable types, see [Connection reference variable type](./variable-library/connection-reference-variable-type.md) and [Item reference variable type](./variable-library/item-reference-variable-type.md).

### Workspace item types

Every Fabric solution is a composition of workspace items, and every item is based on a *workspace item type* that defines its capabilities and the user experience it provides in the Fabric service.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices33.png" alt-text="This is a diagram of a Fabric solution composed of items all of which are created from a specifc workspace item type." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices33.png":::

The workspace item type abstraction is what allows artifacts from different Fabric workloads to behave in a similar and consistent manner. Each item type provides Git synchronization so item definitions can be generated and stored in Git, and each item type responds to **Update from Git** by either creating or updating items from the definitions in Git.

Fabric is an evolving platform. Workspace item types are updated regularly, and new item types are continually added. As a result, some item types are more mature than others with respect to their CI/CD capabilities.

There are four primary CI/CD capabilities that some — but not all — workspace item types support:

- Reading variables from a variable library
- Creating and updating items through the Fabric REST APIs using item definitions
- Calling Fabric REST APIs as a service principal
- Auto-binding workspace item dependencies

Whenever possible, use variable libraries to manage parameterization across environments. When you encounter an item type that doesn't support variable libraries, you may need to fall back to direct edits of the item definition — either in Git or through a Fabric REST API call — to achieve the same goal.

You'll also find differences across item types in their support for the Fabric REST APIs. Some item types support creating and updating items using item definitions; others don't. Some item types don't yet support service-principal calls to the Fabric REST APIs. This is especially common while an item type is in public preview.

> [!NOTE]
> For the current per-item-type CI/CD support matrix, see the supported items lists for [Git integration](./git-integration/intro-to-git-integration.md#supported-items) and [deployment pipelines](./deployment-pipelines/intro-to-deployment-pipelines.md#supported-items).

### Auto-binding support

Most Fabric workspace item types can self-manage their relations to other items in the same workspace. *Auto-binding* is the term for this behavior: as part of the Git synchronization process, item types that support auto-binding rebind their dependencies automatically. Item types that don't support auto-binding require your attention — you might need to run a script after **Update from Git** to correctly rebind item dependencies.

Consider a Fabric solution composed of a lakehouse, a notebook, and a pipeline, where the notebook depends on the lakehouse and the pipeline depends on both the lakehouse and the notebook. After you build the solution in the dev workspace, three dependencies exist between these items.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices34.png" alt-text="This is a diagram of a Fabric solution where a notebook and pipeline depend on a lakehouse in the dev workspace." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices34.png":::

Now think about what happens when you use Git synchronization to replicate these three items from the dev workspace to a feature workspace. The outcome you want to avoid is workspace items in the feature workspace that retain dependencies pointing back to items in the dev workspace. Instead, you want each item to resolve its dependencies to the related items in the *same* workspace. With auto-binding, the notebook and pipeline in the feature workspace properly rebind to siblings in their new workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices35.png" alt-text="This is a diagram of auto-binding resolving notebook and pipeline dependencies to related items in a feature workspace." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices35.png":::

Auto-binding maturity varies. Fabric notebooks didn't support auto-binding until the Notebook item type was updated with auto-binding support in March 2026. Notebooks also don't support auto-binding by default; you must extend a notebook's item definition with a `notebook-settings.json` file.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices39.png" alt-text="This is a screenshot of a notebook-settings.json file configured to enable auto-binding for a Fabric notebook." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices39.png":::

If you build a Fabric solution with an item type that lacks auto-binding support, plan to write a post-sync script that reestablishes the item's relations. (See [Identifying what to automate](#identifying-what-to-automate) later in this article.)

When auto-binding fails or item dependencies don't resolve as expected, see [Dependency errors](./git-integration/dependency-errors.md) for diagnostics and remediation.

### Item definitions

*Item definitions* are an essential component of Fabric's CI/CD support. An item definition is a named folder containing a set of files with the settings and metadata that represent an item of a specific workspace item type. The files in an item definition vary significantly from one item type to another, but the idea that every item can be serialized into a folder in Git is the foundation of the Fabric CI/CD infrastructure.

Every item definition requires a file named `.platform` — the *platform file*. The platform file contains metadata such as the item type and display name. It also contains a `logicalId` value that Fabric uses internally to track logical instances of workspace items as they propagate across Git branches and workspaces.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices36.png" alt-text="This is a screenshot of a Fabric platform file containing item type, display name, and logicalId metadata." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices36.png":::

In addition to the platform file, each workspace item type defines its own schema for the files required and allowed in an item definition. For example, the item definition for a lakehouse contains files such as `alm.settings.json`, `lakehouse.metadata.json`, and `shortcuts.metadata.json`.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices37.png" alt-text="This is a screenshot of a lakehouse item definition folder containing platform, alm settings, lakehouse metadata, and shortcut metadata files." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices37.png":::

The minimal item definition for a notebook requires only a `notebook-contents.py` file in addition to the platform file.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices38.png" alt-text="This is a screenshot of a minimal notebook item definition folder containing the platform file and notebook contents file." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices38.png":::

> [!NOTE]
> Fabric allows creating the `notebook-contents` file with a `.ipynb` extension to support the Jupiter notebook format. Fabric additionally supports creating the `notebook-contents` file using extension such as `.sql`, `.scala` and `.r` to support other programming languages. As discussed earlier, the item definition for a notebook requires a `notebook-settings.json` file to add support for auto-binding.

Some item definitions contain a small number of files; others don't. The item definition for a semantic model can include dozens or even hundreds of TMDL files structured across several folders. While a large file count adds complexity, it enables much more granular collaboration: splitting each table into its own file lets one developer work on measures in the **Sales** table while another developer changes the **Calendar** table — essential when multiple developers are working on a large semantic model concurrently.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices40.png" alt-text="This is a screenshot of a semantic model item definition with TMDL files organized across several folders." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices40.png":::

In this section, you've seen examples of item definitions for lakehouses, notebooks, and semantic models. You'll work with other workspace item types as well. For each, build a basic understanding of the item definition files. An easy way to start is to walk through the item definition files in Git that have been created by Git synchronization for an item type you're working with.

For the full item-definition file format and the source-code conventions Fabric uses, see [Item source code format](./git-integration/source-code-format.md).

#### The role of the logicalId

Using Fabric Git integration, you can replicate a set of workspace items from one workspace to another. Imagine you created a lakehouse named `sales` and used Git synchronization to replicate the lakehouse across three workspaces. All three lakehouse instances share the same type and display name, but each is assigned a unique ID.

To support auto-binding, Fabric needs a way to track that these three lakehouse instances are all associated with a single logical item. That's where the `logicalId` comes in. While each lakehouse instance has its own unique ID, all three lakehouses share the same `logicalId`.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices41.png" alt-text="This is a diagram of three replicated lakehouse instances sharing the same logicalId across different workspaces." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices41.png":::

The `logicalId` matters because it allows Fabric to avoid persisting workspace-specific item IDs into Git. This level of indirection is what makes auto-binding possible. Examine the following snippet from the item definition for a pipeline that has a dependency on a lakehouse in the same workspace:

```json
{
  "typeProperties": {
    "linkedService": {
      "name": "sales_lakehouse",
      "properties": {
        "typeProperties": {
          "workspaceId": "00000000-0000-0000-0000-000000000000",
          "artifactId": "<lakehouse-logicalId>",
          "rootFolder": "Files"
        }
      }
    }
  }
}
```

The `workspaceId` is the empty GUID `00000000-0000-0000-0000-000000000000`, which signals that `artifactId` is the `logicalId` of the lakehouse, not a workspace-specific item ID. When you run **Update from Git** on a new workspace, Fabric uses the `logicalId` to look up the workspace-specific lakehouse ID and rebind the pipeline to the lakehouse to reestablish the relation.

It's unlikely you'll ever need to work directly with the `logicalId` — treat it as an internal property used by Fabric behind the scenes. Never modify a `logicalId` value: doing so is unsupported and can lead to unpredictable behavior. For more on logical IDs and the conflicts they can resolve, see [Logical ID conflicts](./git-integration/logical-id-conflict-resolution.md) and [Resolve conflicts](./git-integration/conflict-resolution.md).

### Deployment pipelines

The Fabric platform provides Deployment Pipelines as a continuous deployment mechanism to help manage the lifecycle of workspace items. The key concept behind Deployment Pipelines is that changes to workspace items can be deployed through a set of stages where each stage is assigned to a Fabric workspace associated with an environment. The following screenshot shows the user interface of a Deployment Pipeline which makes it possible to deploy item-level changes across workspaces.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices43.png" alt-text="This is a screenshot of a Fabric deployment pipeline showing stages used to deploy item changes across workspaces." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices43.png":::

Deployment Pipelines are designed to enhance productivity by hiding low-level details associated with propagating changes through a sequence of workspaces. Deployment Pipelines are similar to GIT synchronization in that they automatically rebind item relationships when deploying a Fabric solution with item dependencies between lakehouses, notebooks and pipelines.

Deployment Pipelines were originally introduced in the Power BI service several years before Microsoft first released Fabric. In their initial release, deployment pipelines offered deployment rules as a mechanism used to support parameterization. For example, you can configure a semantic model with a deployment rule to update its datasource location so the semantic model in each of the three workspaces connects to its own environment-specific database.

Deployment Pipelines in Fabric continue to support deployment rules. However, deployment rules should not be your first choice when it comes to parameterization. Instead, you should prefer using variable libraries to solve parameterization problems. The key point is that variable libraries are strategic in the Fabric roadmap while deployment rules are not.

Deployment Pipelines are designed to assist with continuous deployment in building a release process. Keep in mind that the release process only covers the second half of full CI/CD lifecycle. The use of Deployment Pipelines should be combined together with Fabric GIT integration to fully implement a complete end-to-end solution. This topic will be revisited in the upcoming section on building a release pipeline.

## Automation in Fabric CI/CD

Rolling out a Fabric project with end-to-end CI/CD support requires a significant number of steps, so your ability to automate the setup process is essential. The alternative — known as *ClickOps* — involves completing setup tasks by hand using a browser. Relying on ClickOps is bad because it's prone to human error, causing unnecessary delays and the need to troubleshoot configuration errors. It's especially harmful when you're trying to create a fast and reliable plan for disaster recovery.

The Fabric platform provides public APIs that make it possible to automate setup and CI/CD processes for a Fabric project. The primary API used to create workspaces, connections, and workspace items is the Fabric REST APIs. While you can write code that calls the Fabric REST APIs directly, you can also boost your productivity by leveraging developer tools and libraries that call the Fabric REST APIs on your behalf. The folliwing diagram shows the developer tools and libraries discussed in this article.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices44.png" alt-text="This is a diagram of Terraform, Fabric CLI, and Semantic Link Labs calling Fabric REST APIs for automation." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices44.png":::


This section starts by examining the most common scenarios where you need to write automation logic. Then it walks through three of the most useful developer tools and libraries: **Terraform** for infrastructure-as-code provisioning, the **Fabric CLI** for general-purpose automation logic, and **Semantic Link Labs** for in-Fabric scripting. After that, this section explains why using the `fabric-cicd` library has become adopted as a best practice for building a release process. The sections concludes with a primer on Fabric REST APIs programming for developers that need the greatest level of control with Fabric automation

### Identifying what to automate

The first part of your journey to roll out a Fabric CI/CD project is setting up the project's infrastructure. The infrastructure for a Fabric CI/CD project includes tenant-scoped items in Fabric such as workspaces, capacities, connections, gateways, and deployment pipelines. You also need to create a Git repository and configure Git integration support between Fabric workspaces and Git branches.

After a Fabric CI/CD project is up and running, there is often an ongoing need to create and configure feature workspaces. The flow for creating a feature workspace involves creating a new feature branch from the integration branch and then running **Update from Git** against a new workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices45.png" alt-text="This is a diagram of creating a feature branch and running Update from Git to initialize a feature workspace." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices45.png":::

The problem is that **Update from Git** doesn't go far enough. It creates a lakehouse, but it doesn't automatically populate the lakehouse with tables. It creates a semantic model, but it doesn't automatically create and bind a connection to the semantic model's datasource.

Additional automation is often required after a workspace has been initialized or updated with **Update from Git**. As you begin developing these types of scripts, consider two different types of jobs:

- **Post-deploy jobs** run only once after an empty workspace is first initialized with **Update from Git**.
- **Post-sync jobs** run each time after **Update from Git** completes on a workspace.

A post-deploy job is a set of one-time tasks that are part of the workspace initialization process. Common tasks automated in a post-deploy job include setting the active value set for a variable library, running a notebook to populate a lakehouse with data, and creating and binding a connection for each semantic model. If your Fabric solution includes a semantic model built using **Import**, you should also automate the process of running a refresh operation to ingest and store the imported data.

> [!IMPORTANT]
> Avoid putting ETL logic directly into the scripts you write for post-deploy jobs. Instead, factor out ETL logic into workspace items designed for ETL purposes, such as notebooks, pipelines, copy jobs, user-defined functions, and dataflows. If a post-deploy script discovers a notebook by name and runs it, you can update the ETL logic in the notebook without requiring any updates to the script.

A post-sync job contains logic that must run every time after **Update from Git** completes. Consider the common scenario in which you need to automate keeping the dev workspce in sync with the intergation bracnch. The development process flow begins with a developer making a series of changes to items in a feature workspace and committing those changes to a feature branch. After that, a pull request is created and approved to merge those changes into the integration branch.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices46.png" alt-text="This is a diagram of a post-sync workflow triggered by a merged pull request to update the dev workspace from Git." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices46.png":::

When changes are merged to the intergation branch, you need to automate synchronizing the dev workspace with those changes. You can start by developing a workflow that is triggered whenever changes are merged into the integration branch. This workflow should start by running an **Update from GIT operation** to sync changes from the integration branch to the dev workspace. However, some workspace items might require additional modification after the **Update from GIT** operation completes. This can be the case when you are working with workspace items that do not support auto-bind behavior.  It can also be the case when you re working with items which do not yet support variable libraries yet require environment-specific settings.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices46-2.png" alt-text="This is a diagram of a post-sync workflow triggered by a merged pull request to update the dev workspace from Git." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices46-2.png":::

> [!IMPORTANT]
> Regardless of which tool you use or whether you call Fabric APIs directly, you must consider the identity used to run your automation jobs. Every call to a Fabric REST API executes under the identity of a specific Microsoft Entra security principal. Fabric REST APIs support two types of identity: user principals and service principals. Execute API calls as a service principal whenever possible. This is especially important when CI/CD workflow automation logic executes in a cloud-based platform such as Azure Pipelines or GitHub Actions.

### Creating project infrastructure using Terraform

Terraform is an open-source tool built on the principles of *infrastructure as code* (IaC). IaC defines the infrastructure for a software project using configuration files instead of manual processes or procedural programming logic. For organizations managing cloud deployments, the use of IaC is a well-established best practice in DevOps and CI/CD.

Terraform offers the advantage of a deployment process that is both versionable and repeatable. You can version a Terraform configuration by adding its files to a Git repository, just like you'd version any source file. A Terraform deployment process is repeatable — you can run it again and again and it produces the same outcome. A repeatable process provides consistency and the fastest path to disaster recovery.

Architecturally, Terraform plays the role of an orchestrator that completes its work by calling APIs behind the scenes. Terraform doesn't call those APIs directly; instead, it delegates to *Terraform providers*, plug-in components designed to interact with a specific set of cloud-based APIs. The Terraform registry contains thousands of providers covering platforms such as Azure, AWS, and Google Cloud. Microsoft released the Fabric provider for Terraform in 2025, making it possible to set up the infrastructure for a Fabric CI/CD project using an IaC-based provisioning process.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices47.png" alt-text="This is a diagram of Terraform orchestrating infrastructure provisioning through Azure and Fabric providers." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices47.png":::

For more on Terraform, see the [Terraform documentation](https://developer.hashicorp.com/terraform/docs).

A Terraform configuration is defined using a set of files that contain HashiCorp Configuration Language (HCL). A typical project layout uses files such as **`providers.tf`**, **`variables.tf`**, **`main.tf`**, **`output.tf`**, and **`terraform.tfvars`**. The **`variables.tf`** file defines a set of typed variables. The **`terraform.tfvars`** file assigns variable values when Terraform is running locally in a client tool such as Visual Studio Code. Because **`terraform.tfvars`** often contains sensitive information such as a client secret used to authenticate as a service principal, add a **`.gitignore`** entry so that **`terraform.tfvars`** is never uploaded to a Git repository.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices48.png" alt-text="This is a screenshot of a Terraform project folder with providers, variables, main, output, tfvars, and gitignore files." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices48.png":::

The **`providers.tf`** file contains a **`terraform`** block with **`required_version`** and **`required_providers`** arguments. The **`required_providers`** block specifies each provider with its source and version. The same file contains a **`provider`** block that authenticates the Fabric provider as a service principal:

```terraform
terraform {
  required_version = ">= 1.8, < 2.0"
  required_providers {
    fabric = {
      source  = "microsoft/fabric"
      version = "1.8.0"
    }
  }
}

provider "fabric" {
  tenant_id     = var.tenant_id
  client_id     = var.client_id
  client_secret = var.client_secret
}
```

Once the configuration contains what's required to initialize the provider, run **`terraform init`** to download the executable code for the provider along with security configuration used to verify the downloaded provider code hasn't been tampered with.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices49.png" alt-text="This is a screenshot of a terminal showing terraform init downloading and installing the Microsoft Fabric provider." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices49.png":::

After initializing the Fabric provider, the next step is to create resources by adding **`resource`** blocks to **`main.tf`**. Each **`resource`** block requires a provider-specific resource type and a local resource name. The Fabric provider offers the **`fabric_workspace`** resource type for creating workspaces and the **`fabric_workspace_role_assignment`** resource type for configuring access through workspace role assignments:

```terraform
resource "fabric_workspace" "workspace_main" {
  display_name = var.workspace_name
  capacity_id  = var.capacity_id
}
resource "fabric_workspace_role_assignment" "admin_group" {
  workspace_id = fabric_workspace.workspace_main.id
  principal = {
    id   = var.admin_group
    type = "Group"
  }
  role = "Admin"
}

resource "fabric_workspace_role_assignment" "dev_group" {
  workspace_id = fabric_workspace.workspace_main.id
  principal = {
    id   = var.dev_group
    type = "Group"
  }
  role = "Member"
}
```

When you create a **`fabric_workspace`** resource, specify argument values for **`display_name`** and **`capacity_id`**. The **`fabric_workspace`** resource is also assigned a local name (**`workspace_main`** in the previous listing). The local name matters because it lets you create references using syntax such as **`fabric_workspace.workspace_main.id`**.

Once you have a configuration with a set of resources, run **`terraform plan`** to review a list of the steps required to deploy the configuration. **`terraform plan`** doesn't make any changes; it just shows the planned steps. When you're ready to deploy, run **`terraform apply`** to execute the job that creates the resources needed to match your configuration.

The first time you run **`terraform apply`**, Terraform creates a state file named **`terraform.tfstate`** that tracks the property values for each resource as it's created or updated. By default, Terraform creates **`terraform.tfstate`** locally in the root folder of the current configuration, which is convenient for learning and local development. In production, it's critical to initialize the configuration to manage the state file in cloud storage such as an Azure Storage account. The state file often contains sensitive data such as authentication credentials, so limit access to the state file and encrypt its contents.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices50.png" alt-text="This is a screenshot of a terraform.tfstate file tracking Fabric workspace properties such as id, capacity id, and endpoints." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices50.png":::

A configuration commonly contains an **`output.tf`** file with **`output`** blocks that capture property values for resources created by the configuration. The following listing shows two output blocks that provide values for a workspace ID and DFS endpoint:

```terraform
output "workspace_id" {
  description = "Id of Fabric workspace"
  value       = fabric_workspace.workspace_main.id
}

output "workspace_dfs_endpoint" {
  description = "DFS endpoint for Fabric workspace"
  value       = fabric_workspace.workspace_main.onelake_endpoints.dfs_endpoint
}
```

By tracking the state of resources in the current deployment, Terraform compares what's in the configuration against what's been deployed. This lets Terraform determine what resources need to be created, updated, or destroyed each time you run `terraform apply`. Running **`terraform apply`** a second time without any configuration changes does nothing — Terraform determines the state file matches the current deployment.

Terraform also plays a valuable role in managing resources over the lifetime of a project. To update an existing resource, change the configuration and run **`terraform apply`** again. For example, to change the workspace role assignment with the local name `dev_group` from **Member** to **Contributor**, update the resource argument, save your changes, and run `terraform apply`. You don't have to worry about the sequence of API calls required to make the change — Terraform handles that.

Fabric capacities are an Azure resource and must be created with the Terraform provider for Azure. You can authenticate the Azure provider as a service principal in a similar way to the Fabric provider, but you also need to include the **`subscription_id`** for the subscription used to provision the capacity. The service principal must have the appropriate permissions in that subscription to create a Fabric capacity:

```terraform
terraform {
  required_version = ">= 1.8, < 2.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.63.0"
    }
  }
}

provider "azurerm" {
  features        {}
  tenant_id       = var.tenant_id
  client_id       = var.client_id
  client_secret   = var.client_secret
  subscription_id = var.subscription_id
}
```

The following listing demonstrates how to create a Fabric capacity. First create an Azure resource group using an **`azurerm_resource_group`** resource, then create a capacity in that resource group using an **`azurerm_fabric_capacity`** resource:

```terraform
data "azurerm_client_config" "current" {}

locals {
  spn_object_id = data.azurerm_client_config.current.object_id
}

resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.capacity_location
}

resource "azurerm_fabric_capacity" "main" {
  name                = var.capacity_name
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  administration_members = [
    local.spn_object_id,
    var.admin_user_upn
  ]
  sku {
    name = var.capacity_sku_size
    tier = "Fabric"
  }
}
```

In addition to **`resource`** blocks, Terraform supports **`data`** blocks for scenarios where you just need to query a datasource. The previous listing uses a **`data`** block based on the **`azurerm_client_config`** datasource to dynamically retrieve information about the service principal used to run the Terraform deployment. The **`locals`** block creates a local variable named **`spn_object_id`** holding the service principal object ID, which is then used in the **`azurerm_fabric_capacity`** resource to add the service principal to the capacity's administrators group.

When you configure an **`azurerm_fabric_capacity`** resource, configure a **`sku`** block with a **`name`** argument that controls the SKU size. For example, you might provision the capacities for your project environments with an **F4** for dev, an **F16** for test, and an **F64** for prod. If, a few months later, you determine the test environment capacity isn't powerful enough and needs to be bumped to **F32**, update the test configuration with the new SKU and run **`terraform apply`**.

Now consider creating a Fabric capacity and a workspace in the same configuration. You need to load both the Azure provider and the Fabric provider. One detail is tricky: the Azure provider and the Fabric provider use different formats for the capacity ID. Fabric uses a GUID-based ID for capacities while Azure uses a long string identifier containing the subscription ID and resource group name. The best way to handle this is to add a `data "fabric_capacity"` block to query the capacity for its GUID-based ID:

```terraform
data "fabric_capacity" "main" {
  display_name = var.capacity_name
  depends_on   = [azurerm_fabric_capacity.main]
}

resource "fabric_workspace" "main" {
  display_name = var.workspace_name
  capacity_id  = data.fabric_capacity.main.id
  depends_on   = [data.fabric_capacity.main]
}
```

A Terraform configuration for building Fabric CI/CD project environments typically includes workspaces, capacities and connections, but you can also provision other Azure resources such as Azure Storage accounts, Azure SQL databases, or Azure Key Vault instances. The following diagram shows an example of a Fabric CI/CD project plan which includes a standard set of resources required by three environments. Each environment contains the same set of resource types yet parameterization can give each environment its own unique resource settings.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices51.png" alt-text="This is a diagram of dev, test, and prod Terraform configurations provisioning capacities, workspaces, storage accounts, and Fabric connections." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices51.png":::

When using Terraform, it's recommended that you create a separate configuration for each environment. After all, you don't want a problem in the configuration for the dev environment or the test environment to affect the prod environment. You can create configurations for these environments using a common provisioning flow to create resources in the following sequence.

- **`azurerm_fabric_capacity`**: Fabric capacity
- **`fabric_workspace`**: workspace assigned to the capacity
- **`fabric_workspace_role_assignment`**: workspace role assignments for access control
- **`azurerm_storage_account`**: Azure Storage account
- **`azurerm_storage_container`**: container in the Storage account for environment-specific data files
- **`azurerm_storage_account_blob_container_sas`**: SAS token credential to access the Storage container
- **`fabric_connection`**: Fabric connection used to access data files in the Storage account
- **`fabric_connection_role_assignment`**: connection role assignment to configure connection permissions

For more on Terraform providers, see the [Microsoft Fabric Terraform provider](https://registry.terraform.io/providers/microsoft/fabric/latest/docs)and the [Azure provider documentation](/azure/developer/terraform/).

> [!IMPORTANT]
> This example demonstrates how Terraform helps to enforce best practices with respect to security and governance. You have seen how a Terraform configuration can manage access control to project resources by adding role assignments. If anything changes in the future with respect to permissions or access control, you can simply update the configuration and then run the **`terraform apply`** command.
>
> The example also demonstrates a security best practice for credential management. The Terraform configuration creates a credential in the form of a SAS token and then uses that credential to create a Fabric connection. This is a best practice because it eliminates any need to share credentials with other humans or external processes. The Terraform configuration manages credentials behind the scenes in a state file that is locked down and encrypted in cloud storage. Terraform creates the connection, configures its permissions and pass the connection id back to you as output.

### Fabric CLI

The *Fabric CLI* is a developer tool that offers the easiest path to writing automation logic that completes tasks in Fabric for DevOps and CI/CD. The Fabric CLI acts as a façade that abstracts away the underlying Microsoft APIs to provide a single, focused experience for Fabric automation. The Fabric CLI adds value by shielding you from Microsoft Entra authentication, access token acquisition, and the dispatch of HTTP requests. It also simplifies running automation logic under the identity of a service principal.

The Fabric CLI supports both an *interactive mode* and a *scripting mode*. Interactive mode makes it quick and easy to type and execute a sequence of commands. For example, you can use the Fabric CLI in interactive mode to create a new workspace and then create a lakehouse inside that workspace. The Fabric CLI also provides commands to load data into lakehouse tables and to automate running notebooks and pipelines that contain ETL logic.

Scripting mode makes it possible to write and version automation logic for a Fabric CI/CD project. Use a shell script or a programming language such as PowerShell or Python to execute a sequence of Fabric CLI commands. If you decide not to use Terraform, you can write scripts that call Fabric CLI commands to create and manage tenant-level items such as workspaces, capacities, connections, domains, and gateways.

The Fabric CLI offers rich support for creating and managing workspace items. The CLI provides commands such as **`create`**, **`get`**, **`set`**, and **`rm`** to perform CRUD operations on workspace items. There are also two Fabric CLI commands for working with item definitions: **`import`** creates or updates a workspace item using an item definition, and the complementary **`export`** command exports a workspace item to a local folder as a set of definition files.

For more information, see the [Fabric CLI documentation](/fabric/cicd/cli/).

> [!NOTE]
> Another important capability of the Fabric CLI is its integration with cloud-based Git providers such as Azure DevOps and GitHub. If you're developing GitHub Actions workflows, it's relatively straightforward to install and load the Fabric CLI executable so your workflows can call Fabric CLI commands. If you're developing Azure Pipelines workflows, the **Fabric automation tools** extension for Azure DevOps adds the **`FabricCLITask@0`** task that provisions the Fabric CLI into the pipeline agent without manual installation. For more information, see [Fabric automation tools for Azure DevOps](https://marketplace.visualstudio.com/items?itemName=ms-fabric-api.fabric-automation-tools).

### Semantic Link Labs

*Semantic Link* is a capability in Microsoft Fabric that bridges the gap between data science and data analytics by connecting Power BI semantic models with Python and Spark environments. Semantic Link lets data scientists load data from a semantic model into a **`FabricDataFrame`** object that can be used directly with data science libraries such as *pandas* and *scikit-learn*.

*Semantic Link Labs* is a Python library designed for use in Fabric notebooks. It provides access to Semantic Link for developers working in Fabric, but it goes far beyond wrapping Semantic Link: it adds significant functionality for working with workspace items such as semantic models, reports, lakehouses, notebooks, and variable libraries. The goal of Semantic Link Labs is to simplify technical processes so developers can focus on higher-level activities.

Behind the scenes, Semantic Link Labs completes its work by calling public Microsoft APIs such as the Fabric REST APIs, Power BI REST APIs, and the Tabular Object Model. Developers using Semantic Link Labs benefit from the simplicity of a single API surface without having to think about the details of acquiring access tokens or dispatching calls to multiple APIs.

Semantic Link Labs lets you work with workspace items from the Data Engineering workload, such as lakehouses, notebooks, and environments. It simplifies creating lakehouses and notebooks, configuring a notebook to use a specific lakehouse as its default lakehouse, running jobs for notebooks and pipelines, and monitoring job execution to determine whether a job completed successfully.

For more on Semantic Link Labs, see the [Semantic Link Labs project on GitHub](https://github.com/microsoft/semantic-link-labs).

> [!NOTE]
> If you plan to use Semantic Link Labs, keep in mind that it works only inside the Fabric environment. You can't load the Semantic Link Labs library in a non-Fabric environment such as Azure Pipelines or GitHub Actions workflows. To use Semantic Link Labs functionality from an external automation host, maintain the code that calls Semantic Link Labs in a Fabric notebook, and automate running that notebook from your external host.

## Building a release process

Earlier you learned the fundamentals of building a development process using feature branches and feature workspaces. The goal of the development process is to create a single source of truth in the integration branch that is always ready for deployment. Now it's time to turn your attention to what comes next. You must build a release process to complete the full CI/CD lifecycle. The question now becomes how to deploy a Fabric solution to workspaces in environments for testing and production.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices52.png" alt-text="This is a diagram of a Fabric release process deploying solution updates from development into test and production environments." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices52.png":::

The first and easiest option for building a release process is using a deployment pipeline. Deployment pipelines offer a low-code approach that can be a good fit for small to medium-sized projects, especially those exclusively focused on semantic models and Power BI reports.

As mentioned earlier, deployment pipelines assist with building a release process and not a development process. When you choose to use a deployment pipeline in a Fabric CI/CD project, you still need to plan how to build a development process. At a minimum, you should connect the first workspace in a deployment pipeline to a GIT branch just to gain the basic CI/CD capability such as versioning items and reverting to an earlier versions of an item if something goes wrong with an update.

The following diagram shows a deployment pipeline with the first workspace connected to the **main** branch in a GIT repository. In the simplest development scenario you can make direct updates to items in the dev workspace and then use **Commit to GIT** operations to commit changes to GIT where they can be used to revert to earlier item versions when necessary.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices53.png" alt-text="This is a diagram of a Fabric deployment pipeline with three stages assigned to three different workspaces. The first workspace is also connected to GIT." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices53.png":::

If you decide to build the release process using a deployment pipeline, you still have the ability to design a scalable development process using feature branches and feature workspaces. As an example, you can use the **main** branch as the integration branch for your development process. This makes it possible to create new feature workspaces by using the **Branch out to workspace** command to branch out from the dev workspace. As changes are committed and merged to the **main** branch using a pull request, the changes can be synchronized to items in the dev workspace using an **Update from GIT** operation. Once item-level changes have propagated to the dev workspace, they can be deployed across the other workspaces in the deployment pipeline using the **Deploy** command.

While deployment pipelines provides the easiest path to build a release process, they also have noteworthy limitations which limits their use in certain scenarios. For example, a deployment pipeline and all of its associated workspaces must exist inside the same Entra Id tenant. If you have a requirement create the dev workspace in one Entra Id tenant and the prod workspace in another, you cannot use deployment pipelines to build your release process.

There are some other factors to consider as well when deciding whether to use deployment pipelines. Setting up a deployment pipeline always requires some degree of manual configuration because certain setup tasks are not supported through public APIs. Deployment pipelines are also limited when it comes to setting up manual approval processes. This is especially true when compared to what's possible with pull requests and approval gates in a GIT repository. Moreover, deployment pipelines are not well suited to handle larger projects where there are workspaces containing 100s of items.

A second option for building a release process using GIT synchronization. For example, you can use a GitFlow branching strategy in which a GIT repository is configured with three long-lived branches named **dev**, **test** and **main**. The **dev** branch serves as the integration with code ready for deployment. Pull requests are used push changes to the **test** branch and to the **main** branch which act as dedicated release branches. You can then use an **Update from GIT operation** to deploy changes from a release branch to its target workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices54.png" alt-text="This is a diagram of a Fabric release process built using GIT synchronization." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices54.png":::

There are two important issues to consider when deciding whether to build a release process using GIT synchronization. First, there are issues caused by environment-specific settings that have been committed to GIT. For example, the datasource location for semantic models in the dev workspace will be committed to GIT and then propagated to semantic models in the test workspace and the prod workspace. That means you need to run a post-sync job after an **Update from GIT** operation completes to update each semantic model with the datasource path that is appropriate for its target environment.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices55.png" alt-text="This is a diagram of a Fabric deployment pipeline with stages assigned to three different workspaces. The first workspace is also connected to GIT." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices55.png":::

A second issue with GIT synchronization is that it requires a connection between the target workspace and a GIT branch. When a workspace is connected to a GIT branch, the Fabric user interface lights up the workspace with indicators and messages to control and monitor GIT synchronization. For example, the workspace summary page displays the **Status** column with values such as **Synced** or **Update required** which you might prefer to hide from users in a production workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices56.png" alt-text="This is a diagram of a Fabric deployment pipeline with stages assigned to three different workspaces. The first workspace is also connected to GIT." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices56.png":::

The third option for building a release process is using an *API-driven approach*. Using an API-driven approach has become increasingly popular because it offers significant advantages over the other two choices. For example, API-driven approach is much better from a performance perspective because it can scale to accommodate larger project in which workspaces contains 100s of items.

Using an API-driven release process also provides the flexibility to use whatever GIT branching strategy your organization prefers. Some organizations prefer using a GitFlow branching strategy while other organizations have standardized on trunk-based development. Once you've chosen a branching strategy, you can then adapt an API-driven release process to accommodate it.

> [!IMPORTANT]
> In theory, you could spend the time and energy required to implement a release process using the Fabric REST APIs. However, this would require a non-trivial coding effort. Fortunately, there's a much better option to build a release process which is leveraging the **fabric-cicd** library. You will learn about the **fabric-cicd** library in greater depth in the next section. Before that, we'll examine how the **fabric-cicd** library can be used with popular branching strategies.

Consider the release process shown in the following screenshot which uses a GitFlow branching strategy. The **dev** branch serves as integration branch. This release process uses pull requests to push changes to **test** and **main** which serve as release branches. The use of pull requests makes it possible to configure manual approval processes and to trigger workflows that run when a pull request completes. When a pull request completes and merges its changes to one of the release branches, a workflow runs automatically and implements the release process by using **fabric-cicd** to deploy changes to the target workspace.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices57.png" alt-text="This is a diagram of a GitFlow release process using pull requests and fabric-cicd deployments to test and production workspaces." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices57.png":::

This example is similar to Git synchronization example shown earlier in the sense that they both use the GitFlow branching strategy. However, there are significant benefits in using **fabric-cicd** over GIT synchronization when building a release process. The first benefit is that **fabric-cicd** supports parameterization which makes it possible to dynamically update environment-specific settings as part of the deployment process. Compare this to GIT synchronization which pushes these types of environment-specific settings to items in the target workspace which then requires additional automation after an **Update from GIT** operation completes to apply a fix.

Now let's examine building a release process using a different branching strategy. Consider a scenario in which an organization has standardized on trunk-based development using a branching strategy based on a single long-lived branch. The following diagram show a release process in which the **main** branch acts as both the integration branch and the release branch. The **fabric-cicd** library provides the flexibility of building a release process with a one-to-many mapping between a release branch and multiple target workspaces.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices58.png" alt-text="This is a diagram of trunk-based development using one release branch to deploy to multiple target workspaces." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices58.png":::

> [!IMPORTANT]
> When building a release process, you are often required to configure a manual approval process in which one or more approvers need to sign off on a release before it's deployed to a target workspace. When using a GitFlow branching strategy, it's easy because you can configure pull requests with whatever manual approval process is required. However, things are not as obvious when using trunk-based development because the release process doesn't involve pull requests.
>
> Fortunately, Azure DevOps and GitHub provide another way to configure a manual approval process without using pull requests. The way to accomplish this is by extending a GIT repository by creating *environments*. An environment in a GIT repository acts as a deployment target which can be configured with a manual approval process. When you run a workflow that targets an environment which has been configured with a manual approval process, the workflow is paused until the approval process completes. Once the approval is complete, the workflow resumes and can use the fabric-cicd library to deploy the new release to the target workspace.

### The fabric-cicd library

***fabric-cicd*** is an open-source Python library designed by Microsoft to enable code-first deployment. Code-first deployment is based on the idea that you can create the single source of truth for a Fabric solution by assembling a set of item definitions. Once you've assembled the item definitions for a Fabric solution in a root folder, you can then use these item definitions as templates to deploy a matching set of items to a target workspace.

> [!IMPORTANT]
> While **fabric-cicd** is an open source project, it's officially supported and recommended as a best practice by the Fabric product team. You can have confidence that **fabric-cicd** will continue to evolve along with the Fabric platform.

A stated goal of the **fabric-cicd** project is to assist CI/CD developers who prefer not to interact directly with the Fabric REST APIs. You will find that running a deployment job with fabric-cicd only requires a few lines of code. However, the logic this library provides is quite mature as its been continually refined over the last few years. Let's walk through a few examples of how **fabric-cicd** works internally to give you an appreciation for how much work it does on your behalf.

To run a deployment job using **fabric-cicd**, you must supply two important input parameters. The first parameter points to a root folder containing items definitions. The second parameter references a target workspace. When you start a deployment job, **fabric-cicd** starts by enumerating through the root folder to discover every item definition including those item definition in child folders. For each item definition that **fabric-cicd** discovers, it must run a check on the target workspace to determine whether a matching items already exists. This check is required so **fabric-cicd** can decide whether to create a new item or to update an existing item.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices59.png" alt-text="This is a diagram of fabric-cicd reading item definitions from a repository folder and deploying them to a target Fabric workspace." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices59.png":::

**fabric-cicd** is able to automatically reestablish relationships for items that support auto-binding. The **fabric-cicd** library implements auto-binding behavior in a manner similar to GIT synchronization by using the **`logicalId`** found in the platform file of an item definition. When a **fabric-cicd** deployment job runs, it uses the **`logicalId`** to map out and rebind item dependencies between items in the target workspace.

When **fabric-cicd** runs a deployment job, it processes items in a specific sequence to deal with item dependencies. After all, **fabric-cicd** cannot create an item with a dependency until after it has created the items on which it depends. As an example, notebooks can have dependencies on lakehouses. Pipelines can have dependencies on lakehouses and on notebooks. The **fabric-cicd** deals this dependency tree using a sequence which deploys lakehouses first followed by notebooks and then pipelines. The **fabric-cicd** library even includes logic to detect whether any pipelines have dependencies on other pipelines. The sequencing ensures that pipelines with dependencies are always deployed after other pipelines on which they depend.

Now that understand how **fabric-cicd** works, let's more ahead and walk through an example running a deployment job. You can start by adding a pair of YAML files to the same root folder which holds the item definitions for a Fabric solution. The first file named **`deploy.yml`** allows you to configure settings for **fabric-cicd** deployment jobs. The second file named **`parameter.yml`** is used to configure parameterization to deal with environment-specific setting that have been committed to GIT.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices60.png" alt-text="This is a screenshot of a repository folder containing Fabric item definitions with deploy.yml and parameter.yml configuration files." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices60.png":::

Let's walk through a simple example of configuration-based deployment using **fabric-cicd**. We'll start by examining the contents of a simple configuration file named **`deploy.yml`**. The YAML content contains a top-level key named **`core`** which has child keys named **`workspace_id`**, **`repository_directory`**, **`item_types_in_scope`** and **`parameter`**. Underneath the **`workspace_id`** key, there are two subkeys named **`test`** and **`prod`** which define target environments for a deployment job.

```yaml
core:
  workspace_id:
    test: "11111111-1111-1111-1111-111111111111"
    prod: "22222222-2222-2222-2222-222222222222"

  repository_directory: "/workspace"

  item_types_in_scope:
    - VariableLibrary
    - Lakehouse
    - Notebook
    - DataPipeline
    - SemanticModel
    - Report

  parameter: "parameter.yml"
```

*Targets environments* are an important concept when using **fabric-cicd**. The configuration shown in this example defines two target environments named **`test`** and **`prod`**. The configuration maps each environment to a target workspace using a unique **`workspace_id`** value. Whenever you start a deployment job, you must pass a parameter indicating which environment you want to target. You will experience an error if you try to start a deployment job using an environment name hasn't been defined in **`deploy.yml`**.

**fabric-cicd** provides special treatment when deploying variable libraries. If your solution contains a variable library, **fabric_cicd** will always deploy it first before any other items that depend on it. After creating a variable library, **fabric-cicd** goes one step further making it possible to activate a specific value set. You should be aware that value set activation requires you use matching names between environment passed to **fabric-cicd** and the name of the value set to be activated. If you run a **fabric-cicd** deployment job with a target environment name such as **`test`** or **`prod`**, you must ensure the variable library also contains value sets with the matching names of **`test`** or **`prod`**.

Now let's walk through writing the Python code required to start a **fabric-cicd** deployment job. First, you must install the **fabric-cicd** library which is available in the Python Package Index (PyPI). That means you can install **fabric-cicd** using the **`pip install`** command.


```bash
pip install fabric-cicd
```

The **fabric-cicd** library is commonly used to build release processes in workflows that run in Azure DevOps or in GitHub. Fortunately, the ability to install on demand using **`pip install`** makes it easy to integrate **fabric-cicd** with Azure pipelines or GitHub Custom Actions.

The following Python code demonstrates how few lines of code it takes to run a deployment job with **fabric-cicd**. First, you need to create credentials to authenticate with an Entra Id security principal. This example demonstrates authenticating as a service principal using credentials stored in environment variables. You start a deployment job by calling the **`deploy_with_config`** function and passing parameter values for **`token_credential`**, **`config_file_path`** and **`environment`**.

```python
import os
from azure.identity import ClientSecretCredential
from fabric_cicd import deploy_with_config

credential = ClientSecretCredential(
    tenant_id=os.environ["AZURE_TENANT_ID"],
    client_id=os.environ["AZURE_CLIENT_ID"],
    client_secret=os.environ["AZURE_CLIENT_SECRET"],
)

deploy_with_config(
    token_credential=credential,
    config_file_path="./workspace/deploy.yml",
    environment="test",
)
```

When calling **`deploy_with_config`**, you must pass an **`environment`** value which maps to target environment defined in **`deploy.yml`**. The environment name determines the id of the target workspace. The ability to define multiple deployment targets in configuration is powerful because it allows you to build a release process with a one-to-many mapping between a single release branch and multiple target workspaces. The previous code listing demonstrates how easy it is to switch between deploying to the **test** workspace and the **prod** workspace. You just need to pass a different environment name when calling **`deploy_with_config`**.

Now it is time to examine the **fabric-cicd** support for parameterization which makes it possible to dynamically update settings in item definition files during a deployment job. This parameterization support is essential because it provides a reliable way to deal with environment-specific settings that have been committed to GIT.

Consider an example of a semantic model that connects to an Azure storage account but requires different datasource paths for the three environments **`dev`**, **`test`** and **`prod`**. The problem is that the datasource path for the semantic model in the dev workspace is committed to GIT in the integration branch. Parameterization is required to ensure that the semantic models in the **test** workspace and the **prod** workspace are updated with their environment-specific datasource paths.

The following YAML listing demonstrates configuring **`parameter.yml`** with a **`find-and-replace`** operation. There is a top-level key named **`find_replace`** with nested keys named **`find_value`** and **`replace_value`**. The **`find_value`** key is assigned a string value from the dev workspace that you want to replace. The **`replace_value`** key contains a nested key-value pair for each target environment. In this example, the configuration defines two target environments named **`test`** and **`prod`**.

```yaml
find_replace:
  - find_value: "https://storage4dev.dfs.core.windows.net/productsales/"
    replace_value:
      test: "https://storage4test.dfs.core.windows.net/productsales/"
      prod: "https://storage4prod.dfs.core.windows.net/productsales/"
```

By default, **fabric-cicd** processes a *find_replace* operation by inspecting every file in every item definition whose type is include in the **`item_type_in_scope`** property setting. This can lead to longer processing times in scenarios in which you have a large number of items or you have item definitions such as a semantic model with a large number files.

To optimize performance, **fabric-cicd** parameterization allows you to define filters to control which items and files are examined during a replacement operation. For example, you can extend the **`find_value`** key by adding filtering keys such as **`item_type`**, **`item_name`** and **`file_path`**. The following configuration demonstrates how to filter a **`find_replace`** operation to restrict processing to a single file named **`expressions.tmdl`** found in the item definition for a single semantic model identified by its display name.

```yaml
find_replace:
  - find_value: "https://storage4dev.dfs.core.windows.net/productsales/"
    replace_value:
      test: "https://storage4test.dfs.core.windows.net/productsales/"
      prod: "https://storage4prod.dfs.core.windows.net/productsales/"
    item_type: "SemanticModel"
    item_name: "Product Sales Imported Model"
    file_path: "/definition/expressions.tmdl"
```

The parameterization support in **fabric-cicd** goes beyond adding simple *find-and-replace* operations. **fabric-cicd** also supports parameterization using regular expressions and dynamic variables. In order to demonstrate these advanced parameterization capabilities, let's examine a common scenario in which a semantic model is designed to connect to the SQL endpoint of a lakehouse in the same workspace. This scenario requires parameterization because each semantic model requires a uniuqe datasource path to references the SQL endpoint for the lakehouse in the same workspace.

The item definition for the semantic model includes an **`expressions.tmdl`** file which connects to the SQL endpoint using a call to the PowerQuery function name **`SQL.Database`**. The **`SQL.Database`** function accepts two parameters. The first parameter is for the SQL endpoint server path and second parameter references a lakehouse.

```yml
Sql.Database(<SQL Endpoint Connect String>, "12341234-1234-1234-1234-123412341234")
```

Let's start by looking at the second parameter passed to **`SQL.Database`** which is used to identify the target lakehouse. For this lakehouse parameter, you can pass the either the lakehouse id or the lakehouse display name. Using the lakehouse id is problematic because it will always be different across envorinemnts requiring additional parmeterization. Instead, you should always pass the lakehouse name instead of the lakehouse id. That's because the lakehouse name will be the same across all environments eliminating the need for parametization.

```yml
Sql.Database(<SQL Endpoint Connect String>, "sales")
```

Now let's turn our attention to the first parameter passed to **`Sql.Database`**. This parameter is used to pass the SQL endpoint connect string which ends with **`.datawarehouse.fabric.microsoft.com`**. However, the first part of the SQL endpoint connection string is always unique to a specific workspace. This means the SQL endpoint connection string requires parameterization.

```yml
Sql.Database(4zzdkw4hunvuhp4ttpl5hvkkzm-6pexp5fkuvjedkrx773mxow6z4.datawarehouse.fabric.microsoft.com,
```

**fabric-cicd** supports parameterization using regular expressions to identity *capture zones* used in *find_replace operations*. The following regular expression demonstrates defining a capture zone for the SQL endpoint connection string.

```yml
Sql\.Database\(\s*"([^"]*datawarehouse\.fabric\.microsoft\.com[^"]*)"\s*,
```

Once you have created the regular expression with a capture zone to replace the SQL endpoint connection string, you can use it in a *find_replace* operation as long as you add an **`is_regex`** key with a value set to **`true`**.

```yml
find_replace:

    - find_value: 'Sql\.Database\(\s*"([^"]*datawarehouse\.fabric\.microsoft\.com[^"]*)"\s*,'
      replace_value:
        test: $items.Lakehouse.sales.$sqlendpoint
        prod: $items.Lakehouse.sales.$sqlendpoint
      is_regex: "true"
      item_type: ["SemanticModel"]
      file_path: "**/expressions.tmdl"

```

In addition to demonstrating the use of a regular expression with a capture zone, this example also uses a dynamic variable for the replace_value key. The dynamic variable is created to reference the lakehouse names sales using the syntax **`$items.Lakehouse.sales`**. In this example, the **`$sqlendpoint`** property of the dynamical lakehouse variable is used to provide the replacement value.

There is one final improvement that can be made to this example. In the previous listing, you can see that both environments use the same dynamic variable expression as the replace_value. In a scenario like this in which all environment keys have the same value, you can use the **`_ALL_`** key instead of adding multiple environment keys with redundant values.


```yml
find_replace:

    - find_value: 'Sql\.Database\(\s*"([^"]*datawarehouse\.fabric\.microsoft\.com[^"]*)"\s*,'
      replace_value:
        _ALL_: $items.Lakehouse.sales.$sqlendpoint
      is_regex: "true"
      item_type: ["SemanticModel"]
      file_path: "**/expressions.tmdl"
```

This example demonstrates the power of configuring parameterization using regular expressions, capture zones and dynamic variables. Keep in mind you can use dynamical variables to obtain the **`$id`** property for any type of workspace item. You can also use a dynamic **`$workspace`** variable which provides the **`$id`** property and the **`$name`** in cases in which you need to obtain the id or name of the current workspace.

> [!NOTE]
> To learn more about **fabric-cicd**, see [Home page](https://microsoft.github.io/fabric-cicd/0.1.7/), [Code Samples](https://microsoft.github.io/fabric-cicd/0.1.7/code_sample/), [Code Reference](https://microsoft.github.io/fabric-cicd/0.1.7/code_reference/) and [Parameterization](https://microsoft.github.io/fabric-cicd/0.1.33/how_to/)

### Data orchestration

Fabric provides rich support for building data-centric solutions. This support is built on top of one set of workspace item types that act as a data containers and a complimentary set of workspace item types which focus on running ETL processes and data integration. When designing a Fabric solution with a data container item such as a lakehouse, you must plan how to build and run a ETL logic to populate the lakehouse. You can choose between several different types of ETL-focused item types such as notebooks, pipelines, copy jobs, user-defined functions, dataflows and Spark job definitions.

It is a best practice to author and maintain the ETL logic for a Fabric solution using workspace items that are part of the same solution. When you build ETL logic using workspace items such as notebooks or pipelines, the ETL logic can be tracked in a GIT branch and versioned over time along with all other items in the Fabric solution. For this reason, it's recommended that you avoid designing Fabric solutions that depend on external ETL processes or scripts that are not part of the current solution.

You have several different options for deploying a Fabric solution with a lakehouse to a target workspace. One option is to deploy the Fabric solution using a deployment pipeline. A second option is is using an **Update from GIT** operation. A third option is using the **fabric-cicd** library. Whichever of these three deployment options you choose, the deployment result will always be the same. The lakehouse is created as an empty data container which requires your attention.

Let's walk through an example of data orchestration in a Fabric solution shown in the following diagram. This solution is designed with a medallion architecture in which data flows across three lakehouses which serve as storage for the bronze, silver and gold layers. The bronze lakehouse is designed with an ADLS Gen2 shortcut to enable access to data files in an Azure storage account container. There is one notebook which builds the silver layer by loading the data files from the bronze lakehouse and persisting them as delta tables in the silver lakehouse. There is second notebook which builds the gold layer by loading tables from the silver lakehouse and transforming them into a star schema before saving them as delta tables in the gold lakehouse.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices61.png" alt-text="This is a diagram of a medallion data orchestration process that uses notebooks and a pipeline to populate bronze, silver, and gold lakehouses." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices61.png":::

An important aspect of this data orchestration design is that it exposes a top-level pipeline named **Create Lakehouse Tables**. When you run this pipeline, it runs the two notebooks in sequence to execute all the necessary ETL logic to populate all lakehouses with data. After you deploy this Fabric solution which creates lakehouses in an empty state, you can then automate running this pipeline to execute the data orchestration which moves the workspace into a state where it's ready for use.

> [!IMPORTANT]
> To summarize this best practice, a Fabric solution should expose a single item with top-level ETL logic which can be run to populate data container items with data. Whether you implement a data orchestration strategy using notebooks versus pipelines versus something else is up to you. What's important is that when you first deploy a Fabric solution, you can follow up deployment by running a single job with the data orchestration required to prepare the workspace so it's ready for users. When a Fabric solution includes a top-level notebook or pipeline, you can automate running it after a deployment using fabric-cicd.

When designing a data orchestration strategy, you must also consider data refresh requirements. Think about what happens after the initial deployment once data container items have been populated with data for the first time. How do you plan to keep the data in a lakehouse up to date? The recommended approach is to automate the processes for refreshing data using scheduled jobs.

For example, you could schedule a pipeline that processes a full data refresh to run once a day while designing another pipeline with incremental refresh logic to run as a scheduled job to run every 15 minutes. The key point is that adding scheduled jobs should be part of the data orchestration plan because it enables a Fabric solution to self-manage all its data refresh requirements.

Fabric makes it possible to extend the item definition of notebooks and pipelines using a **`.schedules`** file. The .schedules file contains JSON content with a schedules element containing a list of one or more scheduled jobs. By extending the item definitions for ETL items in a Fabric solutions with **`.schedules`** files, you can automate the data refresh strategy for a Fabric solution.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices62.png" alt-text="This is a screenshot of a Fabric item definition that includes a schedules file for scheduled data refresh jobs." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices62.png":::

One final issue to consider with data orchestration is how to address changes to schemas over time. Think about a scenario where need to update the table schema for a lakehouse that's already been deployed and populated with data. Pushing out table schema changes to a lakehouse is challenging because Fabric doesn't track any metadata for the table schema in the underlying item definition for a lakehouse. The most common way to update the table schema for a lakehouse is a run a notebook with procedural programming logic to add and remove tables and table columns.

Now imagine you begin working in a feature workspace to write code in a notebook to update the lakehouse table schema. First, you write the code and test it using a lakehouse in the feature workspace. After than, you commit your changes to GIT and use a pull request to push the notebook to the integration branch. Now you can use an **Update from GIT** operation to push the notebook to the dev workspace. Likewise, you can use **fabric-cicd** to push the notebook to the test workspace or the prod workspace. However, just pushing the notebook to the workspace with the lakehouse is not enough. You still need to actually run the notebook in the target workspace to update the lakehouse schema.

> [!NOTE]
> Managing schema and pushing schema changes with Fabric warehouse is much different than with a lakehouse. The best practice for implementing CI/CD processes with a warehouse schema involves using the **SqlPackage** and **DacFx** tooling provided by Microsoft. For more informtion, see the [SqlPackage documentation](/sql/tools/sqlpackage/sqlpackage).

### Fabric REST API programming

Fabric offers several developer tools and libraries that are effective in boosting your productivity. You have seen examples of Terraform, Fabric CLI, Semantic Link Labs and **fabric-cicd** which hide many of the low-level details associated with authentication, acquiring access tokens and executing HTTP requests. However, you might encounter scenarios in which you need a greater degree of control. This can be achieved using ***Fabric REST API programming*** to execute API calls using a familiar programming language such as C# or Python.

This guidance recommends using Terraform as your first choice to create infrastructure in Fabric CI/CD projects. But there could be factors that make you decide against using Terraform in a particular Fabric CI/CD project. As an alternative, you can write automation scripts which call the Fabric REST APIs to create tenant-level Fabric items such as workspaces, connections, gateways and deployment pipelines. Using Fabric REST APIs also makes it possible to fully automate the process of setting up GIT integration for a Fabric CI/CD project. This is achieved by creating a GIT source control connection and then using that connection to bind workspaces to GIT branches.

Fabric REST API Programming provides the richest support for managing the lifecycle of workspace items. This is due to the ability to work directly with items definitions when calling CRUD APIs to create and update workspace items. This makes it possible to deploy a Fabric solution to a target workspace using a repeatable process which ensures workspace items are created with the correct relationships to other items in the same workspace. For example, you can create a lakehouse and a notebook in such as way that the notebook is configured to use the new lakehouse as its default lakehouse.

The Fabric REST APIs can be used to control GIT synchronization. There is an **`Update From Git`** API which can be used to initialize or update a feature workspace from an underlying GIT branch. It's also common to call **`Update From Git`** on the dev workspace to synchronize changes merged from feature branches to the integration branch. There is a complimentary **`Commit To Git`** API used in scenarios in which you need to automate the commit operation to push workspace item changes to its underlying GIT branch.

For API fundamentals, see [Fabric REST APIs](/rest/api/fabric/articles/).

> [!CAUTION]
> If you decide to program directly against the Fabric REST APIs, there are a few fundamental you need to learn right away. The Fabric REST APIs are built upon three essential concepts which are ***paginated results***, ***long-running operations (LROs)*** and ***throttling***. Developers who engage in Fabric REST API programming must understand these concepts in order to write code that is correct and reliable. Fortunately, this is something you don't have to worry about when using developer tools such as Terraform and Fabric CLI which deal with paginated results, long-running operations and throttling behind the scenes. For more details, see [pagination](/rest/api/fabric/articles/pagination), [long-running operations](/rest/api/fabric/articles/long-running-operation), and [throttling](/rest/api/fabric/articles/throttling).

Microsoft offers two Software Development Kits (SDKs) for the Fabric REST APIs. Microsoft provides a ***Fabric .NET SDK**** for C# developers and a ***Fabric Python SDK*** for Python developers. These SDKs provide a productivity boost by hiding low-level details for executing HTTP requests, transmitting access tokens and converting back and forth between JSON and strongly-typed objects. The two SDKs also provide the convenience of dealing with paginated results, long running operations and throttling errors behind the scenes.

Let's examine a simple listing showing the traditional *'hello world'* code for using the Fabric Python SDK to create a new workspace. This code begins by creating a **`ClientSecretCredential`** object which is used to authenticate as a service principal. After that, the code creates a **`FabricClient`** object named **`fabric_client`** which exposes methods to execute calls Fabric REST API endpoints. The code in the listing creates a workspace by calling **`create_workspace`**. After that, the code calls **`list_workspaces`** and uses the result to enumerate through the existing set of workspaces.

```python
"""Hello World for Fabric REST API Pythn SDK"""

import os
from azure.identity import ClientSecretCredential
from microsoft_fabric_api import FabricClient

credential = ClientSecretCredential(
    tenant_id = os.getenv("AZURE_TENANT_ID"),
    client_id = os.getenv("AZURE_CLIENT_ID"),
    client_secret = os.getenv("AZURE_CLIENT_SECRET")
)

fabric_client = FabricClient(credential)

# create request body for creating new workspace
create_workspace_request = {
    "displayName": "Hello Python SDK",
    "description": "This isn't so hard",
    "capacityId": os.getenv("FABRIC_CAPACITY_ID")
}

# call Create Workspace API
workspace = fabric_client.core.workspaces.create_workspace(create_workspace_request)

# enumerate workspaces by calling List Workspaces API
workspaces = fabric_client.core.workspaces.list_workspaces()
for workspace in workspaces:
    print(f'{workspace.display_name} - [{workspace.id}]')
```

> [!NOTE]
> For more SDK details, see [Microsoft Fabric Python SDK](https://pypi.org/project/microsoft-fabric-api/) and [Microsoft Fabric .NET SDK](https://www.nuget.org/packages/Microsoft.Fabric.Api).

Let's examine a Fabric REST API programming scenario in which you need to deploy a Fabric solution to a target workspace which involves creating a lakehouse and a notebook. In this scenario, assume you already have an existing item definition for a notebook and you need to create the notebook in such a way that it references the new lakehouse as its default lakehouse. This means you need to update the item definition file named notebook-contents.py with the lakehouse id before you can use the item definition to create the notebook.

There is a timing issue involved in this scenario. You cannot determine the new lakehouse Id until after the lakehouse has been created. Once the lakehouse has been created, you must then update the item definition file notebook-contents.py with the Id of the new lakehouse. The best way to deal with this scenario when programming with the Fabric REST APIs is loading the item definition files for the notebook into memory. Once you have loaded the contents of notebook-contents.py into memory, you can update it dynamically before using it to create a new notebook item. The ability to dynamically update an item definition file provides a more seamless experience when developing a Fabric solution where you're required to create a set of workspace items with dependencies between them.

The Fabric REST APIs offer three primary APIs which involve programming with item definitions. First, the Create Item API allows you to pass an item definition when creating a workspace item. Second, the Update Item Definition API allows you to pass an item definition when updating a workspace item. Third, the Get Item Definition API allows you to retrieve the item definition for an existing item.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices63.png" alt-text="This is a diagram of Fabric REST APIs creating, retrieving, and updating workspace items with item definitions." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices63.png":::

For more API details, see [Fabric item definitions overview](https://learn.microsoft.com/en-us/rest/api/fabric/articles/item-management/definitions/item-definition-overview), [Create Item API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/create-item?tabs=HTTP), [Get Item Definition API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/get-item-definition?tabs=HTTP) and [Update Item Definition API](https://learn.microsoft.com/en-us/rest/api/fabric/core/items/update-item-definition?tabs=HTTP).

Let's walk through a scenario in which creating a workspace item requires dynamically updating an item definition file. Imagine you have just used the Fabric REST API to create a workspace and a lakehouse inside that workspace. Now it's time to create the notebook. However, you have to create a notebook in such a way that it's configured to use the lakehouse as its default lakehouse.

As discussed earlier, the notebook-content.py file in the item definition for a notebook contains metadata which tracks the configuration of its default lakehouse. The metadata is hidden from users when the notebook is opened in the web-based notebook editor provided by the Fabric service. However, you can view this metadata by examining the raw file contents of notebook-content.py file as shown in the following screenshot.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices64.png" alt-text="This is a screenshot of notebook-content.py metadata that stores a notebook's default lakehouse configuration." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices64.png":::

Before calling Create Item, you first need to update the contents of notebook-content to include the correct workspace id and lakehouse id. This means your code must track the ids of the workspace and the lakehouse as they are created. Once you have determined what the ids are, you can perform some type of find-and-replace operation to substitute the workspace id, lakehouse id and lakehouse display name into the contents of notebook-content.py.

```python
# Fabric notebook source
# METADATA ********************
# META {
# META   "dependencies": {
# META     "lakehouse": {
# META       "default_lakehouse": "{LAKEHOUSE_ID}",
# META       "default_lakehouse_name": "{LAKEHOUSE_NAME}",
# META       "default_lakehouse_workspace_id": "{WORKSPACE_ID}",
# META       "known_lakehouses": []
# META     }
# META   }
# META }
```

Directly manipulating item definition files is a powerful technique that assumes you know what you are doing. If you update an item definition file with invalid syntax, you will experience errors when calling Create Item and Update Item Definition. 

Now let's walk through the programming steps required to call Create Item using an item definition. First, you need to enumerate through all the files in the item definition folder and load their contents and their relative files paths into memory. At this point you can updated the contents of notebook-content.py in memory with the metadata for the default lakehouse you'd like to use.

The next step is to parse together a JSON payload that allows you to pass the item definition across the network. Things get a bit tricky because there's a need to pass the contents of multiple files across the network in a single HTTP request. The way to accomplish this is to convert the contents of each file into a base64 encoded format. By formatting file contents using base64 encoding, you can add the content of any file into a JSON element as a standard string value. This makes it possible to transmit all the files required for an item definition across the network in a call the Create Item API.

The following JSON listing shows an example of an HTTP request body that can be passed across the network in a call to the Create Item API. The definition element contains a parts collection which contains a part element for each file in the item definition. Each part element includes a relative path value and a payload for the file contents in a base64 encoded format.

```json
{
  "displayName": "Create Lakehouse Tables",
  "type": "Notebook",
  "definition": {
    "parts": [
      {
        "path": ".platform",
        "payload": "********************<base64-encoded-file-content>"******************** ",
        "payloadType": "InlineBase64"
      },
      {
        "path": "notebook-content.py",
        "payload": "********************<base64-encoded-file-content>"******************** ",
        "payloadType": "InlineBase64"
      },
      {
        "path": "notebook-settings.json",
        "payload": "********************<base64-encoded-file-content>"******************** ",
        "payloadType": "InlineBase64"
      }
    ]
  }
}
```

At this point, you should be able to make an observation about the difference between Fabric REST API programming and using Fabric developer tools and libraries such a Terraform, Fabric CLI or fabric-cicd. Programming the Fabric REST APIs provides more control but at the expense of significantly more complexity. When you use Terraform, Fabric CLI or fabric-cicd, all the work of formatting file contents back and forth between base64 encoding and clear text is handled for you behind the scenes. While these tools and libraries don't provide as much control, they sure makes things a lot easier.

Consider another convenience provided by Terraform, Fabric CLI and fabric-cicd. You don't have to worry about whether target workspace items already exists or not because these tools and libraries contain logic to detect whether a target item already exists. To implement the same logic with the Fabric REST APIs requires multiple steps. First, you need to call the Get Items API to determine whether the target item exists. You must follow that with conditional logic which determines whether to call Create Item versus Update Item Definition.

Let's examine another common scenario in which you need to program the Fabric REST APIs to update an existing workspace item. Perhaps you'd like to update the metadata for an existing notebook to switch its default lakehouse from one lakehouse to another. You can accomplish this using the following steps.

- Call Get Item Definition to retrieve JSON response with the current item definition
- Extract the notebook-content.py file content and convert it from base64 to plain text
- Perform a find-and-replace operation on notebook-content.py to update the workspace id and lakehouse id
- Convert the updated file contents for notebook-content.py back into the base64 encoded format
- Update the JSON element for the item definition with the base64 encoded content for notebook-content.py.
- Call Update Item Definition passing the JSON with the updated item definition.

When you begin to program the Fabric REST APIs, you might notice that it provides item-generic APIs as well as item-specific APIs. You've already seen examples of using item-generic APIs such as Create Item and Update Item Definition. However, the Fabric REST APIs also offer item-specific APIs such as Create Lakehouse and Create Notebook. This begs the question should you use item-generic APIs or item-specific APIs.

With respect to creating, updating and deleting workspace items, it doesn't really matter whether you use the item-generic APIs or item-specific APIs because the result will be the same. For this reason, many developers prefer using item-generic APIs to create and update workspace items because it allows them to write generic code which can be reused with any type of workspace item. 

The only time where it's essential to use item-specific APIs is when you need to a query a workspace item to retrieve its properties. As an experiment, let's compare the results of calling Get Item versus Get Lakehouse. A call to Get Item results in an HTTP GET request to a URL which targets the items endpoint for a specific workspace.

```yml
https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/items/{itemId}
```

The response from Get Item returns a JSON element which contains properties common to all types of workspace items.

```json
{
  "displayName": "sales",
  "description": "Contain for sales data",
  "type": "Lakehouse",
  "workspaceId": "{LAKEHOUSE_ID}",
  "id": "{WORKSPACE_ID}"
}
```
Now let's compare this to a call to Get Lakehouse which uses an URL targeting the lakehouses endpoint for a workspace.

```yml
https://api.fabric.microsoft.com/v1/workspaces/{workspaceId}/lakehouses/{lakehouseId}
```

The response from Get Lakehouse includes a properties element with additional properties that are specific to lakehouses.

```json
{
  "displayName": "sales",
  "description": "Contain for sales data",
  "type": "Lakehouse",
  "workspaceId": "{LAKEHOUSE_ID}",
  "id": "{WORKSPACE_ID}"
  "properties": {
    "oneLakeTablesPath": "https://onelake.dfs.fabric.microsoft.com/{workspaceId}-{ItemId}/Tables",
    "oneLakeFilesPath": "https://onelake.dfs.fabric.microsoft.com/{workspaceId}-{ItemId}/Files",
    "sqlEndpointProperties": {
      "connectionString": "{SQL_ENDPOINT_UNIQUE_PATH}.datawarehouse.fabric.microsoft.com",
      "id": "{LAKEHOUSE_ID}",
      "provisioningStatus": "Success"
    }
  }
}
```

There is a common scenario in which you need to create a lakehouse along with a semantic model which connects to that lakehouse using the OneLake URL or the SQL endpoint. After creating the lakehouse, you need to call the Get Lakehouse API to discover its properties so that you can update the semantic model with the correct connection settings.

### Choosing the best automation approach

TODO: Add dev tools/libraries matrix

A final design decision is how much custom code your team wants to own. Deployment pipelines minimize custom code but limit flexibility. Direct REST API programming maximizes flexibility but increases maintenance. `fabric-cicd` sits between those extremes: you own the workflow and configuration, while the library owns the repeatable deployment mechanics. For many Fabric CI/CD projects, that balance provides the best path from a proof of concept to a governed release process. Revisit the decision periodically as Fabric capabilities and item type support evolve.

## Developing CI/CD workflows

Now that you've seen your options for writing automation logic for CI/CD processes, the next question to answer is where to deploy that code. The answer to this question is simple. You want to deploy the code which runs CI/CD processes in the same GIT repository that holds the item definitions. This design keeps workflow logic, deployment configuration, and solution source files versioned together.

Fabric supports Git integration with Azure DevOps and GitHub. Both platforms can run workflow jobs on demand, on a schedule, or in response to Git events such as commits and pull-request completion. Both platforms also provide a way to store variables, protect secrets, model deployment environments, and require manual approvals before a workflow deploys to a target environment.

A Fabric CI/CD repository commonly contains three categories of files: item definitions under a workspace folder, deployment scripts such as Python files, and workflow YAML files that invoke those scripts. The exact folder names differ by platform, but the goal is the same: source files and automation code move through review and release together.

### Azure Pipelines

Azure DevOps is a Microsoft cloud platform that provides a suite of development tools for software teams to plan, build, test, and deploy applications. It's designed with CI/CD features to support the entire software development lifecycle. When you create a new project in Azure DevOps, the project is automatically created with a GIT repository of the same name. For example, creating a project named Product Sales will automatically create a repository named Product Sales.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices65.png" alt-text="This is a screenshot of an Azure DevOps repository with pipeline YAML files, Python scripts, and Fabric item definitions." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices65.png":::

Azure Pipelines is the workflow development platform that's built into Azure DevOps. Azure pipelines provide the capability to integrate, test and deploy code using the principals of CI/CD. When you create an Azure pipeline, you can configured it to run on-demand or on a schedule such as every night at midnight. Alternately, you can define an Azure pipeline with a trigger to run automatically in response to a code commit or the completion of a pull request.

To create an Azure pipeline, you must first add a YAML file with a pipeline definition into your repository. Once you have added the YAML file, you must then explicitly register it with the Azure DevOps project for it to be recognized as an Azure pipeline. This registration can be accomplished through the Azure DevOps user interface or it can be automated using the Azure DevOps CLI or the Azure REST APIs. Once a pipeline has been registered, you should be able to see it in the Pipelines page in an Azure DevOps project.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices66.png" alt-text="This is a screenshot of an Azure DevOps repository with pipeline YAML files, Python scripts, and Fabric item definitions." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices66.png":::

Use different triggers for different parts of the lifecycle. A validation pipeline can run on every pull request to check files, run tests, or validate item definitions before reviewers approve the merge. A release pipeline can run only after changes land in a release branch or after a release manager starts it manually. Keeping validation and release responsibilities separate makes the workflow easier to govern.

You can write the code for an Azure pipeline in a programming language such as Python. This is accomplished by referencing a script with Python code from the YAML file. The following screenshot shows an example of YAML files for Azure pipelines in the .pipelines folder and their associated Python scripts in the src folder. Below these two folders you can also see the workspace folder which contains the item definitions for a specific Fabric solution.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices67.png" alt-text="This is a screenshot of an Azure DevOps repository with pipeline YAML files, Python scripts, and Fabric item definitions." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices67.png":::

As you begin to develop Azure pipelines for a Fabric CI/CD project, you will find your code requires access to environment settings such as the ids for workspaces, capacities, users and groups. You might also need credentials for an Entra Id application to authenticate as  a service principal. The Azure pipelines service allows you to create variables and secrets in order to make these types of environment settings available to the code in your workflows.

The way to create variables and secrets that are accessible to Azure pipelines is to create a variable group. Once you have created a variable group, you can then add a set of named variables along with their values. The following screenshot shows a typical set of variables and secrets created for Azure pipelines in a Fabric CI/CD project.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices68.png" alt-text="This is a screenshot of an Azure DevOps repository with pipeline YAML files, Python scripts, and Fabric item definitions." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices68.png":::

When you create a variable that contains sensitive data, you can mark that variable as a secret. For example, your pipeline might need to access to a client secret used to authenticate as a service principal. When you mark a variable as a secret, the Azure DevOps service does several important things to protect its value. First, the variable is persisted in the cloud using an encrypted format rather than plain text. Second, the value cannot be viewed in the Azure DevOps user interface after saving. Third, the secret value is masked in logs. If the secret value appears in pipeline output, Azure Pipelines replaces it with *** so it won't be visible to users or in build logs.

Variable groups in Azure DevOps can also be integrated Azure Key Vault. That means you can configure access to secrets in Azure Key Vault from code running in an Azure pipeline. This makes it possible to benefit from Azure Key Vault features such as conditional access policies, auditing, and secret rotation.

By default, an Azure pipeline does not have access to the variables in a variable group. Instead, you must configure permissions for a variable group to ensure the code in your Azure pipelines has access to its variables and secrets. These permissions can be configured using the Pipeline permissions dialog in the Azure Devops user interface or by using the Azure DevOps CLI or the Azure REST APIs.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices69.png" alt-text="This is a screenshot of an Azure DevOps repository with pipeline YAML files, Python scripts, and Fabric item definitions." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices69.png":::

When building a release process using an Azure pipeline, you are often required to configure a manual approval process. This can be tricky when using trunk-based development because the release process doesn't involve pull requests. However, you can solve the problem by creating environments which can be configured with a manual approval process. As an example, you can extend an Azure DevOps project by adding two environments named test and prod.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices70.png" alt-text="This is a screenshot of Azure DevOps environments that can be used as approval gates for Fabric CI/CD deployments." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices70.png":::

Once you have configured the two environments with a manual approval process, you can create Azure pipelines for a release process that target these environments. When you run an Azure pipeline targeting an environment with a manual approval process, the pipeline job pauses until the approval process completes. Once the approval is complete, the pipeline resumes and can use the fabric-cicd library to deploy the new release to a target workspace.

This guidance is intended to get you started developing Azure pipelines for Fabri CI/CD projects. As you begin to get up to speed on developing with Azure pipelines, you should become familiar with monitoring pipeline runs in order to test and debug your code. It's important to add logging to your code to report on successful operations and to provide diagnostic information about any errors that occur. The following screenshot shows how an Azure pipeline can log its progress in a Fabric CI/CD workflow.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices71.png" alt-text="This is a screenshot of Azure DevOps environments that can be used as approval gates for Fabric CI/CD deployments." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices71.png":::

For more details on Azure DevOps and Azure pipelines, see [Azure Pipelines documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/?view=azure-devops), [Key concepts](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops), [YAML pipeline editor](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/yaml-pipeline-editor?view=azure-devops), [Manage variable groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=azure-pipelines-ui%2Cyaml) and [Use Azure Key Vault secrets in Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=managedidentity%2Cyaml).

For a complete walkthrough, see [Tutorial: CI/CD using Azure DevOps and the fabric-cicd library](./tutorial-fabric-cicd-azure-devops.md).

### GitHub Actions

GitHub is different from Azure DevOps with respect to how you work with projects and repositories. GitHub allows you to create repositories without first creating a project. While GitHub supports creating projects to assist with organizing repositories, the use of projects in GitHub is optional. When using GitHub for a Fabric CI/CD project, you just need to create a repository to get started.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices72.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices72.png":::

GitHub Actions is the platform in GitHub that allows you to develop workflows for CI/CD processes. You can create workflows using GitHub Actions that can be run on-demand or run on a schedule. You can also configure workflows to run in response to events such as a code commit or the completion of a pull request.

With GitHub Actions, you create a new workflow by copying a YAML file with  workflow definition into a special repository folder named .github/workflows. Unlike Azure pipelines, there is no need to register the YAML file with GitHub. Just coping to YAML file into the .github/workflows folder is all you need to do for GitHub to recognize it as a Workflow Action. Inside the YAML file, it is possible to reference a script written in a language such as Python. The following screenshot shows a typical example of the file structure of workflow files for a Fabric CI/CD project in a GitHub repository.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices73.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices73.png":::

As you begin to develop GitHub Actions for a Fabric CI/CD project, you need a way to track environment settings such as the ids for workspaces, capacities, users and groups. Your code might also need authentication credentials to authenticate as  a service principal. GitHub allows you to create secrets and variables to track these types of environment settings at two different levels. You can create secrets and variables at the scope of the repository or at the scope of a specific environment.

You can create secrets in GitHub at repository scope by navigating to Settings and selecting the Secrets and variables > Actions. When you create a secret, GitHub will encrypt its value and redact it from logs. A GitHub Actions workflow decrypts a secret at runtime, injects it into your workflow and then immediately discards it from memory when the workflow completes.


:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices74.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices74.png":::

You can also add variables to a GitHub repository for environment settings that are not sensitive. The following screenshot shows an example of variables created for Azure pipelines in a Fabric CI/CD project.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices75.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices75.png":::

While creating secrets and variables at the repository level is adequate for some projects, you might find you need more granularity. GitHub supports a feature which allows you to create environments in a GitHub repository and then to create secrets and variables at environment scope.

Consider a CD release process in which workspaces don't exists within the same Entra Id tenant. For example, imagine the dev workspace, the test workspace and the prod workspace all exist in different Entra Id tenants. In this scenario, you will need different authentication credentials for each Entra Id tenant. You can start by creating three different environments in your GitHub repository named dev, test and prod.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices76.png" alt-text="This is a screenshot of GitHub repository environments named dev, test, and prod for deployment configuration." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices76.png":::

After that, you can configure each environment with the authentication credentials specific to its Entra Id tenant.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices77.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices77.png":::

In addition to tracking environment-specific variables and secrets, an environment can be configured with protection rules. For example, you can create a protection rule for the test environment and the prod environment to require a manual approval process with one or more approvers. Once you have configured the two environments with a manual approval process, you can create GitHub Actions for a release process that target these environments. When you run a GitHub Action targeting an environment with a manual approval process, the GitHub Actions job pauses until the approval process completes. Once the approval is complete, the GitHub Actions job resumes and can use the fabric-cicd library to deploy the new release to a target workspace.

Once you have added the YAML file for a GitHub Actions into the .github/workflows folder, you can view those GitHub Actions in the left navigation of the Actions page. You can also select a workflow see its run history.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices78.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices78.png":::

After selecting an action in the left navigation, you can drill into a specific workflow runs and view its logs. You can also run a action on demand by dropping down the Run workflow menu and clicking the Run workflow button. If you need to collect information from the user before running the workflow, you can define input fields in the workflow's YAML file. The following screenshot shows an example of a workflow with input parameters that prompt the user with a textbox and two checkboxes.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices79.png" alt-text="This is a screenshot of a GitHub repository with workflow YAML files, scripts, and Fabric item definitions for a CI/CD project." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices79.png":::

As you begin to develop workflows using GitHub Actions, you should become familiar with monitoring workflow runs in order to test and debug your code. It's important to add logging to your code to report on successful operations and to display diagnostic information about any errors that occur. The following screenshot shows an example of an GitHub Action written to log its progress in a Fabric CI/CD workflow.

:::image type="content" source="media/fabric-cicd-best-practices/fabric-cicd-best-practices80.png" alt-text="This is a screenshot of a GitHub Actions workflow run log for a Fabric CI/CD deployment." lightbox="media/fabric-cicd-best-practices/fabric-cicd-best-practices80.png":::


## Summary of Best Practices

Let's review the best practices covered in this article.

- Isolating each environment on its own Fabric capacity.
- Single source of truth in the integration branch, with the "Commit to GIT from dev is a one-time op" discipline.
- Multi-workspace solutions sharing a single GIT branch via different Git folder settings.
- Variable libraries as the strategic parameterization story; deployment rules and find-and-replace as fallbacks.
- Running automation as a service principal.
- Letting Terraform create credentials (e.g., SAS tokens) so secrets never leave encrypted state.
- Keeping ETL inside solution items, plus the "top-level orchestration item" pattern for data population post-deploy.

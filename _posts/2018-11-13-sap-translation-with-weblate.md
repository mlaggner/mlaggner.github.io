---
title: 'SAP DDIC translation with Weblate - Part 1'
categories:
  - programming
tags:  
  - sap
  - abap
  - weblate
summary:
  - image: sap.gif  
---

At my main job we often had difficulties translating ABAP reports. This is caused by two main problems:

* Dynpro (and ALV) texts can be offered from different sources:
  * DDIC objects
  * Report texts
  * Dynpro texts

* Translating via SE63 can be hard if you do not know the real source of the text (see above). SE63 is also not a very handy tool to do translations.

Because of this two reasons (and my good experience with [Weblate](https://www.weblate.org)) I researched for a solution to translate ABAP DDIC and report/dynpro texts with Weblate.

The first problem (identify all needed DDIC objects for translations) was rather easy to solve. I've developed a report which searches (recursively) reports/development packages for all used components.

For the second problem I had to dig a bit deeper, since there is no official interface between SAP and Weblate. Weblate itself only uses different source code management tools like [Git](https://git-scm.com/), [Mercurial](https://www.mercurial-scm.org/) and [Subversion](https://subversion.apache.org/
) to import and export translations.

![Weblate workflow](/images/2018/11/weblate_workflow.png){: .align-center}

1. Import translations from Git
2. Translate in Weblate
3. Merge translation changes from Git
4. Export translations to Git

To use weblate as the translation interface I needed to export/import the translations from the SAP system into Git. SAP itself already offers a XLIFF exporter (report `RS_LXE_TEXTEXT_EXPORT`) and importer (report `RS_LXE_TEXTEXT_IMPORT`) via transaction `LXE_MASTER`, so I had only to adopt the code for XLIFF marshalling/unmashalling.

To export those generated XLIFF files to Git you either need the Git toolkit installed on your application server (which we did not have) or use a Git server with an API. We use [Gitlab](https://gitlab.com/) at work and could use the [Gitlab API](https://docs.gitlab.com/ee/api/) for transferring files from/to Git.

With the XLIFF export/import functions from SAP and the API from Gitlab we were able to create a workflow which allows us to fully integrate Weblate into the SAP translation workflow.

![Weblate workflow](/images/2018/11/sap_weblate_workflow.png){: .align-center}

1. Export Translations from SAP (XLIFF)
2. Import translations from Git
3. Translate in Weblate
4. Merge translation changes from Git
5. Export translations to Git
6. Import translations into SAP

In this series I will explain which steps had to be done to integrate Weblate into the SAP translation workflow.

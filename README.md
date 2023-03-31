# Hutte Recipe - PMD

> This recipe uses [pmd](https://github.com/pmd/pmd) via the [pmd-bin](https://github.com/amtrack/pmd-bin) distribution to run static code analysis automatically using GitHub Actions and manually in Hutte using a Custom Button.

## Prerequisites

- a GitHub repository with a valid sfdx project
- a `hutte.yml` file (e.g. the default one shown in the `CONFIGURATION` tab)

## Steps

### Step 1

Create the following 3 files:

`apex-pmd-ruleset.xml`

```xml
<?xml version="1.0"?>

<ruleset name="Ruleset for Apex"
         xmlns="http://pmd.sourceforge.net/ruleset/2.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://pmd.sourceforge.net/ruleset/2.0.0 http://pmd.sourceforge.net/ruleset_2_0_0.xsd">

    <description>Ruleset for Apex</description>

    <rule ref="category/apex/bestpractices.xml/ApexUnitTestShouldNotUseSeeAllDataTrue" />
    <rule ref="category/apex/errorprone.xml/AvoidHardcodingId" />

</ruleset>
```

`.github/workflows/pmd.yml`

```yaml
name: Run PMD

on:
  workflow_dispatch:
  workflow_call:

jobs:
  default:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Install PMD
        run: |
          npm install --global pmd-bin
          pmd --version
      - name: Run PMD
        run: |
          pmd -language apex -R apex-pmd-ruleset.xml -dir force-app
```

`.github/workflows/main.yml`

```yaml
name: main

on:
  push:
    branches:
      - main

jobs:
  pmd:
    name: Run PMD Workflow
    uses: ./.github/workflows/pmd.yml
    secrets: inherit
```

### Step 2

- Edit the `hutte.yml` file in your default branch
- Add the following custom script/button named `Run PMD` in `custom_scripts > scratch_org`

```yaml
custom_scripts:
  scratch_org:
    "Run PMD": |
      npm install --global pmd-bin
      pmd --version
      pmd -language apex -R apex-pmd-ruleset.xml -dir force-app
```

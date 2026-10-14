# My Impact Tracker

Unpacked source for the unmanaged **My Impact Tracker** Power Platform solution.

## Repository layout

- `src/MyImpactTracker/` - source produced by `pac solution unpack`
  - `AppModules/` and `AppModuleSiteMaps/` - model-driven app and site map
  - `Entities/` - Dataverse tables, forms, views, and ribbons
  - `OptionSets/` - global choices
  - `Workflows/` - cloud flow definitions
  - `WebResources/` - solution web resources
  - `bots/` and `botcomponents/` - My Impact Coach agent source
  - `aiskillconfigs/` - AI skill configuration records
  - `Assets/` - agent connection-reference bindings
  - `Other/` - solution manifest, relationships, AI models, and remaining customizations
- `artifacts/` - ignored local output for exported and packed ZIP files

The checked-in snapshot is solution `MyImpactTracker`, version `1.0.0.2`, exported unmanaged from the `lou-dev-gold` development environment (`8b8315e8-7fd9-e639-a66c-d6b32a63c216`).

## Prerequisites and authentication

Install the [Microsoft Power Platform CLI](https://learn.microsoft.com/power-platform/developer/cli/introduction), then authenticate interactively. Do not place passwords, client secrets, or exported authentication data in this repository.

```powershell
$Environment = "<environment-id-or-url>"
pac auth create --name my-impact-dev --environment $Environment --deviceCode
pac auth select --name my-impact-dev
```

Use `pac auth list` to inspect existing profiles and `pac env list` to find accessible environments.

## Export and unpack

Run these commands from the repository root:

```powershell
$Environment = "<source-environment-id-or-url>"
$Solution = "MyImpactTracker"
$Zip = "artifacts\MyImpactTracker_unmanaged.zip"

New-Item -ItemType Directory -Force artifacts | Out-Null
pac solution list --environment $Environment
pac solution export --environment $Environment --name $Solution --path $Zip --overwrite
pac solution unpack --zipfile $Zip --folder "src\MyImpactTracker" --packagetype Unmanaged --clobber
```

The ZIP is a generated artifact and must not be committed.

## Pack and import

Pack the checked-in source and import it into a target environment:

```powershell
$TargetEnvironment = "<target-environment-id-or-url>"
$Zip = "artifacts\MyImpactTracker_unmanaged.zip"

New-Item -ItemType Directory -Force artifacts | Out-Null
pac solution pack --zipfile $Zip --folder "src\MyImpactTracker" --packagetype Unmanaged
pac solution import --environment $TargetEnvironment --path $Zip --publish-changes
```

Review connection references and other environment-specific bindings in the target environment after import. Use a deployment settings file with `pac solution import --settings-file` when a target requires explicit bindings.

## Source validation notes

The unpacked solution includes:

- Model-driven app and site map: `ljr_MyImpactTracker`
- Dataverse tables: `ljr_Customer`, `ljr_Contract`, and `ljr_Impact`
- Daily `Deactivate Expired Contracts` cloud flow, which deactivates active contracts after their End Date
- My Impact Coach agent, its Dataverse MCP tool component, and feedback settings
- Global choices: `ljr_impactsourcetype` and `ljr_impactworktype`
- Agent Dataverse connection-reference binding
- AI models: `Work Statement Extractor` and `Create 1:1 Impactful Summary Agenda`
- Web resources: `ljr_contracts.svg`, `ljr_customers`, and `ljr_impact`

PAC unpack completed without warnings or errors. It does not split every modern component into a dedicated folder: AI model definitions remain in `Other\Customizations.xml`, and the agent connection-reference binding is stored in `Assets\botcomponent_connectionreferenceset.xml`. `Other\Solution.xml` also records dependencies on Microsoft-managed components that must exist in the target environment.

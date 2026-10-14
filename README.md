# My Impact Tracker

My Impact Tracker is a Power Platform solution for recording customer and internal work, connecting it to customers and contracts, and turning factual work notes into leadership-ready impact statements.

The solution includes:

- A model-driven app for Customers, Contracts, and Impacts.
- **My Impact Coach**, a Copilot Studio agent that can create, review, search, and summarize Impact records.
- A native **Capture Impact Record** business skill and fast agent path that create a complete draft from Customer, Contract, Impact Work Type, and short notes.
- AI Builder prompts for work-statement extraction and 1:1 summary generation.
- A daily cloud flow that deactivates Contracts after their End Date.

The checked-in snapshot is the unmanaged `MyImpactTracker` solution, version `1.0.0.4`, exported from the `lou-dev-gold` development environment (`8b8315e8-7fd9-e639-a66c-d6b32a63c216`).

## How to use the solution

### 1. Maintain Customers

Use **Customers** to store the organizations or internal groups associated with your work.

- Create one Customer per account or organization.
- Use the `INTERNAL` Customer for internal projects and team enablement.
- Assign the appropriate CSAM.

### 2. Maintain Contracts

Use **Contracts** to track an engagement's dates, yearly hours, package number, assigned CSAM, and contract type.

- Primary EDE contracts represent ongoing customer work, typically over a one-year period.
- VBD or workshop contracts represent packaged, short-duration deliveries.
- Populate the Customer Success Plan URL only for Primary EDE contracts.
- The **Deactivate Expired Contracts** flow runs daily at 02:00 UTC and deactivates active Contracts whose End Date is before the current date.

### 3. Create an Impact manually

Create an Impact from the model-driven app and provide:

- Customer
- Contract, except for the `INTERNAL` Customer
- Impact Work Type
- Impact Date
- Work Statement or short notes

Use the remaining fields to capture:

- **Change Realized** - the supported outcome or change.
- **Impact Statement** - a concise leadership-ready summary.
- **Evidence** - explicit proof such as validation, approval, delivery, metrics, or artifacts.
- **Enables Next** - the next capability or step supported by the completed work.

New Impacts remain in **Draft** until reviewed.

### 4. Create an Impact with My Impact Coach

Ask the agent to create, add, log, or capture an Impact. The fast-capture skill:

1. Collects Customer, Contract, Impact Work Type, and short notes in one interaction.
2. Resolves only the required Customer and Contract records.
3. Drafts all Impact fields directly without invoking another agent or prompt.
4. Shows one compact preview and asks once for confirmation.
5. Creates the Impact with a single Dataverse write, sending scalar values in `fields` and Customer/Contract GUIDs in the separate `lookups` map.

The agent uses only supplied facts. It does not invent metrics, evidence, business value, approvals, or future work. Missing evidence or outcome details are flagged in the preview without blocking creation.

Example:

> Create an impact for DEMO - Contoso Retail under the Primary EDE contract. Work type is Customer Advisory. Notes: Reviewed the deployment design with the platform team, identified the identity configuration blocking the pilot, and agreed on the remediation steps needed before testing.

### 5. Review and summarize Impacts

My Impact Coach can also:

- Review Draft Impacts for clarity, completeness, unsupported claims, and duplication.
- Search by customer, contract, date range, work type, or status.
- Produce concise 1:1, weekly, monthly, customer, or contract summaries.

## Repository layout

- `src/MyImpactTracker/` - source produced by `pac solution unpack`
  - `AppModules/` and `AppModuleSiteMaps/` - model-driven app and navigation
  - `Entities/` - Dataverse tables, forms, views, and ribbons
  - `OptionSets/` - global choices
  - `Workflows/` - cloud flow definitions
  - `WebResources/` - solution web resources
  - `bots/` and `botcomponents/` - My Impact Coach configuration and tools
  - `skills/` - native Dataverse business skills used by the agent
  - `aiskillconfigs/` - AI skill configuration records
  - `Assets/` - agent connection-reference bindings
  - `Other/` - solution manifest, relationships, AI models, and remaining customizations
- `artifacts/` - ignored local output for exported and packed ZIP files

## Prerequisites and authentication

Install the [Microsoft Power Platform CLI](https://learn.microsoft.com/power-platform/developer/cli/introduction), then authenticate interactively. Do not place passwords, client secrets, tokens, or exported authentication data in this repository.

```powershell
$Environment = "<environment-id-or-url>"
pac auth create --name my-impact-dev --environment $Environment --deviceCode
pac auth select --name my-impact-dev
```

Use `pac auth list` to inspect existing profiles and `pac env list` to find accessible environments.

## Export and unpack

Run from the repository root:

```powershell
$Environment = "<source-environment-id-or-url>"
$Solution = "MyImpactTracker"
$Zip = "artifacts\MyImpactTracker_unmanaged.zip"

New-Item -ItemType Directory -Force artifacts | Out-Null
pac solution list --environment $Environment
pac solution export --environment $Environment --name $Solution --path $Zip --overwrite
pac solution unpack --zipfile $Zip --folder "src\MyImpactTracker" --packagetype Unmanaged --clobber
```

The exported ZIP is generated and must not be committed.

## Pack and import

```powershell
$TargetEnvironment = "<target-environment-id-or-url>"
$Zip = "artifacts\MyImpactTracker_unmanaged.zip"

New-Item -ItemType Directory -Force artifacts | Out-Null
pac solution pack --zipfile $Zip --folder "src\MyImpactTracker" --packagetype Unmanaged
pac solution import --environment $TargetEnvironment --path $Zip --publish-changes
```

Review connection references and environment-specific bindings after import. Use a deployment settings file with `pac solution import --settings-file` when a target requires explicit bindings.

## Source validation notes

The unpacked solution includes:

- Model-driven app and site map: `ljr_MyImpactTracker`
- Dataverse tables: `ljr_Customer`, `ljr_Contract`, and `ljr_Impact`
- My Impact Coach and its Microsoft Dataverse MCP Server tool
- Native `Capture Impact Record` business skill
- Daily `Deactivate Expired Contracts` cloud flow
- Global choices: `ljr_impactsourcetype` and `ljr_impactworktype`
- AI models: `Work Statement Extractor` and `Create 1:1 Impactful Summary Agenda`
- Web resources: `ljr_contracts.svg`, `ljr_customers`, and `ljr_impact`

PAC does not split every modern component into a dedicated folder. AI model definitions remain in `Other\Customizations.xml`, and the agent connection-reference binding is stored in `Assets\botcomponent_connectionreferenceset.xml`.

## Version history

### 1.0.0.4

- Added the native **Capture Impact Record** Dataverse business skill to the solution and source.
- Corrected the agent's `create_record` contract so scalar values are sent in `fields`.
- Moved the `ljr_customer` and `ljr_grm` GUIDs to the separate `lookups` map and explicitly prohibited sending lookup GUIDs in `fields`.
- Preserved the fast path of one preview, one confirmation, and one create operation with exact Dataverse error reporting.

### 1.0.0.3

- Added the My Impact Coach fast Impact-capture skill.
- Reduced creation to four inputs, one preview, and one Dataverse create operation.
- Added factual-only drafting rules and explicit handling for missing evidence or outcomes.
- Relabeled the legacy `ljr_grm` Contract lookup from **GRM** to **Contract**.
- Expanded this README with solution usage, deployment instructions, and version history.

### 1.0.0.2

- Added and enabled the daily **Deactivate Expired Contracts** cloud flow.
- The flow deactivates active Contracts after their End Date.
- Added the cloud flow to the unpacked solution source.

### 1.0.0.1

- Enabled Customer table auditing.
- Improved the Customer table description, form labels, subgrids, Quick Find, and My Customers view.
- Corrected generated labels and improved Customer navigation.

### 1.0.0.0

- Initial unmanaged solution snapshot.
- Added the model-driven app, Customer, Contract, and Impact tables, My Impact Coach, AI models, choices, site map, connection reference, and web resources.

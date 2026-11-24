A new Lamina model starts with a set of cause, control and outcome templates for a cyber risk assessment of an operational technology facility. A cause template names a way an attacker uses a flow to reach a target, and lists the control templates it suggests. An outcome template names a standard outcome and its likelihood reduction. The model owns its copy, so you edit the templates like any other template.

## Starter Files in a New Lamina Model

Creating a Lamina model writes three template files next to the model file, and lists all three under `templates:` in the manifest.

| File | Holds |
|---|---|
| `<id>.causes.template.yaml` | The starter cause templates |
| `<id>.controls.template.yaml` | The control templates that the cause templates suggest |
| `<id>.outcomes.template.yaml` | The starter outcome templates |

The files are ordinary template files. Open them with **Lamina: Edit Templates**, or edit the YAML directly. A change affects this model only.

An existing model does not receive the files. Create a new model, copy its three template files into the existing model folder, then open **Lamina: Edit Templates** and register each file with **Import existing template file…**. [Manage template files](#manage-template-files) describes the file actions.

## Manage Template Files

**Lamina: Edit Templates** registers the template files of the model. A model with no template file shows an empty editor with two actions.

| Action | What it does |
|---|---|
| **Import existing template file…** (**Import file…** once a file is registered) | Registers a YAML file that already lies in the model folder. In VS Code a file dialog opens. In the browser, a prompt asks for the path relative to the model folder. |
| **Create new template file…** (**Create file…** once a file is registered) | Asks for a file name, writes an empty template file with that name, and registers it. The name ends in `.template.yaml`, `.yaml` or `.yml`, and a sub-folder is allowed. The action refuses a name that already exists. |
| **Remove from manifest** | Appears when you choose one file in the **File** list. It removes the file from `templates:`. The file stays on disk. |

The editor reloads after each action. A file outside the model folder is refused, and the editor shows the reason. When the window is read-only, the actions are disabled.

## Catalogue IDs

Every starter template label starts with its catalogue id, then its name, for example `R1 Remote access to an exposed interface`. The letter shows the kind of template.

| Letter | Template |
|---|---|
| `R` | Cause |
| `S` | Control |
| `X` | Outcome |

A cause, control or outcome that uses a template takes its label, so each standard item carries the same id and name in every model.

## Use a Cause Template

1. Add a cause.
2. Select a cause template in the **Template** field of the props drawer.

The cause takes the label and description of the cause template, and its `properties`. You can override any of them on the cause.

The drawer then shows a **Suggested controls** section. Each entry reads *Suggested by* the cause, with the control type and the control template label. Select **+ Add** to add that control to the cause. A control that the cause already holds no longer appears.

A control template of type Inherent adds an inherent control, which you set valid or not valid. A control template of any other type adds an assessed control that you rate. A cause can also add any control template of type Inherent from the **Add control from template** section of its drawer, whether or not the cause suggests it.

Create one cause for each cause template that applies. The cause templates suggest different preventative controls, which is why they stay separate causes.

## The Starter Cause Templates

| Cause template | Covers | ATT&CK for ICS |
|---|---|---|
| R1 Remote access to an exposed interface | An interface provided for remote use, attacked through identity: stolen or guessed credentials, or a trusted insider | T0822, T0886, T0859 |
| R2 Shared management tools or infrastructure control plane | A management tool outside the target that holds privileged access inside it | |
| R3 Shared security tools with privileged access | A security tool, such as an endpoint detection platform, that can run commands in the target | |
| R4 Hijack of an interactive session | An attacker who controls a user's device and uses a session the user opened | |
| R5 Vulnerability in a network-accessible interface | A flaw in a port, an API or another reachable service | T0819, T0866 |
| R6 Compromise of a trusted data source | A database, platform or middleware interface that feeds data into an operational process | |
| R7 Software supply chain compromise | Software or an update that carries malicious code | T0862 |
| R8 Local or physical access | A person with access to the facility who configures or commands field devices | |
| R9 Infected portable device or removable media | A laptop, phone or storage device that spreads malware when it connects | T0864, T0847 |
| R10 Access through a wireless network | An attacker who reaches the radio network and commands remote devices | T0860 |
| R11 Jamming of a wireless network | Radio jamming that disrupts communications | T0814 |
| R12 Drive-by compromise through web content | Web content that runs code on the device that originates the flow | T0817 |
| R13 Spearphishing with a malicious attachment | A targeted message whose attachment runs code on the device | T0865 |

## The Starter Control Templates

Each assessed control template gives the control type, the category, and the target effectiveness: the effectiveness the control has when it works as designed. An inherent control template gives neither a category nor an effectiveness.

| Control template | Type | Category | Target effectiveness |
|---|---|---|---|
| S1 Multi-factor authentication | Preventative | Technology | Partial |
| S2 Privileged access management | Preventative | Technology | Partial |
| S3 Limited user base | Preventative | Process | Low |
| S4 Controlled remote access gateway | Preventative | Technology | Partial |
| S5 Network segmentation and flow filtering | Preventative | Technology | Full |
| S6 Vulnerability and patch management | Preventative | Process | Low |
| S7 Endpoint hardening | Preventative | Technology | Partial |
| S8 Session timeout and lock | Preventative | Technology | Low |
| S9 Validation of incoming operational data | Preventative | Technology | Partial |
| S10 Secure development and supplier assurance | Preventative | Process | Low |
| S11 Portable device and removable media control | Preventative | Process | Full |
| S12 Physical access control | Preventative | Process | Partial |
| S13 Wireless link protection | Preventative | Technology | Partial |
| S14 Email and web content filtering | Preventative | Technology | Low |
| S15 Interface requires authentication | Inherent | | |
| S16 Exposed services are inventoried | Inherent | | |
| S17 Operational data comes from a managed source | Inherent | | |

A control that uses a template takes the target effectiveness from it. Set the current effectiveness on the control.

## The Starter Outcome Templates

The likelihood reduction is the number of steps on the likelihood scale that the outcome sits below the event.

| Outcome template | Likelihood reduction |
|---|---|
| X1 Loss of control of the process | 0 |
| X2 Loss of view of the process | 0 |
| X3 Unsafe operation of equipment | 1 |
| X4 Loss or corruption of operational data | 0 |
| X5 Disclosure of operational information | 1 |

Select an outcome template in the **Template** field of an outcome. The outcome takes the label, description and likelihood reduction of the template.

## ATT&CK for ICS IDs

A cause template with a matching MITRE ATT&CK for ICS technique carries the technique ids in a property named `attack-ics`. The value is one string, with the ids separated by commas, for example `T0822, T0886, T0859`. A cause that uses the cause template shows the property in the props drawer, and a report or a filter can read it. See [Entity properties](/docs/sightline/usage/shared/entity-properties) for how properties work.

A cause template with no matching technique leaves the property out.

## Edit or Replace the Starter Set

Edit a template in **Lamina: Edit Templates** to change its text, its suggested controls, its target effectiveness or its likelihood reduction. A new cause template needs an id that starts with `tpl-cause-`. A new control template needs an id that starts with `tpl-control-`. A new outcome template needs an id that starts with `tpl-outcome-`.

A cause template lists control templates by id in `suggestedControls`. An id that matches no control template appears in the drawer as the bare id. Keep each list and the control templates it names in step.

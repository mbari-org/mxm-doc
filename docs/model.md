[//]: # (hmm mermaid's erDiagram is not very nice)

WiP

```mermaid
flowchart TB
  Provider        -->|defines| MissionTemplate;
  Provider        -->|executes| Mission;
  Provider        -->|reports| MissionStatus;
  Provider        -->|reports| AssetStatus;
  MissionTemplate -->|has| Parameter;
  MissionTemplate -->|applies to| AssetClass;
  Mission         -->|type| MissionTemplate;
  Mission         -->|has| Argument;
  Mission         -->|for| Asset;
  Parameter       -->|has| Unit;
  Argument        -->|for| Parameter;
  Asset           -->|type| AssetClass;
  MissionStatus   -->|refers to| Mission;
  MissionStatus   -->|includes| DataReport;
  MissionStatus   -->|includes| AssetStatus;
```

```mermaid
erDiagram
  Provider ||--o{ MissionTemplate : defines
  Provider ||--o{ Mission : "executes"
  Provider ||--o{ MissionStatus : "reports"
  Provider ||--o{ AssetStatus : "reports"
  MissionTemplate ||--|{ Parameter : "has"
  MissionTemplate ||--|| AssetClass : "applies to"
  Mission ||--o{ MissionTemplate : "type"
  Mission ||--o{ Argument : "has"
  Mission ||--|| Asset : "for"
  Parameter ||--o| Unit : "has"
  Argument ||--|| Parameter : "for"
  Asset ||--o{ AssetClass : "type"
  MissionStatus ||--|| Mission : "refers to"
  MissionStatus ||--o{ DataReport : "includes"
  MissionStatus ||--o{ AssetStatus : "includes"
```

## Provider

This represents the central entity in the data model regarding provider information. As such it primarily (i) indicates
the mission templates and assets that it handles, and (ii) supports the execution of particular missions defined
according to associated mission templates. TethysDash/LRAUV is an example of a provider that can be
represented in the MXM system.

## MissionTemplate

Defines a particular mission template as supported by a particular provider. A mission template applies to assets of
particular asset classes and has associated parameters. The "Science/sci2" mission script is an example of a mission
template associated with the TethysDash/LRAUV provider.

## AssetClass

Defines a family of assets that can be associated with a mission template. In the TethysDash/LRAUV case, "LRAUV" is the
asset class associated with all mission templates in that system.

## Asset

Indicates a particular instance of an asset class. "Daphne" and "Makai" are examples in the TethysDash/LRAUV system,
both of which are instances of the "LRAUV" asset class.

## Parameter

A parameter indicates a particular input parameter in a mission template. Parameter attributes include name, type, units
of measure, and optional default value. As an example, MissionTimeout is one of the parameters associated with the
"sci2" mission template in TethysDash/LRAUV.

## Mission

A mission is a particular instantiation of a mission template. Its purpose is to represent a concrete planned (and
eventually executed) mission. A mission indicates the particular target asset (e.g., "Daphne"), and the concrete
parameter values (either default values or explicitly indicated values via arguments –see below) according to the
corresponding mission template.

## Argument

An argument in a mission captures a particular value for a parameter, either to override its default value (if any) or
to provide a value for a required or optional parameter.

## Unit

Defines a unit of measure, which can be referenced by parameter when the relevant provider indicates that
it uses units of measure for its parameters.

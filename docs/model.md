[//]: # (hmm mermaid's erDiagram is not very nice)

WiP

```mermaid
flowchart TB
  Provider        -->|defines| MissionTemplate;
  Provider        -->|executes| Mission;
  Provider        -->|reports| MissionStatus;
  Provider        -->|reports| AssetStatus;
  MissionTemplate -->|applies to| AssetClass;
  MissionTemplate -->|has| Parameter;
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

## Model

### Provider

This is the central entity in the data model that represents the external
mission execution system.

Basically, a provider indicates the _mission templates_ and _assets_ that it handles,
and supports both the execution of _missions_ defined according to such mission templates,
and associated reporting in terms of mission and asset status and generated data.

#### Provider Capabilities

By default, MXM will assume a minimum set of required capabilities.
At registration, the provider can optionally indicate additional capabilities as described below.
This is intended to allow some flexibility, especially to accommodate various levels
of sophistication of the integrated providers.

The provider can indicate whether it:

- Uses _scheduling_ when issuing missions for execution
- Uses _units of measure_ for the mission template parameters 
- Can validate a mission draft before actual submission
    - yes or not; summary of the validation process
    - result of feasibility analysis, e.g.:
        - will the asset hit the bottom/get too close to shore/etc.?
    - Projected time of mission completion
    - Asset location at mission completion
- Can report the status of previously submitted missions
    - via explicit request from MXM and/or asynchronously via push notifications to MXM 
- Can report the status of a handled asset (not necessarily via mission status report)
    - current position / latest positions; heading; speed; etc.
    - battery level, etc.


### MissionTemplate

Defines a particular mission template as supported by a particular provider.

A mission template:

- applies to assets of a particular asset class
- has associated parameters

### AssetClass

Defines a family of particular assets that can be associated with a mission template.

!!! note
    In the model, all asset classes and assets are defined as a common resource for all registered providers.
    In the future, we envision this common resource to either be kept in sync with or 
    directly refer to the entities defined in the TrackingDB system or similar,
    where an authoritative definition of such asset classes and assets is in place.


### Asset

Indicates a particular instance of an asset class.

### Parameter

A parameter indicates a particular input parameter in a mission template. Parameter attributes include name, type,
optional units of measure (when provider indicates it uses units of measure), and optional default value.

### Mission

A mission is a particular instantiation of a mission template. Its purpose is to represent a concrete planned (and
eventually executed) mission. A mission indicates the particular target asset (e.g., "Daphne"), and the concrete
parameter values (either default values or explicitly indicated values via arguments –see below) according to the
corresponding mission template.

### Argument

An argument in a mission captures a particular value for a parameter, either to override its default value (if any) or
to provide a value for a required or optional parameter.

### Unit

Defines a unit of measure, which can be referenced by mission template parameters when the associated
provider indicates that it uses units of measure for its parameters.

!!! note
    In the model, all possible units of measure are defined as a common resource.
    In the current prototype, we are taking the definitions from TethysDash/LRAUV system
    as a good basis as it is pretty comprehensive. We can revisit this in the future as needed.

### MissionStatus

A provider can indicate ability to report the status of any submitted mission.
The `MissionStatus` entity is used for this report.

This entity includes the following attributes:

- `status`: a string indicating the status of the mission, with possible values including:
    - `SUBMITTED`: the mission has been submitted to the provider
    - `QUEUED`: the mission is currently queued for execution
    - `RUNNING`: the mission is currently running
    - `COMPLETED`: the mission has completed
    - `TERMINATED`: the mission has been terminated
    - `CANCELED`: the mission has been canceled
      
        !!! todo
            CANCELED vs TERMINATED: what is the difference, or keep just one?

    - `FAILED`: the mission has failed
- `dataReports`: a list of `DataReport` entities, each of which indicates a particular data product
  generated by the mission. This is optional and may be empty.
- `assetStatuses`: a list of `AssetStatus` entities, each of which indicates the status of a particular
  asset associated with the mission. This is optional and may be empty.

### DataReport

A `DataReport` entity indicates a particular data product generated by a mission. It includes the following attributes:

- `name`: a string indicating the name of the data product
- `url`: a string indicating the URL of the data product
      
    !!! todo
        Also some explicit `data` attribute for the data itself?

- `date`: a string in ISO format indicating when the data product was generated or updated


### AssetStatus

An `AssetStatus` entity indicates the status of a particular asset.
This can be included in a `MissionStatus` report, or directly for a given asset
if such operation is supported by the provider.

The `AssetStatus` entity includes the following attributes:

- `assetId`: the ID of the asset for which the status is reported
- `positions`: a list of `Position` entities, each of which indicating a particular position of the asset
  at a particular time. This is optional and may be empty.
- Any other attributes to facilitate the reporting for human consumption or by MXM clients.

!!! todo
    - Determine additional attributes (e.g., battery level) that should be characterized explicitly.
    - (incremental) Formalization of common attributes across integrated providers  
    - This information propagated via API 
    - Consumers:
        - MXM UI for a summary of status
        - Client systems (ODSS, etc.)

---

## Examples

Example of models above in the context of various providers:

### TethysDash/LRAUV system

!!! Example ""
    - **Provider**: TethysDash/LRAUV
    - **AssetClass**: LRAUV
    - **Assets**: Daphne, Makai
    - **Mission templates**: Science/sci2, Maintenance/ballast_and_trim
    - **Parameters**: MissionTimeout, ApproachSpeed

### WG FrontTracking

!!! Example ""
    - **Provider**: WG FrontTracking
    - **AssetClass**: Waveglider
    - **Assets**: Tiny, Hansen, SV3-117
    - **Mission template**: TrackFront
    - **Parameters**: temperature_threshold, boundary_polygon

### Dorado system

!!! Example ""
    - **Provider**: Dorado system
    - **AssetClass**: Dorado
    - **Assets**: ...
    - **Mission templates**: ...
    - **Parameters**: ...

---

> just a tentative, alternative diagram representation (mermaid's erDiagram):
> 
> ```mermaid
> erDiagram
>   Provider ||--o{ MissionTemplate : defines
>   Provider ||--o{ Mission : "executes"
>   Provider ||--o{ MissionStatus : "reports"
>   Provider ||--o{ AssetStatus : "reports"
>   MissionTemplate ||--|{ Parameter : "has"
>   MissionTemplate ||--|| AssetClass : "applies to"
>   Mission ||--o{ MissionTemplate : "type"
>   Mission ||--o{ Argument : "has"
>   Mission ||--|| Asset : "for"
>   Parameter ||--o| Unit : "has"
>   Argument ||--|| Parameter : "for"
>   Asset ||--o{ AssetClass : "type"
>   MissionStatus ||--|| Mission : "refers to"
>   MissionStatus ||--o{ DataReport : "includes"
>   MissionStatus ||--o{ AssetStatus : "includes"
> ```

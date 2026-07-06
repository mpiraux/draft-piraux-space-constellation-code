---
title: "A code to describe satellite constellations"
abbrev: "Constellation code"
category: info

docname: draft-piraux-space-constellation-code-latest
submissiontype: IRTF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "IRTF"
workgroup: "Systems and Protocol Aspects for Circumstellar Environments RG"
venue:
  group: "Systems and Protocol Aspects for Circumstellar Environments RG"
  type: "Research Group"
  mail: "space@irtf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/space/"
  github: "mpiraux/draft-piraux-space-constellation-code"
  latest: "https://mpiraux.github.io/draft-piraux-space-constellation-code/draft-piraux-space-constellation-code.html"

author:
 -
    fullname: "Maxime Piraux"
    organization: Aerospacelab
    email: "maxime.piraux@aerospacelab.com"
 -
    fullname: "Juan A. Fraire"
    organization: Inria / Saarland University
    email: "juan.fraire@inria.fr"

normative:
  RFC5234:
  RFC8610:

informative:
  RFC8949:
  TvdLCode:
    author:
      - ins: Tim van der Lee
    title: Constellation Code
    date: July 2023
    target: https://github.com/Tim024/ConstellationCode
  BhSi2019: DOI.10.1145/3359989.3365407
  StFrHe2022: DOI.10.1109/ASMS/SPSC55670.2022.9914716
  VaCrHuKe2006: DOI.10.2514/6.2006-6753
  HoRo1980:
    title: "Spacetrack Report No. 3: Models for Propagation of NORAD Element Sets"
    author:
      - ins: F. R. Hoots
        name: Felix R. Hoots
      - ins: R. L. Roehrich
        name: Ronald L. Roehrich
    date: December 1980
...

--- abstract

When considering a satellite constellation forming a non-terrestrial network, the characteristics of this constellation heavily influences the network topology it forms.
To improve the analysis of such non-terrestrial networks across various tools developed by the network community, this document proposes a notation to describe common constellation patterns.
In addition, this document may serve as an introduction to satellite constellations for IETF participants.

--- middle

# Introduction

A satellite constellation spans three networks to deliver their services as illustrated by {{fig-satellite-constellation-networks}}.
First, an Access Network enables User Equipment (UE) to connect to the constellation.
This is realised by the establishment of a Service Link to exchange UE traffic.
Then, the UE traffic is forwarded over a Core Network consisting of the different interconnected satellites.
Finally, the UE traffic is sent back to ground via the Feeder Link, which connects a satellite to a ground station.
A ground station is usually colocated with the infrastructure required to deliver the service.
In the case of Internet broadband access, this can be a Point-of-Presence (PoP) connecting to the Internet.

~~~~
 Access Network |         Core Network         |    Ground Network
                |                              |
  UE ------- Service ---> *---*---*---* ---- Feeder --> Ground --- PoP
              Link        :   :   :   :       Link      Station
                |         *---*---*---*        |
                |                              |
~~~~
{: #fig-satellite-constellation-networks artwork-align="center" title="A satellite constellation spans three networks"}

The network topology of the Core Network of a satellite constellation is heavily influenced by its orbital characteristics. With recent technologies enabling Optical Inter-Satellite Links (OISL) between satellites, a network is formed in space by establishing links between neighbour satellites. The resulting topology can be dynamic as the distance between neighbour satellites changes throughout their orbital period.

A key characteristic of satellite constellations is the ephemeral nature of the Feeder Links.
They may only be established when a satellite and a ground station are in range of each other.
Typically, ground stations can establish links within a defined cone of coverage.
This cone is often characterised by a Minimum Elevation Angle (MEA), such that Feeder Links can only be established when their elevation is above the MEA.
Consequently, satellites are often engineered such that Feeder Links are feasible within the entire cone of coverage of ground stations.
A ground station often includes several antennas such that a certain number of Feeder Links can be established from a given location.
Satellites may include several antennas as well to establish several Feeder Links or enable make-before-break transitions.

A common notation for the network community to describe these constellations could improve the reproducibility of evaluations, measurements and simulations of satellite constellation networks.
This document focuses on describing some elements of the Core and Ground Networks.

The approach of this document is based on the mission parameters of a satellite constellation. Based on these parameters, the expected position of each satellite within the constellation can then be computed.
Tools using the notation described in this document are free to choose how they propagate the positions of satellites.
This may be revised in later versions of this document.
The two practical options are:

- Keplerian-based propagation, focusing on the theoretical position of satellites.
- Perturbation-based propagation, such as using Simplified General Perturbations 4 (SGP4) or Simplified Deep Space Perturbations 4 (SDP4) {{HoRo1980}} {{VaCrHuKe2006}}.

This version of the specification applies only to circular orbital shells. The rationale for this restriction is that circular orbits are the most common in current satellite constellations and simplify the code syntax. Elliptical orbits, such as those used in Molniya or Flower constellations, are outside the current scope but could be supported in a future extension of this document.

The notation defined in this document can also specify patterns for links within a shell of a constellation. Each pattern is repeated to establish the connectivity of a satellite with its neighbours within the shell. This is inspired by the works of network researchers on constellation network topology design {{BhSi2019}}.

The rest of this document is organised as follows.
{{satellite-constellations}} introduces two variants of the Walker pattern for orbital shells. These are used to define many of the existing satellite constellations. {{constellation-code}} defines the constellation code syntax using an ABNF grammar [RFC5234] and the code semantics. {{examples-of-constellation-codes}} contains examples of existing constellations defined using the constellation code. {{links}} augments the constellation code with a pattern notation to describe links within a shell.
{{ground-stations}} defines how ground stations can be described.
Finally, {{considerations}} concludes with considerations for future versions of this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Satellite constellations

A constellation greatly improves the availability of a satellite service up to global or near-global coverage on Earth.
From the user perspective, a constellation offers more guarantees that a satellite can be reached at all times.
A constellation is composed of a set of orbital planes. Typically, several satellites are present on an orbital plane. They can be close together to perform formation flying or are equally spread within the plane.
Orbital planes are distributed in a complementary manner, i.e., they share some properties (e.g. altitude and inclination) but differ in others (e.g. longitude of ascending node).

When all orbital planes of a constellation are circular orbits sharing the same altitude, they are said to constitute an orbital shell. Constellations often consist of a single orbital shell but more complex deployments can have several shells.

The rest of this section describes two common shells based on the Walker pattern.

## Walker constellations

A Walker constellation consists of circular orbits sharing the same inclination. Two variants of the Walker pattern exist:

- Walker Star, where orbits are distributed over 180 degrees around the equator.
- Walker Delta, where orbits are distributed over 360 degrees around the equator.

### Walker Star

{{fig-walker-star}} is an illustration of a Walker Star constellation considering the Earth equator as horizontal in the Figure. The orbit trajectories are depicted by a dashed line, while satellites and their travel direction are indicated by arrow heads.

The orbits of a Walker Star constellation typically have an inclination close to 90 degrees with respect to the equator plane, though this is not a geometric constraint and other inclinations are possible. Given that they are distributed over 180 degrees around the equator plane, one half-sphere has satellites ascending from the south pole to the north pole while the other has them descending from north pole to south pole. This is depicted on the two sides of {{fig-walker-star}}.
Over the south and north poles, all orbits are crossing paths before going over the other half-sphere.

~~~~
          /  / \  \
        , - ~ ~ ~ - ,
    , '/    ^   v    \' ,
  ,   ^    /     \    v  ,
 ,   /    ^       v    \   ,
,   ^     |       |     v   ,
,   |     ^       v     |   ,
,   ^     |       |     v   ,
 ,   \    ^       v    /   ,
  ,   ^   \       /   v   ,
    ,  \   ^     v   / , '
      ' - , _ _ _ ,  '
          \  \ /  /
~~~~
{: #fig-walker-star artwork-align="center" title="A Walker Star constellation"}

In a Walker Star constellation, a seam can be observed at the start and end of the orbit distribution around the equator plane. That is the first orbit (resp. last orbit) is next to the last orbit (resp. first orbit) going in the opposite direction of the sphere. It can be observed at the center of the {{fig-walker-star}}. The seam effect in Walker Star constellations may limit cross-plane ISL links at the seam boundary, though cross-plane links are still possible elsewhere; for instance, the Iridium constellation uses a Walker Star pattern with cross-plane ISLs. However, the Delta variant is often preferred for OISL-capable constellations due to the absence of the seam effect.

{{fig-walker-star-topo}} illustrates a part of a possible network topology for Walker Star constellations, with four orbital planes depicted vertically, each containing three satellites. In this example, links are only established in-plane, i.e., within the same orbit, though cross-plane links are also possible. Each orbit forms a ring, where the last satellite is connected to the first satellite.

~~~~
  :        :        :        :
  |        |        |        |
+~~~+    +~~~+    +~~~+    +~~~+
[0/0]    [1/0]    [2/0]    [3/0]
+~~~+    +~~~+    +~~~+    +~~~+
  |        |        |        |
  |        |        |        |
+~~~+    +~~~+    +~~~+    +~~~+
[0/1]    [1/1]    [2/1]    [3/1]
+~~~+    +~~~+    +~~~+    +~~~+
  |        |        |        |
  |        |        |        |
+~~~+    +~~~+    +~~~+    +~~~+
[0/2]    [1/2]    [2/2]    [3/2]
+~~~+    +~~~+    +~~~+    +~~~+
  |        |        |        |
  :        :        :        :
~~~~
{: #fig-walker-star-topo artwork-align="center" title="A Walker Star constellation network topology"}

### Walker Delta

{{fig-walker-delta}} is an illustration of a Walker Delta constellation with only two orbits due to graphical constraints. The orbits of a Walker Delta constellation typically have an inclination ranging from 45 to 65 degrees with respect to the equator plane, though any inclination is geometrically valid. Combined with the altitude, the inclination directly limits the latitude coverage of a constellation, while Walker Star constellations have a complete latitude coverage.

Given that the orbits are distributed around the entire equator plane, there is no seam effect as in the Walker Star pattern. Instead, each orbit progresses in the same direction and cross paths twice with every other orbit. In this case, satellites can establish links with neighbouring orbits in addition to links within the same orbit.

~~~~
    /   , - ~ ~ ~ - ,   \
    , '               ' ,
  ,   \                   ,
 ,      ^           /      ,
,         \        v        ,
,           ^   /           ,
,             \v            ,
 ,           /  ^          ,
  ,         v     \       ,
    ,    /             , '
      ' - , _ _ _ ,  '
      \               /
~~~~
{: #fig-walker-delta artwork-align="center" title="A Walker Delta constellation"}

{{fig-walker-delta-topo}} illustrates a part of a possible network topology for Walker Delta constellations, with four orbital planes depicted vertically, each containing three satellites. Links are established in-plane and cross-plane, i.e., from one orbit to the other.

~~~~
      :        :        :        :
      |        |        |        |
    +~~~+    +~~~+    +~~~+    +~~~+
..--[0/0]----[1/0]----[2/0]----[3/0]--..
    +~~~+    +~~~+    +~~~+    +~~~+
      |        |        |        |
      |        |        |        |
    +~~~+    +~~~+    +~~~+    +~~~+
..--[0/1]----[1/1]----[2/1]----[3/1]--..
    +~~~+    +~~~+    +~~~+    +~~~+
      |        |        |        |
      |        |        |        |
    +~~~+    +~~~+    +~~~+    +~~~+
..--[0/2]----[1/2]----[2/2]----[3/2]--..
    +~~~+    +~~~+    +~~~+    +~~~+
      |        |        |        |
      :        :        :        :
~~~~
{: #fig-walker-delta-topo artwork-align="center" title="A Walker Delta constellation network topology"}

# Constellation code

{{constellation-code-abnf-def}} defines the constellation code using an ABNF grammar [RFC5234]. The code can define a constellation with multiple shells. Each shell can follow a Walker Star or Walker Delta pattern.

~~~ abnf
constellation = shell [ "+" constellation ]
shell = walker ":" altitude ":" inclination ":" plane-params
        [ ":" mean-anomaly ]
walker = "D" / "S"
altitude = float
inclination = float
plane-params = no-sats "/" no-planes "/" phasing-factor
no-sats = int
no-planes = int
phasing-factor = int
mean-anomaly = float

int = 1*DIGIT
float = 1*DIGIT [ "." 1*DIGIT ]
DIGIT = %x30-39

~~~
{: #constellation-code-abnf-def artwork-align="center" artwork-name="syntax" title="ABNF Grammar for the constellation code"}
[comment]: Compiled by https://author-tools.ietf.org/abnf

In addition to the grammar presented above defining the syntax of the code, a number of requirements on the semantics of the code are listed below.

- The altitude is expressed in kilometres with reference to the Earth's surface.
- The inclination is expressed in degrees and MUST be within the range of \[0, 180\] degrees. Inclinations greater than 90° represent retrograde orbits.
- The number of satellites must be evenly divisible by the number of planes.
- The phasing factor must be within the range \[0, no-planes - 1\]. It represents the relative offset between satellites in adjacent orbital planes. It determines how satellites in one plane are shifted in their orbital position compared to the satellites in the neighbouring plane, enabling optimal coverage patterns.
- The mean anomaly is expressed in degrees and MUST be within the range of \[0, 360\] degrees. It is optional and represents the orbital position of the first satellite in the first plane of the constellation. When absent it is considered equal to zero. The reference epoch for the mean anomaly is defined by the user's simulation environment or application context.

# Examples of constellation codes

This section provides some examples of how the constellation code can be used to define existing satellite constellations sourced from public information. In some cases, when the phasing factor is not known, it is speculative.

 | Name | Description | Constellation code |
 |:-----|:------------|:-------------------|
 | Iridium | Walker Star, 780 km altitude, 86.4° inclination, 66 satellites, 6 planes | S:780:86.4:66/6/1 |
 | OneWeb | Walker Star, 1 200 km altitude, 87.9° inclination, 672 satellites, 12 planes | S:1200:87.9:672/12/11 |
 | Starlink (shell 1) | Walker Delta, 550 km altitude, 53° inclination, 1584 satellites, 72 planes | D:550:53:1584/72/39 {{StFrHe2022}} |
 | GPS | Walker Delta, 20 180 km, 55° inclination, 24 satellites, 6 planes | D:20180:55:24/6/1 |
{: #example-table title="Examples of constellation codes"}

# Describing links in a shell {#links}

In this section, we extend the code notation with the following CDDL schema
{{RFC8610}} to specify the patterns of links within a shell.

~~~ cddl
constellation-specs = {
  version: tstr,
  shells: [+ shell-entry],
}

shell-entry = {
  code: tstr,  ; Constellation code as specified in this document
  link-patterns: [* link-pattern],
}

link-pattern = {
  (
    (rank-offset: int, ? plane-offset: int) //
    (? rank-offset: int, plane-offset: int)
  ),
  ? conditions: [* condition],
}

condition = { eq: [expression, expression] }

expression = int
           / context-element
           / operation

context-element = "rank" / "plane"

operation = { mod: [expression, expression] }
~~~
{: #constellation-cddl-def artwork-align="left" artwork-name="syntax" title="CDDL schema for constellation links"}

Each data item specifies a constellation that may be composed of several shells.
An example specifying a two-shell constellation is given below in Extended
Diagnostic Notation (EDN) {{RFC8949}}:

~~~ edn
{
  "version": "draft-piraux-space-constellation-code-03",
  "shells": [
    {
      "code": "D:1200:55:400/20/19",
      "link-patterns": [
        { "rank-offset": 1 },             / in-plane link to the next satellite /
        {
          "plane-offset": 1,              / cross-plane link in a staggered pattern /
          "conditions": [                 / e.g., when only three links are possible /
            { "eq": [{ "mod": ["rank", 2] }, { "mod": ["plane", 2] }] }
            / rank % 2 == plane % 2 /
          ]
        }
      ]
    },
    {
      "code": "S:1210:89:52/4/1",
      "link-patterns": [
        { "rank-offset": 1 }
      ]
    }
  ]
}
~~~
{: #constellation-edn-example artwork-align="center" artwork-name="example" title="Example constellation-specs data item"}

{{constellation-edn-example}} specifies a two-shell constellation. The first shell is a Walker Delta shell in which satellites have three links towards neighbours. The second is a Walker Star pattern with two in-plane links per satellite.

These patterns are encoded through the `link-patterns` key. It contains a list of patterns with optional conditions. Each pattern specifies how to reach a neighbour given local plane and rank offsets to establish a bidirectional link. For instance, the first pattern of the first shell specifies that a link is formed with the next satellite in the same orbit.

For each pattern, a list of conditions can be expressed with the `conditions` key. These are evaluated for each satellite within the shell to determine whether the corresponding pattern should be applied to form a link. By applying each pattern to all satellites, the set of links within the constellation shell is established.

## Detailed specification

`version`

: Indicates the version of this I-D that the data item should be interpreted with.

`shells`

: A list of shell entries.

### Shell entry

`code`

: The shell code following the specification in {{constellation-code}}.

`link-patterns`

: A list of link patterns applied to every satellite in the shell.

### Link pattern

`rank-offset`

: An integer specifying the offset in rank to reach the neighbour for this link.

`plane-offset`

: An integer specifying the offset in plane to reach the neighbour for this link. When this offset causes the plane index to wrap around to the first plane, the rank index of the target satellite is adjusted according to the phasing factor of the shell.

At least one of the two offsets MUST be present and non-zero.
The other defaults to zero when absent.
They naturally wrap around at the boundaries of a shell.

`conditions`

: A list of conditions that must all be met for the corresponding link to be added to a given satellite.

### Condition

A condition is a predicate applied to two expressions. This version of the document only specifies the equality predicate, indicated by the `eq` key.

### Expression

An expression is one of: an integer literal, a context element, or an operation
on two sub-expressions.
Context elements are represented by strings and two of them are defined.
`rank` refers to the current rank index and `plane` refers to the current plane index of the satellite being evaluated.
This version of the document only specifies the modulo
operation, indicated by the `mod` key.

# Describing ground stations {#ground-stations}

In this section, we describe ground stations using the following CDDL schema
{{RFC8610}}.

~~~ cddl
ground-stations-specs = {
  version: tstr,
  ground-stations: [+ ground-station],
}

ground-station = {
  name: tstr,
  latitude: float,      ; degrees
  longitude: float,     ; degrees
  altitude: float,      ; metres above Earth surface
  min-elevation: float, ; degrees
  antennas: uint,
}
~~~
{: #ground-stations-cddl-def artwork-align="left" artwork-name="syntax" title="CDDL schema for ground stations"}

An example specifying a single ground station is as follows:

~~~ json
{
  "version": "draft-piraux-space-constellation-code-02",
  "ground-stations": [
    {
      "name": "Charleroi",
      "latitude": 50.40,
      "longitude": 4.25,
      "altitude": 109.0,
      "min-elevation": 10.0,
      "antennas": 8
    }
  ]
}
~~~
{: #ground-stations-json-example artwork-align="left" artwork-name="example" title="Example ground-stations-specs data item"}

{{ground-stations-json-example}} specifies a single ground station with an associated location. It has a MEA of 10 degrees and 8 antennas that can be used simultaneously.

## Detailed specification

`version`

: Indicates the version of this I-D that the data item should be interpreted with.

`ground-stations`

: A list of ground stations.

### Ground station

`name`

: A string to identify the ground station.

`latitude`

: The latitude of the ground station location, expressed in degrees.

`longitude`

: The longitude of the ground station location, expressed in degrees.

`altitude`

: The altitude of the ground station location, expressed in metres above the Earth's surface.

`min-elevation`

: The Minimum Elevation Angle above which Feeder Links can be established, expressed in degrees.

`antennas`

: The number of antennas available to establish Feeder Links simultaneously.

# Considerations for future versions of this document {#considerations}

The code presented in this document does not consider the capabilities of satellites within a constellation to establish links. It focuses on defining the stable network topology that is expected for a constellation. Future versions of this document could consider means to define the capabilities of Optical Communication Terminals (OCTs) used to establish ISLs. This is complementary to the description of the network topology, which forms more of an intent, while capabilities define the space of possible links.

# Security Considerations

As the code specified in this document is foreseen as a user input into software that performs simulations, evaluations and analysis of satellite constellations, implementers SHOULD consider validation and sanitisation measures.


# IANA Considerations

This document has no IANA actions.

--- back

# Changelog

## Since draft-piraux-space-constellation-code-00

* Add YAML format to specify link patterns within shells of a constellation.
* Improvement of the text and examples.

# Acknowledgments
{:numbered="false"}

We thank Tim van der Lee for his work on a code [TvdLCode] that served as the basis for this document.

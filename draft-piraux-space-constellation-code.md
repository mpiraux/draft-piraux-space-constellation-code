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

normative:
  RFC5234:

informative:
  TvdLCode:
    author:
      - ins: Tim van der Lee
    title: Constellation Code
    date: July 2023
    target: https://github.com/Tim024/ConstellationCode
...

--- abstract

When considering a satellite constellation forming a non-terrestrial network, the characteristics of this constellation heavily influences the network topology it forms.
To improve the analysis of such non-terrestrial networks across various tools developed by the network community, this document proposes a notation to describe common constellation patterns.
In addition, this document may serve as an introduction to satellite constellations for IETF participants.

--- middle

# Introduction

The network topology of a satellite constellation is heavily influenced by its orbital characteristics. With recent technologies enabling Optical Inter-Satellite Links (OISL) between satellites, a network is formed by establishing links between neighbour satellites. The resulting topology can be dynamic as the distance between neighbour satellites changes throughout their orbital period. A common notation for the network community could improve the reproducibility of evaluations, measurements and simulations of satellite constellation networks.

The true position of a satellite is often represented using a Two-Line Element set (TLE). A TLE contains a number of fields describing the orbital elements at a given time of a given satellite. Combined with a simplified perturbation model, the TLE can be used to predict the future position and velocity of the satellite relatively accurately. However, when studying satellite constellations, TLEs may not be appropriate. First, they assume each satellite has a known absolute position, which is derived from the launch time and parameters. Second, they involve complex calculations given the chosen perturbation model which may not scale well to large-scale experiments. Third, TLEs are not sufficient to determine how the links are established within the constellation as they do not indicate its characteristics but only the position of its satellites.

The approach of this document is based on the mission parameters of a satellite constellation. Based on these parameters, the expected position of each satellite within the constellation can then be computed. While this approach does not capture the small discrepancies that can occur during the launch and operation of the satellites, we argue that it is sufficient in our context.

The rest of this document is organised as follows.
{{satellite-constellations}} introduces two variants of the Walker pattern for orbital shells. These are used to define many of the existing satellite constellations. {{constellation-code}} defines the constellation code syntax using an ABNF grammar [RFC5234] and the code semantics. {{examples-of-constellations-codes}} contains examples of existing constellations defined using the constellation code. Finally, {{considerations}} concludes with considerations for future versions of this document.

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

The orbits of a Walker Star constellation often have an inclination close to 90 degrees with respect to the equator plane. Given that they are distributed over 180 degrees around the equator plane, one half-sphere has satellites ascending from the south pole to the north pole while the other has them descending from north pole to south pole. This is depicted on the two sides of {{fig-walker-star}}.
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

In a Walker Star constellation, a seam can be observed at the start and end of the orbit distribution around the equator plane. That is the first orbit (resp. last orbit) is next to the last orbit (resp. first orbit) going in the opposite direction of the sphere. It can be observed at the center of the {{fig-walker-star}}. When communication between satellites of neighbour orbits is desired, a Walker Star pattern may not be suitable due to this effect and inter-satellite links may be limited within the same orbit.

{{fig-walker-star-topo}} illustrates a part of a possible network topology for Walker Star constellations, with four orbital plane depicted vertically, each containing three satellites. Links are only established in-plane, i.e., within the same orbit. Each orbit forms a ring, where the last satellite is connected to the first satellite.

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

{{fig-walker-delta}} is an illustration of a Walker Delta constellation with only two orbits due to graphical constraints. The orbits of a Walker Delta constellation often have an inclination ranging from 45 to 65 degrees with respect to the equator plane. Combined with the altitude, the inclination directly limits the latitude coverage of a constellation, while Walker Star constellations have a complete latitude coverage.

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

{{fig-walker-delta-topo}} illustrates a part of a possible network topology for Walker Delta constellations, with four orbital plane depicted vertically, each containing three satellites. Links are established in-plane and cross-plane, i.e., from one orbit to the other.

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

The code only considers circular orbits but future versions of this document could extend it to include the apogee, perigee and argument of the periapsis such that any elliptical orbit can be defined.

~~~ abnf
constellation = shell [ "+" constellation ]
shell = walker ":" altitude ":" inclination ":" plane-params
shell =/ [ ":" mean-anomaly ]
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
- The inclination is expressed in degrees and MUST be within the range of \[0, 90\] degrees.
- The number of satellites must be evenly divisible by the number of planes.
- The phasing factor must be within [0, no-planes[.
- The mean anomaly is expressed in degrees and MUST be within the range of \[0, 360\] degrees. It is optional and represents the current orbital position of the constellation. When absent it is considered equal to zero.

# Examples of constellations codes

This section provides some examples of how the constellation code can be used to define existing satellite constellations sourced from public information. In some cases, when the phasing factor is not known, it is speculative.

 | Name | Description | Constellation code |
 |:-----|:------------|:-------------------|
 | Iridium | Walker Star, 780 km altitude, 86.4° inclination, 66 satellites, 6 planes | S:780:86.4:66/6/1 |
 | OneWeb | Walker Star, 1 200 km altitude, 87.9° inclination, 672 satellites, 12 planes | S:1200:87.9:672/12/11 |
 | GPS | Walker Delta, 20 180 km, 55° inclination, 24 satellites, 6 planes | D:20180:55:24/6/1 |
{: #example-table title="Examples of constellation codes"}

# Considerations for future versions of this document {#considerations}

The code presented in this document does not consider yet the link configuration within a constellation. For instance, in the case of a Walker Delta constellation, satellites may only be able to establish three OISLs, e.g., two in-plane links and a single cross-plane link. Instead, it is assumed that the network topology is fully meshed as illustrated in {{fig-walker-star-topo}} and {{fig-walker-delta-topo}}. Future versions of this document should consider means to indicate how links are established within a constellation, for instance using adjacency matrices.

# Security Considerations

As the code specified in this document is foreseed as a user input into software that performs simulations, evaluations and analysis of satellite constellations, implementers SHOULD consider validation and sanitisation measures.


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

We thank Tim van der Lee for his work on a code [TvdLCode] that served as the basis for this document.

# Chapter 1 — Fundamentals of Low-Voltage Distribution Network Faults

**Week 1 — Fault Detection & Localisation Project**

This chapter develops the physical and mathematical foundations needed to reason about faults in low-voltage (LV) distribution networks before comparing fault-detection and localisation algorithms. The emphasis is deliberately on *what a fault does to the network*, *what conventional and smart-meter instrumentation can actually observe*, and *which simplifying assumptions are safe or unsafe in realistic LV networks*.

The project context supplied for this study—known network topology, smart-meter measurements, OpenDSS simulation, and later algorithmic detection/localisation—is consistent with current research directions in LV monitoring. A particularly relevant Australian study by Flynn *et al.* used voltage–current sensitivity relationships derived from smart meters both for topology estimation and for detecting abrupt changes in network behaviour, validating the approach on Australian LV measurements. [[1]](#ref-01) The [project roadmap](../../ROADMAP.md) defines the prototype’s sensing and simulation scope. The [project instructions](../../AGENTS.md) describe how this chapter supports the Week 1 study and experiments.

## Physical picture of an LV distribution network

At its simplest, an LV distribution network is a conductive tree connecting the secondary terminals of a distribution transformer to customers. The transformer establishes a relatively stiff voltage source; conductors between the transformer and customers have nonzero resistance and reactance; customers draw or inject power at different locations and phases. Any current flowing through those conductors produces voltage drop. A fault is an abnormal electrical connection—or, for some fault classes, an abnormal disconnection—that changes those current paths dramatically.

A useful mental model is therefore:

```text
                Distribution transformer
                nominally 400/230 V
                       │
                       │  z1
                       ▼
                     Bus 1
                    /     \
                  z2       z3
                 /           \
             Customer A      Bus 2
                            /    \
                          z4      z5
                         /          \
                    Customer B   Customer C
                                      │
                                      ×  fault
                                      │
                                    earth /
                                   neutral /
                                 another phase
```

In Australia, the LV classification extends up to 1 kV AC, and Victorian public LV networks normally supply approximately 230 V phase-to-neutral and 400 V phase-to-phase. CitiPower/Powercor explicitly describe Victorian LV systems in these terms, while AS IEC 60038 adopts the IEC standard-voltage framework used nationally. [[2]](#ref-02) [[3]](#ref-03) [[4]](#ref-04) Victorian networks are supplied from distribution substations connected to higher-voltage feeders; CitiPower, for example, states that its final supply to small consumers is through 230/400 V LV systems and that its higher-voltage distribution feeders are generally operated radially, with tie points available for reconfiguration. [[2]](#ref-02)

Most fault-detection reasoning becomes easier if the LV feeder is viewed as a **rooted graph**

$$
G=(\mathcal{V},\mathcal{E}),
$$

where the transformer is the root, $\mathcal{V}$ is the set of buses or connection points, and each edge $e\in\mathcal{E}$ represents a line segment with complex impedance

$$
Z_e=R_e+jX_e\quad [\Omega].
$$

In this radial model, the series impedance between two buses is the sum of the impedances along the unique path joining them. The transfer impedance that couples a current change at one bus to a voltage change at another comes from the part of their two root-to-bus paths that they share, assuming a fixed root voltage and a simple series-impedance model. This is why topology matters to localisation. Customers that share a long section of conductor upstream tend to experience correlated voltage changes caused by currents flowing through that shared impedance. Modern topology-estimation work exploits exactly this property using voltage sensitivity matrices and pairwise electrical distances. [[5]](#ref-05) [[6]](#ref-06)

For a single conductor section carrying current $\underline{I}$,

$$
\underline{V}_s-\underline{V}_r=Z\underline{I}=(R+jX)\underline{I}.
$$

For a predominantly resistive LV feeder, $R/X$ is often much larger than in transmission systems, so active-power changes can have a strong effect on voltage magnitude. The original DistFlow formulation was specifically developed for radial distribution systems and remains a foundation for deriving approximate distribution-voltage relationships. [[7]](#ref-07) For a branch carrying downstream real and reactive power $P$ and $Q$, the first-order voltage-drop approximation is

$$
\Delta V \approx \frac{RP+XQ}{V},
$$

Here $P,Q$ are the receiving-end active and reactive powers, defined positive for downstream consumption. $V$ is the receiving-end voltage magnitude, which can be approximated by nominal voltage for a small drop. Use per-phase quantities and the effective series impedance of the corresponding circuit. This approximation neglects second-order loss terms and assumes moderate voltage deviations; exact three-phase LV power flow is more complicated because of mutual impedance, neutral current, phase unbalance, single-phase loads, inverter injections and unequal conductor impedances. Three-phase sensitivity models have therefore been developed specifically for LV networks rather than treating them as balanced single-phase equivalents. [[8]](#ref-08)

Consider a simple 230 V phase-to-neutral circuit with a customer drawing $P=5\,\mathrm{kW}$ at unity power factor through a circuit with total series resistance $R=0.20\,\Omega$. Ignoring reactance,

$$
I\approx \frac{P}{V}=\frac{5000}{230}=21.7\,\mathrm{A},
$$

so

$$
\Delta V\approx RI=0.20(21.7)=4.35\,\mathrm{V}.
$$

The receiving voltage is roughly $225.7\,\mathrm{V}$ using the unrounded first-order drop. A second customer added downstream increases current in the shared conductor, so *both* customers can see additional voltage drop. That shared-path behaviour is crucial later: a disturbance at one location often leaves a spatial voltage signature across many meters.

Victorian nominal voltage should not be confused with an exact constant voltage. CitiPower/Powercor describe a normal operating range associated with the 230 V standard and report voltage variation caused by loading, line length, solar export, network impedance and upstream conditions. [[9]](#ref-09) Flynn *et al.* found transformer-voltage variation sufficiently important that assuming a constant LV-transformer voltage materially degraded estimated voltage sensitivities; their revised formulation explicitly estimated transformer-voltage variation. [[1]](#ref-01) Thus, a voltage change at every customer at once is not automatically evidence of a local fault.

**Earthing is part of the circuit, not merely a safety detail.** IEC 60364 treats earthing arrangements and protective conductors as fundamental parts of LV installation design, while Australian practice is governed principally through AS/NZS 3000 and local regulations. [[10]](#ref-10) [[11]](#ref-11) [[12]](#ref-12) Victoria uses the Multiple Earthed Neutral (MEN) arrangement. Energy Safe Victoria states that a required LV installation must have an earthed-neutral or MEN connection, and historical ESV material describes MEN as the normal supply-reticulation method in Melbourne. [[13]](#ref-13) [[14]](#ref-14)

Conceptually:

```text
Transformer star point
        │
      Neutral ─────────────── Neutral conductor ────────────┐
        │                                                    │
       ⏚ transformer earth                                MEN link
                                                             │
Protective earth ────────────────────────────────────────────┤
                                                             │
                                                            ⏚
                                                     customer electrode
```

The important fault-analysis consequence is that an earth fault requires a **return path**. Its current depends on the impedance of that path through conductors, earthing connections, the source neutral and, depending on the situation, soil and bonded metallic systems. Grounding arrangement therefore directly influences earth-fault current magnitude and protective-device behaviour. IEC short-circuit guidance explicitly treats zero-sequence networks separately because ground-return behaviour differs from positive- and negative-sequence behaviour. [[15]](#ref-15) Australian MEN networks provide multiple neutral/earth connections, but this does not imply zero earth-path impedance, nor does it guarantee that every line-to-earth fault produces a very large current. [[13]](#ref-13)

This immediately establishes the first principle for fault localisation:

$$
\boxed{
\begin{aligned}
\text{Fault signature}=f(&\text{fault type, fault impedance, source impedance,}\\
                        &\text{topology, earthing, loads, DER})
\end{aligned}
}
$$

There is no universal voltage or current threshold that uniquely identifies every LV fault.

## What electrical faults are and how they arise

A **fault** is an abnormal electrical condition that changes the intended circuit topology or impedance. Classical short-circuit analysis focuses on unintended conductive connections between phase conductors, neutral and earth. Other practically important faults include open conductors, failing joints, intermittent arcs and insulation degradation. IEC 60909 provides the standard framework for calculating balanced and unbalanced short-circuit currents in LV and HV three-phase AC systems. Its 2026 edition explicitly applies to 50/60 Hz low-voltage systems. [[16]](#ref-16)

Physical causes include insulation breakdown, moisture ingress, vegetation or foreign-object contact, mechanical cable damage, conductor breakage, loose or corroded joints, animal contact, incorrect installation and thermal degradation. Overhead networks are additionally exposed to weather, lightning-related flashover and vegetation. Distribution protection literature consequently distinguishes permanent faults from temporary faults that may disappear after de-energisation. [[17]](#ref-17)

The classical shunt-fault classes are:

| Fault | Conductive connection | Balance |
|---|---|---|
| Single line-to-ground (SLG, L-G) | one phase to earth/neutral return path | unbalanced |
| Line-to-line (L-L) | two phases | unbalanced |
| Double line-to-ground (L-L-G) | two phases to ground | unbalanced |
| Three-phase (L-L-L or L-L-L-G) | all three phases | approximately balanced if impedances equal |

The principal open-circuit classes are one-phase-open, two-phase-open and complete interruption. Unbalanced short circuits and open conductors create negative- and sometimes zero-sequence quantities, which is why symmetrical components are so useful for fault analysis. IEC 60909's worked guidance explicitly models positive-, negative- and zero-sequence networks, while standard power-system texts treat unsymmetrical fault calculations through those sequence networks. [[15]](#ref-15) [[18]](#ref-18)

For phase quantities,

$$
\begin{bmatrix}
V_a\\V_b\\V_c
\end{bmatrix}
=
\begin{bmatrix}
1&1&1\\
1&a^2&a\\
1&a&a^2
\end{bmatrix}
\begin{bmatrix}
V_0\\V_1\\V_2
\end{bmatrix},
\qquad
a=e^{j120^\circ}.
$$

Here:

- $V_1$: positive-sequence voltage, representing the normal balanced sequence;
- $V_2$: negative-sequence voltage, associated with phase unbalance;
- $V_0$: zero-sequence voltage, whose three components are in phase and which is strongly influenced by neutral/earth return paths.

With a balanced pre-fault source and decoupled sequence networks, an exactly balanced three-phase short circuit excites only the positive-sequence network. An SLG fault couples all three sequence networks in series. A line-to-line fault involves positive and negative sequence but no zero-sequence current if there is no ground connection. These results are established symmetrical-component fault theory. [[18]](#ref-18) [[19]](#ref-19)

For a three-phase fault at a bus represented by a pre-fault Thévenin source $\underline{V}_\mathrm{th}$, Thévenin impedance $Z_\mathrm{th}$, and per-phase fault impedance $Z_f$,

$$
\underline{I}_f
=
\frac{\underline{V}_\mathrm{th}}{Z_\mathrm{th}+Z_f}.
$$

For a resistive contact $Z_f=R_f\ge 0$ and a passive source path $Z_\mathrm{th}=R_\mathrm{th}+jX_\mathrm{th}$, the current magnitude is

$$
|I_f|=\frac{|V_\mathrm{th}|}
{\sqrt{(R_\mathrm{th}+R_f)^2+X_\mathrm{th}^2}}.
$$

At fixed source voltage and path impedance, lowering $R_f$ increases the fault current. Along a passive radial path with non-negative series resistance and inductive reactance, adding line sections increases the source-path impedance and reduces the fault current under the same source conditions. Consequently, the same bolted fault can produce substantially different currents close to and far from the transformer.

**Worked example — low-resistance balanced three-phase fault.** Assume a 400 V line-to-line source, so the per-phase RMS voltage is

$$
V_{\phi}=\frac{400}{\sqrt{3}}=230.9\,\mathrm{V}.
$$

Let the source-plus-transformer impedance to the feeder origin be

$$
Z_s=0.02+j0.05\,\Omega,
$$

the line impedance to the fault be

$$
Z_\ell=0.08+j0.03\,\Omega,
$$

and per-phase fault resistance be $R_f=0.02\,\Omega$. Then

$$
Z_\mathrm{total}=(0.12+j0.08)\,\Omega,
$$

whose magnitude is approximately $0.144\,\Omega$. Therefore,

$$
|I_f|\approx\frac{230.9\,\mathrm{V}}{0.1442\,\Omega}\approx1.60\,\mathrm{kA}.
$$

This is qualitatively what conventional overcurrent protection expects from a low-impedance short circuit. IEC 60909 exists precisely because real calculations must also account systematically for source, transformer, motor/generator contributions, sequence networks and maximum/minimum operating conditions rather than relying on such a simple lumped equivalent. [[16]](#ref-16)

Now replace the $0.02\,\Omega$ contact by $10\,\Omega$. Using the same simplified source,

$$
|I_f|\approx
\frac{230.9\,\mathrm{V}}{|10.10+j0.08|\,\Omega}
\approx22.9\,\mathrm{A}.
$$

The physical event is still a fault, but its current may now be comparable with an ordinary residential load. This is the central problem of **high-impedance faults (HIFs)**. Reviews of HIF behaviour note that fault currents can remain close to load current and therefore evade conventional overcurrent protection. HIFs may also be nonlinear, intermittent and arcing rather than well represented by a constant resistor. [[20]](#ref-20) [[21]](#ref-21)

Real arcing faults complicate the simple $R_f$ model because an arc may extinguish and reignite, produce asymmetric current, generate harmonics, vary stochastically and display a nonlinear voltage–current characteristic. Ghaderi *et al.* review these properties and emphasise the dependence of HIF behaviour on contact material, moisture and arc conditions. [[20]](#ref-20) A constant-resistance OpenDSS fault is therefore a useful experiment but not a complete physical model of every real fault.

A further distinction is **shunt versus series fault**. A shunt fault creates an unintended low- or high-impedance connection across conductors or to ground and generally increases current. A series fault increases or breaks the intended conduction path. A cracked conductor, blown fuse or failing connector may therefore *reduce* downstream current while producing abnormal downstream voltage or phase unbalance. Published smart-meter work has investigated such faults using voltage unbalance specifically because current-based substation protection can have poor sensitivity to broken or high-impedance conductors. [[22]](#ref-22)

This distinction matters greatly for the eventual project: a detector trained only on “fault = current spike” is solving a much narrower problem than “fault detection in LV networks.”

## How faults change voltages, currents and spatial patterns

Fault detection ultimately depends on **observability**: the fault must change something that a sensor can measure.

The most direct electrical effect of a shunt fault is additional current,

$$
\Delta I \approx I_f,
$$

which produces additional voltage drop through every impedance segment carrying that current. In a single-source radial series model with fixed transformer-terminal voltage and unchanged load currents, the voltage perturbation at bus $k$ is

$$
\Delta \underline{V}_k
\approx
-\sum_{e\in\mathcal{P}(k)\cap \mathcal{P}(f)}
Z_e\,\underline{I}_f,
$$

where $\mathcal{P}(k)$ is the path from the transformer to meter $k$ and $\mathcal{P}(f)$ is the path from the transformer to the fault. A source or transformer impedance upstream of the chosen root adds a common drop if it is included in the model.

This expression is especially useful for localisation. The line sections **shared by the path to the meter and the path to the fault** determine that meter’s direct fault-current-induced feeder drop in this model. A radial topology therefore imposes structure on the voltage disturbance.

Consider:

```text
T ── A ── B ── C ── D
          │         │
          E         × fault
```

A fault near D forces fault current through $T\!-\!A$, $A\!-\!B$, $B\!-\!C$ and $C\!-\!D$. Meters at C and D therefore experience voltage depression across more shared fault-current path impedance than a meter at E, whose path branches at B. The exact values also depend on load current, neutral effects, three-phase coupling and the source impedance, but the *branch-dependent spatial pattern* remains the key localisation cue.

The relationship appears formally in voltage-sensitivity models. Around a pre-fault operating point, voltage-magnitude changes can be approximated by a linear combination of the real and imaginary components of current changes,

$$
\Delta\mathbf v=S_r\Re\{\Delta\underline{\mathbf I}\}+S_x\Im\{\Delta\underline{\mathbf I}\},
$$

Here $\mathbf v$ is the vector of voltage magnitudes and $\underline{\mathbf I}$ is the vector of nodal current phasors. The sensitivity matrices $S_r$ and $S_x$ encode how current changes at one node affect voltage magnitudes elsewhere. An unmodelled fault current can appear as a voltage residual when predictions use only the measured load currents. Flynn *et al.* use abrupt changes in estimated voltage sensitivities for fault detection and account for transformer-voltage variation. Their method illustrates how network structure converts distributed measurements into information about abnormal current paths. [[1]](#ref-01)

A useful simplified derivation starts with the fixed-impedance relation for voltage phasors

$$
\underline{\mathbf V}
=
\underline{V}_0\mathbf 1-Z_\mathrm{bus}\underline{\mathbf I},
$$

where $Z_\mathrm{bus}$ captures shared-path network impedances. A small change gives

$$
\Delta\underline{\mathbf V}
=
\Delta\underline{V}_0\mathbf 1
-
Z_\mathrm{bus}\Delta\underline{\mathbf I}.
$$

The first term represents a transformer/source-voltage change affecting many customers coherently. The second represents changes caused by nodal currents. With current defined positive for withdrawal, a shunt fault adds a current withdrawal at a normally unmodelled location:

$$
\Delta\underline{\mathbf V}_{f}
\approx
-\mathbf z_f\underline{I}_f,
$$

where $\mathbf z_f$ is a vector describing the shared impedance between each observed meter and the fault path.

That is the mathematical reason topology is useful. Different fault positions can produce distinct $\mathbf z_f$ at the observed meters. Locations with the same shared paths can have identical signatures up to a scale factor when fault current is unknown.

**Worked spatial example.** Consider one phase of a radial feeder with source voltage $230\,\mathrm{V}$, source impedance $0.05\,\Omega$, and three identical line sections of $0.08\,\Omega$. A fault with $R_f=1.0\,\Omega$ occurs after the third section.

The current is approximately

$$
I_f=
\frac{230}
{0.05+3(0.08)+1.0}
=
178.3\,\mathrm{A}.
$$

Ignoring ordinary load and reactance, the voltages after each successive line section are approximately

$$
V_1=230-I_f(0.05+0.08)\approx206.8\,\mathrm{V},
$$

$$
V_2=230-I_f(0.05+0.16)\approx192.6\,\mathrm{V},
$$

$$
V_3=230-I_f(0.05+0.24)\approx178.3\,\mathrm{V}.
$$

The voltage depression becomes larger as the observation point approaches the fault along the same branch. A real three-phase LV network will not produce such a perfectly monotonic scalar profile because loads, other phases, neutral impedance, DER and transformer regulation perturb it. Nevertheless, this toy calculation captures why distributed voltage measurements may contain positional information.

Fault resistance changes both magnitude and absolute spatial contrast. For the same purely resistive circuit,

$$
I_f=\frac{230\,\mathrm{V}}{R_\mathrm{path}+R_f}.
$$

Here $R_{\mathrm{path}}=0.05+3(0.08)=0.29\,\Omega$. As $R_f\rightarrow\infty$,

$$
I_f\rightarrow0,
$$

so the shunt fault becomes progressively less visible in ordinary RMS voltage and current. This explains why fault resistance is routinely included as a sensitivity parameter in fault-location studies. Zhang *et al.* tested fault location in simulation across fault resistance, fault type, measurement error, distributed generation and grounding conditions. Their method uses both secondary-side voltage measurements and substation voltage and current. [[23]](#ref-23)

The pre-fault load state also matters. If a meter's voltage moves from $228$ to $220\,\mathrm{V}$, that eight-volt change might indicate a fault, a large motor starting, an EV charger switching on, transformer-voltage movement, a neighbouring PV system ceasing export, or some combination. Flynn *et al.* use voltage–current sensitivities to account for load variation and look for changes in the estimated relationships. [[1]](#ref-01)

Three-phase unbalance adds additional information. Define the voltage-unbalance factor

$$
\mathrm{VUF}
=
\frac{|V_2|}{|V_1|}\times100\%.
$$

An asymmetric fault tends to introduce negative sequence and, for ground faults, potentially zero sequence. However, residential LV networks already contain substantial normal unbalance because many customers are connected single phase. Therefore, a nonzero $V_2$ is not itself proof of a fault. What may be informative is an abrupt or spatially coherent deviation from the usual unbalance pattern. Vieira *et al.* exploited voltage-unbalance changes at LV customer meters in simulated broken-conductor scenarios on a 13.8 kV primary feeder. [[22]](#ref-22)

DER complicates the direction of current flow. In a passive radial network, fault current is dominated by the upstream source. With inverter-connected PV, batteries or other distributed sources, fault contributions may originate at multiple locations and power-flow direction may reverse during normal operation. Research on fault characterisation in distributed-energy systems shows that multiple sources can alter fault currents and conventional protection assumptions. [[24]](#ref-24) Inverters also have control and current-limiting behaviour unlike synchronous machines, so “fault current proportional to $1/Z$” is best understood as the network Thévenin idealisation, not a universal device-level law.

There are also two different temporal regimes:

$$
\boxed{\text{electromagnetic transient}}
\quad \longrightarrow \quad
\boxed{\text{fault-on RMS/phasor state}}.
$$

The first includes sub-cycle travelling waves, DC offset, arc transients and harmonic content. The second concerns the change in RMS voltage/current while the fault is still present after the initial transient. Clearing the fault can create a separate post-clearing state, such as a de-energised branch. IEC 60909 short-circuit calculations primarily address current quantities relevant to short-circuit duty rather than reconstructing every electromagnetic transient. [[16]](#ref-16) Minute-scale smart-meter algorithms can respond to a fault-on state if it lasts long enough to affect the recorded quantities. They may also observe the post-clearing state.

This distinction will become crucial when deciding whether a phenomenon is observable at all.

## Conventional detection, localisation and protection

Three terms should remain separate throughout this project.

**Detection** asks:

$$
\text{“Has an abnormal fault condition occurred?”}
$$

**Localisation** asks:

$$
\text{“Where in the network did it occur?”}
$$

**Protection** asks:

$$
\text{“What must be disconnected, and how quickly, to keep the system safe?”}
$$

A protection system therefore includes detection but is not equivalent to a fault-detection algorithm. Protection additionally requires speed, selectivity, dependability, security, interrupting capability and coordination with adjacent devices. Modern protection literature treats these as central design attributes. [[17]](#ref-17) [[25]](#ref-25)

A fuse operates because sufficiently high current heats its fuse element until it melts. IEC 60269 covers current-limiting LV fuses used for AC circuits up to 1 kV. [[26]](#ref-26) Because heating approximately follows $I^2R$, higher currents produce much faster operation; actual fuse behaviour is described through manufacturer time-current characteristics rather than a single threshold.

An LV circuit breaker detects overcurrent and mechanically interrupts the circuit. Household and similar AC overcurrent breakers are standardised under IEC 60898-1, which covers rated currents up to 125 A and defines short-circuit performance requirements. [[27]](#ref-27) Breakers may combine slower thermal overload operation with fast magnetic short-circuit operation.

An RCD senses the residual sum of the currents in the circuit’s live conductors, including neutral where present. An RCCB is an RCD without integral overcurrent protection. A residual indicates current escaping through some other path, potentially including a person or protective earth. IEC 61008 identifies shock protection as the principal purpose of RCCBs. [[28]](#ref-28) RCDs therefore solve a different problem from phase overcurrent protection: a relatively small earth-leakage current may trip an RCD without ever approaching a breaker's overcurrent threshold.

At network scale, distribution relays may measure current, voltage, direction or impedance and operate breakers or reclosers. Reclosers are useful where faults may be transient: the device trips, waits, then attempts to re-energise the feeder. Modern feeder-protection systems coordinate time-overcurrent characteristics so the device nearest the fault ideally clears first while upstream devices remain closed. [[17]](#ref-17)

A simplified coordination chain is:

```text
source
  │
breaker / relay
  │
recloser
  │
feeder fuse
  │
service fuse
  │
customer breaker / RCD
  │
fault
```

Each device has a protection zone and a time-current characteristic. “Fault detected” therefore does not imply that the system knows the geographic fault position. A feeder relay may know only that current through its measurement point exceeded a criterion. When a fuse operates, operators may know that some downstream section lost supply but still require field patrols, fault indicators, customer calls, SCADA or other measurements to pinpoint the damaged asset.

This motivates smart-meter localisation: thousands of existing measurement points can potentially reveal which customers experienced which voltage or outage pattern. Smart-meter fault-location research has used customer voltage measurements to disambiguate multiple candidate branches produced by impedance-based fault-location methods. [[29]](#ref-29)

Conventional overcurrent protection also illustrates a fundamental limitation. If normal load current is $20\,\mathrm{A}$ and the total current seen by a relay with a resistive fault is $25\,\mathrm{A}$, a relay cannot simply trip at $21\,\mathrm{A}$ without risking operation during ordinary load variation. High-impedance faults are consequently difficult precisely because fault and load distributions overlap. Reviews identify this as one of the defining protection challenges of HIFs. [[20]](#ref-20)

For this project, a smart-meter detection/localisation layer should therefore be viewed primarily as **monitoring and diagnostic intelligence**, unless it is explicitly engineered and certified as part of a protection scheme. An algorithm that flags a probable abnormal segment several minutes after an event could be valuable for outage management and fault investigation while still being far too slow for personnel protection or short-circuit clearing.

## Smart meters as fault sensors

A smart meter is not equivalent to a protection relay, PMU or waveform recorder. What it can contribute depends strongly on which channels the utility exposes and at what temporal resolution.

Victorian AMI is unusually extensive. Victorian government information states that smart meters remotely report customer electricity usage and can provide outage notifications to distribution businesses. [[30]](#ref-30) Jemena reports smart-meter deployment to more than 99% of homes and businesses in its north-west Melbourne network, while AusNet states that its meters record usage in either five- or thirty-minute intervals depending on meter age. [[31]](#ref-31) [[32]](#ref-32)

It is important, however, to separate several fundamentally different kinds of meter information:

| Information | Typical meaning | Fault relevance |
|---|---|---|
| interval active energy $E_P$ [kWh] | energy imported/exported during interval | sustained outage/load-change evidence |
| interval reactive energy $E_Q$ [kvarh] | reactive energy over interval, where available | operating-state context |
| RMS voltage statistics | magnitude, average/min/max depending on meter/system | voltage sag/swell or persistent abnormality |
| current/power | instantaneous or aggregated depending on device | possible loading/fault evidence |
| outage/“last gasp” event | asynchronous supply-loss notification | strong outage indicator |
| restoration event | supply return | restoration verification |
| power-quality event logs | sag/swell, phase or other events on capable meters | richer fault evidence |
| waveform/high-rate samples | generally not normal billing interval data | transient and harmonic analysis |

Jemena specifically reports using AMI “last gasp” information for near-real-time outage management and notes capabilities involving voltage/current fluctuation and sag/swell monitoring. [[33]](#ref-33) Victorian government material likewise states that meters can notify distributors when power is out. [[30]](#ref-30) These event channels are fundamentally different from interval energy.

### The five-minute misconception

Five-minute metering does **not** normally mean one instantaneous sample every five minutes.

AEMO defines five-minute interval metering in terms of **energy recorded over each five-minute interval**. There are 288 such intervals per settlement day, and the reported active-energy value can be in kWh, with units identified in the data record. [[34]](#ref-34) AEMO's five-minute-settlement transition required specified new and replacement NEM meters to support five-minute trading-interval energy data, with requirements phased in from 2018 onward. [[35]](#ref-35) Legacy Victorian AMI may still provide 30-minute data, which is why current Victorian distributor documentation refers to both 5- and 30-minute intervals. [[31]](#ref-31) [[32]](#ref-32)

Thus a five-minute record such as

$$
E_k=0.20\,\mathrm{kWh}
$$

means, with $P(t)$ in kW and $t$ measured in seconds,

$$
E_k=\frac{1}{3600\,\mathrm{s/h}}\int_{t_k}^{t_k+300\,\mathrm{s}} P(t)\,\mathrm{d}t,
$$

not

$$
P(t_k)=0.20\,\mathrm{kW}.
$$

The average power over the interval is

$$
\bar P_k=\frac{E_k}{(5/60)\,\mathrm{h}}
=2.4\,\mathrm{kW}\quad\text{for }E_k=0.20\,\mathrm{kWh}.
$$

This distinction determines which faults are visible.

Suppose a home normally consumes $2\,\mathrm{kW}$ continuously and a fault-related interruption removes supply for only $10\,\mathrm{s}$ within a 300 s interval. The measured interval-average power becomes

$$
\bar P =
2\frac{290}{300}
=
1.933\,\mathrm{kW}.
$$

The event appears only as a $3.3\%$ reduction in interval energy—a change easily confused with normal household variability. If the interruption lasted only $0.2\,\mathrm{s}$,

$$
2\frac{299.8}{300}
=
1.9987\,\mathrm{kW},
$$

a change of only $0.067\%$.

A similar argument applies to a separate interval voltage channel. Suppose the short-window RMS magnitude is $1.0$ pu for 299.8 s and $0.2$ pu for 0.2 s. Approximating the waveform’s mean-square voltage from those two levels gives a whole-interval RMS of

$$
V_\mathrm{rms,300s}
=
\sqrt{
\frac{
299.8(1)^2+0.2(0.2)^2
}{300}
}
\approx0.99968\,\mathrm{pu}.
$$

A severe 200 ms sag has almost vanished in the five-minute RMS. An arithmetic mean of the short-window RMS magnitudes would give approximately $0.99947$ pu. Settlement energy and voltage statistics are separate quantities.

Therefore:

$$
\boxed{
\begin{gathered}
\text{Five-minute interval data cannot reliably}\\
\text{preserve sub-second fault transients.}
\end{gathered}
}
$$

unless the meter separately reports high-rate minimum/maximum statistics, power-quality events or outage notifications.

This is a sampling and aggregation problem rather than merely a machine-learning problem. No algorithm can reconstruct a transient signature that was never retained in the available data.

Five-minute or 30-minute interval data are much better suited to **persistent or sustained effects**: an outage lasting one or more intervals, a persistent conductor disconnection, sustained abnormal voltage, lasting topology change, a prolonged high-resistance condition that shifts average voltages measurably, or a post-fault state that remains until repair/reconfiguration. Interval energy can register sustained demand loss, while observing sustained abnormal voltage or topology-dependent voltage patterns requires a voltage channel. This interpretation is consistent with research that tracks changes in voltage sensitivities estimated from smart-meter operating points. [[1]](#ref-01)

Topology increases information content because a single meter's abnormal voltage is ambiguous, while a set of meters can reveal a branch-dependent pattern. If meters $i$ and $j$ share network path impedance, their voltage sensitivities are related; this principle underlies smart-meter topology identification and subsequent fault inference. [[1]](#ref-01) [[6]](#ref-06) Accurate topology is therefore not merely useful metadata—it constrains the set of spatial disturbance patterns that are physically plausible.

Yet observability is never guaranteed. Let the available measurement vector be

$$
\mathbf y=h(\mathbf x,\theta,f)+\boldsymbol\epsilon,
$$

where

- $\mathbf x$ is the operating state: loads, PV, voltages and currents;
- $\theta$ contains topology and network parameters;
- $f$ describes the fault state;
- $\boldsymbol\epsilon$ represents measurement error.

A fault is observable only if its effect on $\mathbf y$ is distinguishable from plausible changes in $\mathbf x$, uncertainty in $\theta$, and $\boldsymbol\epsilon$.

Several limitations follow.

**Temporal resolution.** Fast arcs, inception angle, DC offset, transient harmonics and travelling waves occur far faster than ordinary AMI settlement data. HIF literature shows that many useful arc signatures exist in waveform shape and frequency content; those features are unavailable if only multi-minute aggregates are retained. [[20]](#ref-20)

**Fault resistance.** As $R_f$ rises, current and voltage disturbance shrink. A sufficiently high-resistance fault can fall inside ordinary load/noise variability. [[20]](#ref-20) [[23]](#ref-23)

**Load variability.** Switching appliances, EV charging and commercial load changes can resemble step changes caused by faults. Model-based methods attempt to condition expected voltage on load specifically to reduce this ambiguity. [[1]](#ref-01)

**DER variability.** PV changes both net demand and voltage and can reverse branch power flow. Consequently, a daytime voltage rise or fall cannot automatically be attributed to a fault. Victorian utilities already identify solar export and two-way flow among normal causes of local voltage variation. [[9]](#ref-09)

**Measurement noise and quantisation.** Small fault signatures can be masked by meter error. Smart-meter topology and fault-location literature routinely tests algorithm robustness under measurement noise for this reason. [[1]](#ref-01) [[23]](#ref-23)

**Missing and asynchronous data.** Communication delay, substitution and missing readings matter. AEMO interval records explicitly contain status fields distinguishing actual, estimated and substituted values. [[34]](#ref-34) A detector should therefore not silently treat every interval as an equally trustworthy synchronous measurement.

**Unknown transformer voltage.** A feeder-wide voltage shift can result from the source rather than a fault. Australian research has demonstrated that modelling transformer-voltage variability materially improves sensitivity estimation. [[1]](#ref-01)

**Topology error.** If a meter is assigned to the wrong phase or branch, its “unexpected” voltage pattern may be created by the model rather than the network. This is particularly relevant to LV systems, where topology records have historically been incomplete and smart-meter topology inference has become an active research area. [[5]](#ref-05) [[6]](#ref-06)

**Parameter uncertainty.** Cable type, length, service impedance, neutral impedance and transformer impedance affect predicted voltage signatures. A physically correct topology with poor impedances may still produce inaccurate localisation.

**Partial deployment.** An unmetered lateral may contain a fault whose effect is visible only weakly at distant meters. Location resolution is therefore bounded by sensor placement.

**Protection action.** A low-impedance fault may be electrically dramatic but extremely brief because a fuse or breaker clears it. Paradoxically, slower five-minute data may therefore observe *less* of the electrically severe fault than of a mild but persistent abnormality.

A useful way to frame candidate Week 2 algorithms is therefore by their measurement assumptions:

$$
\text{algorithm capability}
\subseteq
\text{information retained by the sensing system}.
$$

A travelling-wave method requiring MHz waveform acquisition and a five-minute AMI residual method are not competing solutions to exactly the same sensing problem.

## Modelling faults for study and OpenDSS

OpenDSS is well suited to foundational experiments because its circuit representation preserves phase connectivity, transformer connections, conductor impedances, loads and user-defined faults. EPRI describes an OpenDSS `Fault` element as a two-terminal multiphase resistive network; by default its second terminal is ground, allowing line-to-ground faults, while alternative terminal connections can represent line-to-line and multi-phase faults. [[36]](#ref-36) OpenDSS also has a dedicated fault-study mode that evaluates all-phase, SLG, L-L and L-L-G faults and reports bus voltages and currents. [[37]](#ref-37)

That model should be interpreted carefully.

A fixed `R` fault represents

$$
V_f=R_fI_f.
$$

Use a small positive resistance to approximate a bolted fault with an OpenDSS `Fault` element. The fixed-resistance model is appropriate for studying sensitivity to fault resistance and for conventional short-circuit behaviour. It does **not** automatically reproduce arc reignition, stochastic HIF resistance, conductor melting, dynamic inverter controls, relay operation or switching transients. HIF reviews explicitly describe nonlinear and time-varying behaviours that require richer models. [[20]](#ref-20)

A productive modelling hierarchy is:

```text
Level A: ideal balanced Thévenin circuit
        ↓
Level B: unbalanced three-phase feeder + neutral
        ↓
Level C: real line/transformer parameters + loads + PV
        ↓
Level D: fault resistance variation
        ↓
Level E: protection opening / topology change
        ↓
Level F: stochastic or dynamic arc model if needed
```

Starting at Level A makes the physics transparent. Jumping directly to Level F can produce simulation output without understanding why the output looks the way it does.

The fault calculation itself can be interpreted through the network bus-impedance matrix. For a shunt fault at bus $f$ in a linear single-phase or per-phase equivalent,

$$
V_f^\mathrm{pre}
=
V_f^\mathrm{post}
+
Z_{ff} I_f.
$$

Using $V_f^\mathrm{post}=Z_f I_f$ for the fault branch gives

$$
I_f=
\frac{V_f^\mathrm{pre}}{Z_{ff}+Z_f}.
$$

At another bus $k$,

$$
V_k^\mathrm{post}
=
V_k^\mathrm{pre}
-
Z_{kf}I_f.
$$

The superscript $\mathrm{post}$ here denotes the fault-on state after inception and before clearing. The transfer impedance $Z_{kf}$ measures electrical coupling between measurement bus $k$ and fault bus $f$. This compact equation explains nearly the entire localisation problem:

$$
\boxed{
\Delta V_k=-Z_{kf}I_f
}
$$

under the linear Thévenin approximation.

If $Z_{kf}$ were perfectly known and $I_f$ measurable, the voltage disturbance vector could identify the fault position where candidates have distinct transfer-impedance vectors at the observed meters. In real LV networks, neither is perfect: the network is unbalanced, loads change during the event, the fault itself may be nonlinear, smart meters provide incomplete measurements and source voltage varies. Consequently, practical localisation is a statistical/model-selection problem built on this deterministic electrical backbone.

Symmetrical components provide the analogous description for unsymmetrical faults. For an SLG fault on phase $a$, the fault-branch sequence currents satisfy

$$
I_1=I_2=I_0,
$$

and

$$
I_a=I_0+I_1+I_2=3I_1.
$$

For decoupled sequence networks with a balanced pre-fault source, the classical result is

$$
I_a=
\frac{3V_1^\mathrm{pre}}
{Z_1+Z_2+Z_0+3Z_f}.
$$

This equation is extremely informative. Unlike a balanced three-phase fault, an SLG fault depends on $Z_0$, so neutral and earth-return modelling can strongly change predicted fault current. IEC 60909's sequence-network treatment is therefore especially important for ground faults. [[15]](#ref-15)

For a bolted three-phase fault,

$$
I_f^{3\phi}
=
\frac{V_1^\mathrm{pre}}{Z_1}.
$$

For an L-L fault,

$$
I_0=0,
$$

so ground-return impedance does not enter in the same way. These distinctions explain why two faults at the same physical location can produce very different meter signatures.

When experimenting with OpenDSS, it is therefore important to retain:

- phase-specific bus voltages rather than only a single average;
- neutral conductor representation where applicable;
- pre-fault, fault-on and post-clearing states where applicable;
- fault type and resistance;
- exact electrical location along a branch;
- transformer and line impedances;
- customer phase connectivity;
- DER state;
- whether protection has already operated.

Otherwise, the simulation may erase the very features later needed for detection/localisation.

## From Week 1 to the algorithm-comparison problem

The central conclusion of this chapter is that an LV fault is best understood as a **network perturbation**.

Before the fault,

$$
\mathbf y_t
\approx
h(\mathbf x_t,\theta)
+\boldsymbol\epsilon_t.
$$

During the fault or in the altered network state after clearing,

$$
\mathbf y_t
\approx
h(\mathbf x_t,\theta,f)
+\boldsymbol\epsilon_t.
$$

Fault detection asks whether introducing $f\neq0$ explains the observed measurements substantially better than normal variability. Fault localisation asks which candidate $f$—especially which network position—best explains the *spatial* disturbance pattern.

The strongest physical cues depend on the fault and sensing regime:

| Phenomenon | Protection relay / high-rate recorder | 5–30 min AMI intervals | AMI event/PQ channel |
|---|---:|---:|---:|
| sub-cycle inception transient | strong | effectively absent | sometimes |
| DC offset | strong | absent | generally absent |
| arc harmonics | potentially strong | absent | device-dependent |
| instantaneous kA fault current | strong | usually not retained | device-dependent |
| 100 ms voltage sag | strong | highly diluted in an interval voltage aggregate | possibly |
| last-gasp outage | indirect | not necessarily in interval energy | strong |
| sustained outage | strong | strong | strong |
| persistent abnormal voltage | strong | potentially strong with interval voltage | strong |
| broken/open conductor | depends on relay and sensor placement | often observable if persistent and measurements are affected | depends on available events |
| branch-dependent voltage pattern | possible with distributed sensors | potentially useful with interval voltage | useful with spatially distributed events |
| high-resistance fault with tiny steady-state signature | difficult | difficult | sometimes richer |

This distinction should govern Week 2. Algorithms should not be compared only by reported classification accuracy. They should first be grouped by what information they assume exists: waveforms, phasors, RMS samples, interval energy, outage events, transformer measurements, known line parameters, known topology, labelled faults or simulation-generated training data.

### Suggested exploratory experiments

**Experiment A — fault resistance, distance and voltage footprint.**

Build a simple radial three-phase OpenDSS LV feeder with perhaps 10–20 customers and a known transformer. Place the same SLG fault at several distances from the transformer and sweep

$$
R_f \in
\{0.01,0.1,1,5,10,50\}\ \Omega.
$$

For each case record fault current and every customer's phase voltage before and during the fault. Plot

$$
\Delta V_i
=
V_i^\mathrm{fault}-V_i^\mathrm{pre}
$$

against electrical distance and feeder branch.

The objective is not to detect the fault automatically. It is to understand the geometry of the disturbance. You should expect low-$R_f$ faults to produce larger current and stronger voltage depression; distant faults should generally experience greater source-path impedance; and customers sharing more of the fault path should display more similar voltage changes. In a realistic unbalanced model those relationships will not be perfectly monotonic.

Repeat after adding PV or changing phase loading. The expected lesson is that the *topological pattern* is often more stable than a single global voltage threshold.

**Experiment B — destroy information deliberately with AMI aggregation.**

Create a high-resolution synthetic time series containing several events: a 100 ms voltage sag, a 2 s outage, a 30 s outage, a 3 min outage and a persistent open conductor. Then transform the same signal into:

$$
1\,\mathrm{s},\quad
1\,\mathrm{min},\quad
5\,\mathrm{min},\quad
30\,\mathrm{min}
$$

aggregates using appropriate RMS or energy integration.

Compare the original signal with each representation. The objective is to determine which physical signatures survive the measurement process before designing any detector. Expect short events to disappear almost entirely in five- and thirty-minute averages, while sustained outages and persistent post-fault changes remain visible. Add a separate Boolean “last gasp” event stream and observe how dramatically it changes outage observability.

This experiment is arguably more important than initially testing a sophisticated classifier because it establishes the information-theoretic ceiling imposed by the data source.

### Conceptual questions

1. Why can two faults at the same physical location produce different fault currents if one is L-L and the other SLG?

2. In an MEN network, which impedances form the return path for an earth fault, and which of those are normally missing from a simplistic three-wire model?

3. Why might a severe low-resistance fault be *less visible* in five-minute interval data than a mild open-conductor fault?

4. If every customer voltage falls by approximately the same amount at the same time, what hypotheses other than a feeder fault should be considered?

5. Suppose two candidate fault locations produce almost identical voltage changes at all available meters. What does this imply about localisation observability?

6. Why can an incorrectly estimated phase connection produce behaviour that resembles a fault-model residual?

7. At what point does increasing $R_f$ make “fault” and “ordinary load variation” statistically indistinguishable?

8. How should missing or substituted smart-meter intervals affect confidence in a fault alarm?

9. How would rooftop PV downstream of a fault alter the assumptions behind a purely upstream-source Thévenin model?

10. Which is more useful for localisation: a very accurate voltage measurement at one customer or noisier measurements at twenty topologically dispersed customers? Under what conditions could the answer reverse?

### Essential knowledge before Week 2

Before comparing algorithms, you should be comfortable deriving and interpreting

$$
\underline{V}_s-\underline{V}_r=Z\underline{I},
$$

$$
I_f=\frac{V_\mathrm{th}}{Z_\mathrm{th}+Z_f},
$$

$$
I_\mathrm{SLG}
=
\frac{3V_1}
{Z_1+Z_2+Z_0+3Z_f},
$$

and

$$
\Delta\mathbf V\approx-\mathbf z_fI_f.
$$

You should also be able to explain, without referring to a specific algorithm, why fault type, fault resistance, topology, grounding, source strength, load state, DER, measurement noise and temporal aggregation alter the observable signature.

The most important measurement lesson is:

$$
\boxed{
\text{A detector cannot exploit information that the metering system has discarded.}
}
$$

For conventional five-minute AMI energy data, transient waveform phenomena are generally outside scope. Sustained outages and slower changes that affect energy use are more realistic targets. Sustained voltage changes and topology-dependent voltage patterns require voltage measurements or appropriate power-quality events. Separate outage-event or power-quality channels must therefore be identified explicitly in the eventual dataset. AEMO's own data model reinforces this distinction by defining settlement interval records as energy quantities rather than waveform samples. [[34]](#ref-34)

### Curated supplementary study

For fault calculations and symmetrical components, MIT OpenCourseWare's *Introduction to Electric Power Systems* contains dedicated material on symmetrical components and is useful for developing the mathematical transformation from first principles. [[19]](#ref-19) [[38]](#ref-38)

Soman’s NPTEL *Power System Protection* course covers faults, protection-system attributes, sequence components, fuse protection, overcurrent protection and earth-fault relays in a coherent sequence. [[39]](#ref-39) Pradhan’s NPTEL protection course begins directly with “Faults in Power System” and sequence-component fault review before progressing to numerical and overcurrent relaying. [[40]](#ref-40)

For distribution modelling, Gönen's *Electric Power Distribution Engineering* is a broad reference on practical radial distribution systems, conductor modelling, voltage regulation and protection. [[41]](#ref-41) Grainger and Stevenson's *Power System Analysis* is a standard reference for Thévenin equivalents, symmetrical components, bus impedance and fault calculations. [[18]](#ref-18) Blackburn and Domin's *Protective Relaying: Principles and Applications* provides the complementary protection perspective: relay operating principles, zones, coordination and fault response. [[25]](#ref-25)

For OpenDSS work, EPRI's official documentation on fault objects and fault-study mode should be preferred over secondary tutorials because it precisely documents how the simulator electrically represents each fault. [[36]](#ref-36) [[37]](#ref-37)

For the eventual project direction, Flynn *et al.* is particularly relevant because it links Australian LV smart-meter measurements, topology estimation and fault detection without requiring high-frequency PMUs. It should be read during Week 1 for its physical measurement model and again during Week 2 when algorithm assumptions are compared. [[1]](#ref-01)

### References

1. <a name="ref-01"></a>Flynn, D., Pengwah, A. B., Razzaghi, R., & Andrew, L. L. H. (2023). An improved algorithm for topology identification of distribution networks using smart meter data and its application for fault detection. *IEEE Transactions on Smart Grid*, 14(5), 3850–3861. [DOI](https://doi.org/10.1109/TSG.2023.3239650) · [Monash University record](https://research.monash.edu/en/publications/an-improved-algorithm-for-topology-identification-of-distribution/).

2. <a name="ref-02"></a>CitiPower. (2019). *Distribution Annual Planning Report*, §2.2, Network description. [Report](https://media.powercor.com.au/wp-content/uploads/2020/01/22153349/Final-CitiPower-Distribution-Annual-Planning-Report-2019.pdf).

3. <a name="ref-03"></a>Standards Australia. (2022). *AS IEC 60038:2022, Standard voltages*. [Standards Australia catalogue](https://store.standards.org.au/product/as-iec-60038-2022) · [Standard record](https://www.intertekinform.com/en-us/standards/as-iec-60038-2022-120169_saig_as_as_3210059/).

4. <a name="ref-04"></a>International Electrotechnical Commission. (2021). *IEC 60038:2009+AMD1:2021, IEC standard voltages*. [IEC catalogue](https://webstore.iec.ch/en/publication/72877).

5. <a name="ref-05"></a>Deka, D., Backhaus, S., & Chertkov, M. (2018). Structure learning in power distribution networks. *IEEE Transactions on Control of Network Systems*, 5(3), 1061–1074. [DOI](https://doi.org/10.1109/TCNS.2017.2673546).

6. <a name="ref-06"></a>Pappu, S. J., Bhatt, N., Pasumarthy, R., & Rajeswaran, A. (2018). Identifying topology of low voltage distribution networks based on smart meter data. *IEEE Transactions on Smart Grid*, 9(5), 5113–5122. [DOI](https://doi.org/10.1109/TSG.2017.2680542) · [Author preprint](https://arxiv.org/abs/1609.02678). The paper provides background on phase and topology inference from smart-meter measurements.

7. <a name="ref-07"></a>Baran, M. E., & Wu, F. F. (1989). Optimal sizing of capacitors placed on a radial distribution system. *IEEE Transactions on Power Delivery*, 4(1), 735–743. [DOI](https://doi.org/10.1109/61.19266).

8. <a name="ref-08"></a>Munikoti, S., Jhala, K., Lai, K., & Natarajan, B. (2020). Analytical voltage sensitivity analysis for unbalanced power distribution system. *2020 IEEE Power & Energy Society General Meeting*. [DOI](https://doi.org/10.1109/PESGM41954.2020.9282112) · [Author manuscript](https://www.osti.gov/servlets/purl/1907349).

9. <a name="ref-09"></a>CitiPower & Powercor. (2023). *CitiPower and Powercor Autumn 2023 Voltage Report*. [Report](https://media.powercor.com.au/wp-content/uploads/2023/10/04164117/CPPC-Voltage-Report-Autumn-2023.pdf).

10. <a name="ref-10"></a>International Electrotechnical Commission. (2025). *IEC 60364-1:2025, Low-voltage electrical installations — Part 1: Fundamental principles, assessment of general characteristics and definitions*. [IEC catalogue](https://webstore.iec.ch/en/publication/63699).

11. <a name="ref-11"></a>International Electrotechnical Commission. (2021). *IEC 60364-5-54:2011+AMD1:2021, Low-voltage electrical installations — Part 5-54: Selection and erection of electrical equipment — Earthing arrangements and protective conductors*. [IEC catalogue](https://webstore.iec.ch/en/publication/68865).

12. <a name="ref-12"></a>Standards Australia & Standards New Zealand. (2018). *AS/NZS 3000:2018, Electrical installations (Australian/New Zealand Wiring Rules)*. [Standards Australia catalogue](https://store.standards.org.au/product/as-nzs-3000-2018).

13. <a name="ref-13"></a>Energy Safe Victoria. *Battery installation – Neutral continuity and MEN connection* (EIS 004). [Technical guidance](https://www.energysafe.vic.gov.au/industry-guidance/electrical/electrical-technical-information/eis-004-battery-installation-neutral-continuity-and-men-connection).

14. <a name="ref-14"></a>Energy Safe Victoria. (2014). *Victorian Electrolysis Committee Resource Manual*. [Manual](https://www.esv.vic.gov.au/sites/default/files/2022-12/VEC_Resource_Manual-July2014-1.pdf).

15. <a name="ref-15"></a>International Electrotechnical Commission. (2021). *IEC TR 60909-4:2021, Short-circuit currents in three-phase AC systems — Part 4: Examples for the calculation of short-circuit currents*. [IEC catalogue](https://webstore.iec.ch/en/publication/66482).

16. <a name="ref-16"></a>International Electrotechnical Commission. (2026). *IEC 60909-0:2026, Short-circuit currents in three-phase AC systems — Part 0: Calculation of currents*. [IEC catalogue](https://webstore.iec.ch/en/publication/68454).

17. <a name="ref-17"></a>Blair, J., Hataway, G., & Mattson, T. (2018). Solutions to common distribution protection challenges. *2018 IEEE Rural Electric Power Conference*. [DOI](https://doi.org/10.1109/REPC.2018.00021) · [Author copy](https://selinc.com/api/download/114346/?lang=en).

18. <a name="ref-18"></a>Grainger, J. J., & Stevenson, W. D., Jr. (1994). *Power System Analysis*. McGraw-Hill. [University library record](https://library.iitgoa.ac.in/cgi-bin/koha/opac-detail.pl?biblionumber=681).

19. <a name="ref-19"></a>Kirtley, J. L., Jr. (2011). Introduction to symmetrical components. In *Introduction to Electric Power Systems*, ch. 4. MIT OpenCourseWare. [Chapter PDF](https://www.ocw.mit.edu/courses/6-061-introduction-to-electric-power-systems-spring-2011/7603169f5d8514b769a46fa900b79a57_MIT6_061S11_ch4.pdf).

20. <a name="ref-20"></a>Ghaderi, A., Ginn, H. L., III, & Mohammadpour, H. A. (2017). High impedance fault detection: A review. *Electric Power Systems Research*, 143, 376–388. [DOI](https://doi.org/10.1016/j.epsr.2016.10.021).

21. <a name="ref-21"></a>Depew, A. C., Parsick, J. M., Dempsey, R. W., Benner, C. L., Russell, B. D., & Adamiak, M. G. (2006). Field experience with high-impedance fault detection relays. *2005/2006 IEEE PES Transmission and Distribution Conference and Exhibition*. [DOI](https://doi.org/10.1109/TDC.2006.1668612) · [Author paper](https://wprcarchives.org/wp-content/uploads/2024/07/Alvin-C.-Depew_Field-Experience-with-High-Impedance-Fault-Detection-Relays_2006.pdf).

22. <a name="ref-22"></a>Vieira, F. L., Santos, P. H. M., Carvalho Filho, J. M., Leborgne, R. C., & Leite, M. P. (2019). A voltage-based approach for series high impedance fault detection and location in distribution systems using smart meters. *Energies*, 12(15), 3022. [DOI](https://doi.org/10.3390/en12153022).

23. <a name="ref-23"></a>Zhang, S., Chen, H., Tang, J., Zhang, W., Zang, T., & Xiao, X. (2022). Fault location based on voltage measurement at secondary side of low-voltage transformer in distribution network. *IEEE Transactions on Instrumentation and Measurement*, 71, 3522512. [DOI](https://doi.org/10.1109/TIM.2022.3201230).

24. <a name="ref-24"></a>Keller, J., & Kroposki, B. (2010). *Understanding fault characteristics of inverter-based distributed energy resources* (NREL/TP-550-46698). National Renewable Energy Laboratory. [DOI](https://doi.org/10.2172/971441) · [Report PDF](https://docs.nrel.gov/docs/fy10osti/46698.pdf).

25. <a name="ref-25"></a>Blackburn, J. L., & Domin, T. J. (2014). *Protective Relaying: Principles and Applications* (4th ed.). CRC Press. [Publisher](https://www.routledge.com/Protective-Relaying-Principles-and-Applications/Blackburn-Domin/p/book/9781439888117).

26. <a name="ref-26"></a>International Electrotechnical Commission. (2024). *IEC 60269-1:2024, Low-voltage fuses — Part 1: General requirements*. [IEC catalogue](https://webstore.iec.ch/en/publication/99295).

27. <a name="ref-27"></a>International Electrotechnical Commission. (2015). *IEC 60898-1:2015, Electrical accessories — Circuit-breakers for overcurrent protection for household and similar installations — Part 1: Circuit-breakers for a.c. operation*. [IEC catalogue](https://webstore.iec.ch/en/publication/21972).

28. <a name="ref-28"></a>International Electrotechnical Commission. (2024). *IEC 61008-1:2024, Residual current operated circuit-breakers without integral overcurrent protection for household and similar uses (RCCBs) — Part 1: General rules*. [IEC catalogue](https://webstore.iec.ch/en/publication/67980).

29. <a name="ref-29"></a>Trindade, F. C. L., & Freitas, W. (2017). Low voltage zones to support fault location in distribution systems with smart meters. *IEEE Transactions on Smart Grid*, 8(6), 2765–2774. [DOI](https://doi.org/10.1109/TSG.2016.2538268).

30. <a name="ref-30"></a>Victorian Government. *Smart meters*. [Consumer information](https://www.energy.vic.gov.au/households/save-energy-and-money/smart-meters).

31. <a name="ref-31"></a>Jemena. *My Smart Meter*. [Distributor information](https://www.jemena.com.au/electricity/existing-connections/usage-and-costs/my-smart-meter/).

32. <a name="ref-32"></a>AusNet. *Smart meters*. [Distributor information](https://www.ausnetservices.com.au/smart-meters).

33. <a name="ref-33"></a>Jemena Electricity Networks (Vic) Ltd. (2025). *Advanced Metering Infrastructure*, attachment 10-01 to its Electricity Distribution Price Review regulatory proposal, p. 10. [Australian Energy Regulator document record](https://www.aer.gov.au/documents/jen-att-10-01-advanced-metering-infrastructure-20250131).

34. <a name="ref-34"></a>Australian Energy Market Operator. (2024). *Meter Data File Format Specification NEM12 & NEM13*, version 2.6, §§3.3 and 4.4 and Appendices B–C. [Specification PDF](https://www.aemo.com.au/-/media/files/electricity/nem/retail_and_metering/market_settlement_and_transfer_solutions/2024/mdff-specification-nem12-nem13-v26-clean-final.pdf).

35. <a name="ref-35"></a>Australian Energy Market Operator. *5MS and GS metering and metering data changes*. [Five-minute settlement FAQ](https://www.aemo.com.au/initiatives/major-programs/past-major-programs/five-minute-settlement/participant-toolbox/frequently-asked-questions-faqs/participant-implementation/5ms-and-gs-metering-and-metering-data-changes).

36. <a name="ref-36"></a>Electric Power Research Institute. *OpenDSS Documentation — Defining Faults*. [Documentation](https://opendss.epri.com/DefiningFaults.html).

37. <a name="ref-37"></a>Electric Power Research Institute. *OpenDSS Documentation — Fault Studies*. [Documentation](https://opendss.epri.com/FaultStudies.html).

38. <a name="ref-38"></a>Kirtley, J. L., Jr. (2011). *Introduction to Electric Power Systems*. MIT OpenCourseWare, course 6.061. [Textbook](https://www.ocw.mit.edu/courses/6-061-introduction-to-electric-power-systems-spring-2011/resources/6_061_s11_textbook_pdf/).

39. <a name="ref-39"></a>Soman, S. A. *Power System Protection*. IIT Bombay, National Programme on Technology Enhanced Learning, course 108101039. [Course and lectures](https://nptel.ac.in/courses/108101039).

40. <a name="ref-40"></a>Pradhan, A. K. *Power System Protection*. IIT Kharagpur, National Programme on Technology Enhanced Learning, course 108105167. [Course and lectures](https://nptel.ac.in/courses/108105167) · [Opening lecture](https://www.youtube.com/watch?v=WPmOB31UTkI).

41. <a name="ref-41"></a>Gönen, T. (2014). *Electric Power Distribution Engineering* (3rd ed.). CRC Press. [DOI](https://doi.org/10.1201/b16455) · [Publisher-supplied book record](https://books.google.com/books?id=cRzlAgAAQBAJ).

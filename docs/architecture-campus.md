# Decision note - architecture of the Atlas Industrie campus

## 1. Context and requirements

Atlas Industrie has two buildings about 150 m apart: Building A (head office and IT, 200 users, ERP and messaging) and Building B (production, 100 users, PLCs and supervision). The business directive is that the network must never go down, especially for production. Building B must keep manufacturing even if Building A is isolated. The budget is capped, growth is modest (25 % margin on ports and addresses), and production assets must be protected from IT threats such as ransomware.

## 2. Technical targets retained

- Availability: 99.9 % for office and IT, 99.99 % targeted for production (about 53 minutes of downtime per year).
- Recovery: RTO of 15 minutes for production, verified by a timed failover drill (detect, failover, load, verify).
- Autonomy: Building B keeps its gateways, inter-VLAN routing and OT services without Building A.
- Sizing: 14 devices (2 cores, 2 firewalls, 10 access switches), about 16 ports per core, 20 with the 25 % margin.
- Addressing: 10.42.0.0/16 split by trust zone (see data/vlans.yml), IPv6 ULA fdf2:4d35:180a::/48.

## 3. The two architectures compared

Three scenarios were priced:

- S1, three tiers: 2 modular cores, 4 distribution switches, 10 access switches. Two hops from access to core, high CAPEX and operating load for a modest campus.
- S2, distributed collapsed core: 2 fixed cores (CC-A in A, CC-B in B) linked by two fibres on distinct routes, 10 access switches. One hop, and each building keeps a local core.
- S3, single-site collapsed core: the same 2 cores, both in Building A. Cheapest, but Building B loses its gateways and routing if A fails.

## 4. Five-year costing

All amounts in MAD, excluding tax. TCO = CAPEX + 5 x (support + operations), with support = 10 % of CAPEX per year and operations = devices x 1.5 d x 5,400 MAD (8,100 MAD per device per year).

| | S1 | S2 | S3 |
|---|---|---|---|
| Cores | 324,000 | 172,800 | 172,800 |
| Distribution | 216,000 | 0 | 0 |
| Access (10) | 270,000 | 270,000 | 270,000 |
| Firewall pair | 216,000 | 216,000 | 216,000 |
| Optics and fibre | 129,600 | 108,000 | 86,400 |
| **CAPEX** | **1,155,600** | **766,800** | **745,200** |
| Support, 5 years | 577,800 | 383,400 | 372,600 |
| Devices | 18 | 14 | 14 |
| Operations, 5 years | 729,000 | 567,000 | 567,000 |
| **TCO, 5 years** | **2,462,400** | **1,717,200** | **1,684,800** |

Gaps: S2 is 745,200 MAD cheaper than S1 (about 30 %). S2 costs 32,400 MAD more than S3 over five years (about 2 %).

## 5. Decision and justification

**Decision: S2, distributed collapsed core.**

- S1 is rejected: it costs 745,200 MAD more than S2 and adds a tier the campus size does not justify.
- S3 is the cheapest, but it is not an equivalent solution. It fails a stated requirement: production must continue if Building A is isolated. If A is lost, B loses its default gateways, inter-VLAN routing and external exits. S3 serves to price the requirement.
- Is the extra cost worth it? At an assumed 108,000 MAD per hour of production outage, 32,400 MAD equals 0.3 h, that is 18 minutes. If S2 avoids more than 18 minutes of production downtime over five years, it pays for itself.

Security design:

- Firewall placement: a pair of firewalls, FW-A and FW-B, one per building. Every flow from OT to the rest of the network crosses a firewall.
- DMZ: exposed servers sit in VLAN 60, separated from internal networks, so inbound web traffic cannot pivot inward.
- Default deny between zones: OT to Internet is denied, OT to IT is limited to a named ERP export exception, guest to internal is denied. Every allowed flow is a named, logged exception.
- Trust zones are carried by the address plan (OT = 10.42.64.0/20, IT = 10.42.16.0/20), so an OT-to-IT rule is written on one prefix.
- Unused ports go to VLAN 99, which has no addressing and routes nothing.

## 6. Risks and points of failure

- The 99.99 % figure is a target, not a demonstrated result. Three devices in series at 99.9 % give 0.999^3 = 0.997, about 99.7 %. Doubling the core and the firewall helps, but each non-redundant access switch remains a single point of failure and caps the end-to-end figure near 99.9 %.
- The parallel model assumes independent failures. Common causes remain: same software version on both cores, and both fibre routes meeting in the cable chamber under the courtyard (assumed, see the physical view). The theoretical figure overstates reality.
- Still to analyse: power supplies and UPS, real fibre routes, failover timing against the 15-minute RTO.
- OT/IT trade-off: IT ports use 802.1X, OT ports use static VLANs, MAC filtering and logging, so a replaced PLC at 3 AM does not block production. This is an accepted, documented decision.

## 7. Assumptions and limits

- Exchange rate 1 EUR = 10.8 MAD, prices are orders of magnitude, not quotations.
- Production downtime valued at 108,000 MAD per hour.
- Horizon of five years; support 10 % of CAPEX per year; operations 1.5 days per device per year at 5,400 MAD.
- The cabling plan (north technical gallery, south outer facade, shared cable chamber, separate UPS per core) is invented for the exercise, not surveyed.
- Excluded from costing: power, cooling, civil works, network cards, and any item not listed in the price table.
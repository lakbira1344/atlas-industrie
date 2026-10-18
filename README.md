# Atlas Industrie - network architecture repository
Design of the Atlas Industrie network, the fictitious company of the
"Advanced Network Architectures" course. One folder per object type;
the addressing plan is authoritative in `data/vlans.yml`.
Local check: `python3 scripts/check_vlans.py` from the repository root.
ULA prefix (RFC 4193) kept for the whole course: `fdf2:4d35:180a::/48`
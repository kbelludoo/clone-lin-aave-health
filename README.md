# clone-lin-aave-health

Experimental i64-scale LIN clone of [aave/aave-v3-core](https://github.com/aave/aave-v3-core) `v1.19.4` health-factor math:
`GenericLogic.sol` (`percentMul(coll, avgLT).wadDiv(debt)` / `calculateAvailableBorrows`) plus `PercentageMath.sol` and `WadRayMath.sol`.

- **LIN clone:** [`src/lin_aave_health.lin`](src/lin_aave_health.lin)
- **C11 oracle:** [`test/oracles/aave_health_c11.c`](test/oracles/aave_health_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_AAVE_HEALTH_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** Aave, **not** a Pool / Oracle / aToken, and does **not** claim Solidity uint256 / mainnet / `rayMul` parity (`RAY=1e27` exceeds signed i64 and is rejected). It is **not** the Liquity ICR kernel (`clone-lin-liquity-cr`) and **not** Compound III `isBorrowCollateralized` (`clone-lin-comet-price`).

Upstream was BUSL-1.1 with Change Date 27 January 2023; the Change License is MIT (this run is 2026-10-12). This LIN kernel is an original uint64-scale implementation of the published percentMul+wadDiv wrap, not a verbatim Solidity dump.

Pinned upstream: commit `b74526a7bc67a3a117a1963fc871b3eb8cea8435` (tag `v1.19.4`).

Canonical goldens (uint64-scale, 128-bit product, half-up):

- `percent_mul(10000, 8000)` = **8000** (80.00% of 10000)
- `percent_mul(1, 5000)` = **1**; floor without `HALF_PERCENTAGE` = **0**
- `wad_div(8000, 5000)` = **1600000000000000000** (1.6 WAD)
- `wad_div(2, 3)` = **666666666666666667**; floor = **666666666666666666**
- `health(10000, 8000, 5000)` = **1600000000000000000** (healthy)
- `health(10000, 8000, 8500)` = **941176470588235294** (liquidatable, HF < 1e18)
- `skip_lt(10000, 8500)` = **1176470588235294118** (falsely healthy — named rejected path; Liquity ICR has no LT and matches skip-LT)
- `available(10000, 7500, 1000)` = **6500**; `available(10000, 7500, 8000)` = **0**
- `health(..., debt=0)` = **0** with `no_debt=1` (Aave returns `uint256.max`, unrepresentable in i64)
- `1e18 * 1e18 / 1` fail-closes to **0**

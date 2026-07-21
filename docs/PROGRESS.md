# Progress

## 2026-07-21 — hellc:3001 upstream sync and refresh recovery

- Deployed source commit: `238ff61c984fcf9cfcad494eb8edef135b46b316`
- Upstream base: `53a3637565e79da0e56838d34b9767fc96440829`
- Local fix commit: `36bd43f` (`isNil` import required by the new pingzhongdata fallback)
- Toolchain commit: `238ff61` (pnpm 11.10.0, `pnpm-lock.yaml`, explicit dependency build allowlist)
- Production root: `/var/www/real-time-fund`
- Rollback root: `/var/www/real-time-fund.backup-20260721-143458-b13ada1`
- Previous untracked server lockfile backup: `/root/real-time-fund.pnpm-lock.pre-sync-20260721`

### Root cause and fix

The original `fundgz` and `F10DataApi.aspx` endpoints no longer supplied usable data. Upstream replaced the latest NAV path with `pingzhongdata`, but the new fallback referenced Lodash `isNil` without importing it. The broad fallback `catch` converted that programming error into a generic fund loading failure. Importing `isNil` restored the fallback.

### Verification

- `pnpm install --frozen-lockfile`: passed.
- `pnpm exec eslint app/api/fund.js --max-warnings 999`: passed.
- `pnpm exec next build --webpack`: passed and generated static `out`.
- Nginx configuration test and reload: passed; service remains active.
- Local and public `index.html` SHA-256: `8634fbc236ece312696031270b17910b51f1ee61f91ceb6d9dd039f3287691a6`.
- Browser smoke test restored all 14 screenshot funds and all 14 favorites from the legacy localStorage shape.
- All 14 funds returned a latest NAV and NAV date through the fallback path; the browser reported no unhandled page errors.

### Remaining behavior

Data source 1 still points at the unavailable `fundgz` endpoint. Funds without a saved `dataSource` fall back to the latest published NAV, so intraday estimate fields remain empty until the user selects data source 2 or 3. This deployment does not guess an estimate source because accuracy differs by fund.

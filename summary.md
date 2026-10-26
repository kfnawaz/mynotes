Delete the orphaned AssetCatalogPage, AssetCatalogGrid, AssetCatalogToolbar and assetListAtoms (plus their tests
X
and doc references)?
These are genuinely orphaned (no route registers them, superseded by AssetExplorerPage). If you confirm, I will:
• Delete src/pages/assets/AssetCatalogPage.tsx+_tests_/AssetCatalogPage.test.tsx
• Delete src/components/assets/AssetCatalogGrid.tsx, AssetCatalogToolbar.tsx, and src/atoms/assetListAtoms. ts (+ any tests)
• Remove their references from docs/cp-spa-specifications.md (project tree, 512.7, 518.6 orphan list) and docs/op-spa-what-am-i.md (orphan row)
• Keep PlatformBadge (used elsewhere) and re-run the full tsc/eslint/vitest gate
Note: this removes the only caller of the GET /assets (non-explorer) list endpoint.
1 No, leave them
Keep the orphaned code as-is.
2 Yes, delete them - Remove the orphaned components, atoms, tests, and doc references,
3 Enter custom answer
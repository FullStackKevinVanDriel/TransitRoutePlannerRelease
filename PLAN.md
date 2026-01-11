# TransitRoutePlannerRelease - Implementation Plan

> Last Updated: January 2026
> Version: 0.2.38 → 1.0.0 (Target)

## Repository Purpose

This repository serves as the **release distribution hub** for Transit Route Planner WordPress plugin. It handles:

- GitHub Releases for Pro version updates
- Version tagging and changelog management
- Update server endpoint for Pro license holders
- Release assets (ZIP files) for manual installation

## Ecosystem Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TRANSIT ROUTE PLANNER                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  TransitRoutePlannerPlugin/     Vandromeda API        This Repo    │
│  ├── src/ (React 19)            ├── Stripe ✅         ├── Releases │
│  ├── wordpress-plugin/          ├── License (TODO)    ├── Tags     │
│  └── build scripts              └── Webhooks ✅       └── Updates  │
│                                                                     │
│         │                              │                    │       │
│         ▼                              ▼                    ▼       │
│  ┌─────────────┐              ┌──────────────┐      ┌────────────┐ │
│  │ WordPress   │◄────────────►│  License     │      │  Pro Users │ │
│  │ .org (Free) │              │  Validation  │◄────►│  Updates   │ │
│  └─────────────┘              └──────────────┘      └────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Current State

| Aspect | Status | Details |
|--------|--------|---------|
| Repository | Active | Releases-only, minimal structure |
| Version Tags | v0.1.x - v0.2.x | Multiple versions tagged |
| Release Assets | None | ZIP files not yet published |
| Update Server | Not configured | Needed for Pro auto-updates |
| Changelog | None | Needs CHANGELOG.md |

## Implementation Phases

### Phase 1: Repository Structure Setup

**Priority: HIGH | Dependencies: None**

Set up the foundational structure for release management.

- [ ] Create proper directory structure
- [ ] Add CHANGELOG.md with version history
- [ ] Add release asset templates
- [ ] Configure GitHub Actions for release automation
- [ ] Add release checklist documentation

**Target Structure:**
```
TransitRoutePlannerRelease/
├── README.md                    # Repository overview
├── PLAN.md                      # This document
├── CHANGELOG.md                 # Version history
├── releases/
│   └── .gitkeep                 # Placeholder (assets via GH Releases)
├── .github/
│   ├── workflows/
│   │   └── release.yml          # Auto-create release on tag
│   └── RELEASE_TEMPLATE.md      # Release notes template
└── update-server/
    ├── info.json                # Plugin metadata for update checker
    └── README.md                # Update server documentation
```

---

### Phase 2: Release Automation

**Priority: HIGH | Dependencies: Phase 1**

Automate the release process with GitHub Actions.

- [ ] Create release workflow triggered by version tags
- [ ] Auto-generate release notes from commits
- [ ] Attach ZIP assets to releases
- [ ] Validate ZIP structure before publishing
- [ ] Notify on successful release

**GitHub Actions Workflow:**
```yaml
name: Create Release
on:
  push:
    tags:
      - 'v*'
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true
          files: |
            transit-route-planner-*.zip
```

---

### Phase 3: Update Server Configuration

**Priority: HIGH | Dependencies: Phase 2, Vandromeda API License Endpoints**

Enable Pro users to receive automatic updates through WordPress.

- [ ] Create `info.json` with plugin metadata
- [ ] Set up update endpoint URL structure
- [ ] Integrate license validation with Vandromeda API
- [ ] Test update flow in WordPress
- [ ] Document update server API

**Update Server Endpoint:**
```
GET /update-server/info.json
Response:
{
  "name": "Transit Route Planner Pro",
  "slug": "transit-route-planner-pro",
  "version": "1.0.0",
  "download_url": "https://github.com/.../releases/download/v1.0.0/transit-route-planner-pro.zip",
  "requires": "6.0",
  "tested": "6.7",
  "requires_php": "7.4",
  "sections": {
    "changelog": "..."
  }
}
```

**License-Gated Updates:**
```
Plugin Request → Update Server → Vandromeda API (validate license)
                      │                    │
                      │◄───────────────────┘
                      ▼
              Return update info (if valid)
              or "license required" message
```

---

### Phase 4: Version Strategy & Tagging

**Priority: MEDIUM | Dependencies: Phase 1**

Establish consistent versioning aligned with WordPress.org submission.

**Version Numbering:**
| Version | Milestone |
|---------|-----------|
| 0.2.x | Current development |
| 1.0.0 | WordPress.org submission (Free) |
| 1.0.0-pro | Pro version with licensing |
| 1.x.x | Feature releases |
| x.x.1 | Patch/bugfix releases |

**Tag Format:**
- Free: `v1.0.0`, `v1.1.0`
- Pro: `v1.0.0-pro`, `v1.1.0-pro`

**Release Branches:**
```
main ─────────────────────────────────────────► (stable)
  │
  ├── release/1.0 ─────────────────────────────► (1.0.x patches)
  │
  └── release/1.1 ─────────────────────────────► (1.1.x patches)
```

---

### Phase 5: Free vs Pro Distribution

**Priority: MEDIUM | Dependencies: WordPress.org submission, Vandromeda API**

Manage two distribution channels from this repository.

**Free Version (WordPress.org):**
- Hosted on WordPress.org SVN
- Updates via WordPress.org
- No license required
- Limited features (10 routes/day, 1 map per page)

**Pro Version (This Repo):**
- Hosted on GitHub Releases
- Updates via custom update server
- License validation required
- Full features unlocked

**Release Asset Naming:**
```
transit-route-planner-free-1.0.0.zip    → WordPress.org
transit-route-planner-pro-1.0.0.zip     → GitHub Releases
```

---

### Phase 6: Integration with Vandromeda API

**Priority: HIGH | Dependencies: Vandromeda API License Endpoints**

Connect release distribution with license validation.

**Required Vandromeda API Endpoints:**
| Endpoint | Status | Purpose |
|----------|--------|---------|
| `POST /api/license/validate` | TODO | Validate before serving updates |
| `POST /api/license/activate` | TODO | Activate on domain |
| `GET /api/license/info` | TODO | Check tier for feature access |

**Update Request Flow:**
```
1. WordPress plugin checks for updates
         │
         ▼
2. Request to update-server/info.json
   Headers: { X-License-Key: "XXXX-XXXX-XXXX-XXXX", X-Domain: "example.com" }
         │
         ▼
3. Update server validates with Vandromeda API
         │
         ▼
4. If valid: Return download URL
   If invalid: Return "License required" notice
```

---

## Release Checklist

Use this checklist for each release:

```markdown
## Release v{VERSION} Checklist

### Pre-Release
- [ ] All features tested in WordPress
- [ ] Version bumped in plugin header
- [ ] Version bumped in package.json
- [ ] CHANGELOG.md updated
- [ ] Build script executed successfully
- [ ] ZIP file tested (fresh WordPress install)

### Release
- [ ] Create git tag: `git tag v{VERSION}`
- [ ] Push tag: `git push origin v{VERSION}`
- [ ] Verify GitHub Action completed
- [ ] Download and verify release asset
- [ ] Test update flow for existing installs

### Post-Release
- [ ] Update info.json version
- [ ] Notify customers (if major release)
- [ ] Update documentation
- [ ] WordPress.org SVN commit (Free version)
```

---

## Milestones & Timeline

| Milestone | Target Version | Key Deliverables |
|-----------|----------------|------------------|
| **M1: Repo Setup** | - | Structure, CHANGELOG, Actions |
| **M2: First Automated Release** | v0.3.0 | Working release pipeline |
| **M3: Update Server Live** | v0.4.0 | Pro users get auto-updates |
| **M4: WordPress.org Submission** | v1.0.0 | Free version on WP.org |
| **M5: Pro Launch** | v1.0.0-pro | Licensed Pro version live |

---

## Dependencies on Other Repositories

### TransitRoutePlannerPlugin (Source)
| This Repo Needs | From Plugin Repo |
|-----------------|------------------|
| Built ZIP files | `build-wordpress-plugin.sh` output |
| Version info | `package.json`, plugin header |
| Changelog entries | Commit history |

### Vandromeda API (Backend)
| This Repo Needs | From API |
|-----------------|----------|
| License validation | `POST /api/license/validate` |
| License info | `GET /api/license/info` |

---

## Security Considerations

- [ ] ZIP files signed or checksummed
- [ ] HTTPS-only for update server
- [ ] License keys validated server-side
- [ ] Rate limiting on update checks
- [ ] No sensitive data in release assets

---

## Next Actions (Priority Order)

### Immediate (This Session)
1. ✅ Create PLAN.md (this document)
2. Create CHANGELOG.md with version history
3. Set up basic GitHub Actions workflow

### Short-Term (Next Session)
1. Complete Vandromeda API license endpoints
2. Implement update server info.json
3. Test end-to-end update flow

### Medium-Term
1. Prepare WordPress.org submission assets
2. Set up Pro sales page integration
3. Launch v1.0.0

---

## Related Documents

| Document | Location | Purpose |
|----------|----------|---------|
| Plugin Source | TransitRoutePlannerPlugin/ | React + WordPress code |
| API Docs | vandromeda-api.md | Backend endpoints |
| Priorities | priority-subagents.md | Task prioritization |
| Inventory | repo-inventory.md | Cross-repo tracking |

---

## Notes

- Current version tags (v0.1.x - v0.2.x) point to initial commit; real releases start with structure
- WordPress.org requires SVN, but development stays in Git
- Pro version updates bypass WordPress.org entirely
- Consider GitHub Pages for hosting info.json (free, reliable)

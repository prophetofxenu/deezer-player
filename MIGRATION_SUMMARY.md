# Spotify to Deezer Migration - Quick Reference

## Executive Summary

This document provides a high-level overview of migrating the spotify_player CLI application to use Deezer instead of Spotify. For detailed issue descriptions, see [MIGRATION_ISSUES.md](./MIGRATION_ISSUES.md).

## Current State Analysis

### Architecture Overview
```
spotify_player (Rust)
├── Authentication: librespot-oauth + rspotify
├── API Client: rspotify (Spotify Web API wrapper)
├── Streaming: librespot-playback + librespot-connect
├── UI: ratatui (terminal UI)
├── State Management: Custom Rust modules
└── Config: TOML-based configuration
```

### Key Dependencies to Replace
1. **rspotify** (~15 crate versions) - Spotify Web API
2. **librespot-*** family (~7 crates) - Spotify client library
3. Spotify-specific authentication and OAuth

### Codebase Statistics
_(Estimated from repository analysis)_
- Main client module: ~1,900 lines
- Client handlers: ~250 lines  
- Spotify client wrapper: ~140 lines
- State/model layer: ~1,400 lines
- Total: ~10,000+ lines including UI and other modules

**Files and Dependencies:**
- **Files to Modify:** ~50+ files (20 major changes, 15 moderate, 15 minor)
- **Dependencies to Replace:** ~12+ crates (rspotify family + librespot family)
- **New Dependencies:** 3-5 crates (Deezer SDK, OAuth client, audio codecs if needed)

## Migration Path

### Phase 1: Research & Planning (Weeks 1-2)
**Goal**: Understand Deezer API and available tools

**Key Questions to Answer**:
- What Rust crates exist for Deezer?
- How does Deezer authentication work?
- How can we stream audio from Deezer?
- What features does Deezer support vs. Spotify?

**Issues**: #1, #2, #3

### Phase 2: Core Migration (Weeks 3-8)
**Goal**: Replace Spotify integration with Deezer

**Critical Path**:
1. Authentication system (#4)
2. API client replacement (#5)
3. Data model updates (#6)
4. Streaming implementation (#7)
5. Remove Spotify Connect (#8)

**Issues**: #4-11

### Phase 3: Feature Parity (Weeks 9-11)
**Goal**: Ensure all features work with Deezer

**Tasks**:
- Media controls
- CLI commands
- Configuration system
- Search and library management

**Issues**: #12-16

### Phase 4: Polish & Release (Weeks 12-13)
**Goal**: Production-ready release

**Tasks**:
- Documentation
- Project rename
- CI/CD updates
- Testing
- Release preparation

**Issues**: #17-25

## Key Technical Challenges

### 🔴 Critical: Audio Streaming
**Problem**: No "librespot" equivalent for Deezer

**Possible Solutions**:
1. Use Deezer's official SDK (if available for audio streaming)
2. Reverse engineer Deezer's streaming protocol (legal concerns)
3. Use web player with audio extraction (legal concerns)
4. API-only mode with external player integration

**Impact**: May fundamentally change how the app works

### 🟡 High: Rust Ecosystem
**Problem**: Limited Deezer Rust libraries

**Solution**: 
- Evaluate existing crates (deezer-rs, etc.)
- Build custom API wrapper if needed
- Async/await compatible design

### 🟡 High: Feature Parity
**Problem**: Deezer may not support all Spotify features

**Features to Verify**:
- Connect/device control → **Unlikely to exist**
- Podcasts/shows → Check Deezer support
- Synced lyrics → Check Deezer API
- Collaborative playlists → Check support
- High-quality audio (FLAC) → Check Premium features

**Solution**: Document limitations, provide graceful degradation

### 🟢 Medium: API Differences
**Problem**: Different data structures and endpoints

**Solution**: Create translation layer in data models

## File-by-File Migration Map

| File Category | Files to Change | Complexity | Priority |
|--------------|----------------|------------|----------|
| Authentication | `auth.rs`, `token.rs` | Medium | Critical |
| API Client | `client/*.rs` (4 files) | High | Critical |
| Data Models | `state/*.rs` (8 files) | High | Critical |
| Streaming | `streaming.rs` | Very High | Critical |
| UI | `ui/*.rs` (multiple) | Low-Medium | High |
| CLI | `cli/*.rs` (4 files) | Medium | High |
| Config | `config/*.rs`, docs | Low | High |
| Docs | All `.md` files | Low | High |
| Dependencies | `Cargo.toml` | Medium | Critical |
| Tests | Test files | Medium | Medium |

## Dependencies to Remove

```toml
# Remove these from Cargo.toml:
librespot-connect = "0.8.0"
librespot-core = "0.8.0"
librespot-oauth = "0.8.0"
librespot-playback = "0.8.0"
librespot-metadata = "0.8.0"
rspotify = "0.15.3"
```

## Dependencies to Add

```toml
# Add Deezer SDK (exact crate TBD):
# Option 1: Existing crate (if suitable)
deezer-rs = "x.y.z"  # Example, needs verification

# Option 2: Custom implementation
# reqwest = "0.12" (already present)
# serde = "1.0" (already present)
# + custom API wrapper
```

## Breaking Changes for Users

### Configuration Changes
- `client_id` → `app_id` (Deezer terminology)
- New authentication setup required
- Different cache structure
- Spotify-specific options removed

### Feature Removals (Potential)
- Spotify Connect (device switching)
- Some podcast features (if not supported)
- Certain playlist collaboration features

### URL/URI Format Changes
- `spotify:track:...` → `deezer:track:...`
- Different link formats

## Quick Issue Reference

| # | Title | Priority | Effort | Dependencies |
|---|-------|----------|--------|--------------|
| 1 | Research Deezer API | Critical | 1 week | None |
| 2 | Evaluate Deezer SDK | Critical | 1 week | #1 |
| 3 | Research Streaming | Critical | 1 week | #1 |
| 4 | Replace Auth | High | 1 week | #1, #2 |
| 5 | Replace Client | Critical | 2 weeks | #2, #4 |
| 6 | Update Data Models | High | 1 week | #5 |
| 7 | Implement Streaming | Critical | 2-3 weeks | #3, #5 |
| 8 | Remove Connect | Medium | 3 days | #7 |
| 9 | Update Search | High | 3 days | #5 |
| 10 | Update Playlists | High | 5 days | #5 |
| 11 | Update Library | High | 5 days | #5 |
| 12 | Media Control | Medium | 3 days | #7 |
| 13 | Lyrics Support | Low | 3 days | #5 |
| 14 | CLI Commands | Medium | 5 days | #5 |
| 15 | Command System | Medium | 3 days | #14 |
| 16 | Configuration | Medium | 3 days | #4, #5 |
| 17 | Documentation | High | 1 week | All |
| 18 | Rename Project | High | 2 days | #17 |
| 19 | CI/CD | Medium | 3 days | #18 |
| 20 | Dependencies | High | 2 days | #2, #5 |
| 21 | Issue Templates | Low | 1 day | #17 |
| 22 | Tests | High | 1 week | #5, #6, #7 |
| 23 | Feature Gaps | Medium | 3 days | #1 |
| 24 | Performance | Low | 3 days | #5 |
| 25 | Security Audit | High | 3 days | All |

## Implementation Recommendations

### Step 1: Validate Feasibility (Week 1-2)
**Before starting major work, answer these questions:**

1. ✅ Can we legally stream audio from Deezer?
2. ✅ Is there a suitable Rust SDK or can we build one?
3. ✅ Does Deezer API provide enough functionality?
4. ✅ What are the Premium requirements?

**Action**: Complete Issues #1, #2, #3

**Decision Point**: If streaming isn't feasible, reconsider approach

### Step 2: Proof of Concept (Week 3-4)
**Create a minimal working version**:

1. Basic authentication with Deezer
2. Fetch and display user playlists
3. Search for tracks
4. Play a single track (if possible)

**Action**: Parallel work on Issues #4, #5, #6 (minimal versions)

**Decision Point**: Does the basic integration work?

### Step 3: Core Implementation (Week 5-8)
**Full feature implementation**:

1. Complete authentication system
2. All API endpoints
3. Streaming functionality
4. Data model updates

**Action**: Complete Issues #4-11

### Step 4: Polish (Week 9-13)
**Make it production-ready**:

1. UI refinements
2. Documentation
3. Testing
4. Bug fixes

**Action**: Complete Issues #12-25

## Success Metrics

### Minimum Viable Product (MVP)
- ✅ Authenticate with Deezer
- ✅ Browse library (playlists, albums, artists)
- ✅ Search music
- ✅ Play tracks locally
- ✅ Basic playback controls (play, pause, skip)
- ✅ Volume control

### Feature Complete
- ✅ All MVP features
- ✅ Playlist management (create, edit, delete)
- ✅ Like/save tracks and albums
- ✅ Queue management
- ✅ Seek in tracks
- ✅ Media controls (MPRIS)
- ✅ CLI commands work

### Production Ready
- ✅ All features complete
- ✅ Documentation complete
- ✅ Tests passing
- ✅ CI/CD working
- ✅ Performance acceptable
- ✅ Security audit passed

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Streaming not feasible | Medium | Critical | Research early (Phase 1) |
| No suitable SDK | Medium | High | Build custom wrapper |
| API limitations | High | Medium | Document, workaround |
| Legal issues | Low | Critical | Review ToS, consult legal |
| Timeline overrun | Medium | Medium | Phased releases |
| User migration issues | High | Low | Good documentation |

## Resource Requirements

### Developer Skills Needed
- Rust (advanced)
- API integration (REST APIs)
- Audio programming (optional but helpful)
- OAuth/authentication flows
- Terminal UI development
- Async/await patterns

### Tools and Services
- Deezer Premium account (for testing)
- Deezer Developer account (API access)
- Rust toolchain (already present)
- Testing infrastructure

### Estimated Developer Time
- **Single developer**: 8-13 weeks full-time
- **Small team (2-3)**: 5-8 weeks
- **With custom SDK creation**: Add 2-4 weeks

## Next Steps

1. **Review this document** with stakeholders
2. **Create GitHub issues** from MIGRATION_ISSUES.md
3. **Set up Deezer developer account** for API access
4. **Begin Phase 1 research** (Issues #1-3)
5. **Create proof of concept** branch
6. **Schedule regular sync meetings** for progress review

## Questions to Answer Before Starting

1. Do we have access to Deezer Premium for testing?
2. Have we reviewed Deezer's Terms of Service?
3. Do we have resources for 8-13 weeks of work?
4. What is our risk tolerance for incomplete feature parity?
5. Do we need legal review for streaming implementation?
6. What's our release strategy (alpha/beta/stable)?
7. How do we handle existing users of spotify_player?

## References

- [Deezer API Documentation](https://developers.deezer.com/api)
- [Current spotify_player GitHub](https://github.com/aome510/spotify-player)
- [rspotify Documentation](https://docs.rs/rspotify/)
- [librespot GitHub](https://github.com/librespot-org/librespot)

---

**Document Version**: 1.0  
**Last Updated**: 2025-11-23  
**Status**: Proposal - Pending Review  
**Author**: Analysis by Copilot Code Agent

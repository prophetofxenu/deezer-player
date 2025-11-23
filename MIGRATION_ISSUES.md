# Migration Issues: Spotify to Deezer Player

This document outlines the issues that need to be created and addressed to migrate this CLI application from Spotify to Deezer.

## Overview

The current application is a Rust-based terminal music player built on:
- **rspotify** - Spotify Web API wrapper for Rust
- **librespot** - Open-source Spotify client library (for streaming, connect features)
- **ratatui** - Terminal UI framework
- ~2,400 lines of client code
- ~1,900 lines in main module
- Multiple state management, UI, and configuration modules

## Migration Strategy

The migration requires a multi-phase approach:

### Phase 1: Research and API Integration
### Phase 2: Core Functionality Migration
### Phase 3: Feature Parity
### Phase 4: Documentation and Polish

---

## Issue List

### Phase 1: Research and API Integration

#### Issue 1: Research Deezer API Capabilities and Limitations
**Priority:** Critical  
**Labels:** research, api

**Description:**
Research the Deezer API to understand:
- Available API endpoints and their equivalents to Spotify
- Authentication mechanisms (OAuth flow, access tokens)
- Available Rust libraries/SDKs for Deezer integration
- Rate limits and API constraints
- Audio streaming capabilities
- Differences in data models (track, album, artist, playlist structures)
- Premium vs Free account requirements
- MPRIS/media control support possibilities

**Acceptance Criteria:**
- Document Deezer API capabilities
- Identify feature gaps compared to Spotify
- List available Rust crates for Deezer
- Create API mapping document (Spotify endpoints → Deezer endpoints)

---

#### Issue 2: Evaluate or Create Deezer Rust SDK
**Priority:** Critical  
**Labels:** dependencies, api, research

**Description:**
Determine if there's a suitable Rust SDK for Deezer or if we need to create one.

**Tasks:**
- Search for existing Deezer Rust crates on crates.io
- Evaluate quality, maintenance status, and feature completeness
- If none suitable, design and implement a minimal Deezer API client
- Ensure it supports:
  - Authentication (OAuth 2.0)
  - User profile operations
  - Playlist operations
  - Search functionality
  - Playback control
  - Track/album/artist metadata retrieval

**Acceptance Criteria:**
- Have a working Deezer API client in Rust
- Can authenticate users
- Can retrieve basic music data
- Has async support (tokio compatible)

---

#### Issue 3: Research Deezer Audio Streaming Options
**Priority:** Critical  
**Labels:** research, streaming, audio

**Description:**
Investigate how to implement audio streaming for Deezer since there's no equivalent to librespot.

**Research Areas:**
- Deezer's official SDKs and streaming support
- Legal and ToS implications of direct streaming
- Available audio formats (MP3, FLAC, etc.)
- Audio quality options (128kbps, 320kbps, FLAC)
- Streaming protocols used by Deezer
- Possible integration with existing audio playback libraries (rodio, cpal, gstreamer)
- MPRIS/media control requirements

**Acceptance Criteria:**
- Document streaming approach
- Identify legal constraints
- Propose technical solution for audio playback
- Document required dependencies

---

### Phase 2: Core Functionality Migration

#### Issue 4: Replace Authentication System
**Priority:** High  
**Labels:** authentication, breaking-change

**Description:**
Replace Spotify OAuth authentication with Deezer authentication.

**Files to Modify:**
- `spotify_player/src/auth.rs`
- `spotify_player/src/token.rs`
- Configuration files

**Tasks:**
- Remove `librespot-oauth` dependency
- Remove Spotify OAuth scopes
- Implement Deezer OAuth 2.0 flow
- Update credential caching mechanism
- Modify `SPOTIFY_CLIENT_ID` → `DEEZER_APP_ID`
- Update OAuth scopes for Deezer
- Update redirect URI handling
- Modify token refresh logic

**Acceptance Criteria:**
- Users can authenticate with Deezer
- Access tokens are properly cached
- Token refresh works correctly
- Remove all Spotify-specific authentication code

---

#### Issue 5: Replace Spotify Client with Deezer Client
**Priority:** Critical  
**Labels:** api, refactoring, breaking-change

**Description:**
Replace the core Spotify API client with Deezer API client.

**Files to Modify:**
- `spotify_player/src/client/spotify.rs`
- `spotify_player/src/client/mod.rs`
- `spotify_player/src/client/handlers.rs`
- `spotify_player/src/client/request.rs`

**Tasks:**
- Remove `rspotify` dependency
- Add Deezer SDK dependency
- Replace `Spotify` struct with `Deezer` struct
- Update `SPOTIFY_API_ENDPOINT` → `DEEZER_API_ENDPOINT`
- Reimplement all API methods:
  - User profile operations
  - Playlist CRUD operations
  - Search operations
  - Track/album/artist retrieval
  - Favorites/liked tracks
  - Follow/unfollow operations
  - Queue operations

**Acceptance Criteria:**
- All Spotify API calls replaced with Deezer equivalents
- Core client operations work
- Error handling adapted to Deezer responses
- Rate limiting handled appropriately

---

#### Issue 6: Update Data Models for Deezer
**Priority:** High  
**Labels:** data-model, refactoring

**Description:**
Adapt data structures to work with Deezer's data format.

**Files to Modify:**
- `spotify_player/src/state/model.rs`
- `spotify_player/src/state/data.rs`
- `spotify_player/src/state/player.rs`

**Tasks:**
- Replace `rspotify::model::*` types with Deezer equivalents
- Update ID types (AlbumId, ArtistId, TrackId, PlaylistId, etc.)
- Adapt Context enum for Deezer structure
- Update SearchResults structure
- Modify Track, Album, Artist, Playlist structures
- Update serialization/deserialization logic
- Handle Deezer-specific fields (if any)

**Acceptance Criteria:**
- Data models compatible with Deezer API responses
- Serialization/deserialization works correctly
- All type conversions implemented
- No Spotify-specific model dependencies remain

---

#### Issue 7: Implement Deezer Streaming/Playback
**Priority:** Critical  
**Labels:** streaming, audio, feature

**Description:**
Implement audio streaming functionality for Deezer (most complex issue).

**Files to Modify:**
- `spotify_player/src/streaming.rs`
- `spotify_player/Cargo.toml` (dependencies)
- Configuration files

**Tasks:**
- Remove `librespot-playback` and `librespot-connect` dependencies
- Design new streaming architecture for Deezer
- Implement audio decoding (likely MP3/FLAC)
- Integrate with audio backends (rodio, alsa, pulseaudio, etc.)
- Implement playback controls (play, pause, seek, volume)
- Handle audio buffering and caching
- Implement queue management
- Consider: May need to use Deezer's official SDK if available

**Acceptance Criteria:**
- Can play Deezer tracks locally
- Audio quality settings work
- Playback controls functional
- Queue management works
- Volume control works
- Seeking works
- Audio caching optional feature works

**Note:** This is the most technically challenging part and may require significant research into Deezer's streaming protocols.

---

#### Issue 8: Remove Spotify Connect Feature
**Priority:** Medium  
**Labels:** feature-removal, breaking-change

**Description:**
Remove or replace Spotify Connect functionality as Deezer may not have an equivalent.

**Files to Modify:**
- `spotify_player/src/client/mod.rs`
- `spotify_player/src/streaming.rs`
- README.md
- docs/config.md

**Tasks:**
- Remove `librespot-connect` dependency
- Remove Spotify Connect device switching
- Research if Deezer has device control API
- Either:
  - Implement Deezer equivalent (if available)
  - Or document feature removal
- Update documentation to reflect changes
- Update configuration options

**Acceptance Criteria:**
- Spotify Connect code removed
- Documentation updated
- If Deezer equivalent exists, implement and document
- Device switching works (if supported by Deezer)

---

#### Issue 9: Update Search Functionality
**Priority:** High  
**Labels:** feature, api

**Description:**
Adapt search functionality to use Deezer API.

**Files to Modify:**
- `spotify_player/src/client/handlers.rs`
- `spotify_player/src/event/page.rs`
- `spotify_player/src/ui/page.rs`

**Tasks:**
- Update search API calls to use Deezer
- Adapt search result parsing
- Update search result display
- Handle Deezer-specific search parameters
- Update fuzzy search if needed

**Acceptance Criteria:**
- Can search for tracks, albums, artists, playlists
- Search results display correctly
- Search performance acceptable
- All search types supported by Deezer work

---

#### Issue 10: Update Playlist Management
**Priority:** High  
**Labels:** feature, api

**Description:**
Update playlist operations to use Deezer API.

**Files to Modify:**
- `spotify_player/src/client/handlers.rs`
- `spotify_player/src/playlist_folders.rs`

**Tasks:**
- Replace playlist CRUD operations with Deezer equivalents
- Update playlist track operations (add, remove, reorder)
- Update playlist metadata operations
- Handle Deezer playlist permissions
- Update playlist folders (if Deezer supports them)

**Acceptance Criteria:**
- Can create, read, update, delete playlists
- Can add/remove tracks
- Can reorder tracks in playlists
- Playlist metadata updates work
- User playlists load correctly

---

#### Issue 11: Update Library/Collection Management
**Priority:** High  
**Labels:** feature, api

**Description:**
Adapt library/collection operations for Deezer.

**Files to Modify:**
- `spotify_player/src/client/handlers.rs`
- `spotify_player/src/client/mod.rs`

**Tasks:**
- Update "liked tracks" functionality
- Update "saved albums" functionality
- Update "followed artists" functionality
- Handle Deezer's favorites system
- Update collection sync operations

**Acceptance Criteria:**
- Can like/unlike tracks
- Can save/unsave albums
- Can follow/unfollow artists
- User library loads correctly
- Changes sync properly

---

### Phase 3: Feature Parity

#### Issue 12: Update Media Control Integration
**Priority:** Medium  
**Labels:** feature, media-control

**Description:**
Ensure media control (MPRIS/OS integration) works with Deezer.

**Files to Modify:**
- `spotify_player/src/media_control.rs`

**Tasks:**
- Update media metadata for MPRIS
- Ensure OS media controls work
- Update notification system
- Test on Linux, macOS, Windows

**Acceptance Criteria:**
- MPRIS controls work on Linux
- OS media controls work on macOS/Windows
- Metadata displays correctly
- Play/pause/skip controls work

---

#### Issue 13: Update Lyrics Support
**Priority:** Low  
**Labels:** feature, nice-to-have

**Description:**
Update or remove lyrics functionality for Deezer.

**Files to Modify:**
- `spotify_player/src/client/mod.rs`
- `lyric_finder/` module

**Tasks:**
- Remove `librespot-metadata` for lyrics
- Research Deezer lyrics API
- Either:
  - Implement Deezer lyrics (if available)
  - Use external lyrics service
  - Or remove feature and document
- Update UI to display lyrics

**Acceptance Criteria:**
- Lyrics work if Deezer supports them
- Or gracefully handle absence of lyrics
- Documentation updated

---

#### Issue 14: Update CLI Commands
**Priority:** Medium  
**Labels:** cli, refactoring

**Description:**
Update CLI commands to work with Deezer.

**Files to Modify:**
- `spotify_player/src/cli/commands.rs`
- `spotify_player/src/cli/handlers.rs`
- `spotify_player/src/cli/client.rs`

**Tasks:**
- Update command help text (Spotify → Deezer)
- Update command implementations
- Update URI parsing (Spotify URIs → Deezer URIs)
- Test all CLI commands

**Acceptance Criteria:**
- All CLI commands work with Deezer
- Help text accurate
- URI parsing works for Deezer
- Command output correct

---

#### Issue 15: Update Command System
**Priority:** Medium  
**Labels:** commands, refactoring

**Description:**
Update command descriptions and functionality for Deezer.

**Files to Modify:**
- `spotify_player/src/command.rs`
- `spotify_player/src/event/mod.rs`

**Tasks:**
- Update `OpenSpotifyLinkFromClipboard` → `OpenDeezerLinkFromClipboard`
- Update command descriptions
- Ensure all commands work with Deezer API
- Update keyboard shortcuts if needed

**Acceptance Criteria:**
- All commands work with Deezer
- Command descriptions accurate
- Link opening works for Deezer URLs
- No Spotify-specific command references

---

#### Issue 16: Update Configuration System
**Priority:** Medium  
**Labels:** config, breaking-change

**Description:**
Update configuration files and options for Deezer.

**Files to Modify:**
- `spotify_player/src/config/mod.rs`
- `docs/config.md`
- `examples/app.toml`

**Tasks:**
- Rename `client_id` → `app_id` (or similar Deezer term)
- Update configuration options
- Remove Spotify-specific settings
- Add Deezer-specific settings
- Update default values
- Update config documentation

**Acceptance Criteria:**
- Configuration schema updated
- Documentation reflects changes
- Example configs provided
- Migration guide for existing users

---

### Phase 4: Documentation and Polish

#### Issue 17: Update All Documentation
**Priority:** High  
**Labels:** documentation

**Description:**
Update all documentation to reflect Deezer migration.

**Files to Modify:**
- `README.md`
- `docs/config.md`
- `examples/README.md`
- `checklist.md`
- All inline documentation

**Tasks:**
- Replace "Spotify" with "Deezer" throughout
- Update feature descriptions
- Update screenshots/examples
- Update installation instructions
- Update requirements (Deezer Premium?)
- Update authentication setup guide
- Create migration guide for existing users
- Update links and references

**Acceptance Criteria:**
- All documentation accurate and up-to-date
- No Spotify references remain
- Clear setup instructions
- Feature documentation complete
- Migration guide available

---

#### Issue 18: Rename Project and Binary
**Priority:** High  
**Labels:** breaking-change, project-structure

**Description:**
Rename the project from spotify_player to deezer_player.

**Files to Modify:**
- `Cargo.toml` (workspace and package names)
- `spotify_player/Cargo.toml`
- Folder structure (`spotify_player/` → `deezer_player/`)
- All module imports
- Binary name
- GitHub repository settings

**Tasks:**
- Rename workspace members
- Rename binary output
- Update package metadata
- Update imports throughout codebase
- Rename folders
- Update CI/CD references
- Update Docker configurations

**Acceptance Criteria:**
- Project name is `deezer-player`
- Binary name is `deezer_player`
- All imports updated
- CI/CD works with new names
- No broken references

---

#### Issue 19: Update Build and CI/CD
**Priority:** Medium  
**Labels:** ci-cd, infrastructure

**Description:**
Update build configurations and CI/CD pipelines.

**Files to Modify:**
- `.github/workflows/ci.yml`
- `.github/workflows/cd.yml`
- `.github/workflows/docker.yml`
- `Dockerfile`
- `Cross.toml`

**Tasks:**
- Update workflow names
- Update build scripts
- Update Docker image names
- Update release process
- Update package publishing
- Test all CI/CD pipelines

**Acceptance Criteria:**
- CI/CD pipelines work
- Builds succeed
- Docker images build correctly
- Releases work
- Package publishing works

---

#### Issue 20: Update Dependencies and Cargo Metadata
**Priority:** High  
**Labels:** dependencies, project-structure

**Description:**
Update all dependencies and package metadata.

**Files to Modify:**
- `spotify_player/Cargo.toml`
- `lyric_finder/Cargo.toml`
- `Cargo.lock`

**Tasks:**
- Remove Spotify-specific dependencies:
  - `rspotify` and related
  - `librespot-*` family of crates
- Add Deezer SDK dependency
- Update package description
- Update repository URL
- Update keywords
- Update authors/maintainers
- Run `cargo update`

**Acceptance Criteria:**
- No Spotify dependencies remain
- Deezer dependencies added
- Metadata accurate
- Project builds successfully
- Dependencies secure and up-to-date

---

#### Issue 21: Update Issue Templates and Contributing Guidelines
**Priority:** Low  
**Labels:** documentation, project-maintenance

**Description:**
Update GitHub issue templates and contributing docs.

**Files to Modify:**
- `.github/ISSUE_TEMPLATE/bug_report.md`
- `.github/ISSUE_TEMPLATE/feature_request.md`
- Any CONTRIBUTING.md file

**Tasks:**
- Update references to Spotify
- Update example usage
- Update bug report template
- Update feature request template

**Acceptance Criteria:**
- Templates reference Deezer
- Examples are accurate
- Contributing guidelines updated

---

#### Issue 22: Update Tests
**Priority:** High  
**Labels:** testing, quality

**Description:**
Update test suite to work with Deezer.

**Tasks:**
- Identify existing tests
- Update test fixtures
- Update mocked API responses
- Update test data
- Add integration tests for Deezer API
- Update unit tests

**Acceptance Criteria:**
- All tests pass
- Test coverage maintained or improved
- Integration tests work with Deezer API
- Mock data reflects Deezer responses

---

#### Issue 23: Handle Feature Gaps and Deprecations
**Priority:** Medium  
**Labels:** feature, analysis

**Description:**
Document and handle features that may not be available in Deezer.

**Tasks:**
- Create feature comparison matrix (Spotify vs Deezer)
- Document unsupported features
- Add graceful degradation for missing features
- Add feature flags for optional features
- Update documentation with limitations

**Features to Verify:**
- Podcasts/Shows support
- Connect/device switching
- Synced lyrics
- Radio stations
- Collaborative playlists
- Playlist folders
- High quality audio (FLAC)
- Canvas/video
- Social features

**Acceptance Criteria:**
- Feature comparison documented
- Missing features handled gracefully
- Feature flags implemented where appropriate
- Users informed of limitations

---

#### Issue 24: Performance Optimization
**Priority:** Low  
**Labels:** performance, optimization

**Description:**
Optimize performance for Deezer API.

**Tasks:**
- Profile API call patterns
- Implement caching strategies
- Optimize network requests
- Reduce redundant API calls
- Implement batch operations where possible
- Test rate limiting behavior

**Acceptance Criteria:**
- Application performs well
- API rate limits not exceeded
- Caching reduces redundant calls
- User experience smooth

---

#### Issue 25: Security Audit and Best Practices
**Priority:** High  
**Labels:** security, quality

**Description:**
Ensure security best practices for Deezer integration.

**Tasks:**
- Audit token storage
- Review credential handling
- Ensure HTTPS for all API calls
- Review dependencies for vulnerabilities
- Implement secure configuration storage
- Add security documentation

**Acceptance Criteria:**
- Credentials stored securely
- No security vulnerabilities found
- Dependencies up-to-date and secure
- Security best practices documented

---

## Migration Priorities

### Must Have (Critical Path)
1. Research Deezer API (Issue 1)
2. Deezer SDK evaluation/creation (Issue 2)
3. Streaming solution research (Issue 3)
4. Replace authentication (Issue 4)
5. Replace Spotify client (Issue 5)
6. Update data models (Issue 6)
7. Implement streaming (Issue 7)
8. Update documentation (Issue 17)
9. Rename project (Issue 18)

### Should Have (Important Features)
10. Update search (Issue 9)
11. Update playlists (Issue 10)
12. Update library (Issue 11)
13. Update CLI commands (Issue 14)
14. Update configuration (Issue 16)
15. Update dependencies (Issue 20)

### Nice to Have (Polish)
16. Media control (Issue 12)
17. Lyrics support (Issue 13)
18. Remove/adapt Connect (Issue 8)
19. Tests (Issue 22)
20. Performance optimization (Issue 24)

### Can Be Deferred
21. Issue templates (Issue 21)
22. Feature gaps analysis (Issue 23)
23. Security audit (Issue 25)
24. CI/CD updates (Issue 19)
25. Command system polish (Issue 15)

## Technical Challenges

### Critical Challenges:
1. **Streaming Implementation**: Without a librespot equivalent for Deezer, implementing audio streaming will be complex
2. **API Feature Parity**: Deezer API may not support all Spotify features
3. **Rust Ecosystem**: Limited Rust libraries for Deezer compared to Spotify

### Moderate Challenges:
1. **Authentication Flow**: Different OAuth implementation
2. **Data Model Mapping**: Different field names and structures
3. **Rate Limiting**: Different rate limit policies
4. **Device Control**: May lack Spotify Connect equivalent

### Minor Challenges:
1. **Documentation**: Extensive text updates
2. **Testing**: Need to mock Deezer API
3. **Configuration Migration**: Breaking changes for users

## Estimated Effort

- **Phase 1 (Research)**: 1-2 weeks
- **Phase 2 (Core Migration)**: 4-6 weeks
- **Phase 3 (Feature Parity)**: 2-3 weeks
- **Phase 4 (Polish)**: 1-2 weeks

**Total Estimated Time**: 8-13 weeks for a complete migration

## Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| No suitable Deezer Rust SDK | High | Create minimal custom SDK |
| Streaming not feasible | Critical | Research Deezer SDK, seek alternatives |
| API feature gaps | Medium | Document limitations, provide alternatives |
| Breaking changes for users | Medium | Provide migration guide, version clearly |
| Legal/ToS issues with streaming | Critical | Review Deezer ToS, consult legal if needed |

## Success Criteria

- Application builds and runs successfully
- Can authenticate with Deezer
- Can browse and search music
- Can play music locally (streaming)
- Can manage playlists and library
- Documentation is complete and accurate
- User experience similar to original
- All critical features functional

## Notes

- This is a significant undertaking requiring substantial changes
- Some features may need to be deprecated if Deezer doesn't support them
- The streaming implementation is the biggest technical challenge
- Consider phased rollout with alpha/beta releases
- May need to maintain separate branches during transition
- User migration path should be well-documented

---

**Created**: 2025-11-23  
**Status**: Proposal  
**Next Steps**: Review and prioritize issues, begin Phase 1 research

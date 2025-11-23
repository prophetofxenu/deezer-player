# GitHub Issues Creation Script

This file contains template commands to create GitHub issues for the Spotify to Deezer migration.
You can use the GitHub CLI (`gh`) or create these manually in the GitHub web interface.

## Usage with GitHub CLI

Make sure you have the GitHub CLI installed and authenticated:
```bash
gh auth login
```

Then run these commands to create all issues:

---

## Phase 1: Research and API Integration

### Issue 1: Research Deezer API Capabilities and Limitations
```bash
gh issue create \
  --title "Research Deezer API Capabilities and Limitations" \
  --label "research,api,priority:critical,phase:1" \
  --body "## Description
Research the Deezer API to understand capabilities and limitations for migrating from Spotify.

## Research Areas
- [ ] Available API endpoints and their equivalents to Spotify
- [ ] Authentication mechanisms (OAuth flow, access tokens)
- [ ] Available Rust libraries/SDKs for Deezer integration
- [ ] Rate limits and API constraints
- [ ] Audio streaming capabilities
- [ ] Differences in data models (track, album, artist, playlist structures)
- [ ] Premium vs Free account requirements
- [ ] MPRIS/media control support possibilities

## Deliverables
- Document Deezer API capabilities
- Identify feature gaps compared to Spotify
- List available Rust crates for Deezer
- Create API mapping document (Spotify endpoints → Deezer endpoints)

## References
- [Deezer API Documentation](https://developers.deezer.com/api)
- See MIGRATION_ISSUES.md for detailed information"
```

### Issue 2: Evaluate or Create Deezer Rust SDK
```bash
gh issue create \
  --title "Evaluate or Create Deezer Rust SDK" \
  --label "dependencies,api,research,priority:critical,phase:1" \
  --body "## Description
Determine if there's a suitable Rust SDK for Deezer or if we need to create one.

## Tasks
- [ ] Search for existing Deezer Rust crates on crates.io
- [ ] Evaluate quality, maintenance status, and feature completeness
- [ ] If none suitable, design minimal Deezer API client
- [ ] Ensure support for:
  - [ ] Authentication (OAuth 2.0)
  - [ ] User profile operations
  - [ ] Playlist operations
  - [ ] Search functionality
  - [ ] Playback control
  - [ ] Track/album/artist metadata retrieval

## Acceptance Criteria
- [ ] Have a working Deezer API client in Rust
- [ ] Can authenticate users
- [ ] Can retrieve basic music data
- [ ] Has async support (tokio compatible)

## Dependencies
- Depends on #1

## References
- See MIGRATION_ISSUES.md Issue #2"
```

### Issue 3: Research Deezer Audio Streaming Options
```bash
gh issue create \
  --title "Research Deezer Audio Streaming Options" \
  --label "research,streaming,audio,priority:critical,phase:1" \
  --body "## Description
Investigate how to implement audio streaming for Deezer since there's no equivalent to librespot.

## Research Areas
- [ ] Deezer's official SDKs and streaming support
- [ ] Legal and ToS implications of direct streaming
- [ ] Available audio formats (MP3, FLAC, etc.)
- [ ] Audio quality options (128kbps, 320kbps, FLAC)
- [ ] Streaming protocols used by Deezer
- [ ] Possible integration with existing audio playback libraries (rodio, cpal, gstreamer)
- [ ] MPRIS/media control requirements

## Deliverables
- Document streaming approach
- Identify legal constraints
- Propose technical solution for audio playback
- Document required dependencies

## ⚠️ Critical
This is the most technically challenging aspect of the migration. The project's viability depends on finding a legal and technically feasible streaming solution.

## Dependencies
- Depends on #1

## References
- See MIGRATION_ISSUES.md Issue #3"
```

---

## Phase 2: Core Functionality Migration

### Issue 4: Replace Authentication System
```bash
gh issue create \
  --title "Replace Authentication System with Deezer OAuth" \
  --label "authentication,breaking-change,priority:high,phase:2" \
  --body "## Description
Replace Spotify OAuth authentication with Deezer authentication.

## Files to Modify
- \`spotify_player/src/auth.rs\`
- \`spotify_player/src/token.rs\`
- Configuration files

## Tasks
- [ ] Remove \`librespot-oauth\` dependency
- [ ] Remove Spotify OAuth scopes
- [ ] Implement Deezer OAuth 2.0 flow
- [ ] Update credential caching mechanism
- [ ] Modify \`SPOTIFY_CLIENT_ID\` → \`DEEZER_APP_ID\`
- [ ] Update OAuth scopes for Deezer
- [ ] Update redirect URI handling
- [ ] Modify token refresh logic

## Acceptance Criteria
- [ ] Users can authenticate with Deezer
- [ ] Access tokens are properly cached
- [ ] Token refresh works correctly
- [ ] Remove all Spotify-specific authentication code

## Dependencies
- Depends on #1, #2

## References
- See MIGRATION_ISSUES.md Issue #4"
```

### Issue 5: Replace Spotify Client with Deezer Client
```bash
gh issue create \
  --title "Replace Spotify Client with Deezer Client" \
  --label "api,refactoring,breaking-change,priority:critical,phase:2" \
  --body "## Description
Replace the core Spotify API client with Deezer API client.

## Files to Modify
- \`spotify_player/src/client/spotify.rs\`
- \`spotify_player/src/client/mod.rs\`
- \`spotify_player/src/client/handlers.rs\`
- \`spotify_player/src/client/request.rs\`

## Tasks
- [ ] Remove \`rspotify\` dependency
- [ ] Add Deezer SDK dependency
- [ ] Replace \`Spotify\` struct with \`Deezer\` struct
- [ ] Update \`SPOTIFY_API_ENDPOINT\` → \`DEEZER_API_ENDPOINT\`
- [ ] Reimplement API methods:
  - [ ] User profile operations
  - [ ] Playlist CRUD operations
  - [ ] Search operations
  - [ ] Track/album/artist retrieval
  - [ ] Favorites/liked tracks
  - [ ] Follow/unfollow operations
  - [ ] Queue operations

## Acceptance Criteria
- [ ] All Spotify API calls replaced with Deezer equivalents
- [ ] Core client operations work
- [ ] Error handling adapted to Deezer responses
- [ ] Rate limiting handled appropriately

## Dependencies
- Depends on #2, #4

## References
- See MIGRATION_ISSUES.md Issue #5"
```

### Issue 6: Update Data Models for Deezer
```bash
gh issue create \
  --title "Update Data Models for Deezer API" \
  --label "data-model,refactoring,priority:high,phase:2" \
  --body "## Description
Adapt data structures to work with Deezer's data format.

## Files to Modify
- \`spotify_player/src/state/model.rs\`
- \`spotify_player/src/state/data.rs\`
- \`spotify_player/src/state/player.rs\`

## Tasks
- [ ] Replace \`rspotify::model::*\` types with Deezer equivalents
- [ ] Update ID types (AlbumId, ArtistId, TrackId, PlaylistId, etc.)
- [ ] Adapt Context enum for Deezer structure
- [ ] Update SearchResults structure
- [ ] Modify Track, Album, Artist, Playlist structures
- [ ] Update serialization/deserialization logic
- [ ] Handle Deezer-specific fields (if any)

## Acceptance Criteria
- [ ] Data models compatible with Deezer API responses
- [ ] Serialization/deserialization works correctly
- [ ] All type conversions implemented
- [ ] No Spotify-specific model dependencies remain

## Dependencies
- Depends on #5

## References
- See MIGRATION_ISSUES.md Issue #6"
```

### Issue 7: Implement Deezer Streaming/Playback
```bash
gh issue create \
  --title "Implement Deezer Streaming/Playback" \
  --label "streaming,audio,feature,priority:critical,phase:2" \
  --body "## Description
Implement audio streaming functionality for Deezer (most complex issue).

## Files to Modify
- \`spotify_player/src/streaming.rs\`
- \`spotify_player/Cargo.toml\` (dependencies)
- Configuration files

## Tasks
- [ ] Remove \`librespot-playback\` and \`librespot-connect\` dependencies
- [ ] Design new streaming architecture for Deezer
- [ ] Implement audio decoding (likely MP3/FLAC)
- [ ] Integrate with audio backends (rodio, alsa, pulseaudio, etc.)
- [ ] Implement playback controls (play, pause, seek, volume)
- [ ] Handle audio buffering and caching
- [ ] Implement queue management
- [ ] Consider: May need to use Deezer's official SDK if available

## Acceptance Criteria
- [ ] Can play Deezer tracks locally
- [ ] Audio quality settings work
- [ ] Playback controls functional
- [ ] Queue management works
- [ ] Volume control works
- [ ] Seeking works
- [ ] Audio caching optional feature works

## ⚠️ Note
This is the most technically challenging part and may require significant research into Deezer's streaming protocols.

## Dependencies
- Depends on #3, #5

## References
- See MIGRATION_ISSUES.md Issue #7"
```

### Issue 8: Remove or Replace Spotify Connect Feature
```bash
gh issue create \
  --title "Remove or Replace Spotify Connect Feature" \
  --label "feature-removal,breaking-change,priority:medium,phase:2" \
  --body "## Description
Remove or replace Spotify Connect functionality as Deezer may not have an equivalent.

## Files to Modify
- \`spotify_player/src/client/mod.rs\`
- \`spotify_player/src/streaming.rs\`
- \`README.md\`
- \`docs/config.md\`

## Tasks
- [ ] Remove \`librespot-connect\` dependency
- [ ] Remove Spotify Connect device switching
- [ ] Research if Deezer has device control API
- [ ] Either:
  - [ ] Implement Deezer equivalent (if available)
  - [ ] Or document feature removal
- [ ] Update documentation to reflect changes
- [ ] Update configuration options

## Acceptance Criteria
- [ ] Spotify Connect code removed
- [ ] Documentation updated
- [ ] If Deezer equivalent exists, implement and document
- [ ] Device switching works (if supported by Deezer)

## Dependencies
- Depends on #7

## References
- See MIGRATION_ISSUES.md Issue #8"
```

---

## Additional Issues 9-25

Continue with similar format for remaining issues...

## Bulk Creation Alternative

If you prefer to create all issues at once, you can use a script like this:

\`\`\`bash
#!/bin/bash
# Create all migration issues
# 
# NOTE: This is a template. You need to populate the arrays
# with actual issue data from the examples above.

# Example arrays (incomplete - add all 25 issues)
declare -a titles=(
  "Research Deezer API Capabilities and Limitations"
  "Evaluate or Create Deezer Rust SDK"
  # ... add remaining 23 issues
)

declare -a labels=(
  "research,api,priority:critical,phase:1"
  "dependencies,api,research,priority:critical,phase:1"
  # ... add remaining 23 label sets
)

declare -a bodies=(
  "## Description\nResearch the Deezer API..."
  "## Description\nDetermine if there's a suitable..."
  # ... add remaining 23 bodies (properly escaped)
)

# Loop through and create each issue
for i in "\${!titles[@]}"; do
  gh issue create \\
    --title "\${titles[$i]}" \\
    --label "\${labels[$i]}" \\
    --body "\${bodies[$i]}"
  
  echo "Created issue $((i+1))"
  sleep 1  # Rate limit protection
done

echo "Created \${#titles[@]} issues successfully"
\`\`\`

**Note:** The above script is a template. For actual implementation, copy the full issue descriptions from the templates above into the arrays.

---

## Manual Creation

If you prefer to create issues manually via the GitHub web interface:

1. Go to your repository's Issues page
2. Click \"New Issue\"
3. Copy the title and body from the templates above
4. Add the appropriate labels
5. Link dependencies using \"Depends on #X\" in the description
6. Assign to appropriate milestone/project

## Labels to Create

Make sure these labels exist in your repository:
- \`research\`
- \`api\`
- \`priority:critical\`
- \`priority:high\`
- \`priority:medium\`
- \`priority:low\`
- \`phase:1\`
- \`phase:2\`
- \`phase:3\`
- \`phase:4\`
- \`breaking-change\`
- \`dependencies\`
- \`streaming\`
- \`audio\`
- \`authentication\`
- \`refactoring\`
- \`data-model\`
- \`feature\`
- \`feature-removal\`
- \`documentation\`
- \`testing\`
- \`ci-cd\`
- \`infrastructure\`
- \`security\`
- \`performance\`
- \`optimization\`
- \`project-structure\`
- \`project-maintenance\`
- \`config\`
- \`cli\`
- \`commands\`
- \`media-control\`
- \`nice-to-have\`
- \`quality\`
- \`analysis\`

---

**Note**: For the complete issue descriptions, body text, and acceptance criteria, please refer to MIGRATION_ISSUES.md

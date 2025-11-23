# Technical Architecture: Spotify vs Deezer Migration

This document provides a detailed technical comparison of the current Spotify-based architecture and the proposed Deezer-based architecture.

## Table of Contents
- [Overview](#overview)
- [Dependency Comparison](#dependency-comparison)
- [API Architecture](#api-architecture)
- [Authentication Flow](#authentication-flow)
- [Data Models](#data-models)
- [Streaming Architecture](#streaming-architecture)
- [Module Structure](#module-structure)
- [Configuration Changes](#configuration-changes)

---

## Overview

### Current Stack (Spotify)
```
Application: spotify_player
Language: Rust (Edition 2021)
UI: ratatui 0.29.0
Primary Dependencies:
  - rspotify 0.15.3 (Spotify Web API)
  - librespot-* 0.8.0 (Spotify Client Library)
    - librespot-core
    - librespot-oauth
    - librespot-playback
    - librespot-connect
    - librespot-metadata
  - rodio 0.21.1 (Audio Backend)
  - tokio 1.48.0 (Async Runtime)
```

### Target Stack (Deezer)
```
Application: deezer_player
Language: Rust (Edition 2021)
UI: ratatui 0.29.0 (no change)
Primary Dependencies:
  - TBD: Deezer SDK (custom or existing)
  - Custom OAuth 2.0 implementation
  - Audio playback library (TBD)
  - rodio 0.21.1 (likely keeps)
  - tokio 1.48.0 (no change)
```

---

## Dependency Comparison

### Dependencies to REMOVE

```toml
# Spotify-specific dependencies
rspotify = "0.15.3"
rspotify-http = "0.15.3"
rspotify-macros = "0.15.3"
rspotify-model = "0.15.3"

# Librespot family
librespot-core = "0.8.0"
librespot-oauth = "0.8.0"
librespot-playback = "0.8.0"
librespot-connect = "0.8.0"
librespot-metadata = "0.8.0"
librespot-audio = "0.8.0"
librespot-protocol = "0.8.0"
```

**Total to Remove:** ~8-12 crates plus transitive dependencies

### Dependencies to ADD

```toml
# Deezer API client (TBD - exact crate name)
# Option 1: Existing crate
deezer-rs = "x.y.z"  # If suitable crate exists

# Option 2: Custom implementation using existing tools
# reqwest (already present) - HTTP client
# serde (already present) - JSON serialization
# oauth2 (possibly add) - OAuth 2.0 client

# Audio streaming (TBD based on research)
# May need additional audio codec libraries
mp3 = "x.y.z"  # If streaming MP3
flac = "x.y.z"  # If streaming FLAC

# Possible additional dependencies
url = "2.5"  # URL parsing (may already be present)
base64 = "0.22"  # For API authentication
```

### Dependencies to KEEP

These will remain unchanged:
```toml
anyhow = "1.0"
clap = "4.5"  # CLI framework
crossterm = "0.29"  # Terminal handling
tokio = "1.48"  # Async runtime
ratatui = "0.29"  # TUI framework
serde = "1.0"
serde_json = "1.0"
reqwest = "0.12"
chrono = "0.4"
parking_lot = "0.12"
tracing = "0.1"

# Optional features (may keep)
souvlaki = "0.8"  # Media control
viuer = "0.9"  # Image rendering
notify-rust = "4.11"  # Desktop notifications
```

---

## API Architecture

### Current (Spotify)

```rust
// spotify_player/src/client/spotify.rs
pub struct Spotify {
    creds: Credentials,
    oauth: OAuth,
    config: Config,
    token: Arc<Mutex<Option<Token>>>,
    http: HttpClient,
    session: Arc<tokio::sync::Mutex<Option<Session>>>,
}

impl BaseClient for Spotify {
    // Implements rspotify::BaseClient trait
}

impl OAuthClient for Spotify {
    // Implements rspotify::OAuthClient trait
}
```

**API Endpoint:** `https://api.spotify.com/v1`

**Key Methods:**
- `current_user()`
- `current_user_playlists()`
- `search()`
- `track()`, `album()`, `artist()`
- `start_playback()`, `pause_playback()`
- And many more from rspotify

### Target (Deezer)

```rust
// deezer_player/src/client/deezer.rs (proposed)
pub struct Deezer {
    access_token: Arc<Mutex<Option<String>>>,
    token_expiry: Arc<Mutex<Option<DateTime>>>,
    http: reqwest::Client,
    app_id: String,
    app_secret: Option<String>,
}

impl Deezer {
    // Custom implementation
    pub async fn current_user(&self) -> Result<User>;
    pub async fn user_playlists(&self) -> Result<Vec<Playlist>>;
    pub async fn search(&self, query: &str) -> Result<SearchResults>;
    pub async fn track(&self, id: u64) -> Result<Track>;
    pub async fn album(&self, id: u64) -> Result<Album>;
    pub async fn artist(&self, id: u64) -> Result<Artist>;
    // etc.
}
```

**API Endpoint:** `https://api.deezer.com`

**Key Differences:**
- Deezer uses numeric IDs (u64) vs Spotify's string URIs
- Different authentication mechanism
- Different response structures
- Different pagination approach
- Different rate limiting rules

---

## Authentication Flow

### Current (Spotify)

```rust
// spotify_player/src/auth.rs
const SPOTIFY_CLIENT_ID: &str = "65b708073fc0480ea92a077233ca87bd";

// Uses librespot-oauth for authentication
pub fn get_creds(auth_config: &AuthConfig) -> Result<Credentials> {
    let client_builder = OAuthClientBuilder::new(
        SPOTIFY_CLIENT_ID,
        &auth_config.login_redirect_uri
    );
    // ... OAuth flow
}
```

**Flow:**
1. User provides client_id (optional)
2. OAuth 2.0 PKCE flow
3. Browser opens for authorization
4. Redirect to localhost:8989
5. Token cached in `~/.cache/spotify-player/`
6. Automatic token refresh

**Scopes Required:**
- user-read-playback-state
- user-modify-playback-state
- streaming
- playlist-read/modify
- user-library-read/modify
- user-follow-read/modify
- etc. (33 scopes total)

### Target (Deezer)

```rust
// deezer_player/src/auth.rs (proposed)
const DEEZER_APP_ID: &str = "...";  // User-provided

pub async fn authenticate(
    app_id: String,
    redirect_uri: String
) -> Result<Token> {
    // Custom OAuth 2.0 implementation
    // Note: Unlike Spotify's OAuthClientBuilder pattern,
    // Deezer requires a more manual OAuth flow implementation
    let auth_url = format!(
        "https://connect.deezer.com/oauth/auth.php\
         ?app_id={}&redirect_uri={}&perms={}",
        app_id, redirect_uri, permissions
    );
    
    // 1. Open browser to auth_url
    // 2. Start local server to receive callback
    // 3. Extract code from callback
    // 4. Exchange code for access token
    let token = exchange_code_for_token(code, app_id).await?;
    
    Ok(token)
}

async fn exchange_code_for_token(
    code: String, 
    app_id: String
) -> Result<Token> {
    // POST to https://connect.deezer.com/oauth/access_token.php
    // with code and app_id parameters
    // ...
}
```

**Flow:**
1. User provides app_id (Deezer application ID)
2. OAuth 2.0 flow
3. Browser opens for authorization
4. Redirect to localhost (similar)
5. Token cached in `~/.cache/deezer-player/`
6. Token refresh (if supported)

**Permissions Required:**
- basic_access
- email
- offline_access
- manage_library
- delete_library
- listening_history

**Key Differences:**
- Different OAuth endpoints
- Different parameter names (app_id vs client_id)
- Different permission/scope names
- May not support PKCE
- Different token lifetimes

---

## Data Models

### Current (Spotify)

```rust
// Uses rspotify::model types directly
use rspotify::model::{
    AlbumId, ArtistId, TrackId, PlaylistId,
    FullTrack, SimplifiedAlbum, FullArtist,
    FullPlaylist, SearchResult,
};

pub struct Track {
    pub id: TrackId<'static>,
    pub name: String,
    pub artists: Vec<SimplifiedArtist>,
    pub album: SimplifiedAlbum,
    pub duration: Duration,
    // ... many more fields
}
```

**ID Format:** URI-based
- Track: `spotify:track:6rqhFgbbKwnb9MLmUQDhG6`
- Album: `spotify:album:2ODvWsOgouMbaA5xf0RkJe`
- Artist: `spotify:artist:0OdUWJ0sBjDrqHygGUXeCF`

### Target (Deezer)

```rust
// Custom types or from deezer SDK
pub type TrackId = u64;
pub type AlbumId = u64;
pub type ArtistId = u64;
pub type PlaylistId = u64;

pub struct Track {
    pub id: TrackId,
    pub title: String,  // Note: "title" not "name"
    pub artist: Artist,
    pub album: Album,
    pub duration: u32,  // seconds, not Duration
    // Different field structure
}
```

**ID Format:** Numeric
- Track: `3135556` (just a number)
- Album: `302127`
- Artist: `27`

**Key Differences:**
- Different field names (title vs name)
- Different structures (nested vs flat)
- Different ID types (numeric vs URI)
- Different JSON response format
- Different date formats
- Different image URL structures

**Migration Impact:**
- All model types need updating
- Serialization/deserialization changes
- ID parsing/formatting changes
- URI scheme changes throughout codebase

---

## Streaming Architecture

### Current (Spotify)

```rust
// spotify_player/src/streaming.rs
use librespot_playback::{
    audio_backend,
    player::Player,
    config::PlayerConfig,
};
use librespot_connect::Spirc;

pub struct StreamConnection {
    spirc: Arc<Mutex<Option<Spirc>>>,
    player: Player,
}

impl StreamConnection {
    pub fn new(session: Session, ...) -> Self {
        let backend = audio_backend::find(...);
        let player = Player::new(
            config,
            session.clone(),
            None,
            move || backend(...)
        );
        // Spirc handles Spotify Connect protocol
        let spirc = Spirc::new(...);
        // ...
    }
}
```

**Architecture:**
```
User Command
    ↓
Application
    ↓
Spirc (Connect Protocol)
    ↓
Player
    ↓
Audio Backend (Rodio/ALSA/Pulse)
    ↓
Hardware
```

**Features:**
- Spotify Connect (device control)
- Local playback
- Remote control
- Gapless playback
- Crossfade
- Normalization
- Multiple audio backends

### Target (Deezer)

**Challenge:** No equivalent to librespot for Deezer

**Proposed Architecture Option 1 (Official SDK):**
```rust
// IF Deezer provides SDK with streaming
use deezer_sdk::Player;

pub struct StreamConnection {
    player: Player,
    current_track: Option<TrackId>,
}

impl StreamConnection {
    pub fn new(session: DeezerSession) -> Self {
        let player = Player::new(session);
        // ...
    }
    
    pub async fn play(&mut self, track_id: TrackId) {
        self.player.play(track_id).await;
    }
}
```

**Proposed Architecture Option 2 (Custom):**
```rust
// Custom implementation (more complex)
pub struct StreamConnection {
    http: reqwest::Client,
    decoder: AudioDecoder,
    output: AudioOutput,
}

impl StreamConnection {
    pub async fn play(&mut self, track_id: TrackId) {
        // 1. Get track stream URL from Deezer API
        let url = self.get_stream_url(track_id).await?;
        
        // 2. Download/stream audio data
        let audio_data = self.download_audio(&url).await?;
        
        // 3. Decode audio (MP3/FLAC)
        let decoded = self.decoder.decode(audio_data)?;
        
        // 4. Output to audio backend
        self.output.play(decoded)?;
    }
}
```

**Architecture:**
```
User Command
    ↓
Application
    ↓
Custom Player
    ↓
HTTP Stream → Decoder → Buffer
    ↓
Audio Backend (Rodio)
    ↓
Hardware
```

**Missing Features:**
- No Connect equivalent (likely)
- Manual queue management
- Manual buffering
- Manual seek implementation
- No remote control (likely)

**Required Research:**
- [ ] Does Deezer API provide stream URLs?
- [ ] What audio formats does Deezer use?
- [ ] What are the legal implications?
- [ ] Can we use existing audio libraries?
- [ ] How to handle DRM (if any)?

---

## Module Structure

### Current Module Organization

```
spotify_player/
├── src/
│   ├── auth.rs              # Spotify OAuth
│   ├── token.rs             # Token management
│   ├── client/
│   │   ├── mod.rs           # AppClient (uses rspotify)
│   │   ├── spotify.rs       # Spotify wrapper
│   │   ├── handlers.rs      # API call handlers
│   │   └── request.rs       # Request types
│   ├── streaming.rs         # Librespot integration
│   ├── state/
│   │   ├── model.rs         # Data models (uses rspotify::model)
│   │   ├── data.rs          # State management
│   │   └── player.rs        # Player state
│   ├── ui/                  # UI components (mostly unchanged)
│   ├── cli/                 # CLI commands
│   ├── config/              # Configuration
│   ├── command.rs           # Command definitions
│   └── main.rs              # Application entry
└── Cargo.toml
```

### Target Module Organization

```
deezer_player/
├── src/
│   ├── auth.rs              # Deezer OAuth (REWRITE)
│   ├── token.rs             # Token management (MODIFY)
│   ├── client/
│   │   ├── mod.rs           # AppClient (REWRITE)
│   │   ├── deezer.rs        # Deezer API wrapper (NEW)
│   │   ├── handlers.rs      # API call handlers (MODIFY)
│   │   └── request.rs       # Request types (MODIFY)
│   ├── streaming.rs         # Custom streaming (REWRITE)
│   ├── state/
│   │   ├── model.rs         # Data models (REWRITE)
│   │   ├── data.rs          # State management (MODIFY)
│   │   └── player.rs        # Player state (MODIFY)
│   ├── ui/                  # UI components (minor changes)
│   ├── cli/                 # CLI commands (MODIFY)
│   ├── config/              # Configuration (MODIFY)
│   ├── command.rs           # Command definitions (MODIFY)
│   └── main.rs              # Application entry (MODIFY)
└── Cargo.toml               # Dependencies (MAJOR CHANGES)
```

**Change Summary:**
- **REWRITE**: Complete rewrite needed (~5 files)
- **MODIFY**: Significant modifications (~15 files)
- **MINOR**: Minor changes (~20 files)
- **UNCHANGED**: No changes (~10 files)

---

## Configuration Changes

### Current Configuration

```toml
# app.toml
client_id = "optional-user-client-id"
login_redirect_uri = "http://127.0.0.1:8989/login"
client_port = 8080
default_device = "spotify-player"
enable_streaming = "Always"

[device]
name = "spotify-player"
device_type = "speaker"
volume = 70
bitrate = 320
```

### Target Configuration

```toml
# app.toml
app_id = "required-deezer-app-id"  # REQUIRED
app_secret = "optional-secret"      # May be needed
login_redirect_uri = "http://127.0.0.1:8989/login"
client_port = 8080
default_device = "deezer-player"
enable_streaming = "Always"  # May work differently

[device]
name = "deezer-player"
# device_type removed (no Connect)
volume = 70
audio_quality = "FLAC"  # or "MP3_320", "MP3_128"
# bitrate replaced with quality enum
```

**Breaking Changes:**
- `client_id` → `app_id`
- Device settings modified
- Spotify-specific options removed
- New Deezer-specific options added

**Cache Structure:**
```
Before: ~/.cache/spotify-player/
├── credentials.json
├── audio/
├── spotify-player-*.log
└── user_client_token.json

After: ~/.cache/deezer-player/
├── token.json
├── audio/  # May be structured differently
├── deezer-player-*.log
└── user_data.json
```

---

## API Endpoint Comparison

### Spotify API
```
Base: https://api.spotify.com/v1

GET  /me                          # Current user
GET  /me/playlists                # User playlists
GET  /me/tracks                   # Liked tracks
GET  /me/albums                   # Saved albums
GET  /me/following                # Followed artists
GET  /search                      # Search
GET  /tracks/{id}                 # Get track
GET  /albums/{id}                 # Get album
GET  /artists/{id}                # Get artist
PUT  /me/player/play              # Start playback
PUT  /me/player/pause             # Pause
POST /me/player/next              # Next track
POST /me/player/previous          # Previous track
```

### Deezer API
```
Base: https://api.deezer.com

GET  /user/me                     # Current user
GET  /user/{id}/playlists         # User playlists
GET  /user/{id}/tracks            # Liked tracks
GET  /user/{id}/albums            # Saved albums
GET  /user/{id}/artists           # Followed artists
GET  /search                      # Search
GET  /track/{id}                  # Get track
GET  /album/{id}                  # Get album
GET  /artist/{id}                 # Get artist
# Playback control - TBD
```

**Key Differences:**
- Different URL structures
- Deezer requires user ID for some endpoints
- Different parameter names
- Different authentication headers
- Different pagination systems
- No REST endpoints for playback control (likely)

---

## Error Handling

### Current (Spotify)

```rust
use rspotify::ClientError;

match result {
    Err(ClientError::StatusCode(status)) => {
        // Handle HTTP errors
    }
    Err(ClientError::Io(e)) => {
        // Handle IO errors
    }
    // etc.
}
```

### Target (Deezer)

```rust
#[derive(Debug, thiserror::Error)]
pub enum DeezerError {
    #[error("HTTP error: {0}")]
    Http(#[from] reqwest::Error),
    
    #[error("API error: {code} - {message}")]
    Api { code: u32, message: String },
    
    #[error("Authentication error")]
    Auth,
    
    #[error("Rate limit exceeded")]
    RateLimit,
    
    // Custom error types
}
```

**Changes Needed:**
- Define custom error types
- Map Deezer API errors
- Update error handling throughout
- Update user-facing error messages

---

## Testing Strategy

### Current Testing
```rust
#[cfg(test)]
mod tests {
    // Uses rspotify test utilities
    // Mocks Spotify API responses
}
```

### Target Testing
```rust
#[cfg(test)]
mod tests {
    use mockito;  // Or similar
    
    #[tokio::test]
    async fn test_get_track() {
        let mut server = mockito::Server::new();
        let mock = server.mock("GET", "/track/123")
            .with_body(r#"{"id": 123, "title": "Test"}"#)
            .create();
        
        // Test implementation
    }
}
```

**Testing Needs:**
- Mock Deezer API responses
- Unit tests for all API methods
- Integration tests with test account
- UI tests (can mostly remain same)
- Performance/load tests

---

## Performance Considerations

### API Rate Limits

**Spotify:** 
- Rate limits not clearly documented
- Generally generous for personal use
- Can cache responses

**Deezer:**
- Research needed: What are Deezer's rate limits?
- May be different for different endpoints
- Caching strategy may need adjustment

### Streaming Performance

**Spotify (librespot):**
- Optimized streaming protocol
- Efficient bandwidth usage
- Prefetching and caching

**Deezer (custom):**
- Need to implement buffering
- Need to handle network issues
- May be less efficient initially
- Room for optimization

---

## Security Considerations

### Token Storage

**Current:**
```rust
// Tokens stored in cache folder
// Uses librespot's secure storage
```

**Target:**
```rust
// Need secure token storage
// Consider using keyring crate
use keyring::Entry;

let entry = Entry::new("deezer-player", "access_token")?;
entry.set_password(&token)?;
```

### API Communication

**Both:**
- HTTPS for all API calls
- Secure OAuth flow
- Token refresh mechanism

**Deezer-specific:**
- Review Deezer's security requirements
- Implement certificate pinning?
- Handle session expiry properly

---

## Migration Checklist

### Phase 1: Dependencies
- [ ] Remove all rspotify dependencies
- [ ] Remove all librespot dependencies
- [ ] Add Deezer SDK or HTTP client
- [ ] Add OAuth 2.0 client library
- [ ] Add audio codec libraries (if needed)
- [ ] Update Cargo.lock
- [ ] Verify no conflicts

### Phase 2: Authentication
- [ ] Implement Deezer OAuth flow
- [ ] Update token storage
- [ ] Update token refresh logic
- [ ] Update configuration for app_id
- [ ] Test authentication flow

### Phase 3: API Client
- [ ] Create Deezer API wrapper
- [ ] Implement all API methods
- [ ] Update error handling
- [ ] Add rate limiting
- [ ] Add response caching

### Phase 4: Data Models
- [ ] Define new ID types
- [ ] Create Track struct
- [ ] Create Album struct
- [ ] Create Artist struct
- [ ] Create Playlist struct
- [ ] Update all type conversions
- [ ] Test serialization

### Phase 5: Streaming
- [ ] Research streaming approach
- [ ] Implement audio download/stream
- [ ] Implement decoder
- [ ] Integrate with audio backend
- [ ] Implement playback controls
- [ ] Test audio quality

### Phase 6: Integration
- [ ] Update all API call sites
- [ ] Update UI to use new models
- [ ] Update CLI commands
- [ ] Update configuration system
- [ ] End-to-end testing

### Phase 7: Polish
- [ ] Update documentation
- [ ] Rename project files
- [ ] Update CI/CD
- [ ] Create migration guide
- [ ] Release preparation

---

## Conclusion

This migration is substantial, touching almost every part of the codebase. The most challenging aspects are:

1. **Streaming implementation** - No ready-made solution
2. **API differences** - Significant structural changes
3. **Feature parity** - Some features may not be possible

Success depends heavily on:
- Deezer API capabilities
- Streaming solution feasibility
- Available Rust libraries
- Development time investment

**Recommendation:** Complete Phase 1 research thoroughly before committing to full migration.

---

**Document Version:** 1.0  
**Last Updated:** 2025-11-23  
**Status:** Technical Analysis  
